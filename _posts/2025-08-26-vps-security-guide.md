---
title: VPS安全措施指北
date: 2025-08-26 19:40 +0800
categories:
  - IT
  - Tutorial
tags:
  - VPS
  - Security
comments: true
---

# 1 更新系统
**登录后的首要任务是更新系统。** 这可以确保所有已安装的软件包都打上了最新的安全补丁。

``` bash
# 更新软件包列表信息
apt update

# 升级所有已安装的软件包
apt upgrade -y

# 可选：build-essential它包含了创建一个 Debian 包（.deb）所需的软件包列表安装gcc/g++/gdb/make 等基本编程工具
apt install build-essential
```

这里的 `-y` 参数会自动确认所有升级提示，节省时间。

# 2 更改SSH端口

如果用`lastb`命令查看SSH登陆失败的记录，可以看到有许多针对SSH服务的爆破 (last 可以查看登陆成功的记录)

可以改ssh的 **端口**并修改**防火墙**。

步骤如下:
- 正常情况下，直接通过`sudo vim /etc/ssh/sshd_config`修改SSH端口，找到 `#Port 22` 这一行。首先去掉 `#`，然后将 `22` 改成一个不常用的端口号（范围建议在 1024-65535 之间，例如 `23432`）。然后再使用`systemctl restart ssh`

``` bash
# 设置 SSH端口
Port 23432
```

- 建议在 `/etc/ssh/sshd_config.d/` 创建一个 conf 文件来自定义 sshd 配置，而不是直接编辑 `/etc/ssh/sshd_config`，防止 OpenSSH 更新后配置冲突。
  
  - 有些云服务商为了启用远程密码登录（sshd 默认禁用 ），会在 `/etc/ssh/sshd_config.d/` 自定义一个 conf 文件，修改 sshd 配置前先要排除它们的干扰。
  
```bash
# 查看 sshd_config.d 是否存在其他 conf 文件
sudo ls /etc/ssh/sshd_config.d/*.conf
# 如果存在，重命名，防止后续自定义配置被覆盖
sudo mv /etc/ssh/sshd_config.d/xxx.conf /etc/ssh/sshd_config.d/xxx.conf.bak
```
  
  - sshd 配置修改完，先用 `sudo sshd -T` 看一下有效配置
  
```bash
# Root 用户登录方式
sudo sshd -T | grep -i "PermitRootLogin"
# 密码认证
sudo sshd -T | grep -i "PasswordAuthentication"
# ssh 端口
sudo sshd -T | grep -i "Port"
```
  
  - 注意`prohibit-password` 是 `without-password` 的别名


# 3 配置基础防火墙

防火墙是服务器的第一道防线。UFW (Uncomplicated Firewall) 是一个非常易于使用的防火墙管理工具，使用系统底层的`iptables`进行设置。

a. **安装UFW** (通常已预装):

```bash
sudo apt install ufw
```
b. **设置默认规则**: 先禁止所有传入连接，允许所有传出连接。这是一个安全的基准。
```bash
sudo ufw default deny incoming 
sudo ufw default allow outgoing
```
c. **允许必要的连接**: 我们需要允许SSH、HTTP和HTTPS的流量进入。
-  如果你没有改SSH端口：
```bash
sudo ufw allow ssh # 22端口
```
        
- 如果你改了SSH端口 (例如 2233)：
```bash
sudo ufw allow 2233/tcp
```
- 如果你计划部署网站，也需要允许HTTP和HTTPS：
```bash
sudo ufw allow http  # 80端口 
sudo ufw allow https # 443端口
```
        
d. **启用防火墙**:
```bash
sudo ufw enable
```
    
系统会警告你这可能会中断现有连接，输入`y`确认。因为我们已经允许了SSH端口，所以连接不会中断。
    
e. **检查防火墙状态**:
```
sudo ufw status verbose
```
    
这将显示所有规则和防火墙的当前状态。

> **Docker 端口映射**：
端口映射会直接修改iptables，而ufw是基于iptables工作的，只指定本机端口到容器端口的映射，会将容器端口直接暴露到所有网络接口（0.0.0.0），绕过ufw防火墙  
**解决方法**：指定回环ip=> `127.0.0.1:3306:3306`，这样只能从本机访问，不会暴露到外网
{: .prompt-warning }

# 4 Fail2ban 防暴力破解 SSH
## 4.1 安装Fail2ban

Fail2ban，顾名思义是防止后台暴力扫描的软件，通过分析系统日志中的异常行为（如多次登录失败），自动封禁可疑 IP 地址，有效抵御暴力破解攻击。

安装`Fail2ban`：

``` bash
sudo apt update && sudo apt install fail2ban
```


官方推荐在配置 fail2ban 的时候应该避免直接更改由 fail2ban 安装创建的`.conf` 文件（例如 `fail2ban.conf` 和 `jail.conf`），应该创建扩展名为`.local` 的新文件（例如 `jail.local`）来进行自定义配置。

`.local` 文件将覆盖`.conf` 文件相同部分的参数。

官方推荐的做法是利用 jail.local 来进行自定义设置：

`sudo vim /etc/fail2ban/jail.local`

可以参照以下配置文件来进行自己的配置（记得删注释）：

```bash
[sshd]
ignoreip = 127.0.0.1/8 # 配置忽略检测的 IP (段)，添加多个需要用空格隔开
enabled = true
filter = sshd
port = 22 # 端口，改了的话这里也要改
maxretry = 5 # 最大尝试次数
findtime = 300 # 多少秒以内最大尝试次数规则生效
bantime = 600 # 封禁多少秒，-1是永久封禁（不建议永久封禁）
action = %(action_)s[port="%(port)s", protocol="%(protocol)s", logpath="%(logpath)s", chain="%(chain)s"] # 不需要发邮件通知就这样设置
banaction = iptables-multiport # 禁用方式
logpath = /var/log/auth.log # SSH 登陆日志位置
```

注意ubuntu与debian需要添加`backend=systemd`, 否则fail2ban找不到日志文件导致启动失败

`:wq`保存退出，然后重启服务生效

```bash
systemctl restart fail2ban
systemctl status fail2ban
```

## 4.2 安装rsyslog：auth.log 日志文件依赖rsyslog

```bash
# 安装 rsyslog
sudo apt-get update
sudo apt-get install rsyslog

# 启动服务
sudo systemctl start rsyslog
sudo systemctl enable rsyslog  # 设置开机自启

# 检查服务状态
sudo systemctl status rsyslog
```

## 4.3 内存节省选项(可选)

详见: [maxmatches: memory saving options by sebres · Pull Request #2402 · fail2ban/fail2ban](https://github.com/fail2ban/fail2ban/pull/2402)

在网站或服务器被DDoS时，由于`findtime`和`bantime`很长，可能导致 fail2ban 在访问量较大的网站上产生大量内存消耗，解决方式就是配置中添加类似这样:  
`fail2ban.local:`

```bash
[DEFAULT]
dbmaxmatches = 0
dbpurgeage = 20d
```

`jail.local:`

```bash
[DEFAULT]
maxmatches = 0
```

## 4.4 验证配置是否生效

`tail -f /var/log/fail2ban.log`或者`fail2ban-client status sshd`或者

就能看到目前正在爆破你和ban掉爆破ssh的ip日志，和目前已经封禁的ip了

```bash
# fail2ban-client status sshd

Status for the jail: sshd
|- Filter
|  |- Currently failed:    0
|  |- Total failed:    2
|  `- Journal matches:    _SYSTEMD_UNIT=sshd.service + _COMM=sshd
`- Actions
   |- Currently banned:    0
   |- Total banned:    0
   `- Banned IP list:    
```

## 4.5 解封

```bash
# 解封所有IP
fail2ban-client unban --all
# 解封指定IP
# fail2ban-client unban <IP> ... <IP>
fail2ban-client unban 1.1.1.1
#删除特定服务的(如sshd)被ban IP
fail2ban-client set sshd delignoreip 1.1.1.1
```
# 5 设置系统时间

将系统时间设置为北京时间，推荐使用 `timedatectl` 命令。

北京时间的正确时区标识符是 Asia/Shanghai。您看到的错误消息是因为该工具要求使用较旧、不太常见的 POSIX 格式，但现代系统使用 Area/Location 格式。

这是大多数现代 Linux 发行版（如 Ubuntu、Debian、CentOS 和 Fedora）的标准系统范围方法。

1. 设置时区
    
    在终端中运行以下命令。
```bash
sudo timedatectl set-timezone Asia/Shanghai
```
    
2. 验证更改
    
    可以通过运行以下任一命令来检查更改是否成功：
```bash
timedatectl
```
显示本地时间、世界标准时间 (UTC) 和当前设置的时区。
或者：
```bash
date
```
输出现在应该显示 CST（中国标准时间）和正确的时间。

# 6 通知服务器SSH登录

在登录 ssh 时候自动发通知到微信，以防偷家

可以通过 PAM 模块在每次ssh登录时触发脚本来实现。

脚本记得加可执行权限chmod +x 脚本路径

编辑`/etc/pam.d/sshd`，在文件末尾添加：

`session    optional    pam_exec.so 脚本路径`

对于提到的用例，脚本大致如下：  

```bash
#!/bin/bash

if [ "$PAM_TYPE" != "open_session" ]; then
    exit 0
fi

ip=$PAM_RHOST
date=$(date +"%e %b %Y, %a %r")
name=$PAM_USER

webhook_url="https://sctapi.ftqq.com/xxxxxx.send"

curl -s -X POST "$webhook_url" \
    -H "Content-Type: application/json" \
    -d "{
    \"title\": \"vps登录提醒\",
    \"desp\": \"> 登录用户: $name\\n> 客户端IP: $ip\\n> 登录时间: $date\"
}"
```

请根据自己的用例替换api及调用方式。

参照：

[Server酱](https://github.com/easychen/serverchan-demo)

[群机器人配置说明 - 文档 - 企业微信开发者中心](https://developer.work.weixin.qq.com/document/path/91770)


# 7 创建非 root 账户

使用以下命令创建一个具有提权能力的账户：

```bash
useradd -m -G sudo -s /bin/bash 用户名
```

`-m`选项会在创建用户账号的同时，为用户创建一个家目录。如果不使用`-m`选项，用户账号会被创建，但不会自动创建家目录。

`-s` 选项用于指定用户账号的登录 shell。当您使用 `useradd` 命令创建用户时，通过 `-s` 选项可以指定用户的默认 shell。默认情况下，如果不指定 `-s` 选项，新用户的默认 shell 可能会根据系统的默认设置而定。

`-G` 选项用于指定要将新用户添加到的附加组。当您使用 `useradd` 命令创建用户时，通过 `-G` 选项可以将新用户添加到一个或多个附加组中，逗号隔开。

然后我们给这个用户设置一个至少为 16 位的随机大小写字母 + 数字的密码（个人建议的最低安全性需求）：

```bash
passwd 用户名
```

- 如果想sudo时不输入密码，那么将**用户移出sudo用户组**（不移除将继续继承sudo的权限，需要输入密码），并执行以下代码
  
  1. 打开终端，并以 root 用户身份编辑 `/etc/sudoers` 文件。最好使用 `visudo` 命令进行编辑，因为它可以确保语法正确性：
  
```bash
sudo visudo
```
  
  2. 在打开的文件中找到类似 `root ALL=(ALL:ALL) ALL` 的行，这是一个 sudoers 条目的示例。
  
  3. 在文件中添加一行，类似于下面的内容：
  
```bash
用户名 ALL=(ALL) NOPASSWD: ALL
```
  
  这行的含义是允许用户 在任何主机上，以任何用户的身份，运行任何命令，并且无需输入密码。
  
  4. 保存并关闭文件。在 visudo 中，按 `Ctrl + X`，然后按 `Y` 保存更改。

注意如果用户没移出**sudo用户组**，即使按上述代码修改也会要求输入密码，除非你将**sudo用户组**也设为NOPASSWD

# 8 ssh设置密钥登陆

执行以下命令编辑 SSH 配置文件, 也可改/etc/ssh/sshd_config.d/里的配置（见1）：

```bash
sudo vim /etc/ssh/sshd_config
```

进行如下设置：

```bash
Match User 用户名
    AuthorizedKeysFile .ssh/authorized_keys
    PubkeyAuthentication yes
    PasswordAuthentication no
```

之后重启 SSH 服务生效：

```bash
sudo systemctl restart ssh
```

# 9 禁用 root SSH 密码远程登陆

执行以下命令编辑 SSH 配置文件, 也可改/etc/ssh/sshd_config.d/里的配置（见1）：

```bash
sudo vim /etc/ssh/sshd_config
```

执行以下命令编辑 SSH 配置文件：

```bash
# 禁止 Root 用户通过密码远程登录
PermitRootLogin prohibit-password
```

之后重启 SSH 服务生效：

```bash
sudo systemctl restart ssh
```

为什么不设置成no：

- 直接禁掉root登录`PermitRootLogin no`我也不常用，更习惯和其他sudoer一样配密钥然后禁止密码登录仅允许密钥登录。

# 10 禁止icmp

ufw本身没有直接支持阻止icmp协议的命令。
一般来说不建议禁止icmp，这只是为了测服务器连通性的，不会涉及到具体端口的服务
要改的话一般使用iptables改，也可以去修改ufw的配置实现

# 11 参考资料：

https://linux.do/t/topic/267502
https://linux.do/t/topic/817769