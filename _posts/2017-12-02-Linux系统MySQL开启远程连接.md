# Linux系统MySQL开启远程连接



```mysql
#进入数据库
mhh123@ubuntu64:/$ mysql -uroot -p
Enter password: 
#创建mysql的普通用户
mysql> CREATE USER 'mhh123'@'%'IDENTIFIED BY 'mhh123';
Query OK, 0 rows affected (0.16 sec)
#第一个'mhh123'表示用户名，'%'表示所有电脑都可以连接,第二个'mhh123'表示密码;
mysql> GRANT ALL ON *.* TO 'mhh123'@'%';
Query OK, 0 rows affected (0.08 sec)
#刷新使命令立即生效
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.39 sec)
#查看MYSQL数据库中所有用户
mysql> SELECT DISTINCT CONCAT('User: ''',user,'''@''',host,''';') AS query FROM mysql.user;
+---------------------------------------+
| query                                 |
+---------------------------------------+
| User: 'mhh123'@'%';                   |
| User: 'root1'@'%';                    |
| User: 'debian-sys-maint'@'localhost'; |
| User: 'mysql.session'@'localhost';    |
| User: 'mysql.sys'@'localhost';        |
| User: 'root'@'localhost';             |
+---------------------------------------+
6 rows in set (0.02 sec)
#查看端口号
mysql> show global variables like 'port';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| port          | 3306  |
+---------------+-------+

```

```
#注意ubuntu和centos的配置文件路径不一样
root@ubuntu64:/# vim /etc/mysql/mysql.conf.d/mysqld.cnf 
root@ubuntu64:/# service mysql restart

将
bind-address            = 127.0.0.1
改成
bind-address            = 0.0.0.0
```

---------------------------------------------------------------------------------------------------------------------------

```
#在cmd中测试ip及端口
windows cmd 
ping ip  #地址看是否能ping 通
#在控制面板-程序-打开Telnet客户端
telnet ip 端口 #是否能连上相应的端口

```



```
#windos窗口远程链接数据库
C:\Users\Administrator>mysql -h192.168.40.128 -P3306 -umhh123 -p
Enter password: ******
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 5.7.20-0ubuntu0.16.04.1 (Ubuntu)
```

