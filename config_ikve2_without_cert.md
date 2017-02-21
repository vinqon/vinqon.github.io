

### Strongswan 配置免证书IKEv2 VPN


之前根据[CentOS/Ubuntu一键安装IPSEC/IKEV2 VPN服务器](https://quericy.me/blog/699/) 搭建一个VPN服务器，这套方案里面提供了两种方式连接IPSec（即IKEv1）和IKEv2，前者可以通过帐号、密码以及密钥PSK 即可连接，比较方便。后者需要证书，比较麻烦。所以我使用了IPsec作为产品的最终方案。

最近突然想起这件事情，加上之前调研的实践让我记得IKEv2方案其实更加安全并且稳定。『稳定』的表现是，可以在切换网络依然保持VPN不断开，这是一种不错的体验。所以，打算把目前方案迁移为IKEv2，所以做了一下调研。

首先是『无证书』问题。对于IKEv2，之前有了解到是可以实现无证书连接，这点很重要，不然得引导用户安装证书才能连接，体验会很差。今天用了蛮多时间调研，发现了自己之前一个误区，无证书并不是指完全不用证书，而是是指客户端连接之前不需要配置证书，但服务器端依然需要。

简单来说，IKEv2有两种连接方案，[CentOS/Ubuntu一键安装IPSEC/IKEV2 VPN服务器](https://quericy.me/blog/699/) 一文中也有提到：
1.  服务器使用自签发证书，客户端需要提前安装配套证书
2. 服务器使用CA证书，则客户端则可免证书

根据[用 strongSwan 搭建免证书的 IKEv2 VPN](https://zorro.im/strongswan-ikev2-for-ios-without-certificate/) 一文描述，确实可以通过PSK代替证书实现服务器和客户端都无证书的方案，但是这个方案在设置GUI中无法配置，只能通过添加描述文件来配置，挺扯蛋的。另外，也不知道通过程序是否可以配置，如果可以，这个方案应该是最优的。

下文只讨论通过CA证书实现客户端免证书配置IKEv2。

首先，弄个CA证书，CA证书很多免费的啦，这边查到了比较多人使用的是Let’s Encrypt SSL，但是有一个缺点就是需要定时续期。以下是配置步骤：

**1.安装Let's Encrypt SSL的客户端**
```
git clone https://github.com/letsencrypt/letsencrypt
cd letsencrypt
./letsencrypt-auto --help
```

**2.执行客户端申请证书，按照提示输入邮箱和域名就完成了**
```
./letsencrypt-auto certonly --server https://acme-v01.api.letsencrypt.org/directory --agree-dev-preview
```
![enter image description here](http://vinqon.com/fms2/ROOT/image/certshell.jpg)

**3.证书三个月后就过期，可以设定一个定时器，执行以下续期脚本**
```
sudo ~/.local/share/letsencrypt/bin/letsencrypt renew  
```

**4.把证书全部复制到strongswan一键安装脚本目录**
```
cp chain.pem /root/ca.cert.pem # 包括根证书的链证书
cp cert.pem /root/server.cert.pem # 服务器证书
cp privkey.pem /root/server.pem # 服务器私钥
```

**5.然后就是安装Strongswan了，注意两点：**  

1) 证书选择自己导入证书，否则脚本会自己生成自签发证书
	
![enter image description here](http://vinqon.com/fms2/ROOT/image/strongswan.jpg)

2) 安装完之后，需要去/usr/local/etc/ipsec.conf修改leftid，leftid需要和证书的域名保持一致，域名可以不指向本机IP，就是说一个有效证书可以用在多个Strongswan VPN的服务器上

![enter image description here](http://vinqon.com/fms2/ROOT/image/leftid.jpg)

6. 启动Strongswan之后，就用手机连接了

![enter image description here](http://vinqon.com/fms2/ROOT/image/iphoneconfig2.jpg)

#### Reference:

[IKEv2 Configuration Profile for Apple iOS 8 and newer](https://wiki.strongswan.org/projects/strongswan/wiki/AppleIKEv2Profile)
[Strongswan IKEV2免导入证书配置及调试笔记](https://linsir.org/post/Strongswan-not-import-certs-and-some-tips)
[用 strongSwan 搭建免证书的 IKEv2 VPN](https://zorro.im/strongswan-ikev2-for-ios-without-certificate/)
[申请免费的Let’s Encrypt SSL证书](https://shanbin.name/lets-encrypt-ssl-certificates-for-free/)
