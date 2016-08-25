两台服务器
**主**:192.168.0.101
**从**:192.168.0.108

------------

#1.主库配置
##1.1创建用户并授权
```sql
GRANT REPLICATION SLAVE,RELOAD,SUPER ON *.* TO 'backup'@'192.168.0.108' IDENTIFIED BY 'abcd1234';
```
##1.2修改my.cnf文件
```shell
vi /etc/my.cnf
--------------
server_id=101
log-bin=mysql-bin
```
##1.3重启数据库
```shell
service mysqld restart
```
##1.4登录主数据库
```shell
mysql -uroot -ppassword
```
查看master的文件和position
```sql
show master status;
```
![](index_files/c72bf6f8-6660-462f-a193-a626a7b1c1fe.png)
从上图可以看到，目前master的bin-log文件存在**mysql-bin.00003**中，position是**670**

#2.从数据库
##2.1修改my.cnf
```shell
vi /etc/my.cnf
--------------
server_id=108
log-bin=mysql
```
##2.2重启数据库
```shell
service mysqd restart
```

##2.3登录从数据库
```shell
mysql -uroot -ppassword
```

##2.4配置master
```sql
change master to master_host='192.168.0.101',master_user='backup',master_password='abcd1234',master_log_file='mysql-bin.000003',master_log_pos=670;
```

##2.5启动slave
```sql
start slave
```
