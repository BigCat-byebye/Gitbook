## 证书相相关概念
RSA和DSA都是加密算法，只是使用范围不一样，RSA算法的范围比较广
SHA和MD5都是一种消息摘要算法，主要是用来校验数据的完整性，防止数据在传输过程中被篡改

## 概念相关
 首先要有一个CA根证书，然后用CA根证书来签发用户证书。

    用户进行证书申请：一般先生成一个私钥，然后用私钥生成证书请求(证书请求里应含有公钥信息)，再利用证书服务器的CA根证书来签发证书。
    特别说明:
1. 自签名证书(一般用于顶级证书、根证书): 证书的名称和认证机构的名称相同.
2. 根证书：根证书是CA认证中心给自己颁发的证书,是信任链的起始点。任何安装CA根证书的服务器都意味着对这个CA认证中心是信任的。
数字证书则是由证书认证机构（CA）对证书申请者真实身份验证之后，用CA的根证书对申请人的一些基本信息以及申请人的公钥进行签名（相当于加盖发证书机构的公章）后形成的一个数字文件。数字证书包含证书中所标识的实体的公钥（就是说你的证书里有你的公钥），由于证书将公钥与特定的个人匹配，并且该证书的真实性由颁发机构保证（就是说可以让大家相信你的证书是真的），因此，数字证书为如何找到用户的公钥并知道它是否有效这一问题提供了解决方案。
## 关于证书以及加密以及私钥公钥的简单理解
其中最重要的概念就是公钥私钥，使用openssl可以创建一对公钥私钥，公钥和私钥是成对存在的，具体原理涉及到数学素数，这里不谈。

    其实证书和加密可以理解为公钥私钥的相对用法，
1. 加密：比如说ssh的加密，首先在客户端生成一对密钥，然后在服务器端用公钥来加密数据，然后客户端用私钥进行解密，由于公钥私钥的成对性，所以保证了数据的安全，在传输数据的时候，同时可以用md5或者sha等算法，保证数据的完整性
2. 证书：就是用CA机构的私钥来加密你的公钥，首先来说CA机构，CA机构生成一对密钥，然后将公钥发送到所有主机上，然后使用CA使用自己的私钥来加密你自己生成的公钥，那么能利用CA的公钥解密数据，则说明此数据是由CA认可的，从而实现了对数据身份的确认，这就是证书，然后解密出来的公钥就可以用来加密数据，从而实现数据加密，这就是ssh，这一整个过程保证了数据传输人的身份，以及数据本身的安全
   
格式相关

.key格式：私有的密钥

.csr格式：证书签名请求（证书请求文件），含有公钥信息，certificate signing request的缩写

.crt格式：证书文件，certificate的缩写

.crl格式：证书吊销列表，Certificate Revocation List的缩写

.pem格式：用于导出，导入证书时候的证书的格式，有证书开头，结尾的格式

x509证书链

x509证书一般会用到三类文件，key，csr，crt。

Key是私用密钥，openssl格式，通常是rsa算法。

csr是证书请求文件，用于申请证书。在制作csr文件的时候，必须使用自己的私钥来签署申请，还可以设定一个密钥。

crt是CA认证后的证书文件（windows下面的csr，其实是crt），签署人用自己的key给你签署的凭证

## 生成自签订证书
```
1. 将本机当作CA的话，需要增加几个配置文件，具体可以看/etc/pki//tls/openssl.cnf

[tt@SWEBMYVMM000210 /etc/pki/CA]$ll
total 20
drwxrwxrwx 2 root root 4096 Mar 12 11:08 certs
drwxrwxrwx 2 root root 4096 Nov 18  2016 crl
-rwxrwxrwx 1 root root    0 Mar 12 11:06 index.txt
drwxrwxrwx 2 root root 4096 Nov 18  2016 newcerts
drwxrwxrwx 2 root root 4096 Nov 18  2016 private
-rwxrwxrwx 1 root root   33 Mar 12 11:12 serial
​

2. 生成一对CA根证书，用来给另一对密钥加密

#生成私钥
openssl genrsa -out ca.key 2048

#生成证书请求
openssl req -new -key ca.key -out ca.csr

#利用私钥和证书请求生成CA证书
openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt

3. 生成密钥，并利用CA证书来加密

#生成私钥
openssl genrsa -out fd.key 2048

#利用私钥生产公钥
openssl rsa -in fd.key -pubout -out fd-public.key

#用自己的私钥生成证书请求文件
openssl req -new -key fd.key -out fd.csr

#用证书请求文件，CA的证书，CA的私钥生成证书
openssl ca -in fd.csr -cert ca.crt -keyfile ca.key -out fd.crt
​```