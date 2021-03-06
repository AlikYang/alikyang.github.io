---
layout: post
title: 加密解密算法、PKI及SSL、创建私有CA
date: 2017-11-10
categories: blog
tags: [数据加密,网络安全]
description: 
---

<h3 id="toc_0">对称加密、非对称加密、单向加密、密钥交换</h3>


对称加密：在加密和解密的过程中使用相同的密钥；

	算法：
	DES：Data Encryption Standard
	3DES：
	AES：Advanced (128bits, 192bits, 256, 384, 512bits)
	Blowfish
	Twofish
	IDEA
	RC6
	CAST5
	缺点：
	    1、密钥过多：每一个用户的连接都要使用一个单独的密钥，防止其他连接到服务器的用户知晓别人的密码
	    2、密钥分发困难：在向用户传输密钥时候很难保证整个过程的安全性


非对称加密：密钥分为一对公钥和私钥，在加密过程中使用公钥加密的数据，只能用私钥解密，反之亦然

    pubkey：可以公开给所有人，从私钥中提取
    secret key：自己留存，必须保证其私密性
    数字签名：主要在于让接收方确认发送方身份；
    密钥交换：发送方用对方的公钥加密一个对称密钥，并发送给对方；
    数据加密：
    算法：RSA, DSA, ELGamal


单向加密：只能加密，不能解密；提取数据指纹；

    特性：定长输出、雪崩效应；
    算法：
	md5: 消息摘要算法，5是版本号，128bits
	sha1: 安全哈希算法，1是版本号，160bits
	sha224	
	sha256
	sha384
	sha512
    功能：主要用来验证数据完整性

密钥交换：IKE

    公钥加密：
	DH (Deffie-Hellman)
		A: p, g
		B: p, g

		A: x
			--> p^x%g

			p^y%g^x = p^xy%g
		B: y
			--> p^y%g

			p^x%g^y = p^xy%g

DH算法，是用于密钥交换的一种算法。公钥加密算法是用公钥把“对称加密的密钥”加密后在互联网上传输给对方。只要在网上传输，就有被破解的可能。而DH算法的优势就在于不需要让密钥在互联网上传输。
简单来讲是这样实现的，A和B双方协商生成一个大素数；
A生成p和g两个数，还有一个x是A自己私下生成的
B也生成p和g，然后自己生成一个y。
A开始做运算：p^x%g。 把这个式子的结果发送给B，这时候都是明文传输，所以C可以通过互联网截获到p，g与“p^x%g”这三个值，但是要用它们反推出一个x次方值是非常困难的。
相应的B也把它的结果p^y%g发给A，A拿到这个结果直接计算x次方的值，即(p^y%g)^x，答案和p^yx%g是一样的；
而B也做相应的运算（p^x%g)^y=p^xy%g；而这个结果就是双方的密钥，不用在网络中传输，而是通过计算得到的。

结论：虽然A和B不知道彼此的x，y值，但是最后也能通过计算得到一致的密钥。


<h3 id="toc_1">PKI: Public Key Infrastructure</h3>

    签证机构：CA
    注册机构：RA
    证书吊销列表：CRL
    证书存取库

    X.509：定义了证书的结构以及认证协议标准
	版本号
	序列号
	签名算法ID
	发行者名称
	有效期限
	主体名称
	主体公钥
	发行者惟一标识
	主体的惟一标识
	扩展
	发行者签名

SSL: Secure Socket Layer

TLS: Transport Layer Security

	1995：SSL 2.0, Netscape
	1996: SSL 3.0
	1999: TLS 1.0 
	2006: TLS 1.1 RFC 4346
	2008：TLS 1.2 
	2015: TLS 1.3 

    分层设计：
	1、最低层：基础算法原语的实现，aes, rsa, md5
	2、向上一层：各种算法的实现
	3、再向上一层：组合算法实现的半成品
	4、用各种组件拼装而成的种种成品密码学协议/软件：tls, ssh, 

OpenSSL：开源项目

    三个组件：
	openssl: 多用途的命令行工具；
	libcrypto: 公共加密库；
	libssl: 库，实现了ssl及tls；

openssl命令：

    openssl version：程序版本号

    标准命令、消息摘要命令、加密命令
    
    标准命令：enc，ca，req，genrsa等，这些命令可以直接被使用来实现不同的功能
    消息摘要命令：主要用于单向加密，要使用dgst来调用不同的方式（比如MD5）进行单向加密
    加密命令：用于对数据加密解密，要使用enc来调用不同的加密方式。

    标准命令：
	enc, ca, req, ...

    对称加密：
	工具：openssl enc, gpg
	算法：3des, aes, blowfish, twofish


enc命令：

	加密：~]# openssl enc -e -des3 -a -salt -in fstab -out fstab.ciphertext
	-e：表示这是一个加密操作
	-des3：使用des3对称加密方式
	-a：使用base64编码方式进行输出（base64是文本格式的编码，可以存放进文本文件里）
	-salt：加密的时候加入一些杂质
	-in：数据输入源
	-out：数据输出位置

	解密：~]# openssl enc -d -des3 -a -salt -in fstab.ciphertext -out fstab
	-d：表示这是一个解密操作

单向加密：

    工具：md5sum, sha1sum, sha224sum, sha256sum,..., openssl dgst

    dgst命令：
	openssl dgst -md5 /PATH/TO/SOMEFILE


    MAC: Message Authentication Code，单向加密的一种延伸应用，用于实现在网络通信中保证所传输的数据的完整性；

	机制：
	    CBC-MAC
	    HMAC：使用md5或sha1算法

生成用户密码：

	openssl passwd -1 -salt SALT  #生成用户密文密码
	-1：表示使用md5算法进行单向加密
	-salt：指定杂质内容，最多8位

输完命令之后就可以输入密码，最后得到一个加盐后的特征码，和shadow中的密码格式一样，直接复制过去就可以。另，openssl不支持-6加密即sha512算法

生成随机数：

	openssl rand -base64|-hex NUM
	NUM: 表示字节数；-hex时，每个字符4位，出现的字符数为NUM*2; 
	openssl rand -base64 10
	-base64 10：使用base64编码生成10位随机数。
	#如果不加这个选项，有可能生成的随机数是二进制格式，无法保存到文本文件中。
	base64编码表示可以使用任何数字或字母表示，但是最后会生成两个“==”，需要手动删除，所以并不常用。

公钥加密：

	加密：
	    算法：RSA, ELGamal
	    工具：gpg, openssl rsautl
	数字签名：
	    算法：RSA, DSA, ELGamal

	密钥交换：
	    算法：dh

	DSA: Digital Signature Algorithm
	DSS：Digital Signature Standard
	RSA：

生成密钥对儿：

	openssl genrsa -out /PATH/TO/PRIVATEKEY.FILE NUM_BITS

	# (umask 077; openssl genrsa -out key.pri 2048)

提取出公钥：

	# openssl rsa -in /PATH/FROM/PRIVATEKEY.FILE -pubout


随机数生成器：

	/dev/random：仅从熵池返回随机数；随机数用尽，阻塞；
	/dev/urandom：从熵池返回随机数；随机数用尽，会利用软件生成伪随机数；非阻塞；




<h3 id="toc_2">建立私有CA</h3>

	OpenCA
	openssl

证书申请及签署步骤：

	1、生成申请请求；
	2、RA核验；
	3、CA签署；
	4、获取证书；

创建私有CA：

	openssl的配置文件：/etc/pki/tls/openssl.cnf

(1) 创建所需要的文件

	# touch index.txt
	# echo 01 > serial
	# 

(2) CA自签证书

(a) 生成私钥；

	# (umask 077; openssl genrsa -out /etc/pki/CA/private/cakey.pem 2048)

(b) 生成自签证书；

	# openssl req -new -x509 -key /etc/pki/CA/private/cakey.epm -days 7300 -out /etc/pki/CA/cacert.pem
	-new: 生成新证书签署请求；
	-x509: 专用于CA生成自签证书；
	-key: 生成请求时用到的私钥文件；
	-days n：证书的有效期限；
	-out /PATH/TO/SOMECERTFILE: 证书的保存路径；
	键入命令后，会提示让用户输入证书中的一些信息，包含：
	第一行为国家
	第二行为省/州的名称
	第三行为城市名称
	第四行为公司名称
	第五行是部门名称
	第六行是证书持有者的名字， 这里是CA自己创建的证书，所以天蝎CA服务器自己的域名
	第七行是邮件地址，可以为空
	生成的证书会以.pem作为标识结尾

(c) 为CA提供所需的目录及文件；

	~]# mkdir  -pv  /etc/pki/CA/{certs,crl,newcerts}  #当不存在时需要创建签发证书、吊销证书、新证书目录
	~]# touch  /etc/pki/CA/{serial,index.txt}  #创建证书序列号文件、证书索引文件
	~]# echo  01 > /etc/pki/CA/serial          # 第一次创建的时候需要给予证书序列号

(3) 发证

(a) 用到证书的主机生成证书请求；

	# (umask 077; openssl genrsa -out /etc/httpd/ssl/httpd.key 2048)
	# openssl req -new -key /etc/httpd/ssl/httpd.key -days 365 -out /etc/httpd/ssl/httpd.csr

(b) 把请求文件传输给CA；

	~]# scp  /etc/httpd/ssl/httpd.csr  root@172.16.249.18:/tmp/  #此处为要发送目标主机地址

(c) CA签署证书，并将证书发还给请求者；

	# openssl ca -in /tmp/httpd.csr -out /etc/pki/CA/certs/httpd.crt -days 365
	*.crt：表示证书文件
	-days ：签署证书的有效期

查看证书中的信息：

	openssl x509 -in /PATH/FROM/CERT_FILE -noout -text|-subject|-serial
	方法一：~]# cat  /etc/pki/CA/index.txt
	V：表示已经签署的
	01：表示证书序列号
	/C=CN/ST=Beijing/O=… ...：  表示主题信息(主题标示)
	方法二：查看证书中的信息(CA或者服务端均可)：
	~]# openssl  x509  -in /etc/pki/CA/certs/httpd.crt  -noout  -serial  -subject
	-serial ：序列号  -subject：主题信息

将CA签署机构的.crt证书发送给服务器

	~]#  scp  /etc/pki/CA/certs/httpd.crt  root@172.16.249.210:/etc/httpd/ssl
	注意：第一次进行主机间基于ssh的scp操作会接收一个证书，Queue要你那认证

	删除服务器和CA主机上签署前的*.csr文件，确保安全
	httpd主机：~]# rm  -rf  /etc/httpd/ssl/httpd.csr
	CA主机：~]# rm  -rf  /tmp/httpd.csr

(4) 吊销证书

(a) 客户端获取要吊销的证书的serial

	# openssl x509 -in /PATH/FROM/CERT_FILE -noout -serial -subject

(b) CA

	先根据客户提交的serial与subject信息，对比检验是否与index.txt文件中的信息一致；
	在/etc/pki/CA/crets/*下生成证书后，会在/etc/pki/CA/newcrets/*以对应证书命名为SERIAL.pem文件存放

吊销证书：

	# openssl ca -revoke /etc/pki/CA/newcerts/SERIAL.pem
	其中SERIAL要换成证书真正的序列号：eg.01.pem

(c) 生成吊销证书的编号(第一次吊销证书时执行)

	# echo 01 > /etc/pki/CA/crlnumber

(d) 更新证书吊销列表

	# openssl ca -gencrl -out thisca.crl

查看crl文件：

	# openssl crl -in /PATH/FROM/CRL_FILE.crl -noout -text





