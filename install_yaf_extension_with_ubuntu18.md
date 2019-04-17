# 使用PECL安装yaf（以Ubuntu18 + php7.3为例）

#### 1. pecl install yaf

```bash
sudo pecl install yaf channel-update pecl.php.net
```

###### 执行结果如下：

```shell
.....

running: make INSTALL_ROOT="/tmp/pear/temp/pear-build-rootrhPLgR/install-yaf-3.0.7" install
Installing shared extensions:     /tmp/pear/temp/pear-build-rootrhPLgR/install-yaf-3.0.7/usr/lib/php/20170718/
running: find "/tmp/pear/temp/pear-build-rootrhPLgR/install-yaf-3.0.7" | xargs ls -dils
1974702    4 drwxr-xr-x 3 root root    4096 May 20 09:52 /tmp/pear/temp/pear-build-rootrhPLgR/install-yaf-3.0.7
2106594    4 drwxr-xr-x 3 root root    4096 May 20 09:52 /tmp/pear/temp/pear-build-rootrhPLgR/install-yaf-3.0.7/usr
2106595    4 drwxr-xr-x 3 root root    4096 May 20 09:52 /tmp/pear/temp/pear-build-rootrhPLgR/install-yaf-3.0.7/usr/lib
2106596    4 drwxr-xr-x 3 root root    4096 May 20 09:52 /tmp/pear/temp/pear-build-rootrhPLgR/install-yaf-3.0.7/usr/lib/php
2106597    4 drwxr-xr-x 2 root root    4096 May 20 09:52 /tmp/pear/temp/pear-build-rootrhPLgR/install-yaf-3.0.7/usr/lib/php/20170718
2106593 1412 -rwxr-xr-x 1 root root 1442040 May 20 09:52 /tmp/pear/temp/pear-build-rootrhPLgR/install-yaf-3.0.7/usr/lib/php/20170718/yaf.so

Build process completed successfully
Installing '/usr/lib/php/20170718/yaf.so'
install ok: channel://pecl.php.net/yaf-3.0.7
configuration option "php_ini" is not set to php.ini location
You should add "extension=yaf.so" to php.ini
```

#### 2. 查看Yaf扩展位置

```
sudo ls /usr/lib/php/20180731/ | grep yaf
```

#### 3. 配置yaf扩展

新建 yaf.ini 文件

```shell
sudo vim /etc/php/7.3/mods-available/yaf.ini
```

在其中输入以下内容

```shell
extension=yaf.so
```

#### 4. 添加软链接

```shell
// fpm
sudo ln -s /etc/php/7.2/mods-available/yaf.ini /etc/php/7.2/fpm/conf.d/20-yaf.ini

// cli
sudo ln -s /etc/php/7.2/mods-available/yaf.ini /etc/php/7.2/cli/conf.d/20-yaf.ini
```

#### 5. 重启FPM服务

```shell
sudo service php7.3-fpm restart
```

#### 6. 验证

验证 yaf 模块是否安装成功

```shell
php -i| grep yaf
```

看到如下输出，说明扩展安装成功了！

```shell
vagrant@homestead:/$ php -i| grep yaf
/etc/php/7.3/cli/conf.d/20-yaf.ini,
yaf
yaf support => enabled
Supports => http://pecl.php.net/package/yaf
yaf.action_prefer => Off => Off
yaf.environ => product => product
yaf.forward_limit => 5 => 5
yaf.library => no value => no value
yaf.lowcase_path => Off => Off
yaf.name_separator => no value => no value
yaf.name_suffix => On => On
yaf.st_compatible => Off => Off
yaf.use_namespace => Off => Off
yaf.use_spl_autoload => Off => Off
```