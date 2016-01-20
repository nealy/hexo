title: setup vpn server on centos by strongswan
date: 2016-01-20 15:47:12
updated: 2016-01-20 15:47:12
tags: 在CentOS/Ubuntu下搭建 IPSec/IKEv2 VPN
categories: Others
---

本文介绍如何利用StrongSwan一步步在CentOS服务器（或者VPS主机）上搭建自己的VPN服务器，支持IPSec协议和最新的IKEv2协议，支持iOS、OS X、Windows7、Android、Linux等，iOS9、OS X10.10以上版本已支持IKEv2协议。
<!-- more -->

VPN （Virtual Private Network），是一种常用于连接中、大型企业或团体与团体间的私人网络的通讯方法。可在不安全的网络中传送可靠、安全的信息，或者透过服务器读取被限制的外界资源，俗称“翻墙”。搭建VPN的服务器配置要求不高，一般的VPS主机即可满足，国外有不少物美价廉的主机可供选择，比如Bandwagon、Host US等。本文以搭载了CentOS6.x的VPS主机为例，其他系统可自行修改相应的命令。

###安装StrongSwan
StrongSwan是一个完整的2.4和2.6的Linux内核下的IPsec和IKEv1 的实现，当然也完全支持新的IKEv2协议的Linux 2.6内核。可以手动到[官网strongswan.org](https://www.strongswan.org)下载发行包，也可以直接编译源码安装，编译源码的好处是可以指定不同的配置来满足我们的需求。

编译并安装：
```
# 若不指定版本号(strongswan.tar.gz)将始终下载最新版本
wget https://download.strongswan.org/strongswan-5.3.5.tar.gz
tar xzf strongswan.tar.gz
cd strongswan-*
./configure --sysconfdir=/etc --enable-openssl --enable-nat-transport --disable-mysql --disable-ldap --disable-static --enable-shared --enable-md4 --enable-eap-mschapv2 --enable-eap-aka --enable-eap-aka-3gpp2 --enable-eap-gtc --enable-eap-identity --enable-eap-md5 --enable-eap-peap --enable-eap-radius --enable-eap-sim --enable-eap-sim-file --enable-eap-simaka-pseudonym --enable-eap-simaka-reauth --enable-eap-simaka-sql --enable-eap-tls --enable-eap-tnc --enable-eap-ttls
make && make install
```
>注意，要编译StrongSwan需要gcc等工具，这里默认系统已经安装，如果没有可使用下述命令一次安装所有依赖：
>yum -y install pam-devel openssl-devel make gcc

###证书配置

 - 生成CA证书的私钥，并使用私钥签名CA证书
```
ipsec pki --gen --outform pem > ca.pem
ipsec pki --self --in ca.pem --dn "C=CN, O=VPN, CN=VPN CA" --ca --lifetime 3650 --outform pem >ca.cert.pem
```
>C 表示国家名，O 表示组织名，CN 表示通用名

 - 生成服务器证书的私钥，并用CA证书签发服务器证书
```
ipsec pki --gen --outform pem > server.pem
ipsec pki --pub --in server.pem | ipsec pki --issue --lifetime 1200 \
    --cacert ca.cert.pem \
    --cakey ca.pem --dn "C=CN, O=VPN, CN=${server_name}" \
    --san="10.0.0.0" \
    --flag serverAuth --flag ikeIntermediate \
    --outform pem > server.cert.pem
```
>- ${server_name}替换成你自己的CN。这里的CN仍然表示通用名，建议使用服务器的 IP 地址或 URL。
>- iOS 客户端要求 CN 必须是你的服务器的 IP 地址或 URL。