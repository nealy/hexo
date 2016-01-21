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
./configure --enable-openssl --enable-nat-transport --disable-mysql --disable-ldap --disable-static --enable-shared --enable-md4 --enable-eap-mschapv2 --enable-eap-aka --enable-eap-aka-3gpp2 --enable-eap-gtc --enable-eap-identity --enable-eap-md5 --enable-eap-peap --enable-eap-radius --enable-eap-sim --enable-eap-sim-file --enable-eap-simaka-pseudonym --enable-eap-simaka-reauth --enable-eap-simaka-sql --enable-eap-tls --enable-eap-tnc --enable-eap-ttls
make && make install
```
>注意，要编译StrongSwan需要gcc等工具，这里默认系统已经安装，如果没有可使用下述命令一次安装所有依赖：
>yum -y install pam-devel openssl-devel make gcc

###证书配置

1. 生成CA证书的私钥，并使用私钥签名CA证书

    ```
    ipsec pki --gen --outform pem > ca.pem
    ipsec pki --self --in ca.pem --dn "C=com, O=vpn, CN=VPN CA" --ca --outform pem > ca.cert.pem
    ```
>C 表示国家名，O 表示组织名，CN 表示通用名

2. 生成服务器证书的私钥，并用CA证书签发服务器证书

    ```
    ipsec pki --gen --outform pem > server.pem
    ipsec pki --pub --in server.pem | ipsec pki --issue --cacert ca.cert.pem \
        --cakey ca.pem --dn "C=com, O=vpn, CN=${server_name}" \
        --san="${server_name}" \
        --flag serverAuth --flag ikeIntermediate \
        --outform pem > server.cert.pem
    ```
>- C和O的值必须与CA证书的一致。CN和san建议使用服务器的 IP 地址或 URL，san可设置多个
>- iOS 客户端要求 CN 必须是你的服务器的 IP 地址或 URL。
>- Windows 7 除了对CN的要求之外，还要求必须显式说明这个服务器证书的用途（如用于与服务器进行认证），–flag serverAuth。
>- 非 iOS 的 Mac OS X 要求了”IP 安全网络密钥互换居间（IP Security IKE Intermediate）“这种增强型密钥用法（EKU），–flag ikdeIntermediate。
>- Android 和 iOS 都要求服务器别名（serverAltName）为服务器的 URL 或 IP 地址，–san。

3. 生成客户端证书

    ```
    ipsec pki --gen --outform pem > client.pem
    ipsec pki --pub --in client.pem | ipsec pki --issue --cacert ca.cert.pem \
        --cakey ca.pem --dn "C=com, O=vpn, CN=VPN Client" \
        --outform pem > client.cert.pem
    ```

4. 生成 pkcs12 证书

    ```
    openssl pkcs12 -export -inkey client.pem -in client.cert.pem -name "client" -certfile ca.cert.pem -caname "VPN Client"  -out client.cert.p12
    ```

5. 安装证书

    ```
    cp -r ca.cert.pem /usr/local/etc/ipsec.d/cacerts/
    cp -r server.cert.pem /usr/local/etc/ipsec.d/certs/
    cp -r server.pem /usr/local/etc/ipsec.d/private/
    cp -r client.cert.pem /usr/local/etc/ipsec.d/certs/
    cp -r client.pem  /usr/local/etc/ipsec.d/private/
    ```

###StrongSwan配置

####**ipsec配置文件**
```/usr/local/etc/ipsec.conf```
```
config setup
    uniqueids=never
conn %default
    keyexchange=ike
    left=%any
    leftsubnet=0.0.0.0/0
    right=%any
conn IKE-BASE
    leftcert=server.cert.pem
    rightsourceip=10.0.0.0/24
# ios etc.
conn by_cert
    also=IKE-BASE
    keyexchange=ikev1
    fragmentation=yes
    leftauth=pubkey
    leftsubnet=0.0.0.0/0
    rightauth=pubkey
    rightauth2=xauth
    rightcert=client.cert.pem
    auto=add
# ios etc.
conn by_psk
    also=IKE-BASE
    keyexchange=ikev1
    leftauth=psk
    rightauth=psk
    rightauth2=xauth
    auto=add
# osx linux android etc.
conn by_key
    also=IKE-BASE
    keyexchange=ikev2
    leftauth=pubkey
    rightauth=pubkey
    rightcert=client.cert.pem
    auto=add
# ikev2 (ios osx win7 etc.)
conn IKEv2-EAP
    also=IKE-BASE
    keyexchange=ikev2
    ike=aes256-sha256-modp1024,3des-sha1-modp1024,aes256-sha1-modp1024!
    esp=aes256-sha256,3des-sha1,aes256-sha1!
    rekey=no
    leftid=${my_cert_cn}
    leftauth=pubkey
    leftsendcert=always
    rightfirewall=yes
    rightsendcert=never
    rightauth=eap-mschapv2
    eap_identity=%any
    dpdaction=clear
    fragmentation=yes
    auto=add
```

####**strongswan配置文件**
```/usr/local/etc/strongswan.conf```
```
charon {
    load_modular = yes
    duplicheck.enable = no #闭冗余检查以同时连接多个设备
    compress = yes
    plugins {
        include strongswan.d/charon/*.conf
    }
    dns1 = 8.8.8.8
    dns2 = 8.8.4.4
    # for windows
    nbns1 = 8.8.8.8
    nbns2 = 8.8.4.4
}
include strongswan.d/*.conf
```

####**密码认证文件**
```vi /usr/local/etc/ipsec.secrets```
```
: RSA server.pem
: PSK "myPSKpw"
: XAUTH "myXAUTHpw"
myUserName %any : EAP "myUserPassWord"
```
>Windows Phone 8.1 设备因其使用EAP连接时会自动在用户名之前加上设备名称，因此若要用于WP8.1设备应追加一条EAP配置：
>```设备名称\用户名 : EAP "密码"```
>其中，设备名称可在“设置-关于-手机信息”中查看。

####**启动StrongSwan**
```ipsec start```


###iptables配置
