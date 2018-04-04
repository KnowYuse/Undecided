# 使用DigitalOcean和SS搭建梯子

---

### **前言**

这篇文章记录了如何创建自己的服务器和安装与配置SS以达到翻墙的目的。

### **注册账号**

DigitalOcean的服务器最低配置价格为\$5/mo，通过[此链接](https://m.do.co/c/0659a88d8588)注册账号可额外获得10\$(如果能申请到GitHub的[学生优惠](https://education.github.com/),可获得50\$的优惠券，但笔者并没有拿到，GitHub发邮件说会尽快修复这个bug)。

注册后选择支付方式，可通过信用卡或者PayPal进行支付(笔者选择了PayPal，建议大家注册一个PayPal账号，很多国外网站都支持PayPal支付)，支付后就可以创建自己的服务器，同时在setting-billing里可看到自己的余额为15\$。

### **创建服务器**

点击Create Droplet选择服务器配置，此处笔者选择的是Ubuntu16.04.4 x64系统、\$5/mo套餐、Singapore节点**(节点选择后不可更改，如果感觉明显卡顿只能销毁当前服务器重新创建)**，附件选项建议选上Monitoring,如果想要在自己电脑上远程登录主机，可选择添加SSH key，如果不添加的话将会收到一封含有登录密码的邮件，此处笔者选择不添加。然后给自己服务器取名后点击create按钮，等待一两分钟就好。
**创建完后就能看到自己服务器的IP地址。**

### **添加新用户**

点击Console弹出控制台，输入账号名`root`和邮件中的密码，登录后再输入一次密码更改密码。
进入系统后默认用户为root用户。
**由于使用root账户有很大风险，因此建议创建一个新账户。**
输入`adduser 用户名`(用户名可随意设置)后输入密码并确认，然后填写个人信息(可输入回车跳过)，输入Y即创建完毕。

然后输入`vim /etc/sudoers`修改文件
```
# User privilege specification
root ALL=(ALL:ALL) ALL
用户名 ALL=(ALL:ALL) ALL
```
输入`:w!`保存，输入`:q`退出
输入`su 用户名`切换到新创建的用户名，输入`cd`切换到家目录

### **配置SS**

系统已经帮我们安装了`Python2`、`Python3`、`git`。
输入以下命令
```
sudo apt-get update
sudo apt-get install python3-pip
sudo pip3  install git+https://github.com/shadowsocks/shadowsocks.git@master
```
安装完成后输入
```
sudo ssserver -p 443  -k password -m aes-256-cfb --user nobody -d start
```
其中443为端口号，也可设置为其他端口
password为自己设置的密码

### **如何使用**

安卓用户可通过[此链接](https://pan.baidu.com/s/19b0X7RyJRvmkkNDsJXDWLw)下载客户端，密码为dp91

iPhone/iPad用户可在应用商店搜索SS，笔者使用的是FirstWingy，但**此应用为其他开发者开发，不保证其安全性**。

Windows用户可通过[此链接](https://github.com/shadowsocks/shadowsocks-windows/releases)下载客户端

OS X用户可通过[此链接](https://github.com/shadowsocks/shadowsocks-iOS/releases)下载客户端

### **后记**

通过上述方式只要有IP地址、端口号、密码即可使用，**流量限制为1TB，超过部分将额外收费**。

可使用配置文件来配置SS，也可设置多个端口并限定每个端口的流量。

除了翻墙服务器也可进行其他开发。

PS:本来想把这篇文章发在简书上的，但刚发布就被和谐，想来其他平台应该也是，此处不予置评。

参考链接就不写惹~






