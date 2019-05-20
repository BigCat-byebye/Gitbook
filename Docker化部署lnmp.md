docker的环境：基于ubuntu

部署思路：创建一个共享目录，将3个容器的/var目录集合到一起，方便存取网页文件，由于使用--link参数实现容器通信，

一，共享卷的设定
``` shell
docker volume create lnmp-share 
chmod 777 -R /var/lib/docker/volumes/lnmp-share
```

二，mysql的设定
``` shell
1. docker run -itd --name mysql -p 3306:3306 -v lnmp-share:/var mysql bash

2. 修改mysql的配置文件/etc/mysql下的文件
  a，将debian.cnf的socket路径修改为/var/lib/mysqld/mysqld.sock
  # 如果修改socket文件，或者pid文件的路径，那么一定要修改该debian.cnf中的路径，否则会报错ssl错误
  b，将mysql.conf.d/mysqld.cnf中的/var/run/mysqld/目录修改为/var/lib/mysqld/目录，这样是为了方便将mysql的数据目录以及socket文件共享出来
3. service msyql start
  a, mysql -uroot
  b, use mysql;
  c, select user,host,authentication_string from user;
  d, update user set authentication_string=password('123456') where user='root';
  e, grant all privileges on *.* to lnmp@'%' identified by '123456';
  f, flush privileges;
4. service mysql restart
# 此处有个问题，当我将/var/目录共享出来后，在容器内使用mysql -uroot -p登陆的时候，报错，必须要使用-S path_to_socket才可以登陆。。。。。暂还没解决
```

三，php-fpm的设定
``` shell
1. docker run -itd --name php-fpm -v lnmp-share:/var --link mysql php-fpm bash
2. 修改php-fpm的配置文件，将/etc/php/7.2/fpm/pool.d/www.conf中的listen改为使用ip：port方式，ip修改为自身ip，172.17.0的那个
3. service php-fpm start
```

四，nginx的设定
``` shell
1. docker run -itd --name nginx -v lnmp-share:/var --link mysql --link php-fpm -p 80:80 nginx bash
2. 修改nginx的配置文件
  a, 配置php-fpm的解析
        location ~ \.php$ {
              fastcgi_pass 172.17.0.3:9000;
              # 此处ip为php-fpm的ip地址
              fastcgi_index index.php;
              include fastcgi_params;
              fastcgi_param SCRIPT_FILENAME        $document_root$fastcgi_script_name;
              # 注意此处，由于php-fpm和nginx不在同一个容器下面，如果没有共享目录的话，那么需要php-fpm解析的php文件应该放到php-fpm的容器里面的/var/www/html/目录下，但是由于我们将lnmp-share挂载到了/var目录下，所以此处不需要担心
      }
  b, 配置nginx配置文件中的index.php，增加一个index.php即可
    # Add index.php to the list if you are using PHP
  index index.html index.htm index.php index.nginx-debian.html;
3. nginx即可
```