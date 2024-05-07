# 如何购买VSP搭建属于自己的节点（以vultr为例的v2ray搭建教程）





## 购买VSP

![截屏2024-05-07 15.41.56](https://github.com/fakerUNcode/V2RAY_BUILDING_TUTORIAL_Newperson-friendly/blob/master/%E6%88%AA%E5%B1%8F2024-05-07%2014.58.48.png)

来到vultr官网并注册，新人充10刀送100刀，很划算。

点击deploy,选择 **cloud compute**,配置选最低配的就够用，backup根据自己的需要选择是否开启，系统推荐Ubuntu 22 lts x64,节点建议东京，大阪的似乎月流量很少，用了2G就提示我到23%了。东京节点100G每月。Vultr只计算OutBound流量，不会重复计算InBound流量，现在还算良心。

最低只需6刀一个月。

## 节点相关信息

![截屏2024-05-07 15.42.50](https://github.com/fakerUNcode/V2RAY_BUILDING_TUTORIAL_Newperson-friendly/blob/master/%E6%88%AA%E5%B1%8F2024-05-07%2015.42.50.png)

进入**server information**界面后，可以看到自己的服务器相关信息，如带宽使用量、IP地址，服务器用户名以及密码等，此时我们需要的信息🈶：username \password

## 连通性确认

我们使用ip测试网站测试服务器的连通性https://www.toolsdaquan.com/ipcheck/

在以上网站输入自己的ip地址以及端口号，端口号先使用22（SSH占用的端口号），以此测试SSH连接是否可用，如果显示IMCP \ TCP国内外皆可用，则说明服务器连通性正常，可以进行下一步工作。

若显示国外IMCP\TCP可用，国内不可用，说明该IP已被封禁，更换服务器即可，只需将当前服务器stop service,然后deploy新服务器即可（顺序不要错，不然可能得到和刚刚相同的IP地址）

若显示国内外IMCP可用，TCP不可用，可能是服务器防火墙打开，使用服务器界面的console登录自己的用户密码进行操作关闭防火墙：

使用命令：

```
ufw disable
```

（注：IMCP是网际报文控制协议，可以支持PING服务，若22端口的PING不通，说明该IP的连接失效请更换，若22端口通，其他端口不通该IP可以通过操作抢救）

## ssh连接服务器操作

### 安装v2ray

SSH（Secure Shell）是一种用于安全远程访问计算机网络服务的协议。它提供了一个加密的、可靠的通信通道，以确保在网络中传输的数据的机密性和完整性。

SSH远程连接服务器，笔者使用Macos,使用终端命令即可（Linux也可以），windows可以借助putty等工具，详细的方法可以参考网上的链接教程。

如果以上工具都不适用，则可以尝试vultr自带的console(在sever infomation界面)操作和下面的教程类似，但是需要使用屏幕左侧的粘贴板才能粘贴命令，无法使用快捷键，这点需要注意。

![截屏2024-05-07 14.39.44](https://github.com/fakerUNcode/V2RAY_BUILDING_TUTORIAL_Newperson-friendly/blob/master/%E6%88%AA%E5%B1%8F2024-05-07%2014.39.44.png)

如图，以Macos为例，输入命令：

```
ssh root@xx.xx.xx.xx
```

root即为用户名，xx.xx.xx.xx是你购买的服务器的ip地址。

接下来会提示输入密码，从服务器信息粘贴输入即可：

![截屏2024-05-07 14.58.48](https://github.com/fakerUNcode/V2RAY_BUILDING_TUTORIAL_Newperson-friendly/blob/master/%E6%88%AA%E5%B1%8F2024-05-07%2014.58.48.png)

显示上图信息说明连接成功。

接下来使用一键安装脚本命令安装v2ray:

```
bash <(wget -qO- -o- https://git.io/v2ray.sh)
```

安装成功后会看见如下界面：

![V2Ray 脚本安装完成](https://camo.githubusercontent.com/f0b343c8aa7fe2642f21090b2bc9fc6bc34e6e997cff0790565af344b0c6633f/68747470733a2f2f766970322e6c6f6c692e696f2f323032332f30352f31312f5745387155735a44677678545661642e706e67)

port是你的节点要使用的端口，以及一些其他重要的参数。此时可以试试这个端口是否可通，**若IMCP通，国内TCP不通，说明端口可能被禁用，按照命令提示根据自己的需要自定义端口即可。**如果国内外都可通说明节点已经搭建成功了。

复制URL导入到**客户端**(自己的电脑）的v2rayN即可开始使用。

### 时钟同步

V2ray 服务器使用的是 Vmess 协议，客户端的时间和 VPS 的时间不一致，就会导致不能正常使用。

同样是在SSH连接下，以中国大陆所用东八区时间为例：

```
date -R
```

以上命令可以查看服务器的时间，如果和用户机的时间不同则需要设置时间同步才可用。

```
rm -rf /etc/localtime
```

```
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

以上命令清除服务器的本机时间并设置时间为东八区。

设置后查看时间，如果和用户主机时间一致则可以正常使用。

### 网络的拥塞控制 （BBR算法安装）

Ubuntu使用以下命令可以安装Google的BBR算法（一种TCP拥塞控制算法）：

```
apt-get update -y && apt-get install wget -y && apt-get install curl -y
```

以上命令更新服务器系统及脚本所依赖的安装包。

```
wget -N --no-check-certificate "https://raw.githubusercontent.com/chiakge/Linux-NetSpeed/master/tcp.sh" && chmod +x tcp.sh && ./tcp.sh
```

以上命令一键安装BBR。

部分系统出现警告提示键盘移到No上即可。

![Linux更换内核提示](https://www.linuxv2ray.com/wp-content/uploads/2022/05/1651652385-abort-kernel-removal-1024x683.jpg)



![BBR切换内核为BBR-PLUS](https://www.linuxv2ray.com/wp-content/uploads/2022/05/1651652380-bbr5in1-one-click-script-2-1024x683.jpg)

安装成功即可选择一个选项使用，添加使用BBRplus。

若提示证书错误可以使用命令：

```
apt-get -y install ca-certificates
```

若需要切换加速选项可以使用以下命令：

```
 ./tcp.sh
```