## yum安装

```bash
# 安装
yum -y install git.x86_64

# 验证
git version
```



![image-20220121234319970](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220121234319970.png)

通过yum安装的git版本停留在1.8.3，要想安装最新版，还得通过压缩包安装



## 编译安装

下载地址：https://git-scm.com/download/linux

![image-20220121234759823](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220121234759823.png)

```bash
# 安装依赖环境
yum install -y zlib-devel bzip2-devel openssl-devel ncurses-devel gcc perl-ExtUtils-MakeMaker package

# 解压缩至 /usr/local/git
tar -zxvf git-2.34.1.tar.gz -C /usr/local/git

# 进入想git的解压路径，编译生成makefile
cd git-2.34.1/
./configure --prefix=/usr/local/git

# 编译
make profix=/usr/local/git

# 安装
make install

# 配置环境变量
vim /etc/profile

# 添加如下内容
export GIT_HOME=/usr/local/git
export PATH=$PATH:$GIT_HOME/bin

# 生效
source /etc/profile

# 验证
git version
```

![image-20220122001805219](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220122001805219.png)

可看到安装的是最新版本的git了



## exe安装

下载地址：https://git-scm.com/download/win

无脑下一步

![image-20220122002516325](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220122002516325.png) 

安装时可选择编辑器，如vim、notepad++等

![image-20220122002643950](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220122002643950.png) 

之后无脑下一步就行了

验证

![image-20220122003950234](https://gitee.com/yangtao8453/picgo/raw/master/img/image-20220122003950234.png) 

