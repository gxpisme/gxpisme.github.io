---
title: 搭建PHP环境（PHP5.6）
date: 2018-04-08 19:52:41
tags:
    - PHP
categories: PHP
---
<!-- more -->
## 背景
搭建简单的PHP5.6环境供调试使用。

## 搭建PHP5.6环境

### 安装PHP5.6
```
下载 wget http://cn2.php.net/get/php-5.6.35.tar.gz/from/this/mirror -O php-5.6.35.tar.gz
解压 tar xzvf php-5.6.35.tar.gz
    cd php-5.6.35
安装三步曲 ./configure --disable-all --prefix=/yourpath/php5
         make
         make install
```

### 安装vld扩展
```
下载 wget http://pecl.php.net/get/vld-0.14.0.tgz
解压 tar xzvf vld-0.14.0.tgz
    cd vld-0.14.0
安装扩展小四步 /yourpath/php5/bin/phpize
             ./configure --with-php-config=/yourpath/php5/bin/php-config --enable-vld
             make
             make install
```
创建/yourpath/php5/lib/php.ini
php.ini文件中的内容为
```
extension_dir="/yourpath/php5/lib/php/extensions/no-debug-non-zts-20131226"
extension="vld.so"
```

### 安装xdebug扩展
```
下载 wget https://xdebug.org/files/xdebug-2.5.5.tgz
解压 tar xzvf xdebug-2.5.5.tgz
    cd xdebug-2.5.5

安装扩展小四步 /yourpath/php5/bin/phpize
            ./configure --with-php-config=/yourpath/php5/bin/php-config --enable-xdebug
             make
             make install
```
添加到php.ini中，由于xdebug必须在zend的扩展中，所以php.ini配置如下
```
extension_dir="/yourpath/php5/lib/php/extensions/no-debug-non-zts-20131226"
extension="vld.so"
zend_extension="/yourpath/php5/lib/php/extensions/no-debug-non-zts-20131226/xdebug.so"
```

## 查看安装的模块
```
[xpisme@aliyun ~]$ /yourpath/php5/bin/php -m

[PHP Modules]
Core
date
ereg
pcre
Reflection
SPL
standard
vld
xdebug

[Zend Modules]
Xdebug
```


vld使用：http://www.php-internals.com/book/?p=C-php-vld
