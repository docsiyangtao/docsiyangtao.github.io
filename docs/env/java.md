## rpm安装

卸载自带JDK

```bash
# 查循已安装的JDK
rpm -qa | grep java
```

![image-20220120000304717](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220120000304717.png)

将java开头的都卸载掉

```bash
yum -y remove java-1.8.0-openjdk-headless.x86_64
...
```

rpm包下载地址：https://www.oracle.com/java/technologies/downloads/#java8

```bash
# rpm命令安装
rpm -ivh jdk-8u321-linux-x64.rpm

# 验证
java -version
```

![image-20220120000639549](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220120000639549.png)



## 压缩包安装

```bash
# 将压缩包解压到/usr/local/java中，若没有此目录则新建
tar -zxvf jdk-8u321-linux-x64.tar.gz -C /usr/local/java

# 配置环境变量，编辑/etc/profile，增加如下内容
JAVA_HOME=/usr/local/java/jdk1.8.0_321
CLASSPATH=$JAVA_HOME/lib/
PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME CLASSPATH

# 让环境变量生效
source /etc/profile

# 验证
java -version
```

![image-20220120000639549](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220120000639549.png)



## yum 安装OpenJDK

```bash
# 查找jdk
yum search java-1.8

# 在搜索结果列表中选择java-1.8.0-openjdk-devel.x86_64，进行安装
yum -y install java-1.8.0-openjdk-devel.x86_64

# 验证
java -version
```

![image-20220120004238575](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220120004238575.png)



## exe安装

下载地址：https://www.oracle.com/java/technologies/downloads/#java8-windows

无脑下一步就行了，会先安装JDK在安装JRE，确保两个都安装完成

![image-20220120005037415](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220120005037415.png) 

配置环境变量

![image-20220120005406804](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220120005406804.png) 

![image-20220120005610345](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220120005610345.png) 

编辑PATH

![image-20220120005743536](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220120005743536.png) 

验证

![image-20220122003855260](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220122003855260.png) 

