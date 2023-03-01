# Linux操作-SCP命令

##  Windows和Linux间的文件传输

​	当我们通过xshell等工具链接上Linux系统的时候，我们想要把windows上的文件传输到Linux上的时候，我们可以使用`rz`命令打开文件会话窗口进行Windows机器上的文件选择，然后就可以将文件传输到当前的Linux目录下了。另一种方法就是使用配套的xtfp工具直接打开Windows和Linux的目录进行传输，个人推荐使用`rz`命令进行上传，方便快捷；

​	当然，上面说了使用`rz`命令将Windows上的文件传输到Linux机器上，下面将如何通过命令将Linux系统上的文件传输到Windows系统上，当时前提是通过Windows系统上的xshell等工具已经连接到了Linux系统了。好了，从Linux系统上传输文件到Windows的命令是`sz`命令；

> sz filename 

使用上述命令同样会打开文件会话窗口，不过这个会话窗口是要你选择将这个filename文件传输放到Windows上的那个目录的。

## Linux 间的文件传输

**前提条件** 传输的两台Linux机器是可以通过22端口进行互通的，也就是同网段的机器，或者开通了相应的网络策略；

实例：将LinuxA机器上/srv/目录下的文件demo.zip 传输到LinuxB机器上的/opt/目录下，其中LinuxB的用户名是root，IP是192.168.1.2

**一定要确保LinuxB上是有/opt目录的**

> 首先登陆LinuxA
>
> cd /srv/
>
> scp demo.zip root@192.168.1.2:/opt/

通过以上操作就可以见LinuxA上的demo.zip文件传输到LinuxB机器上了


## 文件复制

从远程机器`192.168.1.10`上将`/opt/software/`文件a.txt复制到本地机器上` /opt/software/`

> scp root@192.168.1.10:/opt/software/a.txt  /opt/software/