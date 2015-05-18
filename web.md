# 树莓派做 web 服务器

一想到 Linux Web 服务器，我们首先想到的是：Apache + MySql + Php。

- Apache 是世界使用排名第一的 Web 服务器软件。
可以运行在几乎所有广泛使用的计算机平台上，由于其跨平台和安全性被广泛使用，是最流行的 Web 服务器端软件。
- MySQL 是一个关系型数据库管理系统，由瑞典 MySQL AB 公司开发。是最流行的关系型数据库管理系统，在 WEB 应用方面 MySQL 是最好的 RDBMS(关系数据库管理系统)应用软件之一。
- PHP（外文名: Hypertext Preprocessor，中文名：“超文本预处理器”）是一种通用开源脚本语言。语法吸收了 C 语言、Java 和 Perl 的特点，易于学习，使用广泛，主要适用于 Web 开发领域。


树莓派可以安装这个 LAMP 系列，但 Apache 和 MySql 对于树莓派这个小小的机器，太重了，主要是消耗内存多＼速度慢＼占用磁盘大(约 200M)，所可以选择安装一个轻量级的 Web 服务器：
nginx + php + sqlite

- nginx 是个轻量级的 Web 服务器，是一款轻量级的 Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器,上 nginx 的并发能力确实在同类型的网页服务器中表现较好。
- SQLite 是一款轻型的数据库，是遵守 ACID 的关系型数据库管理系统，它的设计目标是嵌入式的，而且目前已经在很多嵌入式产品中使用了它，它占用资源非常的低，在嵌入式设备中，可能只需要几百 K 的内存就够了。

## Apache + MySql + Php 安装

### 安装 Apache

Apache 可以用下面的命令来安装`sudo apt-get install apache2`

Apache 默认路径是`/var/www/`

其配置文件路径为`/etc/apache2/`

可以通过`sudo vi /etc/apache2/ports.conf`修改监听端口号

重启服务生效`sudo service apache2 restart`

### 安装 mysql

`sudo apt-get install mysql-server`
安装过程中，会出现一个提示符让你输入一个密码。
这个密码是 mysql root 用户的密码。

### 安装 PHP

输入下面的命令，就可以安装 PHP 5,以及 PHP 访问 mysql 数据库所需要的库。

```
sudo apt-get install php5
sudo apt-get install php5-mysql
```

### 测试

安装完成后，可以在浏览器中输入你路由器的 IP 或域名，就可以访问你的网站了。你应该能看到一个页面显示“It works”，但是没有其它内容。

创建一个/var/www/index.php

```
<?php  
  print <<< EOT  
<!doctype html>  
<html lang="en">  
<head>  
<meta charset="UTF-8">  
<title>Test successful</title>  
</head>  
<body>  
<h1>Test successful</h1>  
<p>Congratulations.</p>  
<p>Your webserver and PHP are working.</p>  
</body>  
</html>  
EOT;  
  
?> 
```

## nginx + php + sqlite 安装

### 安装 nginx  web服务器 (约6MB)

```
sudo apt-get install nginx
```

### 启动 nginx

```
sudo /etc/init.d/nginx start
```

nginx 的 www 根目录默认在`/usr/share/nginx/www`中

### 修改 nginx 的配置文件

```
sudo vi /etc/nginx/sites-available/default
```

#### 以下几个选项注意一下

**listen 8080;## listen for ipv4; this line is default and implied** - 监听的端口号，如果与其它软件冲突，可以在这里更改。

**root /usr/share/nginx/www;** - nginx 默认路径 html 所在路径。
**index index.html index.htm index.php;** - nginx 默认寻找的网页类型，我们可以增加一个 index.php。

#### PHP 脚本支持

找到 php 的定义段，将这些行的注释去掉 ，修改后内容如下

```
location ~ .php$ {
　fastcgi_pass unix:/var/run/php5-fpm.sock;
　fastcgi_index index.php;
　include fastcgi_params;
}
```

php 段中有一些其它定义，不要去动它，比如

```
#      fastcgi_split_path_info ...
#      fastcgi_pass 127.0.0.1:9000
```

#### 安装 php 和 sqlite (约 3MB)

```
sudo apt-get install php5-fpm php5-sqlite
```

#### 重新加载 nginx 的配置

```
sudo /etc/init.d/nginx reload
```

#### 测试 html

通过主机的 IE 访问树莓派，可以看到主页(表示 Web 服务器已正常启动)

#### 测试 php

在树莓派中生成一`php`文件

```
sudo vi /usr/share/nginx/www/index.php
```

在文件中输入以下内容

```
<?php  
  print <<< EOT  
<!doctype html>  
<html lang="en">  
<head>  
<meta charset="UTF-8">  
<title>Test successful</title>  
</head>  
<body>  
<h1>Test successful</h1>  
<p>Congratulations.</p>  
<p>Your webserver and PHP are working.</p>  
</body>  
</html>  
EOT;  
  
?>  
```