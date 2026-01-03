---
title: 使用Frp进行coder-server部署
date: 2024-07-06 00:34:03 +08:00
filename: 2024-07-06-Frp-coder-server
categories:
  - Misc
tags:
  - Environment
  - Server
dir: Misc
share: true
---

## 步骤概览

1. 首先在本地机上运行code- server。记住端口，待会配置文件要用。

2. 提前先将域名解析到公网服务器，用A类就行，解析好了之后申请一个ssl证书，将证书下载下来，后续要用。

3. 接着配置frp

4. 配好frp之后，现在应该可以通过你设置的域名+端口的方式访问到服务

5. 使用nginx反向代理实现直接用域名访问。

**以下提到的端口都要在安全组里开放!!!**
## code-serve本地运行

首先在[Releases · coder/code-server (github.com)](https://github.com/coder/code-server/releases)上下载最新的包，根据本地机环境选择合适的版本,我个人服务器和本地机器都是使用的Ubuntu，Debian系的小伙伴可以参考我下面的安装方式，其他不同的安装方式[请看这里](https://coder.com/docs/code-server/install)

下载完成之后，传到你的本地服务器上，随便放一个目录都行，然后进入该目录下执行如下命令

>推荐切换到root用户来执行该操作

```shell
sudo dpkg -i 你下载的文件名
sudo systemctl enable --now code-server@$USER
```

这样的话，code-server就安装完成并设置好开机启动了，可执行文件在`/bin/code-server`

接下来我们就要配置code-server了,访问`~/.config/code-server/config.yaml`这个目录下的文件，这就是配置文件

```yaml
bind-addr: 0.0.0.0:2333
auth: password #这是验证方式，我们选择用密码验证
password: xxxxxx #这里输入你的密码就行了
```

再次运行`sudo systemctl enable --now code-server@$USER`就可以通过你的本地机ip+2333来访问code-server了，但是现在我们还没有加ssl证书，所以是http访问了，在http下，code-server上一些插件使用不了的，所以下一步我们需要开启https访问

## 域名解析以及ssl证书配置

域名直接用A类记录解析到你的公网ip

然后去宝塔面板，新建站点，域名就填你刚刚解析的域名，然后点击申请ssl，用Let's Encrypt申请一下就好了，完成之后点击下载证书，我们需要用到公钥和私钥。

解压我们刚刚下载的证书，打开其他证书，发现有两个文件，fullchain.pem和private.key，将这两个文件复制到本地机器的code-server配置文件的文件夹内，例如`~/.config/code-server`，复制到这里面,现在这个文件夹下就有三个文件了，`config.yaml`，和刚刚复制过来的文件

接着我们继续配置code-server，点进去，增加cert

假如你也是用root安装，那么路径会和我一样，如果是用用户安装，则在对应用户的.config目录下

```yaml
cert: /root/.config/code-server/fullchain.pem
cert-key: /root/.config/code-server/privkey.key
```

因为我们挂的是域名的ssl证书，所以我们还需要将服务穿透到公网上用域名访问才能看见https生效，接下来我们就开始配置穿透。

## frp配置

### 服务端

将frps和frps.toml两个文件放置到服务端任意文件夹，我这里放在`/home/frp`路径下

服务端toml的填写

```toml
bindPort = 7000 # frp服务端口
vhostHTTPPort = 18080 # http端口
vhostHTTPSPort = 18443 # https端口

#配置frp仪表盘
webServer.addr = "0.0.0.0"
webServer.port = 7500
webServer.user = "换成你的用户名，双引号不要去掉"
webServer.password = "换成你的密码，双引号不要去掉"
```

接着我们来设置开机自启，这里就用官方给出的方法

1. **安装 systemd**
    
    如果您的 Linux 服务器上尚未安装 systemd，可以使用包管理器如 `yum`（适用于 CentOS/RHEL）或 `apt`（适用于 Debian/Ubuntu）来安装它：
    
```shell
# 使用 yum 安装 systemd（CentOS/RHEL）
yum install systemd

# 使用 apt 安装 systemd（Debian/Ubuntu）
apt install systemd
```
    
2. **创建 frps.service 文件**
    
    使用文本编辑器 (如 vim) 在 `/etc/systemd/system` 目录下创建一个 `frps.service` 文件，用于配置 frps 服务。
    
    `$ sudo vim /etc/systemd/system/frps.service`
    
    写入内容
    
```
[Unit]
# 服务名称，可自定义
Description = frp server
After = network.target syslog.target
Wants = network.target

[Service]
Type = simple
# 启动frps的命令，需修改为您的frps的安装路径
ExecStart =/home/frp/frps -c /home/frp/frps.toml
Restart=always
RestartSec=30
User=root
Group=root

[Install]
WantedBy = multi-user.target
```
    
3. **使用 systemd 命令管理 frps 服务**
    
```shell
# 启动frp
sudo systemctl start frps
# 停止frp
sudo systemctl stop frps
# 重启frp
sudo systemctl restart frps
# 查看frp状态
sudo systemctl status frps

```
    
4. **设置 frps 开机自启动**
    
```shell
sudo systemctl enable frps
```
    

通过遵循上述步骤，您可以轻松地使用 systemd 来管理 frps 服务，实现启动、停止、自动运行和开机自启动。确保替换路径和配置文件名称以匹配您的实际安装。

### 客户端

将frpc和frpc.toml两个文件放置到服务端任意文件夹，我这里放在`/home/frp`路径下

客户端toml的填写

```toml
serverAddr = "x.x.x.x" # 替换为你的服务器公网ip
serverPort = 7000

[[proxies]]
name = "ssh"
type = "tcp"
localIP = "127.0.0.1"
localPort = 22
remotePort = 6000

[[proxies]]
name = "coder"
type = "https"
localPort = 2333 # 刚刚我们code-server的端口
customDomains = ["替换成你的域名"]
```

同样的，我们配一下自启动服务

2. **创建 frpc.service 文件**
    
    使用文本编辑器 (如 vim) 在 `/etc/systemd/system` 目录下创建一个 `frpc.service` 文件，用于配置 frpc 服务。
    
    `$ sudo vim /etc/systemd/system/frpc.service`
    
    写入内容
    
```
[Unit]
# 服务名称，可自定义
Description = frpc server
After = network.target syslog.target
Wants = network.target

[Service]
Type = simple
# 启动frpc的命令，需修改为您的frpc的安装路径
ExecStart =/home/yu/frp/frpc -c /home/yu/frp/frpc.toml
Restart=always
RestartSec=30
User=root
Group=root

[Install]
WantedBy = multi-user.target
```
    
3. **使用 systemd 命令管理 frps 服务**
    
```shell
# 启动frp
sudo systemctl start frpc
# 停止frp
sudo systemctl stop frpc
# 重启frp
sudo systemctl restart frpc
# 查看frp状态
sudo systemctl status frpc

```
    
4. **设置 frps 开机自启动**
    
```shell
sudo systemctl enable frpc
```
    


至此，frp以及配置完成，你应该可以通过你的域名+端口(18080或者18443)来访问网页了，但是这种方法太不优雅了，所以我们下面要进行nginx反向代理的配置

## Nginx反向代理

在服务端宝塔面板的站点设置那里，新增反向代理，将目标url填写为你的域名+刚刚配置的frps端口(18080或者18443)，因为我是https，所以使用域名+18443。发送域名不用管，会自动填好，代理名称随便写一个

这样配置完成之后，通过域名直接访问，大概率会报错的

>Frp的http穿透是通过域名加端口的方式来访问，直接用公网ip加端口是无法访问的，所以在反向代理的时候一定要注意。但是你设置的域名，在nginx反向代理的过程中，还是会给你转换成ip进行处理。

解决思路：反向代理时候，通过域名访问而不是IP访问。

查询文档，发现可以使用配置项：`proxy_ssl_server_name`，该配置项默认值是off，需要将一些内容写到配置文件中：

点进反向代理配置文件中，添加一行`proxy_ssl_server_name on;`即可，现在能愉快的使用域名来访问code-server了

[参考资料1](https://blog.csdn.net/wanger5354/article/details/131934728?ops_request_misc=&request_id=&biz_id=102&utm_term=nginx反向代理%20502&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-3-131934728.142^v100^pc_search_result_base5&spm=1018.2226.3001.4187)

[参考资料2](https://longsheng.org/post/20639.html)