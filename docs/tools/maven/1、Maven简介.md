# Maven简介

## 是什么

项目管理工具，将项目开发和管理过程抽象成一个项目对象模型（POM）

`POM（Project Object Model）：项目对象模型`

![image-20210913090834585](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20210913090834585.png)

## 作用

1. 标准化、跨平台的自动化项目构建方式
2. 方便管理项目依赖，避免资源间的版本冲突
3. 标准、统一的项目结构

## 下载

https://maven.apache.org/download.cgi

下载压缩包后直接解压就可以了

## 配置环境变量

配置Path

![image-20220828170403994](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220828170403994.png)

验证

![image-20220828170641911](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220828170641911.png)

## 仓库

存储资源，即jar包

![image-20210913092116066](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20210913092116066.png)



### 本地仓库

自己电脑上存储资源的仓库，连接远程仓库获取资源

### 中央仓库

Maven团队维护，存储所有资源的仓库

### 私服

部门/公司范围内存储资源的仓库，从中央仓库获取资源，保存具有版权的资源，包含购买或自主研发的jar
中央仓库中的jar都是开源的，不能存储具有版权的资源一定范围内共享资源，仅对内部开放，不对外共享

## 仓库配置

conf\settings.xml

### 本地仓库

![image-20210913093121496](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20210913093121496.png)

### 镜像中央仓库

```xml
<mirrors>
    <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>central</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
</mirrors>
```

`注意：如果是maven安装目录下的conf\settings.xml，则是全局配置`

`如果是repository同级目录下的settings.xml，则是用户配置，优先级更高`

