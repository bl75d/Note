
# 使用AWS-EC2搭梯子
    Reference: 
    https://celerysoft.github.io/2016-01-15.html
    https://fengooge.blogspot.com/2019/01/build-Shadowsocks-service-in-AWS-EC2.html
### 前言
作为研究学术和技术的，必须得使用Google，但是国家为了阻止天朝喷子为害天下，就架起了一道长城困住他们。作为被误伤的我们，想要走出长城，就得自己搭建梯子爬出去。购买VPN是非常危险的，很容易就被误伤，所以最好的方法是购买小众VPS搭建自己的海外服务器，防止被误伤。作为第一次捣鼓这些的人，我是十分不建议上来就花个几十美金购买VPS的，最好的方案是找一个免费的服务器来练练手，那么，亚马逊就站出来了，他家的Amazon Web Services.
## 流程：
### 1.注册AWS账户并且启动EC2实例
注册AWS账户->进入AWS Management Console->选择launch a vitual machine->选择Amazon Linux 2 AMI（为例）->instance type:t2.micro->其他默认->launch

***最后确定开始审核。这时候会提示生成密钥对(.pem文件)，这个很重要，一定要保存好，没有这个密钥对是无法远程登录管理你的服务器的，所以一定要保存好.

在Console选中实例->启动实例->连接

### 2.连接配置服务器端应用和安全设置
#### (Mac OS 系统)通过SSH连接到服务器

首先，打开终端，使用chmod命令确保私有密钥不是公开可见的：

sudo chmod 400 /你.pem文件的路径/lbxvpn.pem 是刚才下载的密钥对.pem文件的路径。

然后，通过ssh命令连接到服务器

    sudo ssh -i /你.pem文件的路径/lbxvpn.pem ec2-user@ec2-18-117-183-6.us-east-2.compute.amazonaws.com
ec2-user 是用户名，Amazon Linux AMI默认的是ec2-user，

ec2-18-117-183-6.us-east-2.compute.amazonaws.com 是你的服务器的公有DNS，这些信息右键点击你的实例，点击连接，弹出的提示框里都写着。

注意：每次重新启动实例之后，@ 后面的字符串部分都会变更，需要在 AWS 的说明页面进行更改再连接。

登录成功后，你能看到如下响应信息：
The authenticity of host 'ecc2-18-117-183-6.us-east-2.compute.amazonaws.com (10.254.142.33)'
can't be established.
RSA key fingerprint is 1f:51:ae:28:bf:89:e9:d8:1f:25:5d:37:2d:7d:b8:ca:9f:f5:f1:6f.
Are you sure you want to continue connecting (yes/no)?
输入yes，然后按回车。

#### 安装 Shadowsocks 服务：

    sudo yum install -y python-setuptools
    
    sudo easy_install pip
    
    sudo pip install shadowsocks

***如果报错 ImportError: No module named typing*** 尝试使用：sudo pip3 install shadowsocks

### 配置 Shadowsocks 服务
使用shell命令行：

    mkdir /etc/shadowsocks 
    #创建 Shadowsocks 目录：
    sudo vim /etc/shadowsocks/config.json
    #创建配置文件

在打开的配置文件中输入以下信息：

{

    "server": "0.0.0.0",
    "server_port": 443,
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "password": "**你的密码**",
    "timeout": 300,
    "method": "aes-256-cfb",
    "fast_open": false,
    "workers": 1
}

上面的「server_port」为服务端口号，默认 443 就可以了，当然换成其它的也行。「password」一栏设置成自己需要的密码。其它信息均默认。编辑完输入「:wq」回车即保存退出。

### 启动Shadowsocks服务器
    sudo /usr/local/bin/ssserver -c /etc/shadowsocks/config.json -d start

**注意上面/usr/local/bin部分是 Shadowsocks 服务所在路径，如果报错请根据自己的实际情况调整。

停止和重启命令：

    sudo /usr/local/bin/ssserver -c /etc/shadowsocks/config.json -d stop
    sudo /usr/local/bin/ssserver -c /etc/shadowsocks/config.json -d restart

#### 设置开机启动

    sudo vi /etc/rc.local
    sudo /usr/local/bin/ssserver -c /etc/shadowsocks.json -d start

### 在亚马逊 AWS-EC2 实例控制面板中设置防火墙规则
    把实例信息的显示滑块拖到右边，点击「安全组」下面的超链接
    在打开的页面中一次点击 操作——编辑入站规则-添加HTTPS
    在弹出的端口信息中，端口范围填入「443」（根据自己的实际情况填写端口号），其它默认
到此，整个 Shadowsocks 服务端的搭建工作就完成了，我们可以使用 Shadowsocks 客户端工具来翻墙上网了。

### 3.Shadowrocket连接自由上网
下载Shadowrocket（US Apple store）:

Add server：

    ip:ec2实例的外网IP
    port:443
    password:你设定的密码
连接即可

https://shadowsockshelp.github.io/Shadowsocks/ios.html
