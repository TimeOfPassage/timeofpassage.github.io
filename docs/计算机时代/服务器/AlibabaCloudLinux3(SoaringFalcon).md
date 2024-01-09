# Alibaba Cloud Linux 3 (Soaring Falcon)

> 操作同CentOS类似

# 安装docker(参考CentOS)

[aliyun安装链接](https://developer.aliyun.com/article/791789?spm=5176.21213303.J_qCOwPWspKEuWcmp8qiZNQ.12.59622f3dXxwKFK&amp;scm=20140722.S_community@@%E6%96%87%E7%AB%A0@@791789._.ID_community@@%E6%96%87%E7%AB%A0@@791789-RL_aliyun%20cloud%20linux%20docker-LOC_llm-OR_ser-V_3-RK_rerank-P0_2)

```bash
# 设置源
sudo dnf config-manager --add-repo=https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 设置dns兼容插件
sudo dnf -y install dnf-plugin-releasever-adapter --repo alinux3-plus
# 安装docker-ce社区版
sudo dnf -y install docker-ce --nobest

# 启动服务并设置开机自启动
systemctl start docker
systemctl enable docker

# 查看状态
systemctl status docker
```

# JDK1.8安装

从oracle官网下载tar.gz文件

```bash
# 下载 (链接从oracle官网获取)
wget https://download.oracle.com/otn/java/jdk/8u202-b08/1961070e4c9b4e26a04e7f5a083f551e/jdk-8u202-linux-x64.tar.gz?AuthParam=1701092429_0d00b36ad7f00fd00258c4393b7afeba -O jdk-8u202-linux-x64.tar.gz
# 解压
tar zvxf jdk-8u202-linux-x64.tar.gz
# 编辑~/.bash_profile文件，增加环境变量
‍```
JAVA_HOME=/xxx/xx/jdk1.8.0_202
export PATH=$JAVA_HOME/bin:$PATH
‍```
source ~/.bash_profile
```

‍
