## 关机、重启、注销

```bash
# 即刻关机
shutdown -h now

# 十分钟后关机
shutdown -h 10			

# 11：00关机
shutdown -h 11:00 	

# 立刻关机
init 0					

# 关机
telinit 0				

# 立刻关机
poweroff

# 关机
halt

# 预定时间关机（10分钟后）
shutdown -h +10			

# 取消指定时间关机
shutdown -c 			

# 重启
shutdown -r now			

# 10分钟后重启
shutdown -r 10			

# 11：00重启
shutdown -r 11:00		

# 重启
reboot 					

# 重启
init 6														

# buff数据同步到磁盘
sync					

# 退出登录Shell
logout					
```



## 系统、硬件、性能、日期

```bash
# 查看内核/OS/CPU信息
uname -a				

# 查看内核版本
uname -r				

# 查看处理器架构
uname -m				

# 查看处理器架构
arch									

# 查看CPU信息
cat /proc/cpuinfo

# 查看系统USB设备信息
lsusb -tv				

# 查看系统PCI设备信息
lipci -tv

# 查看内存总量
grep MemTotal/proc/meninfo	

# 查看空闲内存量
grep MemFree/proc/meminfo

# 动态显示CPU/内存/进程等情况
top						

# 每1秒采一次系统状态，采20次
vmstat 1 20				

# 查看IO读写/CPU使用情况
iostat					

# 查询CPU使用情况（1秒1次，共10次）
sar -u 1 10				

# 查询磁盘性能
sar -d 1 10	

# 查看内存用量和交换区用量
free -m

# 查看中断
cat /proc/interrupts	

# 查看系统负载
cat /proc/loadavg

# 查看Linux版本信息
cat /proc/version

# 查看计算机名
hostname				

# 显示当前登录系统的用户
who						

# 显示登陆时的用户名
who am i				

# 显示当前用户名
whoami			

# 查看系统运行时间、用户数、负载
uptime					

# 查看系统的环境变量
env						

# 查看已加载的系统模块
lsmod									

# 显示系统日期时间
date					

# 显示2021日历表
cal 2021							
```



## 磁盘、分区

```bash
# 查看所有磁盘分区
fdisk -l				

# 查看所有交换分区
swapon -s				

# 查看磁盘使用情况及挂载点
df -h					

# 查看磁盘使用情况及挂载点
df -hl					

# 查看指定目录的大小
du -sh /dir				

# 从高到低依次显示文件和目录大小
du -sk * | sort -rn		

# 挂载hds2盘
mount /dev/hda2/mnt/hda2	

# 指定文件系统类型挂载（如ntfs）
mount -t ntfs /dev/sdc1 /mnt/usbhd1		

# 挂载iso文件
mount -o loop xxx.iso /mnt/cdrom		

# 挂载USB盘/闪存设备
mount /dev/sda1 /mnt/usbdisk			

# 通过设备名卸载
umount -v /dev/sda1		

# 通过挂载点卸载
umount -v /mnt/mymnt	

# 强制卸载
fuser -km /mnt/hda1		
```



## 用户和用户组

```bash
# 创建用户
useradd <username>

# 删除用户yangtao
userdel -r <username>

# 修改用户的组
usermod -g group_name user_name		

# 将用户添加到组
usermod -aG group_name user_name	

# 修改用户yangtao的登录shell、主目录以及用户组
usermod -s /bin/ksh -d /home/yangtao2 -g dev yangtao	

# 查看test用户所在的组
groups test				

# 创建用户组
groupadd group_name		

# 删除用户组
groupdel group_name		

# 重命名用户组
groupmod -n new_name old_name		

# 重命名用户组
su - user_name			

# 修改口令
passwd					

# 修改某用户的口令
passwd codesheep		

# 查看活动用户
w						

# 查看指定用户yangtao的信息
id yangtao				

# 查看用户登录日志
last					

# 查看当前用户的计划任务
crontab -l				

# 查看系统所有用户
cut -d: -f1 /etc/passwd	

# 查看系统所有组
cut -d: -f1 /etc/group	
```



## 网络和进程管理

```bash
# 查看网络接口属性
ifconfig				

# 查看某张网卡的配置
ifconfig eth0			

# 查看路由表
route -n				

# 查看所有监听端口
netstat -lntp			

# 查看已经建立的TCP连接
netstat -antp			

# 查看TCP/UDP的状态信息
netstat -lutp			

# 启动eth0网络设备
ifup eth0				

# 禁用eth0网络设备
ifdown eth0				

# 查看iptables规则
iptables -L				

# 配置IP地址
ifconfig eth0 x.x.x.x netmask x.x.x.0				

# 以dhcp模式启用eth0
dhclient eth0			

# 配置默认网关
route add -net 0/0 gw Gateway_IP					

# 配置静态路由到达网络
route add -net x.x.x.x netmask x.x.x.x gw x.x.x.x	

# 删除静态路由
route del 0/0 gw Gateway_IP							

# 查看主机名
hostname				

# 解析主机名
host www.baidu.com		

# 查看DNS记录，查看域名解析是否正常
nslookup www.baidu.com	

# 查看所有进程
ps -ef					

# 查看指定的进程
ps -ef | grep redis		

# 杀死指定名称的进程
kill -s name			

# 杀死指定pid的进程
kill -s pid				

# 实时显示进程状态
top						

# 每1秒采一次系统状态，采20次
vmstat 1 20				

# 查看IO读写/CPU使用情况
iostat					

# 查询CPU使用情况（1秒1次，共10次）
sar -u 1 10				

# 查询磁盘性能
sar -d 1 10				
```



## 常见系统服务命令

```bash
# 列出系统服务
chkconfig --list		  

# 查看某个服务
service <服务名> status	

# 启动某个服务
service <服务名> start		

# 终止某个服务
service <服务名> stop		

# 重启某个服务
service <服务名> restart	

# 查看某个服务
systemctl status <服务名>	

# 启动某个服务
systemctl start <服务名>	

# 终止某个服务
systemctl stop <服务名>	

# 重启某个服务
systemctl restart <服务名>

# 开启自启动
systemctl enable <服务名>	

# 关闭自启动
systemctl disable <服务名>	
```



## 文件和目录操作

```bash
# 进入某个目录
cd <目录名>				  

# 返回上级目录
cd ..					   	

# 返回上两级目录
cd ../..				   	

# 进入个人主目录
cd							

# 返回上一步所在目录
cd -						

# 显示当前路径
pwd							

# 查看文件目录列表
ls							

# 查看目录中内容（显示是文件还是目录）
ls -F						

# 查看文件和目录的详情列表
ls -l						

# 查看隐藏文件
ls -a						

# 查看文件和目录的详情列表（增强文件大小易读性）
ls -lh						

# 查看文件和目录列表（以文件大小升序查看）
ls -lSr						

# 查看文件和目录的属性结构
tree						

# 创建目录
mkdir <目录名>				  

# 同时创建两个目录
mkdir dir1 dir2				

# 创建目录树
mkdir -p /tmp/dir1/dir2		

# 删除 file1 文件
rm -f file1					

# 删除 dir1 目录
rmdir dir1					

# 删除 dir1 目录及其内容
rm -rf dir1					

# 同时删除两个目录及其内容
rm -rf dir1 dir2			

# 重命名/移动目录
mv old_dir new_dir			

# 复制文件
cp file1 file2				

# 复制某目录下的所有文件至当前目录
cp dir/* .					

# 复制目录
cp -a dir1 dir2				

# 复制一个目录至当前目录
cp -a /tmp/dir1 .			

# 创建指向文件/目录的软链接
ln -s file1 link1			

# 创建指向文件/目录的物理链接
ln file1 link1				

# 从根目录开始搜索文件/目录
find / -name file1			

# 搜索用户user1的文件/目录
find / -user user1			

# 在目录/dir中搜索带有.bin后缀的文件
find /dir -name *.bin		

# 快速定位文件
locate <关键词>			 

# 寻找.mp4结尾的文件
locate *.mp4				

# 显示某二进制文件/可执行文件的路径
whereis <关键词>			  

# 查找系统目录下某二进制文件
which <关键词>				  

# 设置目录所有者(u)、群组(g)及其他人(o)的读(r)写(w)执行(x)权限
chmod ugo+rwx dir1			

# 移除群组(g)与其他人(o)对目录的读写执行权限
chmod go-rwx dir1			

# 改变文件的所有者属性
chown user1 file1			

# 改变目录的所有者属性
chown -R user1 dir1			

# 改变文件群组
chgrp group1 file1			

# 改变文件的所有人和群组
chown user1:group1 file1	
```



## 文件查看和处理

```bash
# 查看文件内容
cat file1					

# 查看内容并标示行数
cat -n file1				

# 从最后一行开始查看文件内容
tac file1					

# 查看一个长文件的内容
more file1					

# 类似于more命令，但允许反向操作
less file1					

# 查看文件前两行
head -2 file1				

# 查看文件后两行
tail -2 file1				

# 实时查看添加到文件中的内容
tail -f /log/msg			

# 在文件hello.txt中查找关键字yangtao
grep yangtao hello.txt		

# 在文件hello.txt中查找以hello开头的内容
grep ^hello hello.txt		

# 选择hello.txt中所有包含数字的行
grep [0-9] hello.txt		

# 将hello.txt文件中的s1替换成s2
sed 's/s1/s2/g' hello.txt	

# 从hello.txt文件中删除所有空白行
sed '/^$/d' hello.txt		

# 从hello.txt文件中删除所有注释和空白行
sed '/*#/d;/^$/d' hello.txt	

# 从文件hello.txt中排除第一行
sed -e '1d' hello.txt		

# 查看只包含关键词 s1 的行
sed -n '/s1/p' hello.txt	

# 删除每一行最后的空白字符
sed -e 's/*$//' hello.txt	

# 从文档中只删除词汇 s1 并保留剩余全部
sed -e 's/s1//g' hello.txt	

# 查看第一行到第五行的内容
sed -n '1,5p;5q' hello.txt	

# 查看第五行
sed -n '5p;5q' hello.txt	

# 合并两个文件或两栏的内容
paste file1 file2			

# 合并两个文件或两栏的内容，中间用 + 区分
paste -d '+' file1 file2	

# 配许两个文件的内容
sort file1 file2			

# 比较两个文件的内容（去除 file1 所含内容）
comm -1 file1 file2			

# 比较两个文件的内容（去除 file2 所含内容）
comm -2 file1 file2			

# 比较两个文件的内容（去除两文件共有部分）
comm -3 file1 file2			
```



## 打包和解压

```bash
# 压缩至zip包
zip xxx.zip file			

# 将多个文件+目录压缩成zip包
zip -r xxx.zip file1 file2 dir1		

# 解压zip包
unzip xxx.zip				

# 创建非压缩tar包
tar -cvf xxx.tar file		

# 将多个文件+目录达成tar包
tar -cvf xxx.tar file1 file2 dir1	

# 查看tar包的内容
tar -tf xxx.tar				

# 解压tar包
tar -xvf xxx.tar			

# 将tar包解压缩至指定目录
tar -xvf xxx.tar -C /dir	

# 创建bz2压缩包
tar -cvfj xxx.tar.bz2 dir	

# 解压bz2压缩包
tar -jxvf xxx.tar.bz2		

# 创建gzip压缩包
tar -zcvf xxx.tar.gz dir	

# 解压gzip压缩包
tar -zxvf xxx.tar.gz		

# 解压bz2压缩包
bunzip2 xxx.bz2				

# 压缩文件
bzip2 filename				

# 解压gzip压缩包
gunzip xxx.gz 				

# 压缩文件
gzip filename				

# 最大程度压缩
gzip -9 filename			
```



## RPM包管理命令

```bash
# 查看已安装的rpm包
rpm -qa						

# 查询某个rpm包
rpm -q pkg_name				

# 显示xxx功能是由哪个包提供的
rpm -q --whatprovides xxx	

# 显示xxx功能被哪个程序包依赖
rpm -q --whatrequires xxx	

# 显示xxx包的更改记录
rpm -q --changelog xxx		

# 查看一个包的详细信息
rpm -qi pkg_name			

# 查询一个包所提供的文档
rpm -qd pkg_name			

# 查看已安装rpm包提供的配置文件
rpm -qc pkg_name			

# 查看一个包安装了那些文件
rpm -ql pkg_name			

# 查看某个⽂件属于哪个包
rpm -qf filename 			

# 查询包的依赖关系
rpm -qR pkg_name 			

# 安装rpm包
rpm -ivh xxx.rpm			

# 测试安装rpm包
rpm -ivh --test xxx.rpm		

# 安装rpm包时忽略依赖关系
rpm -ivh --nodeps xxx.rpm	

# 卸载程序包
rpm -e xxx					

# 升级确定已安装的rpm包
rpm -Fvh pkg_name			

# 升级rpm包（若未安装则会安装）
rpm -Uvh pkg_name			

# RPM包详细信息校验
rpm -V pkg_name 			
```



## YUM包管理命令

```bash
# 显示可用的原仓库
yum repolist enabled		

# 搜索软件包
yum search pkg_name			

# 下载并安装软件包
yum install pkg_name		

# 只下载不安装
yum install --downloadonly pkg_name		

# 显示所有程序包
yum list					

# 查看当前系统已安装包
yum list installed			

# 查看可以更新的包列表
yum list updates 			

# 查看可升级的软件包
yum check-update			

# 更新所有软件包
yum update					

# 升级指定软件包
yum update pkg_name			

# 列出软件包依赖关系
yum deplist pkg_name		

# 删除软件包
yum remove pkg_name			

# 清除缓存
yum clean all				

# 清除缓存的软件包
yum clean packages			

# 清除缓存的header
yum clean headers			
```

