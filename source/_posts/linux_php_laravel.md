
---
title:  Ubuntu 16.04安装Laravel
category:  php
tags: php
---

Laravel是开源的PHP框架，方便用户开发MVC web应用。
Laravel源代码：https://github.com/laravel/laravel
<!--more-->
Ubuntu 16.04安装Laravel

# 安装PHP
```bash
apt update
apt upgrade
apt install php php-mcrypt php-gd php-mbstring php-mysql
```

# 安装composer
```bash
curl -sS https://getcomposer.org/installer | php
chmod +x composer.phar
mv composer.phar /usr/local/bin/composer
```
# 安装MySQL数据库

首先，更新升级系统：
```bash
$ sudo apt update
$ sudo apt upgrade
```
安装MariaDB
```bash
$ sudo apt install mariadb-server
```
启动MariaDB服务
```bash
$ sudo systemctl start mysql
$ sudo systemctl enable mysql
```

# 安装Apache
```bash
apt install apache2
```
启动apache服务
```bash
systemctl start apache2
systemctl enable apache2
```

# 下载Laravel
```bash
cd /var/www/html
git clone https://github.com/laravel/laravel.git
```
# 安装Laravel
```bash
cd laravel
composer install
```
# 设置文件权限
```bash
chown -R www-data: /var/www/html/laravel
# chmod -R 755 /var/www/laravel
```
创建.env配置文件
```bash
mv .env.example .env
```
生成App key
```
$ php artisan key:generate
```
编辑config/app.php，把上面生成的key写入：
![图片](http://www.linuxdiyf.com/linux/uploads/allimg/161031/2-161031222930449.jpg)

添加虚拟主机配置文件
```bash
$ vim /etc/apache2/sites-available/laravel.conf
```
写入内容
```xml
<VirtualHost *:80>
ServerAdmin admin@your_domain.com
DocumentRoot /var/www/html/laravel/public/
ServerName your_domain.com
ServerAlias www.your_domain.com
<Directory /var/www/html/laravel/>
Options FollowSymLinks
AllowOverride All
Order allow,deny
allow from all
</Directory>
ErrorLog /var/log/apache2/your_domain.com-error_log
CustomLog /var/log/apache2/your_domain.com-access_log common
</VirtualHost>
```
注意替换上面的域名。
使生效：
```bash
a2ensite laravel.conf
systemctl restart apache2
```
使用浏览器访问：your_doamin_or_IP
![图片](http://www.linuxdiyf.com//linux/uploads/allimg/161031/2-1610312229415M.JPG)
