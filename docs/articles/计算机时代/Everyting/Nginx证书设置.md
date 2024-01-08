# #Nginx证书设置

# 证书生成

注意：一般生成的目录,应该放在nginx/conf/ssl目录

1.创建服务器证书密钥文件 server.key：

```shell
openssl genrsa -des3 -out server.key 2048
```

输入密码，确认密码，自己随便定义，但是要记住，后面会用到。

2.创建服务器证书的申请文件 server.csr

```shell
openssl req -new -key server.key -out server.csr
```

输出内容为：

```shell
Enter pass phrase for root.key: ← 输入前面创建的密码 
Country Name (2 letter code) [AU]:CN ← 国家代号，中国输入CN 
State or Province Name (full name) [Some-State]:BeiJing ← 省的全名，拼音 
Locality Name (eg, city) []:BeiJing ← 市的全名，拼音 
Organization Name (eg, company) [Internet Widgits Pty Ltd]:MyCompany Corp. ← 公司英文名 
Organizational Unit Name (eg, section) []: ← 可以不输入 
Common Name (eg, YOUR name) []: ← 此时不输入 
Email Address []:admin@mycompany.com ← 电子邮箱，可随意填
Please enter the following ‘extra’ attributes 
to be sent with your certificate request 
A challenge password []: ← 可以不输入 
An optional company name []: ← 可以不输入
```

4.备份一份服务器密钥文件

```shell
cp server.key server.key.org
```

5.去除文件口令

```shell
openssl rsa -in server.key.org -out server.key
```

6.生成证书文件server.crt

```shell
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

‍

# 附录：ssl配置

```nginx
server{
   listen      443 ssl;
   server_name  localhost;
   ssl_certificate /etc/nginx/conf.d/aaaa.pem;
   ssl_certificate_key /etc/nginx/conf.d/aaaa.key;

   location / {
       proxy_redirect off;
       proxy_pass https://www.taobao.com/; 
   }  
}
```

# 附录：http重定向https

```nginx
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    rewrite ^(.*)$ https://$host$1 permanent;
}
```
