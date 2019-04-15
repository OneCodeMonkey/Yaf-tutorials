## 1.关于yaf

yaf是一个C语言编写的PHP框架

> （剑的三层境界：1.手中有剑，心中亦有剑；2.手中无剑，心中有剑；3.手中无剑，心中亦无剑）
>
> 

#### 1_1.yaf的特点

在和其他用PHP框架来比的时候，yaf就是剑的第二层境界，框架不再我们手中，而在PHP的“心”中



用PHP扩展来写PHP框架的难点

1). 难于开发，要完成一个PHP扩展的框架，需要有C语言背景，又要有PHP扩展的开发背景，也要有PHP框架设计背景

2). 目标用户群小，现在国内很多中小型站都是用虚拟主机，并不能随意给PHP添加扩展，所以这部分的用户无法用此框架

3). 维护成本高，要维护PHP扩展，不仅仅需要精通C开发和调试，也要精通Zend API，并且维护升级的周期会变长



用PHP扩展写PHP框架的可行性

1). 扩展逻辑相对稳定，一般不轻易变化，把他们抽象出来用PHP扩展取去实现，不会带来额外的维护负担那

2). 框架逻辑如果比较复杂，那么自检的耗时，耗内存都会加大。但如果用PHP扩展来实现，几乎不会带来额外的性能开销。



#### 1_2.yaf的优点



1. 用C语言开发的PHP框架，相比原生PHP，几乎不会带来额外的性能开销。
2. 所有的框架类，不需要去编译，在PHP启动的时候就加载，并常驻内存中。
3. 更短的内存周转周期，提高内存使用率，降低内存占用率。
4. 灵巧的自动加载机制，同时支持全局和局部加载规则，方便类库共享。
5. 高性能的视图引擎（我们也可以使用其他模板引擎）
6. 高度灵活可扩展的框架，支持自定义视图引擎，支持插件，支持自定义路由等。
7. 内建多种路由，可以兼容目前常见的路由协议
8. 强大而又高度灵活的配置文件支持，并支持缓存配置文件，避免复杂的配置结构带来的性能损失。
9. 在框架本身对危险的操作习惯做了禁止。
10. 更快的执行速度，高性能为最大特色。



#### 1_3.流程图



Yaf提供了完整的一套API，并支持Bootstrap和插件机制，流程图如下：

![img](http://www.laruence.com/manual/images/yaf_sequence.png)



#### 1_4.yaf的性能



用Apache 的 ab 工具可模拟并发量进行测试。



## 2.yaf的安装/配置

#### 2_1.yaf的安装

yaf只支持PHP5.2+的版本

yaf需要 SPL 的支持，SPL在PHP5中是默认开启的扩展模块

yaf需要PCRE的支持，PCRE也是在PHP5中默认开启的扩展模块

###### 在Linux下的安装步骤

1). 下载yaf的最新版本，解压缩以后，进入yaf的源码目录，依次执行这几条命令：

```php
$PHP_BIN phpize

./configure --with-php-config=$PHP_BIN/php-config

make

make install
```

然后在php.ini载入yaf.so，然后重启PHP服务

Yaf_Request_Abstract的getPost，getQuery等方法没有对应的setter方法，并且这些方法是直接从PHP内部的$_POST,$_GET等大变量本身通过只读方式拿过来的查询值。所以就有一个问题，通过在PHP脚本中对这些变量的修改，并不能反映到getPost/getQuery等方法上面。

例子：

```php
<?php
class Index_Controller extends Yaf_Request_Abstract {
    public function indexAction () {
        $_POST['name'] = "new_name";
        // 此时对$_POST的修改，并不能反映到getPost上
        echo $this->getRequest->getPost("name"); // echo old_name
    }
}
```

当然这样设计是经过深思熟虑的，也可以不依赖PHP的variable_orders的配置，但是带来一个问题就是QA和Rd无法通过修改这些变量来做测试数据。

所以Yaf专门提供了一个Debug模式，在这个模式下，getPost / getQuery / getServer / getCookie 将从符号表中的对应变量来查询得到值，从而可以让我们直接对PHP的超级变量做的修改能反映到对应的 Yaf_Request_Abstract 的方法上。

> （注意：请不要在正式的环境中以 Debug 模式来编译 Yaf，这个做一是有一定性能损耗，二是即使这么做了，但这种做法与 $_POST 这类大变量设计之初的"只读"特性相违背。所以我们在 yaf_Application 的 construct() 里，如果当前 yaf 是以 debug 模式编译i的，会触发一个 E_STRICT 的提示:
>
>   Strict Standards: you are running ap in debug mode）
>
> 例子：以 debug 模式编译yaf
>
> ```
> $PHP_BIN/phpize
> ./configure --enable-ab-debug --with-php-config=$PHP_BIN/php-config
> make
> make install
> ```
>
> 

#### 2_2.yaf定义的常量



| 常量（启用命名空间后的常量名）                               | 说明                                                |
| :----------------------------------------------------------- | --------------------------------------------------- |
| YAF_VERSION （Yaf \ VERSION）                                | Yaf 框架的三位版本信息                              |
| YAF_ENVIRON (Yaf \ ENVIRON)                                  | Yaf 的环境常量，指明了要读取的配置节，默认是product |
| YAF_ERR_STARTUP_FAILED （Yaf \ ERR \ STARTUP_FAILED）        | Yaf 的错误代码常量，表示启动失败，值 512            |
| YAF_ERR_ROUTE_FAILED (Yaf \ ERR \ ROUTE_FAILED)              | Yaf 的错误代码常量，表示路由失败，值 513            |
| YAF_ERR_DISPATCH_FAILED (Yaf \ ERR \ DISPATCH_FAILED)        | Yaf 的错误代码常量，表示分发失败，值 514            |
| YAF_ERR_NOTFOUND_MODULE (Yaf \ ERR \ NOTFOUD \ MODULE)       | Yaf 的错误代码常量，找不到指定的模块，值 515        |
| YAF_ERR_NOTFOUND_CONTROLLER (Yaf \ ERR \ NOTFOUD \ CONTROLLER) | Yaf 的错误代码常量，找不到指定的 Controller，值 516 |
| YAF_ERR_NOTFOUND_ACTION (Yaf \ ERR \ NOTFOUD \ ACTION)       | Yaf 的错误代码常量，找不到指定的 Action，值 517     |
| YAF_ERR_NOTFOUND_VIEW (Yaf \ ERR \ NOTFOUD \ VIEW)           | Yaf 的错误代码常量，找不到指定的 View，值 518       |
| YAF_ERR_CALL_FAILED (Yaf \ ERR \ CALL_FAILED)                | Yaf 的错误代码常量，调用失败，值 519                |
| YAF_ERR_AUTOLOAD_FAILED (Yaf \ ERR \ AUTOLOAD_FAILED)        | Yaf 的错误代码常量，自动加载类失败，值 520          |
| YAF_ERR_TYPE_ERROR (Yaf \ ERR \ TYPE_ERROR)                  | Yaf 的错误代码常量，表示关键逻辑的参数错误，值 521  |



#### 2_3.yaf的配置项



###### yaf 的配置选项

| 选项名称             | 默认值  | 可修改范围     | 更新记录                                                     |
| -------------------- | ------- | -------------- | :----------------------------------------------------------- |
| yaf.environ          | product | PHP_INI_ALL    | 环境名称，当用INI作为Yaf的配置文件时，这个指明了Yaf将要在INI配置中读取的节的名字 |
| yaf.library          | NULL    | PHP_INI_ALL    | 全局类库的目录路径                                           |
| yaf.cache_config     | 0       | PHP_INI_SYSTEM | 是否缓存配置文件(只针对INI配置文件生效，打开此选项可在复杂配置的情况下提高新功能) |
| yaf.name_suffix      | 1       | PHP_INI_ALL    | 在处理 Controller, Action, Plugin, Model 的时候，类名中关键信息是否是后缀式，比如 UserModel, 在前缀模式下则变成了 ModelUser |
| yaf.name_separator   | ""      | PHP_INI_ALL    | 在处理 Controller, Action, Plugin, Model 的时候，前缀和名字之间的分隔符，默认为空。也就是说比如 UserPlugin 加入设置为 "_", 则判断的依据变成 "User_Plugin", 这个主要是为了兼容 ST 已有的命名规范 |
| yaf.forward_limit    | 5       | PHP_INI_ALL    | forward 的最大嵌套深度                                       |
| yaf.use_namespace    | 0       | PHP_INI_SYSTEM | 开启时，Yaf 将使用命名空间的方式注册自己的类，比如 Yaf_Application 将会变成 Yaf \ Application |
| yaf.use_sql_autoload | 0       | PHP_INI_ALL    | 开启时，Yaf 在加载不成功的情况下，会继续让PHP的自动加载函数来加载，从性能上考虑，除非特殊情况，否则默认最好保持此选项关闭掉。 |

> （警告：在开启 yaf.cache_config 的情况下，yaf 会使用 INI 文件路径作为 Key，这就有一个陷阱，就是如果在一台服务器上同时运行两个应用，那么它们必须不能使用同一个路径名下的 INI 配置文件，否则就会出现 Application Path 混乱的情况。故尽量不要使用相对路径。）
>
> 

## 3.quick start

#### 3_1.需要些什么？

Yaf 已经正确编译安装好。

#### 3_2.Hello world

###### 目录结构

对于 yaf 的应用，都应遵循类似下面的目录结构

例：一个典型的 yaf 项目目录结构

```
public
|-	index.php  // 入口文件
|-	.htaccess  // 伪静态，重写规则
|+ 	css
|+	img
|+	js
conf
|-	application.ini  // 配置文件
application
|+	controllers
	|-	Index.php  // 默认控制器
|+	views
	|+	index  // 控制器
		|-	index.phtml  // 默认视图
|+	models
|+	modules  // 其他模块
|+	library  // 本地类库
|+	plugins  // 插件目录
```

###### 入口文件

入口文件是所有请求的入口，一般借助于 rewrite 规则，把所有请求都定向到这个如开口文件

> 例：一个经典的入口文件 public / index.php
>
> ```php
> <?php
> define("APP_PATH", realpath(dirname(__FILE__) . '/../'));
> $app = new Yaf_Application(APP_PATH . "/conf/application.ini");
> $app->run();
> ```
>
> ###### 重写规则
>
> 除非我们使用的是基于 query string 的路由协议（Yaf_Route_Simple, Yaf_Route_Supervar）, 否则我们就要使用 WebServer 提供的 Rewrite 规则，把所有应用请求重定向到我们定义的入口文件
>
> 例：Apache Rewrite 规则（httpd.conf中定义，或根目录下 .htaccess 文件中定义）
>
> ```xml
> #.htaccess
> RewriteEngine On
> RewriteCond %{REQUEST_FILENAME} !-f
> RewriteRule .* index.php
> ```
>
> 例：Nginx Rewrite 规则 （nginx.conf中定义）
>
> ```json
> server {
>     listen ****;
>     server_name domain.name;
>     root document_root;
>     index index.php index.html index.htm;
>     
>     if (!-e $request_filename) {
>         rewrite ^/(.*) /index.php/$1 last;
>     }
> }
> ```
>
> ###### 配置文件
>
> 在 Yaf 中，配置文件支持集成，支持分节，并对PHP常量进行支持，我们不必担心配置文件太大而造成解析性能上的问题，因为 Yaf 会在第一个运行时 的时候就载入配置文件，把格式化后的内容保持在内存中，直到配置文件有变化了，才会再次载入。
>
> 例：一个简单的配置文件 application / conf / application.ini
>
> ```
> [product]
> ;支持直接写PHP中的已定义常量
> application.directory=APP_PATH "/application/"
> ```
>
> ###### 控制器
>
> 在 Yaf 中，默认的模块 / 控制器 / 动作，都是以 Index 命名，当然这可以通过 配置文件来修改。
>
> 对于默认模块，控制器的目录是在 application 目录下的 controllers 目录下，Action 的命名规则是 ”名字 + Action“
>
> 例：默认控制器 application / controllers / Index.php
>
> ```php
> <?php
> class IndexController extends Yaf_Controller_Abstract {
>     public function indexAction () {
>         $this->getView()->assign("content", "Hello Yaf!");
>     }
> }
> ```
>
> ###### 视图文件
>
> Yaf 支持简单的视图引擎，并支持用户自定义自己的视图引擎，比如 Smarty 引擎
>
> 对于默认模块，视图文件的路径是在 application 目录下的 views 目录下，以小写的  action 为名的目录中
>
> 例：一个默认 Action 的视图 application / views / index / index.phtml
>
> ```php+HTML
> <html>
> <head>
> 	<title>Hello Yaf</title>        
> </head>
> <body>
> 	<?php echo $content; ?>    
> </body>    
> </html>
> ```
>
> ###### 运行
>
> 在浏览器里我们输入域名：
>
> ```
> http://{hostname}/application/index.php
> ```
>
> 此时如果前面操作无误的话，我们能看到浏览器页面正常显示：Hello Yaf!
>
> 

#### 3_3.使用代码生成工具



yaf 提供了代码生成工具 [yaf_code_generator](https://github.com/laruence/php-yaf/tree/master/tools/cg) ，我们可以用 yaf_cg 这个命令来生成一个简单的 Demo 应用

```shell
php-yaf-src/tools/cg/yaf_cg helloWorld
```

将得到的 helloworld 目录拷贝到 WebServer 的 documentRoot 目录下，然后用浏览器访问：

```shell
http://{hostname}/helloworld
```

效果等同于访问 

```shell
http://{hostname}/helloworld/application/index.php
```



## 4.配置文件

#### 4_1.必要的配置项

#### 4_2.可选的配置项

## 5.自动加载器

#### 5_1.全局类和自身类（本地类）

#### 5_2.类的加载规则

## 6.使用Bootstrap

#### 6_1.简介

#### 6_2.使用Bootstrap

## 7.使用插件

#### 7_1.简介

#### 7_2.Yaf支持的Hook

#### 7_3.定义插件

#### 7_4.注册插件

#### 7_5.插件目录

## 8.路由和路由协议

#### 8_1.概述

#### 8_2.设计

#### 8_3.默认情况

#### 8_4.使用路由

#### 8_5.路由协议详解

#### 8_6.自定义路由协议

## 9.在命令行使用Yaf

#### 9_1.简介

#### 9_2.使用样例

#### 9_3.分发请求

## 10.异常和错误

#### 10_1.概述

#### 10_2.异常模式

#### 10_3.错误模式

## 11.内建的类

#### 11_1.Yaf_Application

#### 11_2.Yaf_Bootstrap_Abstract

#### 11_3.Yaf_Loader

#### 11_4.Yaf_Dispatcher

#### 11_5.Yaf_Plugin_Abstract

#### 11_6.Yaf_Registry

#### 11_7.Yaf_Session

#### 11_8.Yaf_Config_Abstract

#### 11_9.Yaf_Controller_Abstract

#### 11_10.Yaf_Action_Abstract

#### 11_11.Yaf_View_Interface

#### 11_12.Yaf_Request_Abstract

#### 11_13.Yaf_Response_Abstract

#### 11_14.Yaf_Router

#### 11_15.Yaf_Route_Interface

#### 11_16.Yaf_Exception

