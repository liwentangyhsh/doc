# Docker 

先建立数据存放目录（/data/）mysql主从配置

```javascript
--mysql
  --master
     --data  
     --conf
        --my.cnf    
  --slave
     --data  
     --conf
        --my.cnf
```

Master:

```
[mysqld]
server_id=1
log-bin=mysql-bin
read-only=0
replicate-ignore-db=mysql
replicate-ignore-db=sys
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema

#
# include all files from the config directory
#
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
```

Slave:

```
[mysqld]
server_id=2
log-bin=mysql-bin
read-only=1
replicate-ignore-db=mysql
replicate-ignore-db=sys
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema

#
# include all files from the config directory
#
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
```



创建master主容器

```
docker run --name mastermysql -d -p 3307:3306 
-e MYSQL_ROOT_PASSWORD=root 
-v ~/test/mysql_test/master/data:/var/lib/mysql 
-v ~/test/mysql_test/master/conf/my.cnf:/etc/mysql/my.cnf 
mysql:5.7
```

创建slave从容器

```
docker run --name slavemysql -d -p 3308:3306 
-e MYSQL_ROOT_PASSWORD=root 
-v ~/test/mysql_test/slave/data:/var/lib/mysql 
-v ~/test/mysql_test/slave/conf/my.cnf:/etc/mysql/my.cnf 
mysql:5.7
```

master容器设置 

创建主容器的复制账号

```sql
//进入master容器
docker exec -it mastermysql bash
//启动mysql命令，刚在创建窗口时我们把密码设置为：root
mysql -u root -proot
//创建一个用户来同步数据，每个slave使用标准的MySQL用户名和密码连接master。
//进行复制操作的用户会授予REPLICATION SLAVE 权限。

CREATE USER 'slave'@'%' IDENTIFIED BY '123456'; 
GRANT REPLICATION SLAVE ON *.* to 'slave'@'%' identified by '123456';

//这里表示创建一个slaver同步账号slave，允许访问的IP地址为%，%表示通配符
//查看状态，记住File、Position的值，在Slave中将用到
show master status/G;
```

slave容器设置

```
//进入slaver容器
docker exec -it slavemysql bash
//启动mysql命令，刚在创建窗口时我们把密码设置为：root
mysql -u root -proot
//设置主库链接
change master to master_host='172.17.0.2',master_user='slave',master_password='123456',master_log_file='mysql-bin.000001',master_log_pos=0,master_port=3306;//启动从库同步
start slave;//查看状态
show slave status\G;
```

如果 show slave status\G命令结果中出现： 

Slave_IO_Running: Yes 

Slave_SQL_Running: Yes 

以上两项都为Yes，那说明主从配置好了



参考文档：

​	https://cloud.tencent.com/developer/article/1576556

​	https://my.oschina.net/u/3773384/blog/1810111



资料：
如何学习区块链技术？
https://www.zhihu.com/question/51047975/answer/259779402
《Node.js区块链开发》
http://bitcoin-on-nodejs.ebookchain.org/
maven调用本地nodejs命令
https://www.cnblogs.com/xzf-forever/p/6842984.html
区块链2文档，内容比较齐全
https://pan.baidu.com/s/1QDV4LPOlZtGSTIe-as0Y-Q#list/path=%2F
无人驾驶区块链技术指南【5rjs.cn】
https://pan.baidu.com/s/1pMAqGzh?_at_=1615779111636
一个实践项目，并且还配有一本书 《Node.js区块链开发》
https://github.com/Ebookcoin/ebookcoin
https://github.com/ddnlink/ddn