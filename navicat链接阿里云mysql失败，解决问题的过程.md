navicat链接阿里云mysql失败，解决问题的过程

1.进入mysql

mysql -uroot -p

2.切换到mysql服务器

use mysql

3.特定用户host修改

```update user set host='%' where user='root';```

4.指定用户授权

```grant all privileges on test.* to root@'%';```

5.查看云服务器的安全组规则中的3306端口是否开启，未开启则进行开启

6.查看防火墙的http，https端口及mysql是否开启通信，未开启，则进行开启

7.重启实例

8.使用   **端口扫描-站长之家**，测试3306端口是否对外开启

