# MySQL安装

+ 安装与卸载中，用户全部切换成为root，一旦安装，普通用户能使用的.

+ 初期练习，mysql不进行用户管理，全部使用root进行，尽快适应mysql语句，后面学了用户管理，在考虑新建普通用户

## 卸载不需要的环境
查看是否有mysql:ps ajx | grep mysql / ps ajx | grep mariadb

切换成超级用户root

删除前，先关闭:systemctl stop mysqld

查询mysql文件: rpm -qa | grep mysql

卸载mysql相关文件: rpm -qa | grep mysql | xargs yum -y remove

之后查询就没有文件了,然后还有一件事要确认一下:

看看有没有: ls /etc/my.cnf,但是ls var/lib/mysql可能还会有上个mysql的残留文件 

## 下载yum源
打开网站[http://repo.mysql.com/](http://repo.mysql.com/),进去之后会看不全,所以进去右键，查看网页源代码

查询自己的版本:cat /etc/redhat-release

我们选择MySQL要选5.7版本的，不选8.0,如果有对应版本就选对应版本,否则选mysql57-community-release-el7.rpm(centos)

然后我们在Linux环境里面输入:rz.上传刚刚下好的文件

在安装之前我们先: ls /etc/yum.repos.d/ -l,查看一下yum源清单里面是否有MySQL,如果没有可能会安装不上

没有的话就:rpm -ivh mysql57-community-release-el7.rpm(刚刚rz上传上去的那个文件名),然后再次查询(ls /etc/yum.repos.d/ -l)就会出现了

vim /etc/yum.repos.d/mysql-community.repo这个文件发现有各种各样的工具和MySQL,但是我们不需要管，他会自动给我们匹配yum源来匹配系统

yum list | grep mysql查看MySQL的内容,如果出现说明我们下的yum源已经生效了,然后就可以把rm mysql57-community-release-el7.rpm(刚刚rz上传的那个文件)

## 开始安装

yum install -y mysql-community-server

安装会出现的问题

GPG Keys are configured as: file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql ,GPG过期了

那么我们运行这段的代码就行了:rpm --import http://repo.mysql.com/RPM-GPG-KEY-mysql-2022

还有一个问题就是前面的yum源下错了,也会导致某些问题

确认安装成功:看看是否有ls /etc/my.cnf,which mysqld(服务端),which mysql(客户端)

启动一下mysql:systemctl start mysqld

然后我们查看 ps ajx | grep mysqld就看得到一个进程了,netstat -nltp服务端口号

我们尝试登入一下: mysql -uroot -p,登入不上去

##开始登入

1. 获取临时root密码:sudo grep 'temporary password' /var/log/mysqld.log
2. 直接mysql -uroot -p登入
3. vim /etc/my.cnf, 最后新起一行加入一句skip-grant-tables, 然后重启一下mysql:systemctl restart mysqld

当是第一种的时候,还需更改一下密码：

① set global validate_password_policy=0;

② set global validate_password_length=1;

③ alter user 'root'@'localhost' identified by '密码@';

④ flush privileges;

## 设置配置文件

在/etc/my.cnf里面

port:端口号

datadir:未来MySQL建表建库的地方

加入:character-set-server=utf8、default-storage-engine=innodb,配置之后重启一下MySQL就行了

开机自动启动:systemctl enable mysqld、systemctl daemon-reload
