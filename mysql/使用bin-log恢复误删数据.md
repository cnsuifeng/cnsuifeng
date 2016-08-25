```
错误执行update或者delete语句，造成整个表变化，传统使用全量备份+binlog增量备份恢复到删除以前的状态，如果数据量比较大的话，会造成恢复过程很复杂。
如果binlog-format是ROW的话，会有一种相对简单的方式来进行
```

#0准备
##0.1修改binlog-format为ROW
```
修改/etc/my.cnf
binlgo-format=ROW
```

##0.2查看数据库中内容
![](index_files/01108804-c4ba-4b24-89f2-66a5b88abbe5.png)

我们模拟没有加where条件，直接删除tb_a中的数据
![](index_files/2402d246-992b-4c86-8bb5-d74f6d36f264.png)

#1.查看binlog相关内容
```shell
mysqlbinlog --base64-output=decode-rows -v -v --start-datetime='2016-08-25 15:30:00' mysql-bin.000004 | grep -B 50 '### DELETE FROM `test_db`.`tb_a`' | more
```
![](index_files/443248a7-6b5e-48b9-9762-4797b4a2f87e.png)
红色部分可以看到删除时用到的数据，其中@1、@2表示我们测试表中第一个和第二个字段

#2.我们关心中间delete的部分，使用如下命令将中间部分保存下来
```shell
mysqlbinlog --base64-output=decode-rows -v -v --start-datetime='2016-08-25 15:30:00' mysql-bin.000004|sed -n '/end_log_pos 418/,/COMMIT/p'|tail -n +2 > recovery.binlog
```
#3.查看recover.binlog内容
![](index_files/1e4875b6-dff4-43a9-9666-ce919a711fdd.png)

#4.从binlog中反向整理出可执行的sql语句
```shell
cat recovery.binlog | sed -n '/###/p' | sed 's/### //g;s/\/\*.*/,/g;s/DELETE FROM/\nREPLACE INTO/g;s/WHERE/SELECT/g'|sed -r 's/(@2.*),/\1;/g'|sed 's/@[1-9].*=//g'>recory.sql
```
**注**：@2表示最后一个字段
![](index_files/d73bee83-d52d-46d2-b161-63b83685d5e0.png)

#5.执行sql文件
```sql
source /data/mysql/recovery.sql
```

#6查看执行结果
![](index_files/f1b54d38-e2fc-4206-9d42-4c46284e730c.png)

感谢[贺春旸](http://chuansong.me/n/352489851451 "贺春旸")，思路和代码都来自他的文章，根据自己的实际情况，做了一次测试，非常成功。
再次感谢。
