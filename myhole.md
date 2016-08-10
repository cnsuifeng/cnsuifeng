
#开发遇到问题集锦

##1.[**Mysql**]【20160810】Communications link failur
【**问题详细描述**】jdbc链接mysql，频繁操作数据库不定时报如下错误：com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure<br/>
【**问题原因**】出现这个问题有两个原因:<br/>
(1)数据库链接超时<br/>
(2)内存不足，如频繁使用Class.forName()加载数据库驱动<br/>
【**解决办法**】<br/>
(1)修改my.ini(linux为/etc/my.cnf）中的配置项wait_timeout=1814400(21天，21*24*3600)，不一定非得修改成这个值，修改一个比较大的数值即可
修改完成后重启数据库即可生效<br/>
(2)如果第一种办法无法避免该问题，就是程序中存在严重的内存泄漏，如频繁使用Class.forName()加载驱动，建议使用如下方式，在数据库操作类中使用静态块完成驱动的加载，如下所示<br/>

`static{
        Class.forName(driver_name)
    }`
  这样整个系统加载一次驱动就可以实现数据库链接的获取
##2.[**Mysql**]【20160810】    Too many connections<br/>
【**问题描述**】频繁操作数据库，报如下错误:com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException: Data source rejected establishment of connection, message from server: “Too many connections”<br/>
【**问题原因**】并发连接数太小或者系统繁忙导致连接数被占满<br/>
【**解决办法**】MY.INI(linux下为/etc/my.cnf)找到max_connections,修改为500-1000的值即可，修改完成后重启数据库即可
