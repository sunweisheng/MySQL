# 安装MySQL数据库5.7版本

## 安装MySQL数据库社区版

前往MySQL社区版本下载界面[社区版下载页](https://dev.mysql.com/downloads/)
![Alt text](http://static.bluersw.com/images/MySQL/Install-57-01.png)
因为操作系统是CentOS 7所以选择MySQL Yum Repository，进入后选择Red Hat Enterprise Linux 7 / Oracle Linux 7 (Architecture Independent), RPM Package。
![Alt text](http://static.bluersw.com/images/MySQL/Install-57-02.png)
复制下方No thanks, just start my download.链接的地址在服务器上使用Wget下载（[官方安装说明](https://dev.mysql.com/doc/mysql-yum-repo-quick-guide/en/)）。
![Alt text](http://static.bluersw.com/images/MySQL/Install-57-03.png)

进入操作系统执行：

```shell
#添加Yum Repository
wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm

rpm -Uvh mysql80-community-release-el7-3.noarch.rpm

#选择发行版本
yum repolist all | grep mysql

#默认是最新版本
yum-config-manager --disable mysql80-community
#要安装的是5.7版本
yum-config-manager --enable mysql57-community

#再检查一下设置结果
yum repolist all | grep mysql

#开始安装
yum install mysql-community-server

#启动MySQL服务
systemctl start mysqld.service

#查看服务状态
systemctl status mysqld.service

#设置开机启动
systemctl enable mysqld.service

#查看初始密码
grep 'temporary password' /var/log/mysqld.log
9A*:rreu:wuH

#首次登录MySQL修改密码
mysql -uroot -p

#关闭密码策略
set global validate_password_policy=0;

#修改密码
ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';

#退出用新密码重新登录
\q

#创建新用户
CREATE USER 'dba'@'%' IDENTIFIED BY '密码';

#创建数据库
CREATE DATABASE `数据库名称` /*!40100 DEFAULT CHARACTER SET utf8 */;

#将刚创建的数据库所有权限都分配给dba用户
GRANT ALL PRIVILEGES ON 数据库名称.* TO dba@'%' IDENTIFIED BY 'dba密码' WITH GRANT OPTION;

#退出
\q
```

## 测试外部链接数据库

安装MySQL Workbench[下载地址](https://www.mysql.com/products/workbench/)

```shell
#打开端口
firewall-cmd --add-port=3306/tcp --permanent
firewall-cmd --reload

#因为是KVM虚拟机不想操作iptables所以用ssh转发
ssh -CNg -L 3306:192.168.122.2:3306 root@127.0.0.1
```

![Alt text](http://static.bluersw.com/images/MySQL/Install-57-04.png)

## 更改数据库文件的位置

```shell
#停止MySQL服务
systemctl stop mysqld.service
systemctl status mysqld.service

#默认数据库文件是在这里
/var/lib/mysql/
#配置文件在这里
cat /etc/my.cnf

#创建数据库目录（在容量比较大的存储设备里）
mkdir -p /mysql/data
mkdir /mysql/log

#复制文件和文件夹（当然也可以全部复制）
cp /var/lib/mysql/auto.cnf /mysql/data/
cp /var/lib/mysql/ibdata1 /mysql/data/
cp /var/lib/mysql/ib_logfile0 /mysql/data/
cp /var/lib/mysql/ib_logfile1 /mysql/data/
cp -r /var/lib/mysql/mysql /mysql/data/
cp -r /var/lib/mysql/performance_schema /mysql/data/
cp -r /var/lib/mysql/TestDB /mysql/data/
cp /var/log/mysqld.log /mysql/log/

#变更所有者
chown -R mysql:mysql /mysql/

#修改配置文件
vi /etc/my.cnf

datadir=/var/lib/mysql 改为 datadir=/mysql/data
log-error=/var/log/mysqld.log 改为 log-error=/mysql/log/mysqld.log

#关闭setenforce
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

#重启服务
systemctl start mysqld.service
systemctl status mysqld.service
```
