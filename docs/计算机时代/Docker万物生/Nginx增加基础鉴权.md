# 安装htpasswd

```bash
yum install httpd-tools -y
```

# htpasswd命令创建用户和修改密码

------

**语法**
htpasswd(选项)(参数)
**选项**

> - `-c：`创建一个加密文件；

- `-n：`不更新加密文件，只将加密后的用户名密码显示在屏幕上；
- `-m：`默认采用MD5算法对密码进行加密；
- `-d：`采用CRYPT算法对密码进行加密；
- `-p：`不对密码进行进行加密，即明文密码；
- `-s：`采用SHA算法对密码进行加密；
- `-b：`在命令行中一并输入用户名和密码而不是根据提示输入密码；
- `-D：`删除指定的用户。

**参数**
用户：要创建或者更新密码的用户名；
密码：用户的新密码。
**实例**
利用htpasswd命令添加用户

> **htpasswd -bc htpasswd.user admin `123456`**

在bin目录下生成一个 htpasswd.user 文件，用户名admin，密码：123456，默认采用MD5加密方式。

在原有密码文件中增加下一个用户

> **htpasswd -b htpasswd.user Jack `123456`**

去掉-c选项，即可在第一个用户之后添加第二个用户，依此类推。

利用htpasswd命令删除用户名和密码

> **htpasswd -D htpasswd.user Jack**

利用htpasswd命令修改密码

> **htpasswd -b htpasswd.user Jack `123456`**

# Nginx配置

```conf
upstream xxx-server {
        server xxx.xx.x.xx:8080;
}

server {
    listen       80;
    server_name  localhost;

    access_log  /var/log/nginx/xxx.access.log  main;

    location / {
        auth_basic "input your username and password";
        auth_basic_user_file /etc/nginx/conf.d/xxxx.user;
        proxy_pass http://xxx-server/;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-NginX-Proxy true;
    }

}
```

核心配置

```bash
auth_basic "input your username and password";
auth_basic_user_file /etc/nginx/conf.d/xxxx.user;
```

user_file即是上述使用**htpasswd**工具创建的文件
