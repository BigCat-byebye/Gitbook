Lnmp环境

1.lnmp环境安装和正常的环境一样，只是需要特别注意的是，phabricator不支持php7.0，所以此处需要注意
``` shell
yum install mariadb-server mariadb-libs mariadb
yum install nginx nginx-all-modules
yum install php php-fpm
yum install git php-mysql php-process php-devel php-gd php-pecl-apc php-pecl-json php-mbstring 
``` 
2.修改mysql默认密码
``` shell
# 此处有坑，mysql5.7和mysql5.7有较大差异，1是源于初始密码的不同，2是源于mysql表的字段值有所修改，3是对于密码的强度要求不同
``` 

3.复制phabricator的3件套到相关地方
``` shell
mv phabricator/* /usr/share/nginx
# phabriactor的3件套分别为arcanist/,libphutil/,phabricator/
```

4.配置nginx中phabricator的配置文件，下文语法源自官方文档，nginx的和apache的不一样，可查看官方配置
``` shell
server {
  server_name phab.local;
  root        /usr/share/nginx/phabricator/webroot;
# 默认的root目录应该为/usr/share/nginx/html/，此处为修改后的密码，因为后续需要做操作，即将phabricator的3件套移至root原目录的上一级目录后，再将root目录修改至phabricator目录中的webroot目录下
  location / {
    index index.php;
    rewrite ^/(.*)$ /index.php?__path__=/$1 last;
  }
  location /index.php {
    fastcgi_pass   localhost:9000;
    fastcgi_index   index.php;
    fastcgi_param  REDIRECT_STATUS    200;
    fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param  QUERY_STRING       $query_string;
    fastcgi_param  REQUEST_METHOD     $request_method;
    fastcgi_param  CONTENT_TYPE       $content_type;
    fastcgi_param  CONTENT_LENGTH     $content_length;
    fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
    fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
    fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;
    fastcgi_param  REMOTE_ADDR        $remote_addr;
  }
}
```

5.配置phabricator的mysql相关
``` shell
cd /usr/share/nginx/phabricator/
# 进入此目录，执行./bin/config set mysql.host 127.0.0.1，如果不行的，请根据报错进入相关目录，我记得好像是setup目录
./manage_config.php set mysql.host 127.0.0.1
# ./bin/config set mysql.host 127.0.0.1，本来默认执行命令是这个的，但是因为有时候PATH路径的问题，会报错，所以此处直接执行php文件进行设置
./manage_config.php set mysql.user root
./manage_config.php set mysql.pass 123456
./manage_storage.php upgrade
# ./bin/storage upgrade，同上述的./bin/config set mysql.host 127.0.0.1命令
systemctl restart nginx
systemctl restart mariadb
systemctl restart php-fpm
```

6.配置arc工具，很简单，配置一个PATH路径即可
``` shell
export PATH=$PATH:/usr/share/nginx/arcanist/bin
```

7.配置git仓库，结合phabricator
``` shell
git config --global user.name "admin"
git config --global user.email "admin@admin"
git config --list 

mkdir git
cd git/
git init
```