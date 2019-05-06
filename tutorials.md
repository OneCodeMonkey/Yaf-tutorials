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
> Strict Standards: you are running ap in debug mode）
>

例子：以 debug 模式编译yaf

```
$PHP_BIN/phpize
./configure --enable-ab-debug --with-php-config=$PHP_BIN/php-config
make
make install
```



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

例：一个经典的入口文件 public / index.php

```php
<?php
define("APP_PATH", realpath(dirname(__FILE__) . '/../'));
$app = new Yaf_Application(APP_PATH . "/conf/application.ini");
$app->run();
```

###### 重写规则

除非我们使用的是基于 query string 的路由协议（Yaf_Route_Simple, Yaf_Route_Supervar）, 否则我们就要使用 WebServer 提供的 Rewrite 规则，把所有应用请求重定向到我们定义的入口文件

例：Apache Rewrite 规则（httpd.conf中定义，或根目录下 .htaccess 文件中定义）

```xml
#.htaccess
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule .* index.php
```

例：Nginx Rewrite 规则 （nginx.conf中定义）

```php
server {
  listen ****;
  server_name domain.name;
  root document_root;
  index index.php index.html index.htm;
 
  if (!-e $request_filename) {
    rewrite ^/(.*) /index.php/$1 last;
  }
}
```

###### 配置文件

在 Yaf 中，配置文件支持集成，支持分节，并对PHP常量进行支持，我们不必担心配置文件太大而造成解析性能上的问题，因为 Yaf 会在第一个运行时 的时候就载入配置文件，把格式化后的内容保持在内存中，直到配置文件有变化了，才会再次载入。

例：一个简单的配置文件 application / conf / application.ini

```
[product]
;支持直接写PHP中的已定义常量
application.directory=APP_PATH "/application/"
```

###### 控制器

在 Yaf 中，默认的模块 / 控制器 / 动作，都是以 Index 命名，当然这可以通过 配置文件来修改。

对于默认模块，控制器的目录是在 application 目录下的 controllers 目录下，Action 的命名规则是 ”名字 + Action“

例：默认控制器 application / controllers / Index.php

```php
<?php
class IndexController extends Yaf_Controller_Abstract {
 public function indexAction () {
     $this->getView()->assign("content", "Hello Yaf!");
 }
}
```

###### 视图文件

Yaf 支持简单的视图引擎，并支持用户自定义自己的视图引擎，比如 Smarty 引擎

对于默认模块，视图文件的路径是在 application 目录下的 views 目录下，以小写的  action 为名的目录中

例：一个默认 Action 的视图 application / views / index / index.phtml

```php+HTML
<html>
<head>
	<title>Hello Yaf</title>        
</head>
<body>
	<?php echo $content; ?>    
</body>    
</html>
```

###### 运行

在浏览器里我们输入域名：

```
http://{hostname}/application/index.php
```

此时如果前面操作无误的话，我们能看到浏览器页面正常显示：Hello Yaf!

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



yaf 和用户共用一个配置空间，也就是在 Yaf_Application 初始化时给出的配置文件中的配置。作为区别，Yaf 的配置项均以 ap 为前缀，Yaf 的核心必不可少的配置只有一个。

> （注意：yaf 通过在不同的环境中，选取不同的配置节，再结合配置可继承，来实现 “一套配置，多种环境公用” ，比如：线上 -> 测试 -> 开发）

| 名称                  | 值类型 | 说明               |
| --------------------- | ------ | ------------------ |
| application.directory | String | 应用的绝对目录路径 |

#### 4_2.可选的配置项



Yaf 的可选配置项

| 名称                                         | 值类型        | 默认值                             | 说明                                                         |
| -------------------------------------------- | ------------- | ---------------------------------- | ------------------------------------------------------------ |
| application.ext                              | String        | php                                | PHP脚本的扩展名                                              |
| application.bootstrap                        | String        | Bootstrapplication.php             | Bootstrap路径（绝对路径）                                    |
| application.library                          | String        | application.directory + "/library" | 本地类库的绝对路径地址                                       |
| application.baseUri                          | String        | NULL                               | 在路由中，需要忽略的路径前缀，一般不需要设置，Yaf 会自动判断。 |
| application.dispatcher.defaultModule         | String        | index                              | 默认的模块                                                   |
| application.dispatcher.throwException        | Bool          | True                               | 在出错的时候，是否抛出异常                                   |
| application.dispatcher.catchException        | Bool          | False                              | 是否使用默认的异常捕获 Controller, 如果开启，那么在有未捕获的异常时，控制权会交给 ErrorController 的 errorAction() ，可以通过 $request->getException() 获得此异常对象 |
| application.dispatcher.defaultAction         | String        | index                              | 默认的控制器名                                               |
| application.view.ext                         | String        | phtml                              | 使用的视图模板文件的扩展名                                   |
| application.modules  和 application.system.* | String String | Index                          *   | 声明存在的模块名，请注意：如果我们定义这个值，一定要定义 Index Module，通过这个属性可以修改 yaf 的 runtime configure，比如 application.system.lowcase_path, 此处注意只有 PHP_INI_ALL 的配置项才可以在这里被修改。 |



## 5.自动加载器



Yaf 在自启动的时候，会通过 SPL 注册一个自己的 Autoloader， 出于性能的考虑，对于框架相关的 MVC 类，Yaf Autoloader 只以映射目录的方式尝试一次。

注意：Yaf 支持在PHP脚本中触发对 Controller 的自动加载，但因为 Controller 的定位需要根据 Module 路由结果来判断，这就造成了在 Bootstrap 或者 RouteStartUp 之前无法确定。所以对于 Controller 的自动加载，Yaf 将只尝试加载默认的 Module 的 Controller ，也就是只在 “{项目路径} / controllers” 目录下寻找

具体的目录映射规则如下：

###### Yaf 目录映射规则：

| 类型     | 后缀或者前缀，可通过php.ini中的ap.name_suffix切换 | 映射路径                                                     |
| -------- | ------------------------------------------------- | ------------------------------------------------------------ |
| 控制器   | Controller                                        | 默认模块下名为 {项目路径 / controllers / ，否则为 {项目路径} / modules / {模块名} / controllers / |
| 数据模型 | Model                                             | {项目路径} / models /                                        |
| 插件     | Plugin                                            | {项目路径} / plugins /                                       |

而对于非框架 MVC 相关的类，Yaf 支持全局类和自身类的两种加载方式，并且 Yaf 支持大小写敏感和不敏感两种方式来处理文件路径。



#### 5_1.全局类和自身类（本地类）

Yaf 为了方便在一台服务器上部署的不同产品之间共享公司级别的共享库，支持全局类和本地类两种加载方式。

全局类是指所有产品之间共享的类，这些类库的路径是通过 ap.library 在php.ini ，或在PHP编译时支持了 with-config-file-scan-dir ，可以写在 ap.ini 里。

本地类是指产品自身的类，这些类库的路径是通过在产品的配置文件中，通过 ap.library 配置，在 Yaf 中通过调用 Yaf_Loader 的 registerLocalNamespace 的方法，来声明哪些类前缀是本地类。

> 注意：在 use_spl_autoload 此项配置关闭的情况下， Yaf Autoloader 在一次找不到的情况下，会立即返回，而剥夺其后的自动加载器的执行机会。

#### 5_2.类的加载规则

类的加载规则都是一样的： Yaf 规定类名中必须包含路径信息，也就是以 “_” 分割的目录信息，Yaf 将依照类名中的目录信息完成自动加载。如下的例子在没有声明本地类的情况下：

例：一个映射的例子 Zend_Dummy_Foo

```php
// Yaf 将在如下路径寻找类 Foo_Dummy_Bar
{类库路径（php.ini中指定的ap.library）}/Foo/Dummy/Bar.php
```

而通过下面这种方式调用 registerLocalNamespace:

```php
// 声明，凡是以 Foo 和 Local 开头的类，都是本地类
$loader = Yaf_Loader::getIgnstance();
$loader->registerLocalNamespace(array("Foo", "Local"));
```

那么对于刚才的例子，将会在以下路径里寻找 Foo_Dummy_Bar

例：一个映射的例子 Zend_Dummy_Bar

```php
// Yaf 将在如下路径寻找类 Foo_Dummy_Bar
{类库路径（php.ini中指定的ap.library）}/Foo/Dummy/Bar.php
```



## 6.使用Bootstrap

#### 6_1.简介

Bootstrap 也叫引导程序，是 Yaf 提供的一个全局配置的入口，在 Bootstrap 中我们可以做很多全局自定义工作。

#### 6_2.使用Bootstrap

一个 Yaf_Application 被实例化之后，运行 Yaf_Application::run 之前，可选的我们可以运行 Yaf_Application::bootstrap

例：调用 Bootstrap

```php
<?php
$app = new Yaf_Application("conf.ini");
$app->bootstrap()->run();
```

当 bootstrap 被调用时，Yaf_Application 就会默认在 APPLICATION_PATH 下寻找 Bootstrap.php，而在这个文件中，必须定义一个 Bootstrap 类，而这个类也必须继承自 Yaf_Bootstrap_Abstract.

实例化成功之后，所有在 Bootstrap 类中定义的，以 _init 开头的方法，都会被依次调用，而这些方式都可以接受一个 Yaf_Dispatcher 实例做为参数

> 注意：也可以通过在配置文件中修改 application.bootstrap 来变更 Bootstrap 类的位置。

一个 Bootstrap 的例子：

例：Bootstrap

```php
<?php
/**
 * 所有在 Bootstrap 类中，以 _init 开头的方法都会被 Yaf 调用。这些方法都接受一个参数 Yaf_Dispatcher $dispatcher, 调用次序和声明顺序一样
 */
class Bootstrap extends Yaf_Bootstrap_Abstract{
	public function _initConfig ()
    {
        $config = Yaf_Application::app()->getConfig();
        Yaf_Registry::set("config", $config);
    }
    public function _initDefaultName(Yaf_Dispatcher $dispatcher)
    {
        $dispatcher->setDefaultModule("Index")->setDefaultController("Index")->setDefaultAction("index");
    }
}
```



## 7.使用插件

#### 7_1.简介

Yaf 支持用户自定义插件来扩展 Yaf 的功能，这些插件都是一些类，他们都必须继承自 Yaf_Plugin_Abstract. 插件要发挥功效也必须在 Yaf 中进行注册，然后在适当时机 Yaf 就会调用它。

#### 7_2.Yaf支持的Hook

支持 6 个 Hook，如下表所示：

| 触发顺序 | 名称                 | 触发时机               | 说明                                                         |
| -------- | -------------------- | ---------------------- | ------------------------------------------------------------ |
| 1        | routerStartup        | 在路由启动之前触发     | 这是 7 个事件中最早的要给，但是一些全局自定义的工作还是应该放在 Bootstrap 中去完成 |
| 2        | routerShutdown       | 路由结束之后触发       | 此时路由一定正确完成，否则这个事件不会触发                   |
| 3        | dispatchLoopStartup  | 分发循环开始之前被触发 |                                                              |
| 4        | preDispatch          | 分发之前触发           | 如果在一个请求处理过程中，发生了 forward ,则这个事件会被多次触发 |
| 5        | postDispatch         | 分发结束之后触发       | 此时动作已经执行结束，视图也已经渲染完成，和 preDispatch 类似，此事件也可能触发多次 |
| 6        | dispatchLoopShutdown | 分发循环结束之后触发   | 此时表示所有的业务逻辑都已完成，但是响应还没有发送           |



#### 7_3.定义插件

插件类是用户编写的，但是它需要继承自 Yaf_Plugin_Abstract 实例和 Yaf_Response_Abstract 实例，一个插件类的例子如下：

例：插件类

```php
<?php
class UserPlugin extends Yaf_Plugin_Abstract {
    public function routerStartup (Yaf_Request_Abstract $request, Yaf_Request_Abstract $response)
    {
        //
    }
    public function routerShutdown (Yaf_Request_Abstract $request, Yaf_Response_Abstract $response)
    {
        //
    }
}
```

这个例子中插件 UserPlugin 只关心两个事件，所以只定义了两个方法。

#### 7_4.注册插件

只有向 Yaf_Dispatcher注册了插件之后，我们的插件才能生效。一般的插件注册都会放在 Bootstrap 中进行。

例：注册插件

```
<?php
class Bootstrap extends Yaf_Bootstrap_Abstract {
    public function _initPlugin (Yaf_Dispatcher $dispatcher)
    {
        $user = new UserPlugin();
        $dispatcher->registerPlugin($user);
    }
}
```



#### 7_5.插件目录

一般地，插件应该放置在 APPLICATION_PATH 下的 plugins 目录下，这样在自动加载的时候加载器通过类名，发现这是个插件类，就会在这个目录下查找。

当然插件也可以放在任何你想放置的地方，只要最终能达到把这个类加载进来的目的就没问题

> 更多详细内容参考 
>
> [Yaf_Plugin_Abstract]: http://www.laruence.com/manual/yaf.class.plugin.html
>
> 

## 8.路由和路由协议

#### 8_1.概述

路由作为负责解析一个请求，并且决定交给哪个module，哪个controller，哪个action去处理的角色，它同时也定义了一种方法来实现用户自定义路由，这也使得它成为最重要的一个 MVC 组件。

为了方便自定义路由，Yaf 采用更灵活的路由器和路由协议分离的模式。也就是一个固定不变的路由器，配合各种可自定义的路由协议，来实现灵活多变的路由策略。

#### 8_2.设计

作为一个应用中的路由组件是很重要的，理所当然地路由组件是抽象的，这样允许开发者很容易地设计出我们自定义的路由协议。然而默认的路由组件其实已经服务的很好了。

当我们确实需要用一个非标准的路由协议时，我们可以自定义一个自己的路由协议而放弃默认的路由协议。事实上路由组件由两部分组成：路由器（Yaf_Router）和路由协议（Yaf_Route_Abstract）.

路由协议事实上主要负责匹配我们预先定义好的路由协议，意思就是我们只有一个路由器，当我们可以有很多路由协议，路由器主要负责管理和运行路由链，它根据路由协议栈倒序依次调用各个路由协议，直到某个路由协议返回成功以后，就匹配成功。

> 注意：路由注册的顺序很重要，最后注册的路由协议，最先尝试路由，这就有个陷阱

路由的过程发生在 dispatch 过程中的最开始，并且路由解析仅发生一次，路由过程在任何控制器或动作被 dispatch 之前被执行，一旦路由成功，路由器会将解析出的信息传递给请求对象（Yaf_Request_Abstract object），这些信息包括 module，controller，action，用户参数等。然后 dispatcher 会按照这些信息派遣正确的控制器动作。路由器的插件 Hook 是 routerStartup 和 routerShutdown，顾名思义一个在路由解析前，一个在路由解析后被调用。

#### 8_3.默认情况

默认情况下，我们的路由器是 Yaf_Router, 默认使用的路由协议是 Yaf_Route_Static, 这是基于 HTTP 路由的。它期望一个请求是 HTTP 请求并且请求对象使用 Yaf_Request_HTTP

#### 8_4.使用路由

使用路由既可以让其很复杂，同时也可以让其很简单，这取决于我们的应用怎么设计的。然而使用一个路由是很简单的，我们可以添加路由协议给路由器。

不同的路由协议如下：

[Yaf_Route_Simple]: http://www.laruence.com/manual/yaf.routes.usage.html
[Yaf_Route_Supervar]: http://www.laruence.com/manual/yaf.routes.usage.html
[Yaf_Route_Static]: http://www.laruence.com/manual/yaf.routes.usage.html
[Yaf_Route_Map]: http://www.laruence.com/manual/yaf.routes.usage.html
[Yaf_Route_Rewrite]: http://www.laruence.com/manual/yaf.routes.usage.html
[Yaf_Route_Regex]: http://www.laruence.com/manual/yaf.routes.usage.html

首先看下路由器是如何让路由协议与之一起工作的，在我们添加任何路由协议之前，我们必须要拿到 Yaf_Route 实例，我们通过派遣器 Yaf_Dispatcher 的 getRouter() 方法来得到默认的路由器

```php
<?php
$router = Yaf_Dispatcher::getInstance()->getRouter();
```

一旦我们有了路由器实例，我们就能通过它来添加我们自定义的一些路由协议：

```php
<?php
$router->addRoute('myRoute', $route);
$router->addRoute('myRoute1', $route);
```

除此之外，我们还可以直接在配置文件中添加我们的路由协议：

例：配置添加路由协议的例子

```ini
[common]
;自定义路由
;注意，顺序很关键
routes.regex.type="regex"
routes.regex.match="#^/list/([^/]*)/([^/]*)#"
routes.regex.route.controller=Index
routes.regex.route.action=action
routes.regex.map.1=name
routes.regex.map.2=value

;添加一个名为 simple 的路由协议
routes.simple.type="simple"
routes.simple.controller=c
routes.simple.module=m
routes.simple.action=a

;添加一个名为 supervar 的路由协议
routes.supervar.type="supervar"
routes.supervar.varname=r

[product:common]
;product节是Yaf默认关心的节，添加一个名为 rewrite 的路由协议
routes.rewrite.type="rewrite"
routes.rewrite.match="/product/:name/:value"
```

> 注意：路由协议的顺序很重要，用 Yaf 要保证路由协议的添加顺序和在配置文件中的顺序相同

例：在 Bootstrap 中通过调用 Yaf_Router::addConfig 添加定义在配置中的路由协议

```php
<?php
class Bootstrap extends Yaf_Bootstrap_Abstract {
    public function _initRoute (Yaf_Dispatcher $dispatcher)
    {
        $router = Yaf_Dispatcher::getInstance()->getRouter();
        // 添加配置中的路由
        $router->addConfig(Yaf_Registry::get("config")->routes);
    }
}
```

> 其实路由器也提供了不同的方法来得到和设置包含在它内部的信息，我们可以用下面两个方法
>
> getCurrentRoute()    // 在路由结束之后，获取起作用的路由协议
>
> getRoute(), getRoutes()    // 获取路由协议

#### 8_5.路由协议详解

###### 默认路由协议

默认的路由协议 Yaf_Route_Static, 就是分析请求中的 request_uri, 在去除掉 base_uri 以后，获取到真正负载路由信息的 request_uri 片段，具体的策略是根据 "/" 对 request_uri 分段，依次得到 Module, Controller, Action, 在得到 Module 之后，还需要根据 Yaf_Application::$modules 来判断带你 Module 是否是合法的 Module，如果不是那么认为 Module 并没有体现在 request_uri 中，而把原 Module 当做 Controller， 原 Controller 当做 Action：

例：默认路由协议

```php
<?php
/**
 * 对于请求 request_uri 为 "/ap/foo/bar/dummy/1", base_uri 为 "/ap",则最后参加路由的 request_uri 为 "/foo/bar/dummy/1"
 * 然后通过对 URL 分段，得到如下信息 foo, bar, dummy, 1。然后判断 foo 是不是一个合法的 Module，如果不是，则认为结果是
 */
  ['module' => '默认模块', 'controller' => 'bar', 'action' => 'dummy', 'params' => [1 => null]]
/**
 * 而如果在配置文件中定义了 ap.modules = "Index, Foo",
 * 则此处就会认为 foo 是一个合法模块，则结果如下：
 */
  ['module' => 'foo', 'controller' => 'bar', 'action' => 'dummy', 'params' => [1 => null]]
```

注意：当只有一段路由信息时，比如对于上面的例子，请求的 URI 为 /ap/foo, 则默认路由和下面要提到的 Yaf_Route_Supervar 会首先判断 ap.action_prefer, 如果 true，则把 foo 当作 Action，否则当作 Controller

###### Yaf_Route_Simple

Yaf_Route_Simple 是基于请求中的 query_string 来做路由的，在初始化一个 Yaf_Route_Simple 路由协议的时候，我们需要给出 3 个参数，这 3 个参数分别代表在 query string 中的 Module, Controller, Action 的变量名：

例：Yaf_Route_Simple

```php
<?php
/**
 * 指定3个变量名，
 * $router = new Yaf_Route_Simple("m", "c", "a");
 * $router->addRoute("name", $route);
 * 对于此请求：http://{hostname}/index.php?c=index&a=test
 * 解析得到的结果如下
 */
  ['module' => '默认模块', 'controller' => 'index', 'action' => 'test'] 
```

> 注意：只有在 query string 中不包含任何3个参数之一的情况下， Yaf_Route_Simple 才返回失败，将路由控制权交给下一个路由协议

###### Yaf_Route_Supervar

Yaf_Route_Supervar 和 Yaf_Route_Simple 类似，都是在 query string 中获取路由信息，不同的是它获取的是一个类似包含整个路由信息的 request_uri

例： Yaf_Route_Supervar

```php
<?php
/**
 * 指定 supervar 的变量名
 * $router = new Yaf_Route_Supervar("r");
 * $router->addRoute("name", $route);
 * 对于此请求：http://{hostname}/index.php?r=/a/b/c
 * 解析得到的结果如下
 */
  ['module' => 'a', 'controller' => 'b', 'action' => 'c']  
```

> 注意：在 query string 中不包含 supervar 变量的时候，Yaf_Route_Supervar 会返回失败，将路由控制权交给下一个路由协议

###### Yaf_Route_Map

Yaf_Route_Map 协议是一种简单的路由协议，它将 Request_uri 以 “/” 分节，组合在一起，形成一个分层的控制器或动作的路由结果。Yaf_Route_Map 的构造函数接受两个参数，第一个参数表示路由结果是作为动作的路由结果还是控制器的路由结果。默认的是动作路由结果，第二个参数是一个字符串，表示一个分隔符。如果设置了这个分隔符，那么在 Request_uri， 分隔符之前的作为路由信息载体，而之后的作为请求参数。

例：映射路由协议

```php
<?php
/**
 * 对于请求 request_uri 为"/ap/foo/bar", base_uri为"/ap",则最后参加路由的 request_uri 为"/foo/bar"
 * 然后通过对 URL 分节，得到 "foo","bar",组合在一起以后，得到路由结果 foo_bar，然后根据在构造 Yaf_Route_Map 的时候是否指明了 【控制器优先】，如果没有则把结果当作是 Action 的路由结果。否则则认为是 Controller 的路由结果，默认 Controller_first 为 false
 */
```

###### Yaf_Route_Rewrite

Yaf_Route_Rewrite 是一个强大的路由协议，他能满足我们绝大部分的路由需求：

例：Yaf_Route_Rewrite

```php
<?php
// 创建一个路由协议实例
$router = new Yaf_Route_Rewrite(
	'product/:ident',
    [
        'controller' => 'products',
        'action' => 'view',
    ]
);
// 使用路由器转载路由协议
$router->addRoute('product', $route);
```

在这个例子中，我们试图匹配 URL 指定的一个单一的产品，就像 http://{hostname}/product/choclolat-bar. 为了实现这点，我们在路由协议中传递了 2个变量到路由协议 Yaf_Route_Rewrite 的构造函数中，第一个变量（'product/:indent'）就是匹配的路径，第二个变量（array）是路由到的动作控制器；路径使用一个特别的表示来告诉路由协议如何匹配到路径中的每一段，这个标识有两种，可以帮助我们创建我们的路由协议，如下所示：

a):

b)*

" : " 指定了一个段，这个段包含一个变量用于传递到我们动作控制器中的变量，我们要设置好事先的变量名，比如在上面我们的变量名就是 ‘ident’，因此如果我们访问 http://{hostname}/product/chocolate-bar，将会创建一个变量名为 ident 且值是 chocolate-bar 的变量名，然后就可以在我们的 Action 中获取到它的值： $this->getRequest()->getParam('ident');

" * "被用于做一个通配符，意思就是在 Url 中它后面的所有段都将作为一个通配数据被存储。例如我们有路径 ‘path/product/:ident/*’ 并且我们访问的 URL 为 http://{hostname}/product/chocolate-bar/test/value1/another/value2, 那么所有的在 ‘chocolate-bar’ 之后的节都将被当作 key-value ‘键值对’，因此会给我们这样的结果：

```php
ident = chocolate-bar
test = value1
another = value2
```

 这种行为也就是我们平常默认使用的路由协议的行为，注意 key-value 要成对出现。我们有静态的路由协议部分，这些部分简单地被匹配来满足我们的路由协议，在我们的例子中，静态部分就是 product: 就像现在看到的那样，我们的 Yaf_Route_Rewrite 路由协议提供给我们极大的灵活性来控制我们的路由。

###### Yaf_Route_Regex

到目前为止，我们之前的路由协议都很好地完成了基本的路由操作，我们常用的也是此。然而 Yaf_Route_Regex 会有一些限制，这就是我们为什么要引进正则路由 （Yaf_Route_Regex）的原因，正则路由给予我们 preg 正则的全部功能，当同时也使我们的路由协议变得更加复杂了一些。即使是它们有些复杂，我们还是希望能掌握此路由协议的用法，因此此协议更加灵活。

例：Yaf_Route_Regex

```php
<?php
$route = new Yaf_Route_Regex(
	'product/([a-zA-Z-_0-9]+)',
    [
        'controller' => 'products',
        'action' => 'view',
    ]
);
$router->addRoute('product', $route);
```

可以看到，我们现在移动我们的正则到我们的 path（构造函数的第一个参数）中来了。跟之前一样，这个路由协议现在应该是匹配一个数字，字母 ，‘-’ 和 ‘_’ 组成的 ident 变量的字符提供给我们，但是 ident 变量在哪呢？

如果我们使用了 Yaf_Route_Refex 这个路由协议，我们可以通过变量 1(one) 来获取其值，即可以在控制器里用 $this->getRequest()->getParam(1) 来获取，其实这里如果看过正则的都知道这就是反向应用中的 \1 . 然而我们会想为什么要定义这么一个麻烦，我们不能记住或弄清每一个数字代表的是什么变量吗？ 为了改变这点，正则路由协议的构造函数提供了 第三个参数 来完成数字到变量名的映射

例：Yaf_Route_Regex

```php
<?php
$route = new Yaf_Route_Regex(
	'product/([a-zA-Z-_0-9]+)',
    [
        'controller' => 'products',
        'action' => 'view'
    ],
    [
        1 => 'ident',   /// 完成数字到字符变量的映射
    ]
);
$router->addRoute('product', $route);
```

这样，我们就简单地将 变量 1 映射到了 ident 变量名，这样就设置了 ident 变量，同时我们也可以在控制器中获取到它的值。

#### 8_6.自定义路由协议

也会有场景，我们前面提到的 Yaf_Route_Simple, Yaf_Route_Supervar, Yaf_Route_Map, Yaf_Route_Rewrite, Yaf_Route_Regex 都无法满足需求。

我们可以使用自定义的路由协议，方式是实现 

[Yaf_Route_Interface]: http://www.laruence.com/manual/yaf.class.route.html

 接口即可。

## 9.在命令行使用Yaf

#### 9_1.简介

Yaf 支持在命令行下运行，以此来方便调试。

#### 9_2.使用样例

要使得 Yaf在命令行下运行，有两种方式，第一种方式专为用 Yaf 开发 Crontab 等任务脚本而设计，在这种方式下，对 Yaf 唯一的要求是能自动加载所需要的 Module 和类库。所以可以简单地通过 Yaf_Application::execute 来实现。

第二种方式是为了在命令行下模拟请求而设计，运行和web请求一样的流程，从而可以用在命令行下测试我们的 Yaf 应用。对于此种方式，唯一的关键点是请求体，默认的请求体是由 Yaf_Application 实例化并交给 Yaf_Dispatcher 的。而在命令行下，Yaf_Application 并不能正确地实例化一个命令行请求，所以需要变更一下，具体点是请求体需要手动实例化。

例：实例化一个 Yaf_Request_Simple

```php
<?php
$request = new Yaf_Request_Simple();
print_r($request);
```

如上面的例子，Yaf_Request_Simple 的构造函数可以不接受任何参数，在这种情况下 Yaf_Request_Simple 会在命令行参数中寻找一个字符串参数。如果找到了则把请求的 request_uri 置为这个字符串。

> 注意：Yaf_Request_Simple 是可以接受 5个参数的，具体可见 
>
> [Yaf_Request_Simple]: http://www.laruence.com/manual/yaf.class.request.html#yaf.class.request.simple

现在试着运行上面的代码：

```shell
$ php request.php 
     
    
输出:
                    
     Yaf_Request_Simple Object
     (
     [module] => 
     [controller] => 
     [action] => 
     [method] => CLI
     [params:protected] => Array
     (
     )

     [language:protected] => 
     [_base_uri:protected] => 
     [uri:protected] => 
     [dispatched:protected] => 
     [routed:protected] => 
     )
```

现在我们变更下运行方式：

```shell
$ php request.php  "request_uir=/index/hello"
     
    
输出:
                    
     Yaf_Request_Simple Object
     (
     [module] => 
     [controller] => 
     [action] => 
     [method] => CLI
     [params:protected] => Array
     (
     )

     [language:protected] => 
     [_base_uri:protected] => 
     [uri:protected] => index/hello  //注意这里
     [dispatched:protected] => 
     [routed:protected] => 
     )
```

可以看出差别。

当然我们也可以完全指定 Yaf_Request_Simple::__construct 的5个参数

例：带参数实例化一个 Yaf_Request_Simple

```shell
<?php
     $request = new Yaf_Request_Simple("CLI", "Index", "Controller", "Hello", array("para" => 2));
     print_r($requst);
     
    
运行输出:
                    
     $ php request.php 
     Yaf_Request_Simple Object
     (
     [module] => Index
     [controller] => Controller
     [action] => Hello
     [method] => CLI
     [params:protected] => Array
     (
     [para] => 2
     )

     [language:protected] => 
     [_base_uri:protected] => 
     [uri:protected] => 
     [dispatched:protected] => 
     [routed:protected] => 1    //注意这里
     )
```

可以看到一个比较特别的点，routed 属性变为了 true，这代表如果我们手动指定了构造函数的参数，那么这个请求不会再经过 路由，而是直接由路由来完成状态。

#### 9_3.分发请求

现在请求已经改造完成了，那么接下来就比较简单了，我们只需要把我们传统的入口文件

```php
<?php
$app = new Yaf_Application("conf.ini");
$app->bootstrap()->run();
```

改为：

```php
<?php
$app = new Yaf_Application("conf.ini");
$app->getDispatcher()->display(new Yaf_Request_Simple());
```

这样我们就可以在命令行运行 Yaf 了。

参见：

[Yaf_Request_Simple]: http://www.laruence.com/manual/yaf.class.request.html#yaf.class.request.simple



## 10.异常和错误

#### 10_1.概述

Yaf 实现了一套错误和异常捕获机制，主要对常见的错误处理和异常捕获方法做了一个简单抽象，方便应用组织自己的错误统一处理逻辑。

Yaf 自身出错时候，根据配置可以分别采用抛异常或触发错误的方式来通知错误，在 application.dispatcher.throwException（配置文件，或通过 Yaf_Dispatcher::throwException(true)） 打开的情况下，Yaf 会抛异常，否则会触发错误。

对抛异常和触发错误两种方式，也对应两种 handle 处理错误的方式。

#### 10_2.异常模式

在 application.dispatcher.catchException（配置文件，或通过 Yaf_Dispatcher::catchException(true)）开启的情况下，当 Yaf 遇到未捕获异常的时候，就会把运行权限交给当前模块的 Error Controller 的 Error Action 动作，而异常或作为请求的一个参数，传递给 Error Action.

在 Error Action 中可以通过 $request->getRequest()->getParam("exception") 获取当前发生的异常。

> 注意：也可以通过 $request->getException() 来获取当前发生的异常，而如果 Error Action 定义了一个名为 $exception 的参数的话，也可以直接通过这个参数获取当前发生的异常。
>
> 例：Error Controller
>
> ```php
> <?php
> // 当有未捕获的异常时，控制流会走到这里
> class ErrorController extends Yaf_Controller_Abstract
> {
> 	// 也可以通过 $request->getException() 获取发生的异常
>     public function errorAction ($exception)
>     {
>         assert($exception === $exception->getCode());
>         $this->getView()->assign("code", $exception->getCode());
>         $this->getView()->assign("message", $exception->getMessage());
>     }
> }
> ```
>
> 有了这样的最终异常处理逻辑，应用就可以在出错的时候直接抛异常，在统一异常处理逻辑中，根据各种不同的异常逻辑，处理错误，记录日志。
>
> 一个常用的 Error Action 如下：
>
> ```php
> <?php
> // 当有未捕获的异常时，则控制流会流到这里
> class ErrorController extends Yaf_Controller_Abstract
> {
>     // 也可以通过 $request->getException()获取发生的异常
>     public function errorAction ($exception)
>     {
>         switch($exception->getCode()) {
>             case YAF_ERR_LOADFAILED:
>             case YAF_ERR_LOADFAILED_MODULE:
>             case YAF_ERR_LOADFAILED_CONTROLLER:
>             case YAF_ERR_LOADFAILED_ACTION:
>                 header("Not Found");
>                 break;
>             case CUSTOM_ERROR_CODE:
>                 break;
>         }
>     }
> }
> ```
>
> 或者换种更易读的写法：
>
> ```php
> <?php
> // 当有未捕获的异常时，则控制流会流到这里
> class ErrorController extends Yaf_Controller_Abstract
> {
>     // 通过 $request->getException() 获取到发生的异常
>     public function errorAction ()
>     {
>         $exception = $this->getRequest()->getException();
>         try {
>             throw $exception;
>         } catch (Yaf_Exception_LoadFailed $e) {
>             // 加载失败
>         } catch (Yaf_Exception $e) {
>             // 其他类型错误
>         }
>     }
> }
> ```
>
> 

#### 10_3.错误模式

// 略

## 11.内建的类

#### 11_1.Yaf_Application

###### 简介

Yaf_Application 代表一个产品/项目，是Yaf运行的主导者，真正执行的主体。他负责接收请求，协调路由，分发，执行，输出。

在PHP5.3以后，打开了 Yaf.use_namespace 的情况下，也可以使用 Yaf\Application.

```php
final Yaf_Application
{
    protected Yaf_Config _config;
    protected Yaf_Dispatcher _dispatcher;
    protected static Yaf_Application _app;
    protected boolean _run = FALSE;
    protected string _environ;
    protected string _modules;
    public void __construct (mixed $config, string $section = ap.environ);
    public Yaf_Application bootstrap (void);
    public Yaf_Response_Abstract run (void);
    public Yaf_Dispatcher getDispatcher (void);
    public Yaf_Config_Abstract getConfig (void);
    public string envirn (void);
    public string geModules (void);
    public static Yaf_Application app (void);
    public mixed execute (callback $function, mixed $parameter = NULL, mixed $... = NULL);
}
```

###### 属性说明

| 属性        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| _app        | Yaf_Application 通过特殊的方式实现了单例模式，此属性保存当前实例 |
| _config     | 全局配置实例                                                 |
| _dispatcher | Yaf_Dispatcher 实例                                          |
| _modules    | 存在的模块名，从配置文件中 ap.modules 读取                   |
| _environ    | 当前的环境名，也就是Yaf_Application在读取配置的时候，获取的配置节名字 |
| _run        | 布尔值，指明当前的Yaf_Application是否已经运行                |

###### Yaf_Application::_construct()

```php
public void Yaf_Application::__construct(mixed $config, string $section = ap.environ);
```

初始化一个Yaf_Application, 如果$config 是一个 INI 文件，那么 $section 指明要读取的配置节。

参数：

config：关联数组的配置，或者指向一个 INI 格式的配置文件的路径的字符串，或者指向一个 Yaf_Config_Abstract 的实例

返回值：void

例：Yaf_Application::__construct

```php
<?php

$config = array(

	"ap" => array(

		"directory" => "/usr/local/www/ap",

	),

);

$app = new Yaf_Application($config);
```

输出：

```bash
object(Yaf_Application)#1 (6) {
    ...
}
```

###### Yaf_Application::bootstrap

```php
Yaf_Application Yaf_Application::bootstrap(void);
```

指示 Yaf_Application 去寻找Bootstrap(默认在ap.directory/Bootstrap.php),并执行所有在 Bootstrap 类中定义的，以 _init 开头的方法。一般用在处理请求之前，做一些个性化定制。

Bootstrap 并不会调用 run，所以还需要在 Bootstrap 之后调用 Application::run 来运行 Yaf_Application 实例

参数：void

返回值：Yaf_Application

例：Yaf_Application::bootstrap

```php
<?php
$config = array(
	"ap" => array(
    	"directory" => "/usr/local/www/ap",
    ),
);
$app = new Yaf_Application($config);
$app->bootstrap()->run();
```

参见：

[Yaf_Application::run]: http://www.laruence.com/manual/yaf.class.application.run.html
[Yaf_Bootstrap_Abstra]: http://www.laruence.com/manual/yaf.class.bootstrap.html

###### Yaf_Application::app()

```php
static Yaf_Application Yaf_Application::app(void)；
```

获取当前的 Yaf_Application 实例

参数：void

返回值：Yaf_Application

例：Yaf_Application::app

```php
<?php
$config = array(
	"ap" => array(
    	"directory" => "/usr/local/www/ap",
    ),
);
$app = new Yaf_Application($config);
assert($app === Yaf_Application::app();
```

###### Yaf_Application::environ()

```php
string Yaf_Application::environ(void);
```

获取当前 Yaf_Application 环境名

参数：void

返回值：当前的环境名，即 ini_get("yaf.environ")

例：Yaf_Application::environ

```php
<?php
$config = array(
	"application" => array(
    	"directory" => "/usr/local/www/yaf",
    ),
);
$app = new Yaf_Application($config);
print($app->environ());   // 例如配的环境是product，这里即打印 “product”
```

###### Yaf_Application::run()

```php
boolean Yaf_Application::run(void);
```

运行一个 Yaf_Application,开始接受并处理请求，这个方法只能调用一次，多次调用并不会有特殊效果。

参数：void

返回：boolean

例：Yaf_Application::run

```php
<?php
$config = array(
	"application" => array(
    	"directory" => "/usr/local/www/yaf",
    ),
);
$app = new Yaf_Application($config);
$app->run();
```

###### Yaf_Application::execute()

```php
mixed Yaf_Application::execute(callback $function, mixed $parameter = NULL, $parameter $... = NULL);
```

在 Yaf_Application 的环境下，运行一个用户自定义函数过程，主要用在使用 Yaf 做简单的命令行脚本时，应用 Yaf 的外围环境，比如：自动加载，配置，视图引擎等。

> 注意：如果需要使用 Yaf 的路由分发，也就是说，如果是需要在 CLI 下全功能运行 Yaf, 请参考 
>
> [在命令行下使用]: http://www.laruence.com/manual/yaf.incli.html

参数：

$function: 要运行的函数或方法，方法可以通过 array($obj, "method_name") 来定义。

$parameter: 零个或多个要传递给函数的参数。

返回值：

被调用函数或者方法的返回值

例：Yaf_Application::execute

```php
<?php
$config = array(
	"ap" => array(
    	"directory" => "/usr/local/www/ap",
    ),
);
$app = new Yaf_Application($config);
$app->execute("main");

function main(){
    ...
}
    
```

参见：9.在命令行使用 Yaf

###### Yaf_Application::getDispatcher

```php
Yaf_Config_Abstract Yaf_Application::getDispatcher(void);
```

获取当前的分发器

参数：void

返回值：Yaf_Dispatcher 的实例

例：Yaf_Application::getDispatcher 

```php
<?php
define("APPLICATION_PATH", dirname(__FILE__));
$app = new Yaf_Application("conf/application_simple.ini")；
// bootstrap
$app->getDispatcher()->setAppDirectory(APPLICATION_PATH . "/action/")->getApplication()->bootstrap()->run();
// 当然也可以这么写
$dispatcher = Yaf_Dispatcher::getInstance()->setAppDirectory(APPLICATION_PATH . "/action/")->getApplication()->bootstrap()->run();
```

###### Yaf_Application::getConfig

```php
Yaf_Config_Abstract Yaf_Application::getConfig(void);
```

获取 Yaf_Application 读取的配置项。

参数：void

返回值：Yaf_Config_Abstract

例：Yaf_Application::getConfig

```php
<?php
$config = array(
	"ap" => array(
    	"directory" => "/usr/local/www/ap",
    ),
);
$app = new Yaf_Application($config);
print_r($app->getConfig('application'));

// 输出
Yaf_Config Object(
	[_config:private] => array(
    	[ap] => array(
        	[directory] => /usr/local/www/ap
        )
    )
)
```

###### Yaf_Application::getModules

```php
string Yaf_Application::getModules(void);
```

获取在配置文件中声明的模块。

参数：void

返回值：string

例：Yaf_Application::getModules

```php
<?php
$config = array(
	"ap" => array(
    	"directory" => "/usr/local/www/ap",
        "modules" => "Index",
    ),
);
$app = new Yaf_Application($config);
print_r($app->getModules());

// 输出
Array(
	[0] => Index
)
```

#### 11_2.Yaf_Bootstrap_Abstract

###### 简介

Yaf_Bootstrap_Abstract 提供了一个可以定制 Yaf_Application 的最早的时机，它相当于一段引导，入口程序。他本身没有定义任何方法，但任何继承自 Yaf_Bootstrap 的类中以 _init 开头的方法，都会在 Yaf_Application::bootstrap 时被调用，调用的顺序和这些方法在类中的定义顺序相同，Yaf 保证这种调用顺序。

> 注意：这些方法，都可以接受一个  Yaf_Dispatcher 参数

例：Yaf_Bootstrap_Abstract 的例子

```php
<?php
/**
 * 所有在 Bootstrap 类中，以 _init 开头的方法，都会被 Yaf 调用。这些方法都接受一个参数：Yaf_Dispatcher $dispatcher 调用的次序，和声明的次序相同。
 */
class Bootstrap extends Yaf_Bootstrap_Abstract
{
    /**
     * 注册一个插件
     * 插件的目录是在 application_directory/plugins
     */
    public function _initPlugin(Yaf_Dispatcher $dispatcher) {
        $user = new UserPlugin();
        $dispatcher->registerPlugin($user);
    }
    /**
     * 添加配置中的路由
     */
    public function _initRoute(Yaf_Dispatcher $dispatcher) {
        $router = Yaf_Dispatcher::getInstance()->getRouter();
        $router->addConfig(Yaf_Registry::get('config')->routes);
        // 添加一个路由
        $route = new Yaf_Route_Rewrite(
        	"/product/list/:id/",
            array(
            	"controller" => "product",
                "action" => "info",
            )
        );
        $router->addRoute('dummy', $route);
    }
    
    // 自定义视图引擎
    public function _initSmarty(Yaf_Dispatcher $dispatcher){
        $smarty = new Smarty_Adapter(null, Yaf_Registry::get("config")->get("smarty"));
        Yaf_Dispatcher::getInstance()->setView($smarty);
    }
}

// 在入口文件：
<?php
// 默认的，Yaf_Application 将会读取配置文件中在 php.ini 中设置的 ap.environ 的配置节。
$application = new Yaf_Application("conf/sample.ini");
// 如果没有关闭自动 response(通过 Yaf_Dispatcher::getInstance()->autoResponse(FALSE)), 则 $response 会被自动输出，此处也不需要再次输出 Response
$response = $application->bootstrap()->run();
```



#### 11_3.Yaf_Loader

###### 简介

Yaf_Loader 类为 Yaf 提供了自动加载功能，它根据类名中包含的路径信息实现类的定位和自动加载。

Yaf_Loader 也提供了对传统的 require_once 的替代方案，相比传统的 require_once, 因为舍弃了对 required 的支持，所以性能上略微有一丁点优势。

在 PHP5.3 之后，打开 Yaf.use_namespace 的情况下，也可以使用 Yaf\Loader.

```php
final Yaf_Loader {
    protected static Yaf_Loader _instance;
    protected string _library_directory;
    protected string _global_library_directory;
    protected string _local_ns;
    public static Yaf_Loader getInstance (string $local_library_directory = NULL, string $global_library_directory = NULL);
    public Yaf_Loader registerLocalNamespace (mixed $namespace);
    public boolean getLocalNamespace ();
    public boolean clearLocalNamespace ();
    public boolean isLocalName (string $class_name);
    public static boolean import (string $file_name);
    public boolean autoload (string $class_name);
}
```

###### 属性说明

_instance: Yaf_Loader 实现了单例模式，一般的它由 Yaf_Application 负责初始化，此属性保存当前实例。

_library_directory: 本地（自身）类加载路径，一般的，属性的值来自配置文件中的 ap.library

_global_library_directory: 全局类加载路径，一般的，属性的值来自 php.ini 中的 ap.library

_local_ns: 本地类的类名前缀，此属性通过 Yaf_Loader::registerLocalNamespace 来添加新的值。

###### Yaf_Loader::getInstance

```php
public static Yaf_Loader::getInstance (string $local_library_directroy = NULL, string $global_library_directory = NULL);
```

获取当前的 Yaf_Loader 实例

参数：

$local_library_directory: 本地（自身）类库目录，如果留空，则返回已经实例化过的 Yaf_Loader 实例

注意：Yaf_Loader 是单例模式，所以即使第二次以不同的参数实例化一个 Yaf_Loader ,得到的仍然是一个已经实例化的第一个实例。

$global_library_directory: 全局类库目录，如果留空则会认为和 $local_library_directory 相同。

返回值：Yaf_Loader

例：Yaf_Loader::getInstance()

```php
<?php
$loader = Yaf_Loader::getInstance();    
```

###### Yaf_Loader::import

```php
public static boolean Yaf_Loader::import (string $filename);
```

导入一个 PHP 文件，因为 Yaf_Loader::import 只是专注于一次包含，所以要比传统的 require_once 性能好一点。

参数：

$file_name: 要载入的文件路径，可以为绝对路径和相对路径。如果是相对路径，则会以应用的本地类目录（ap.library）为基目录。

返回值：

成功返回 TRUE，失败返回 FALSE.

例: Yaf_Loader::import()

```php
<?php
// 绝对路径
Yaf_Loader::import("/usr/local/foo.php");
// 相对路径，会在 APPLICATION_PATH . "/library"下加载
Yaf_Loader::import("plugins/User.php");
```

###### Yaf_Loader::autoload

```php
public static boolean Yaf_Loader::autoload (string $class_name);
```

载入一个类，这个方法被 Yaf 用作自动加载类的方法， 当然也可以手动调用。

参数：$class_name: 要载入的类名，类名必须包含路径信息，也就是下划线分隔的路径信息和类名。载入的过程中，首先会判断这个类名是否是本地类，如果是本地类，则使用本地类类库目录。否则使用全局类目录。然后判断 yaf.lowcase_path 是否开启，如果开启，则会把类名中的路径部分全部小写，然后加载执行。

```/** yaf.lowcase_path = 0 */  Foo_Bar_Dummy表示这个类存在于类库目录下的 Foo/Bar/Dummy.php```

```/** yaf.lowcase_path = 1 */  Foo_Bar_Dummy表示这个类存在于类库目录下的 foo/bar/Dummy.php```

> 注意：在 php.ini 中的 yaf.lowcase_path 开启的情况下，路径信息中的目录部分都会被转换为小写。

返回值：成功返回true

> 注意：在 php.ini 中的 yaf.use_sql_autoload 关闭的情况下，即使类没有找到，Yaf_Loader::autoload 也会返回 TRUE， 剥夺其后面的自动加载函数的执行权。

例：Yaf_Loader::autoload

```php
<?php
Yaf_Loader::autoload("Baidu_ST_Dummy_Bar");
```

###### Yaf_Loader::registerLocalNamespace

```php
public Yaf_Loader Yaf_Loader::registerLocalNamespace (mixed $local_name_prefix);
```

注册本地类前缀，对于以这些前缀开头的本地类，都从本地类库中加载。

参数：$local_name_prefix: 字符串或是数组格式的类名前缀。不包含前缀后面的下划线。

返回值: Yaf_Loader

例： Yaf_Loader::registerLocalNamespace

```php
<?php
Yaf_Loader::getInstance()->registerLocalNamespace("Foo");
Yaf_Loader::getInstance()->registerLocalNamespace(["Foo", "Bar"]);
```

###### Yaf_Loader::isLocalName

```php
public boolean Yaf_Loader::isLocalName(string $class_name);
```

判断一个类是否是本地类。

参数：$class_name: 字符串的类名。本方法会根据下划线分隔截取出类名的第一部分，然后在 Yaf_Loader 的 _local_ns 中判断是否存在，从而确定结果。

返回值：boolean

例：Yaf_Loader::isLocalName

```php
<?php
Yaf_Loader::getInstance()->registerLocalNamespace("Foo");
Yaf_Loader::getInstance()->isLocalName("Foo_2"); // TRUE
Yaf_Loader::getInstance()->isLocalName("Foo2");  // FALSE
```

###### Yaf_Loader::getLocalNamespace

```php
public array Yaf_Loader::getLocalNamespace(void);
```

获取当前已经注册的本地类前缀

参数：void

返回值：成功返回字符串

例：Yaf_Loader::getLocalNamespace 的例子

```php
<?php

Yaf_Loader::getInstance()->registerLocalNamespace(["Foo", "Bar"]);

print(Yaf_Loader::getInstance()->getLocalNamespace());

输出：

:Foo:Bar:
```

###### Yaf_Loader::clearLocalNamespace

```php
public boolean Yaf_Loader::clearLocalNamespace();
```

清除已注册的本地类前缀

参数：void

返回值：成功返回 TRUE，失败返回 FALSE

例：Yaf_Loader::clearLocalNamespace

```php
<?php
Yaf_Loader::getInstance()->clearLocalNamespace();    
```



#### 11_4.Yaf_Dispatcher

###### 简介

Yaf_Dispatcher 实现了MVC分发中的 C 分发，它由 Yaf_Application 负责初始化，然后由 Yaf_Application::run() 启动，然后 Yaf_Dispatcher 协调路由来的请求，并分发和执行发现的动作，并收集动作产生的响应，输出响应给请求者，并在整个过程完成以后返回响应。

在PHP5.3 以后，打开 yaf.use_namespace 的情况下，也可以使用 Yaf\Dispatcher.

```php
final Yaf_Dispatcher {
    protected static Yaf_Dispatcher _instance;
    protected Yaf_Router_Interface _router;
    protected Yaf_View_Abstract _view;
    protected Yaf_Request_Abstract _request;
    protected array _plugins;
    protected boolean _render;
    protected boolean _return_response = FALSE;
    protected boolean _instantly_flush = FALSE;
    protected string _default_module;
    protected string _default_controller;
    protected string _default_action;
    public static Yaf_Dispatcher getInstance();
    public Yaf_Dispatcher disableView();
    public Yaf_Dispatcher enableView();
    public boolean autoRender (bool $flag);
    public Yaf_Dispatcher returnResponse (boolean $flag);
    public Yaf_Dispatcher flushInstantly (boolean $flag);
    public Yaf_Dispatcher setErrorHandler (mixed $callback, int $error_type = E_ALL | E_STRICT);
    public Yaf_Application getApplication();
    public Yaf_Request_Abstract getRequest();
    public Yaf_Router_Interface getRouter();
    public Yaf_Dispatcher registerPlugin (Yaf_Plugin_Abstract $plugin);
    public Boolean setAppDirectory (string $directory);
    public Yaf_Dispatcher setRequest (Yaf_Request_Abstract $request);
    public Yaf_View_Interface initView();
    public Yaf_Dispatcher setView (Yaf_View_Interface $view);
    public Yaf_Dispatcher setDefaultModule (string $default_module_name);
    public Yaf_Dispatcher setDefaultController (string $default_controller_name);
    public Yaf_Dispatcher setDefaultAction (string $default_action_name);
    public Yaf_Dispatcher throwException (boolean $switch = FALSE);
    public Yaf_Dispatcher catchException (boolean $switch = FALSE);
    public Yaf_Response_Abstract dispatch (Yaf_Request_Abstract $request);
}
```

###### 属性说明

_instance: Yaf_Dispatcher 实现了单例模式，此属性保存当前实例。

_request: 当前请求。

_router: 路由器。

_view: 当前的视图引擎，可通过 Yaf_Dispatcher::setView 来替换视图引擎为自定义的视图引擎（比如 Smarty / Firekylin 等常见引擎）

_plugins: 已经注册的插件，插件一经注册就不能更改和删除了。

_render: 标示着是否在动作执行完成后，调用视图引擎的 render 方法来产生响应。可通过 Yaf_Dispatcher::returnResponse 来切换开关状态。

_return_response: 标示着是否在产生响应以后，不自动输出给客户端，而是返回给调用者。可通过 Yaf_Dispatcher::returnResponse 来切换开关状态

_instantly_flush: 标示着是否在有输出的时候，直接响应给客户端，而不写入 Yaf_Response_Abstract 对象。

> 注意：如果此属性为 TRUE，那么将 忽略 Yaf_Dispatcher::$_return_response

_default_module: 默认的模块名，在路由的时候，如果没有指明模块，则会使用这个值，也可以通过配置文件中的 ap.dispatcher.defaultModule 来指定。

_default_controller: 默认的控制器名，在路由的时候，如果没有指明控制器，则会使用这个值，也可以通过配置文件里的 ap.dispatcher.defaultController 来指定。

_default_action: 默认的动作名。在路由的时候如果没有指明动作，则会使用这个值，看可以通过配置文件中的 ap.dispatcher.defaultAction 来指定。

###### Yaf_Dispatcher::getInstance()

```php
public static Yaf_Dispatcher Yaf_Dispatcher::getInstance();
```

获取当前的 Yaf_Dispatcher 实例

参数：void

返回值：Yaf_Dispatcher

例：Yaf_Dispatcher::getInstance

```php
<?php
$dispatcher = Yaf_Dispatcher::getInstance();    
```

###### Yaf_Dispatcher::disableView

```php
public boolean Yaf_Dispatcher::disableView();
```

关闭自动 Render，默认是开启的，在动作执行完成以后，Yaf 会自动 render 以动作名命令的视图模板文件

参数：void

返回值：成功返回 Yaf_Dispatcher, 失败返回 FALSE

例：Yaf_Dispatcher::disableView

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    // controller 的init方法会首先被调用
    public function init() {
        // 如果是 ajax 请求，则关闭 HTML 输出
        if ($this->getRequest()->isXmlHttpRequest()) {
            Yaf_Dispatcher::getInstance()->disableView();
        }
    }
}
```

###### Yaf_Dispatcher::enableView

```php
public boolean Yaf_Dispatcher::enableView();
```

开启自动 Render. 默认是开启的，在动作执行完成以后，Yaf 会自动 render 以动作名命名的视图模板文件

参数：void

返回值：成功返回 Yaf_Dispatcher, 失败返回 FALSE.

例：Yaf_Dispatcher::enableView

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    // Controller 的init方法会被自动首先调用
    public function init() {
        // 如果不是ajax请求，则开启 HTML 输出
        if (!$this->getRequest()->isXmlHttpRequest()) {
            Yaf_Dispatcher::getInstance()->enableView();
        }
    }
}
```

###### Yaf_Dispatcher::autoRender

```php
public boolean Yaf_Dispatcher::autoRender(boolean $switch);
```

开启/关闭自动渲染功能。在开启的情况下（默认是开启的），等动作执行完以后，Yaf 会自动调用 View 引擎去渲染该 Action 对应的视图模板。

参数：$switch: 开启状态

返回值：成功返回 Yaf_Dispatcher, 失败返回 FALSE

例：Yaf_Dispatcher::autoRender

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function init() {
        if($this->getRequest()->isXmlHttpRequest()) {
            // 如果是ajax请求，则关闭自动渲染（需要由我们手动返回 JSON 响应）。
            Yaf_Dispatcher::getInstance()->autoRender(FALSE);
        }
    }
}
```

###### Yaf_Dispatcher::returnResponse

```php
public void Yaf_Dispatcher::returnResponse(boolean $switch);
```

是否返回Response 对象，如果启用则 Response 对象在分发完成以后不会自动输出给请求端，而是交给程序员自己控制输出。

参数：$switch: 开启状态

返回值：成功返回 Yaf_Dispatcher，失败返回 FALSE

例：Yaf_Dispatcher::returnResponse

```php
<?php
$application = new Yaf_Application("config.ini");
// 关闭自动响应，需要后续主动进行输出
$response = $application->getDispatcher()
    ->returnResponse(TRUE)
    ->getApplication()
    ->run();
// 输出响应
$response->response();
```

###### Yaf_Dispatcher::flushInstantly

```php
public void Yaf_Dispatcher::flushInstantly(boolean $switch);
```

切换自动响应，在 Yaf_Dispatcher::enableView() 的情况下，会使得 Yaf_Dispatcher 调用 Yaf_Controller_Abstract::display 方法，直接输出响应给请求端。

参数：$switch:  开启状态

返回值：成功返回 Yaf_Dispatcher，失败返回 FALSE

例： Yaf_Dispatcher::flushInstantly

```php
<?php
$application = new Yaf_Application("config.ini");
// 立即输出响应
Yaf_Dispatcher::getInstance()->flushInstantly(TRUE);
// 此时会调用 Yaf_Controller_Abstract::display 方法
$application->run();
```

###### Yaf_Dispatcher::setErrorHandler

```php
public boolean Yaf_Dispatcher::setErrorHandler(mixed $callback, int $error_code = E_ALL | E_STRICT);
```

设置错误处理函数。一般在 application.throwException 关闭的情况下，Yaf会在出错的时候触发错误，这个时候如果设置了错误处理函数，则把控制权交给错误处理函数处理。

参数：

$callback: 错误处理函数，这个函数需要最少接受两个参数：错误码($error_code)，错误信息($error_message), 可选的还有三个参数：错误文件($err_file), 错误行($err_line)和错误上下文($err_context)

$error_code: 要捕获的错误类型

返回值：成功返回 Yaf_Dispatcher, 失败返回 FALSE

例：Yaf_Dispatcher::setErrorHandler

```php
<?php
// 一般可放在 Bootstrap 中定义错误处理函数
function myErrorHandler($errno, $errstr, $errfile, $errline) {
    switch ($errno) {
        case YAF_ERR_NOTFOUND_CONTROLLER：
        case YAF_ERR_NOTFOUND_MODULE:
        case YAF_ERR_NOTFOUND_ACTION:
            header("Not Found");
            break;
        default:
            echo "Unknown error type:[$errno] $errstr<br/>\n";
            break;
    }
    return true;
}
Yaf_Dispatcher::getInstance()->setErrorHandler("myErrorHandler");
```

###### Yaf_Dispatcher::getApplication

```php
public Yaf_Application Yaf_Dispatcher::getApplication();
```

获取当前的 Yaf_Application 的实例

参数：void

返回值：Yaf_Application

例：Yaf_Dispatcher::getApplication

```php
<?php
$application = Yaf_Dispatcher::getInstance()->getApplication();
// 不过还是推荐这种用法比较简便
$application = Application::app();
```

###### Yaf_Dispatcher::getRouter

```php
public Yaf_Router Yaf_Dispatcher::getRouter();
```

获取路由器

参数：

返回值：Yaf_Router

例：Yaf_Dispatcher::getRouter

```php
<?php
$router = Yaf_Dispatcher::getInstance()->getRouter();    
```

###### Yaf_Dispatcher::getRequest

```php
public Yaf_Request_Abstract Yaf_Dispatch::getRequest();
```

获取当前的请求实例

参数：void

返回值：Yaf_Request_Abstract

例：Yaf_Dispatcher::getRequest

```php
<?php
$request = Yaf_Dispatcher::getInstance()->getRequest();    
```

###### Yaf_Dispatcher::registerPlugin

```php
public boolean Yaf_Dispatcher::registerPlugin(Yaf_Plugin_Abstract $plugin);
```

注册一个插件

参数：$plugin: 一个 Yaf_Plugin_Abstract 派生类的实例。

返回值：成功返回 Yaf_Dispatcher, 失败返回FALSE

例：Yaf_Dispatcher::registerPlugin

```php
<?php
/**
 * 所有在Bootstrap类中，以 _init 开头的方法都会被 Yaf 调用。
 * 这些方法都接受一个参数：Yaf_Dispatcher $dispatcher
 * 调用的次序和声明的次序相同
 */
class Bootstrap extends Yaf_Bootstrap_Abstract
{
    // 注册一个插件，插件的目录 application_directory/plugins
    public function _initPllugin(Yaf_Dispatcher $dispatcher) {
    	$user = new UserPlugin();
        $dispatcher->registerPlugin($user);
    }
}
// 插件类定义
class UserPlugin extends Yaf_Plugin_Abstract
{
    public function routerStartup(Yaf_Request_Abstract $request, Yaf_Response_Abstract $response) {
        echo "Plugin routerStartup called <br/>\n";
    }
    public function routerShutdown(Yaf_Request_Abstract $request, Yaf_Response_Abstract $response) {
        echo "Plugin routerShutdown called <br/>\n";
    }
    public function dispatchLoopStartup(Yaf_Request_Abstract $request, Yaf_Response_Abstract $response) {
        echo "Plugin DispatchLoopStartup called <br/>\n";
    }
    public function preDispatch(Yaf_Request_Abstract $request, Yaf_Response_Abstract $response) {
        echo "Plugin PreDispatch called <br/>\n";
    }
    public function postDispatch(Yaf_Request_Abstract $request, Yaf_Response_Abstract $response) {
        echo "Plugin postDispatch called <br/>\n";
    }
    public function dispatchLoopShutdown(Yaf_Request_Abstract $request, Yaf_Response_Abstract $response) {
        echo "Plugin DispatchLoopShutdown called <br/>\n";
    }
}
```

###### Yaf_Dispatcher::setAppDirectory

```php
boolean Yaf_Dispatcher::setAppDirectory(string $directory);
```

改变APPLICATION_PATH，在此之后将从新的APPLICATION_PATH 下加载控制器 / 视图，但注意这不会改变自动加载的路径。

参数：$directory: 绝对路径的APPLICATION_PATH

返回值：成功则返回Yaf_Dispatcher, 失败则返回FALSE

例：Yaf_Dispatcher::setAppDirectory

```php
<?php
$config = array(
	"ap" => array(
    	"directory" => "/usr/local/www/ap",
    ),
);
$app = new Yaf_Application($config);
$app->getDispatcher()->setAppDirectory("/usr/local/new/application")->getApplication()->run();
```

###### Yaf_Dispatch::setRequest

```php
public boolean Yaf_Dispatcher::setRequest(Yaf_Request_Abstract $request);
```

设置请求对象

参数：$request: 一个Yaf_Request_Abstract 实例

$error_code: 要捕获的错误类型

返回值：成功返回 Yaf_Dispatcher, 失败则返回 FALSE

例：Yaf_Dispatcher::setRequeset

```php
<?php
$request = new Yaf_Request_Simple("Index", "Index", "index");
Yaf_Dispatcher::getInstance()->setRequest($request);
```

###### Yaf_Dispatcher::initView

```php
public Yaf_View_Interface Yaf_Dispatcher::initView(string $tpl_dir);
```

初始化视图引擎，因为 Yaf 采用延迟实例化视图引擎的策略，所以只有在使用前调用此方法，视图引擎才会被实例化。

> 注意：如果我们需要自定义视图引擎，那么需要在调用 Yaf_Dispatcher::setView 自定义视图引擎之后才可以调用此方法，不然将得不到正确的视图引擎，因为默认的此方法会实例化一个 Yaf_View_Simple 视图引擎

参数：$tpl_dir:  视图的模板目录的绝对路径

返回值：Yaf_View_Interface 实例

例：Yaf_Dispatcher::initView

```php
<?php
class Bootstrap extends Yaf_Bootstrap_Abstract {
    public function _initViewParameter(Yaf_Dispatcher $dispatcher) {
        $dispatcher->initView(APPLICATION_PATH . "/views/")->assign("webroot", WEBROOT);
    }
}
```

###### Yaf_Dispatcher::setView

```php
public boolean Yaf_Dispatcher::setView(Yaf_View_Interface $request);
```

设置视图模板引擎

参数：$view: 一个实现了 Yaf_View_Interface 的视图模板引擎实例

返回值：成功则返回 Yaf_Dispatcher, 失败则返回 FALSE

例：Yaf_Dispatcher::setView

```php
<?php
/**
 * 所有在 Bootstrap 类中，以 _init 开头的方法，都会被 Yaf 调用
 * 这些方法，都接受一个参数：Yaf_Dispatcher $dispatcher
 * 调用的次序和声明的顺序相同
 */
class Bootstrap extends Yaf_Bootstrap_Abstract
{
    // 自定义视图引擎
    public function _initSmarty(Yaf_Dispatcher $dispatcher) {
        $smarty = new Smarty_Adapter(null, Yaf_Registry::get("config")->get("smarty"));
        Yaf_Dispatcher::getInstance()->setView($smarty);
    }
}
/**
 * 视图引擎定义
 * Smarty/Adapter.php
 */
class Smarty_Adapter implements Yaf_View_Interface
{
    /**
     * Smarty object
     * @var Smarty
     */
    public $_smarty;
    /**
     * Constructor
     * @param string $tmplPath
     * @param array @extraParams
     * @return void
     */
    public function __construct($tmplPath = null, $extraParams = array()) {
        require "Smarty.class.php";
        $this->_smarty = new Smarty;
        if (null !== $tmplPath) {
            $this->setScriptPath($tmplPath);
        }
        foreach ($extraParams as $key => $value) {
            $this->_smarty->$key = $value;
        }
    }
    
    /**
     * Assign variables to the template
     * Allow setting a specific key to the specified value, OR passing an array of key=>value pairs to set an masse.
     * @see __set()
     * @param string|array $spec  The assignment strategy to use(key or array of key => value pairs)
     * @param mixed $value (Optional) If assigning a named variable, use this as the value.
     * @return void
     */
    public function assign($spec, $value = null) {
        if (is_array($spec)) {
            $this->_smarty->assign($spec);
            return;
        }
        $this->_smarty->assign($spec, $value);
    }
    
    /**
     * Processes a template and returns the output.
     * @param string $name  The template to process.
     * @return string   The output.
     */
    public function render($name) {
        return $this->_smarty->fetch($name);
    }
}
```

###### Yaf_Dispatcher::setDefaultController

```php
public boolean Yaf_Dispatcher::setDefaultController(string $default_controller_name);
```

设置路由的默认控制器，如果在路由结果中不包含控制器信息，则会使用此默认控制器作为路由控制器结果

参数：$default_controller_name: 默认控制器名，请注意需要首字母大写

返回值：成功返回 Yaf_Dispatcher, 失败返回FALSE

例：Yaf_Dispatcher::setDefaultController

```php
<?php
class Bootstrap extends Yaf_Bootstrap_Abstract
{
    public function _initDefaultName(Yaf_Dispatcher $dispatcher) {
        // 这个仅是为了举例，本身 Yaf 默认的是Index
        $dispatcher->setDefaultModule("Index")->setDefaultController("Index")->setDefaultAction("index");
    }
}
```

###### Yaf_Dispatcher::setDefaultModule

```php
public boolean Yaf_Dispatcher::setDefaultModule(string $default_module_name);
```

设置路由的默认模块，如果在路由结果中不包含模块信息，则会使用此默认模块作为路由结果。

参数：$default_module_name:  默认模块名，注意首字母需要大写

返回值：成功则返回 Yaf_Dispatcher, 失败返回 FALSE

例：Yaf_Dispatcher::setDefaultModule

```php
<?php
class Bootstrap extends Yaf_Bootstrap_Abstract
{
    public function _initDefaultName(Yaf_Dispatcher $dispatcher) {
        // 这里仅是举例，本身 Yaf 默认的就是 Index
        $dispatcher->setDefaultModule("Index")->setDefaultController("Index")->setDefaultAction("index");
    }
}
```

###### Yaf_Dispatcher::setDefaultAction

```php
public boolean Yaf_Dispatcher::setDefaultAction(string $default_module_name)；
```

设置路由的默认动作，如果在路由结果中不包含动作信息，则会使用此默认动作作为路由动作结果。

参数：$default_module_name:  默认动作名，请注意需要全部小写

返回值：成功则返回 Yaf_Dispatcher, 失败返回 FALSE

例：Yaf_Dispatcher::setDefaultAction

```php
<?php
class Bootstrap extends Yaf_Bootstrap_Abstract
{
    public function _initDefaultName(Yaf_Dispatcher $dispatcher)
    {
        $dispatcher->setDefaultController("Index")->setDefaultAction("index");
    }
}
```

###### Yaf_Dispatcher::throwException

```php
public boolean Yaf_Dispatcher::throwException(boolean $switch);
```

切换在 Yaf 出错的时候抛出异常，还是直接触发异常。

当然也可以在配置文件中使用 ap.dispatcher.throwException = $switch 达到同样的效果，默认的是开启状态。

参数：$switch:  如果为 TRUE，则 Yaf 在出错的时候采用抛出异常的方式。如果为 FALSE, 在 Yaf 在出错的时候采用 直接触发异常的方式。

返回值：成功返回 Yaf_Dispatcher, 失败返回 FALSE

例：Yaf_Dispatcher::throwException

```php
<?php
$app = new Yaf_Application("conf.ini");
// 关闭抛出异常
Yaf_Dispatcher::getInstance()->throwException(FALSE);
```

###### Yaf_Dispatcher::catchException

```php
public boolean Yaf_Dispatcher::catchException(boolean $switch);
```

在 ap.dispatcher.throwException 开启的状态下，是否启用默认捕获异常机制

当然也可以在配置文件中使用 ap.dispatcher.catchException = $switch 达到同样效果，默认是开启状态。

参数：$switch:  如果为 TRUE, 则在由未捕获异常时，Yaf 会交给 Error Controller的Error Action 处理。

返回值：成功返回 Yaf_Dispatcher, 失败返回 FALSE.

例：Yaf_Dispatcher::catchException 

```php
<?php
$app = new Yaf_Application("conf.ini");
// 开启捕获异常
Yaf_Dispatcher::getInstance()->catchException(TRUE);
```

###### Yaf_Dispatcher::dispatch

```php
public boolean Yaf_Dispatch::dispatch(Yaf_Request_Abstract $request);
```

开始处理流程，一般的不需要用户调用此方法，Yaf_Application::run 会自动调用此方法。

参数：$request: 一个 Yaf_Request_Abstract 实例

返回值：成功返回一个 Yaf_Response_Abstract 实例，失败则返回 FALSE, 错误则会抛出异常。

#### 11_5.Yaf_Plugin_Abstract

###### 简介

Yaf_Plugin_Abstract 是 Yaf 的插件基类，所有应用在 Yaf 的插件都需要继承实现这个类，这个类定义了 7 个方法，依次在 7 个实际的时候被调用

在 PHP5.3以后，打开 yaf.use_namespace 的情况下，也可以使用 Yaf\Plugin_Abstract

```php
abstract Yaf_Plugin_Abstract
{
    public void routerStartup (Yaf_Request_Abstract $request, Yaf_Response_Abstract $response);
    public void routerShutdown (Yaf_Request_Abstract $request, Yaf_Response_Abstract $response);
    public void dispatchLoopStartup (Yaf_Request_Abstract $request, Yaf_Response_Abstract $response);
    public void preDispatch (Yaf_Request_Abstract $request, Yaf_Response_Abstract $response);
    public void postDispatch (Yaf_Request_Abstract $request, Yaf_Response_Abstract $response);
    public void dispatchLoopShutdown (Yaf_Request_Abstract $request, Yaf_Response_Abstract $response);
}
```

>  注意：插件有两种部署方式，一种部署在 plugins 目录下，通过名称中的后缀（可通过 ap.name_suffix 和 ap.name_separator 来改变具体命名形式）来使得自动加载器可以正确加载。另一种是放置在类库，由普通加载规则加载。但是无论哪种形式，用户自定义的插件都要继承自 Yaf_Plugin_Abstract.

#### 11_6.Yaf_Registry

###### 简介

Yaf_Registry, 对象注册表（或称对象仓库）是一个用于在整个应用空间（applciaiton space）内存储对象和值的容器。通过把对象存储在其中，我们可以在整个项目的任何地方使用同一个对象。这种机制相当于一种全局存储。我们可以通过 Yaf_Registry 类的静态方法来使用对象注册表。另外，由于该类是一个数组对象，我们可以使用数组形式来访问其中的类方法。

在 PHP5.3 以后，打开 yaf.use_namespace 的情况下，也可以使用 Yaf\Registry

```php
Yaf_Registry
{
    public static Yaf_Registry has (string $name);
    public static Yaf_Registry get (string $name);
    public static Yaf_Registry set (string $name, mixed $value);
    public static Yaf_Registry del (string $name);
}
```

###### Yaf_Registry::set

```php
public static Yaf_Registry Yaf_Registry::set(string $name, mixed $value);
```

往全局注册表里添加一个新的项

参数：$name: 要注册的项名

$value: 要注册的项的值

返回值：Yaf_Registry

例：Yaf_Registry::set

```php
<?php
// 存入
Yaf_Registry::set('config', Yaf_Application::app()->getConfig());
// 之后可以在任何地方获取到
$config->Yaf_Registry::get("config");
```

###### Yaf_Registry::get

```php
public static Yaf_Registry Yaf_Registry::get(string $name);
```

获取注册表中寄存的项

参数：$name:  要获取的项的名字

返回值：成功，返回要注册的注册项值，失败则返回 FALSE

例：Yaf_Registry::get

```php
<?php
// 存入
Yaf_Registry::set('config', Yaf_Application::app()->getConfig());
// 之后可以在任何地方获取到
$config->Yaf_Registry::get("config");
```

###### Yaf_Registry::has

```php
public static Yaf_Registry Yaf_Registry::has(string $name);
```

查询某一项目是否存在于注册表中

参数：$name:  要查询的项的名

返回值：存在，返回 TRUE；不存在，返回 FALSE

例：Yaf_Registry::has

```php
<?php
// 存入
Yaf_Registry::set('config', Yaf_Application::app()->hasConfig());
assert(Yaf_Registry::has('config'));
```

###### Yaf_Registry::del

```php
public static Yaf_Registry Yaf_Registry::del(string name);
```

删除存在于注册表的名为 $name 的项目

参数：$name:  要删除的项的名

返回值： 成功，返回TRUE；失败， 返回 FALSE

例：Yaf_Registry::del

```php
<?php
// 存入
Yaf_Registry::set('config', Yaf_Application::app()->delConfig());
Yaf_Registry::del('config');
```



#### 11_7.Yaf_Session

###### 简介

Yaf_Session 是 Yaf 的 Session 的包装，实现了 Iterator, ArrayAccess, Countable 的接口，方便使用。

在 PHP5.3 以后，打开 yaf.use_namespace 的情况下，也可以使用 Yaf\Session

```php
final Yaf_Session implements Iterator,ArrayAccess,Countable
{
    public static Yaf_Session getInstance();
    public Yaf_Session start();
    public mixed get(string $name = NULL);
    public boolean set(string $name, mixed $value);
    public mixed __get(string $name);
    public boolean __set(string $name, mixed $value);
    public boolean has(string $name);
    public boolean del(string $name);
    public boolean __isset(string $name);
    public boolean __unset(string $name);
}
```



#### 11_8.Yaf_Config_Abstract

###### 简介

Yaf_Config_Abstract 被设计在应用程序中简化访问和使用配置数据，config 为在应用程序中访问这样的配置数据提供了一个基于用户接口的嵌入式对象属性。配置数据可能来自于各种支持等级结构数据存储的媒体。Yaf_Config_Abstract 实现了Countable，ArrayAccess 和 Iterator 接口，这样可以基于 Yaf_Config_Abstract 对象使用 count() 函数和 PHP 语句如 foreach，也可以通过数组方式访问 Yaf_Config_Abstract  的元素。

Yaf_Config_INI 为存储在ini文件中的配置数据提供了适配器。Yaf_Config_Simple 为存储在PHP数组中的配置数据提供了适配器。

在 PHP5.3以后，打开 yaf.use_namespace 的情况下，也可以使用 Yaf\Config_Abstract

```php
Abstract Yaf_Config_Abstract implements Iterator,ArrayAccess,Countable
{
    protected array _config;
    protected array _readonly;
    public mixed get(string $name = NULL);
    public mixed __get(string $name);
    public mixed __isset(string $name);
    public mixed __set(string|int $name,mixed $value);
    public mixed set(string|int $name, mixed $value);
    public mixed count();
    public mixed offsetGet(string|int $name);
    public mixed offsetSet(string|int $name,mixed $value);
    public mixed offsetExists(string|int $name);
    public mixed offsetUnset(string|int $name);
    public void rewind();
    public mixed key();
    public mixed next();
    public mixed current();
    public boolean valid();
    public array toArray();
    public boolean readOnly();
}
```

###### 属性说明

_config: 配置实际的保存容器

_readonly: 配置是否容许修改。对于 Yaf_Config_Ini 来说永远都是TRUE

###### The Yaf_Config_Ini class

简介：Yaf_Config_INI 为存储在 Ini 文件的配置数据提供了适配器。

在PHP5.3之后，默认打开 yaf.use_namespace 的情况下，也可以使用 Yaf\Config_Ini

> 注意：当使用 INI 文件作为 Yaf_Application 时，可以打开 ap.cache_config 来提升性能。

说明：Yaf_Config_Ini 允许开发者通过嵌套的对象属性语法，在应用程序中用熟悉的 INI 格式存储和读取配置数据。INI 格式在提供拥有配置数据键的等级结构和配置数据节之间的继承能力方面具有专长。

配置数据等级结构通过 '.' 或 '。' 分离键值。一个节可以扩展或通过在节的名称之后带一个冒号（：）和被继承的配置数据的节的名称来从另一个节继承。

例：INI 文件

```ini
[base]
database.master.host = localhost
[production : base]
;Yaf的配置
application.directory = /usr/local/www/production
;应用的配置
webhost = www.example.com
database.adapter = pdo.mysql
database.params.host = db.example.com
database.params.username = dbuser
database.params.password = secret
database.params.dbname = daname
;开发站点配置数据从生产站点配置数据集成，有需要可以重写
[dev : production]
application.directory = /usr/dev/htdocs
database.params.host = dev.example.com
database.params.username = devuser
database.params.password = devsecret
```

dev 节将得到 production 节的所有配置，并间接获得 base 节的配置。并且覆盖 application.directory 的配置为 “/usr/dev/htdocs”

Yaf_Config_Abstract 实现了 __get 方法，所以获取配置会很容易

例：获取配置

```php
$config = new Yaf_Config_Ini('/path/to/config.ini', 'staging');
echo $config->database->get('params')->host;  // ‘dev.example.com’
echo $config->get("database")->params->dbname; // ‘dbname’
```

###### The Yaf_Config_Simple class

简介：Yaf_Config_Simple 为存储在数组中的配置数据提供了适配器。在PHP5.3以后，打开 yaf.use_namespace 的情况下也可以使用 Yaf\Config_Simple.

#### 11_9.Yaf_Controller_Abstract

###### 简介

Yaf_Controller_Abstract 是 Yaf 的 MVC 体系的核心

Yaf_Controller_Abstract 体系具有可扩展性，可以通过继承已有的类来实现这个抽象类，从而添加应用自己的应用逻辑。

对于 Controller 来说，真正的执行体是在 Controller 中定义的一个个动作，当然这些动作也可以定义在 Controller 外部

与一般框架不同，在 Yaf 中可以定义动作的参数，这些参数的值来自对 Request 的路由结果中的同名参数值，比如对于如下控制器：

例：Yaf_Controller_Abstract 参数动作

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function indexAction ($name, $value)
    {
    }
}
```

在使用默认路由的情况下，对于请求 http://{hostname}/index/index/name/a/value/2 此请求我们知道会在 Request 对象中生成两个参数 name,value,  而注意到动作 indexAction 的参数与此同名，于是在 indexAction 中，可以有如下两种方式来获取这两个参数

例：Yaf_Controller_Abstract 参数动作

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function indexAction ($name, $value)
    {
        // 直接获取参数
        echo $name,$value;
        // 通过 Request 对象获取
        echo $this->getRequest()->getParam('name');
    }
}
```

> 注意：需要注意的是，这些参数是来自用户请求 URL，所以使用前一定要做安全化过滤，另外，为了防止 PHP 抛出参数缺失的警告，请尽量定义有默认值的参数

在PHP5.3 以后，打开 yaf.use_namespace 的情况下，也可以使用 Yaf\Controller_Abstract

```php
abstract Yaf_Controller_Abstract
{
    protected array actions;
    protected Yaf_Request_Abstract _request;
    protected Yaf_Response_Abstract _response;
    protected Yaf_View_Interface _view;
    protected string _script_path;
    private void __construct();
    public void init();
    public string getModuleName();
    public Yaf_Request_Abstract getRequest();
    public Yaf_Response_Abstract getResponse();
    public Yaf_View_Interface getView();
    public Yaf_View_Interface initView();
    public boolean setViewPath (string $view_directory);
    public string getViewPath();
    public Yaf_Response_Abstract render(string $action_name, array $tpl_vars = NULL);
    public boolean display(string $action_name, array $tpl_vars = NULL);
    public boolean forward(string $action, array $invoke_args = NULL);
    public boolean forward(string $controller, string $action, array $invoke_args = NULL);
    public boolean forward(string $module, string $controller, string $action, array $invoke_args = NULL);
    public boolean redirect(string $url);
}
```

###### 属性说明

actions:  有时候为了拆分比较大的 Controller，让代码更清晰可读和方便管理，Yaf 支持将具体的动作分开定义。每个动作都需要实现 Yaf_Action_Abstract 就可以通过定义 Yaf_Controller_Abstract::$actions 来指明哪些动作具体应用于哪些分离的类，

例如：

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public $actions = [
        "index" => "actions/Index.php",
    ];
}
```

这样当路由到达动作 Index 时，就会加载 APPLICATION_PATH . "/actions/Index.php"，并且在这个脚本文件中寻找 IndexAction (可通过 yaf.name_suffix 和 yaf.name_separator 来改变具体命名形式)，继而调用这个类的 execute 方法：

> 注意：在 yaf.st_compatible 打开的情况下，会产生额外的查找逻辑。

_request:  当前的请求实例，属性的值由 Yaf_Dispatcher 保证，一般通过 Yaf_Controller_Abstract::getRequest 来获取此属性

_response:  当前的响应对象，属性的值由 Yaf_Dispatcher 保证，一般通过 Yaf_Controller::getResponse 来获取此属性。

_view:  视图引擎，Yaf 才会延时实例化视图引擎来提高性能，所以这个属性直到显式调用 Yaf_Controller_Abstract::getView 或者 Yaf_Controller_Abstract::initView 以后才可用。

_script_path:  视图文件的目录，默认值由 Yaf_Dispatcher 保证，可通过 Yaf_Controller_Abstract::setViewPath 来改变这个值。

###### Yaf_Controller_Abstract::getModuleName

```php
public string Yaf_Controller_Abstract::getModuleName();
```

获取当前控制器所属的模块名

参数：void

返回值：成功返回模块名，失败返回 NULL

例：Yaf_Controller_Abstract::getModuleName

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function init()
    {
        echo $this->getModuleName();
    }
}
```

###### Yaf_Controller_Abstract::getRequest

```php
public Yaf_Request_Abstract Yaf_Controller_Abstract::getRequest();
```

获取当前的请求实例

参数：void

返回值：Yaf_Request_Abstract

例：Yaf_Controller_Abstract::getRequest

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function init()
    {
        $request = $this->getRequest();
    }
}
```

###### Yaf_Controller_Abstract::getResponse

```php
public Yaf_Response_Abstract Yaf_Controller_Abstract::getResponse();
```

获取当前的响应实例

参数：void

返回值：Yaf_Request_Abstract

例：Yaf_Controller_Abstract::getResponse

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function init()
    {
        $response = $this->getResponse();
    }
}
```

###### Yaf_Controller_Abstract::getView

```php
public Yaf_View_Interface Yaf_Controller_Abstract::getView();
```

获取当前的视图引擎

参数：void

返回值：Yaf_View_Interface

例：Yaf_Controller_Abstract::getView

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function init()
    {
        $view = $this->getView();
    }
}
```

###### Yaf_Controller_Abstract::initView

```php
public Yaf_View_Interface Yaf_Controller_Abstract::initView();
```

初始化视图引擎，因为 Yaf 采用延时实例化视图引擎的策略，所以只有在使用前调用此方法，视图引擎才会被实例化

参数：void

返回值：Yaf_View_Interface

例：Yaf_Controller_Abstract::initView

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function init()
    {
        $view = $this->initView();
        // 此后就可以直接通过获取 Yaf_Controller_Abstract::$_view 来访问当前的视图引擎
        $this->_view->assign("webroot", "http://{hostname}");
    }
}
```

###### Yaf_Controller_Abstract::setViewPath

```php
public boolean Yaf_Controller_Abstract::setViewPath(string $view_directory);
```

更改视图模板目录，之后 Yaf_Controller_Abstract::render 就会在整个目录下寻找模板文件。

参数：void

返回值：成功则返回 Yaf_Controller_Abstract，失败返回 FALSE

例：Yaf_Controller_Abstract::setViewPath

```php
class IndexController extends Yaf_Controller_Abstract
{
    public function init()
    {
        $this->setViewPath("/usr/local/www/tpl");
    }
}
```

###### Yaf_Controller_Abstract::getViewPath

```php
public string Yaf_Controller_Abstract::getViewPath();
```

获取当前的模板目录

参数：void

返回值：成功返回模板目录，失败返回 NULL

例：Yaf_Controller_Abstract::getViewPath

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function init()
    {
        echo $this->getViewPath();
    }
}
```

###### Yaf_Controller_Abstract::render

```php
public Yaf_Response_Abstract Yaf_Controller_Abstract::render(string $action, array $tpl_vars = NULL);
```

渲染视图模板，得到渲染结果

> 注意：此发方法是对 Yaf_View_Interface::render 的封装

参数：$action:  要渲染的动作名

$tpl_vars:  传递给视图引擎的渲染参数，当然也可以使用 Yaf_View_Interface::assign 来替代

返回值：Yaf_Response_Abstract

例：Yaf_Controller_Abstract::render

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function init()
    {
        // 首先关闭自动渲染
        Yaf_Dispatcher::getInstance()-disableView();
    }
    public function indexAction()
    {
        $this->initView();
        // 自己输出响应
        echo $this->render("test.phtml");
    }
}
```

###### Yaf_Controller_Abstract::display

```php
public boolean Yaf_Controller_Abstract::display(string $action, array $tpl_vars = NULL);
```

渲染视图模板，并直接输出渲染结果。

> 注意：此方法是对 Yaf_View_Interface::display 的包装

参数：$action:  要渲染的动作名

$tpl_vars:  传递给视图引擎的渲染参数，当然也可以使用 Yaf_View_Interface::assign 来替代。

返回值：成功返回 TRUE，失败返回 FALSE

例：Yaf_Controller_Abstract::display

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function init()
    {
        // 首先关闭自动渲染
        Yaf_Dispatcher::getInstance()->disableView();
    }
    public function indexAction()
    {
        $this->initView();
        // 输出响应
        $this->display("test.phtml", ["name" => "value"]);
    }
}
```

###### Yaf_Controller_Abstract::forward

```php
public boolean Yaf_Controller_Abstract::forward(string $action, array $params = NULL);
```

```php
public boolean Yaf_Controller_Abstract::forward(string $controller, string $action, array $params = NULL);
```

```php
public boolean Yaf_Controller_Abstract::forward(string $module, string $controller, string $action, array $params = NULL);
```

将当前请求交给另一个 action 处理

> 注意：Yaf_Controller_Abstract::forward 只是登记下要 forward 的目标地，并不会立即执行跳转。而是会等到当前的 action 执行完成以后，才会进行新一轮的 dispatch.
>
> 参数：$module:  要转给动作的模块，注意要首字母大写，如果为空则转给当前模块。
>
> $controller:  要转给动作的控制器，注意要首字母大写，如果为空则转给当前控制器
>
> $action:  要转给的动作，注意要全部小写
>
> $params:  关联数组，附加的参数可通过 Yaf_Request_Abstract::getParams 拿到
>
> 返回值：成功返回 Yaf_Controller_Abstract，失败返回 FALSE
>
> 例：Yaf_Controller_Abstract::forward
>
> ```php
> <?php
> class IndexController extends Yaf_Controller_Abstract
> {
>     public function init()
>     {
>         // 如果用户没登录，则转给登录操作
>         if($user_not_login) {
>             $this->forward("login");
>         }
>     }
> }
> ```
>
> ###### Yaf_Controller_Abstract::redirect
>
> ```php
> public boolean Yaf_Controller_Abstract::redirect(string $url);
> ```
>
> 重定向请求到新的路径
>
> 参数：$url:  要定向的路径
>
> 返回值：成功返回 Yaf_Controller_Abstract， 失败返回 FALSE
>
> 例：Yaf_Controller_Abstract::redirect
>
> ```php
> <?php
> class IndexController extends Yaf_Controller_Abstract
> {
>     public function init()
>     {
>         if($user_not_login) {
>             $this->redirect("/login/");
>         }
>     }
> }
> ```

#### 11_10.Yaf_Action_Abstract

###### 简介

Yaf_Action_Abstract 是 MVC 中 C 的动作，一般而言动作都是定义在 Yaf_Controller_Abstract 的派生类中。但有的时候为使得代码更清晰，分离一些大的控制器，我们选择单独去继承这个 Yaf_Action_Abstract 来实现。

Yaf_Action_Abstract 体系具有可扩展性，可通过继承已有的类来实现这个抽象类，从而添加我们自己的应用逻辑。

在PHP5.3以后，打开 yaf.use_namespace 的情况下，也可以使用 Yaf\Action_Abstract.

```php
abstract Yaf_Action_Abstract extends Yaf_Action_Controller
{
	public abstract void execute();
}
```



#### 11_11.Yaf_View_Interface

###### 简介

Yaf_View_Interface 是为了提供可扩展的，可自定义的视图引擎而设定的视图引擎接口，他定义了用在 Yaf 上的视图引擎需要实现的方法和功能。

在PHP5.3以后，打开 yaf.use_namespace 的情况下，也可以使用 yaf\View_Interface

```php
interface Yaf_View_Interface
{
    public string render(string $view_path, array $tpl_vars = NULL);
    public boolean display(string $view_path, array $tpl_vars = NULL);
    public boolean assign(mixed $name, mixed $value = NULL);
    public boolean setScriptPath(string $view_directory);
    public string getScriptPath();
}
```

###### The Yaf_View_Simple class

简介：Yaf_View_Simple 是 Yaf 自带的视图引擎，它追求性能，所以并没有提供类似 Smarty 那样的多样功能和复杂的语法。

对于 Yaf_View_Simple 的视图模板，就是普通的PHP脚本，对于通过 Yaf_View_Interface::assign 的模板变量，可在视图模板中直接通过变量名使用。

在PHP5.3 之后，打开 yaf.use_namespace 的情况下，也可以使用 Yaf\View_Simple

```php
Yaf_View_Simple extends Yaf_View_Interface
{
    protected array _tpl_vars;
    protected string _script_path;
    public string render(string $view_path, array $tpl_vars = NULL);
    public boolean display(string $view_path, array $tpl_vars = NULL);
    public string getScriptPath();
    public boolean assign(string $name, mixed $value);
    public boolean __set(string $name, mixed $value = NULL);
    public mixed __get(string $name);
}
```

属性说明：_tpl_vars:  所有通过Yaf_View_Simple::assign 分配的变量，都保存在这个变量里。

_script_path:  当前视图引擎的模板文件的根目录

###### Yaf_View_Simple::assign

```php
public boolean Yaf_View_Simple::assign(mixed $name, mixed $value = NULL);
```

为视图引擎分配一个模板变量，在视图模板中可通过 ${ $name } 获取模板变量值

参数：$name:  字符串或关联数组，如果为字符串，则 $value 不能为空，此字符串代表要分配的变量名。如果为数组，则$value 需为空，此参数为变量名和值的关联数组

$value:  分配的模板变量

> 注意：如果 $name 不是合法的PHP变量名，比如整数或是包含 “ | ” 的字符串，那么在视图模板文件中将不能直接通过 ${$name} 来访问这个变量。当然，你还是可以在视图模板文件中通过 $this->_tpl_vars[ $name ] 来访问这个变量。

返回值：成功返回 Yaf_View_Simple，失败返回 FALSE

例：Yaf_View_Simple::assign

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function init()
    {
        $params = array('name' => 'value');
        $this->getView()->assign($params)->assign("foo", "bar");
    }
}
```

###### Yaf_View_Simple::render

```php
public string Yaf_View_Simple::render(string $view_path, array $tpl_vars = NULL);
```

渲染一个视图模板，得到结果。

参数：$view_path:  视图模板的文件，绝对路径，一般这个路径由 Yaf_Controller_Abstract 提供

$tpl_vars:  关联数组，模板变量

返回值：成功返回视图模板执行结果，失败返回 NULL

例：Yaf_View_Simple::render

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function indexAction()
    {
        echo $this->getView()->render($this->_script_path . "/test.phtml");
    }
}
```

###### Yaf_View_Simple::display

```php
public string Yaf_View_Simple::display(string $view_path, array $tpl_vars = NULL);
```

渲染一个视图模板，并直接输出到客户端

参数：$view_path:  视图模板的文件名，绝对路径，一般这个路径由 Yaf_Controller_Abstract 提供

$tpl_vars:  关联数组，模板变量

返回值：成功返回 TRUE，失败返回 FALSE

例：Yaf_View_Simple::display

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function indexAction()
    {
        $this->getView()->display($this->_script_path . "/test.phtml");
    }
}
```

###### Yaf_View_Simple::setScriptPath

```php
public boolean Yaf_View_Simple::setScriptPath(string $view_directory);
```

设置模板文件的根目录，默认的Yaf_Dispatcher 会把这个根目录设成 APPLICATION_PATH . "/views".

参数：$view_directory:  视图模板的根目录，绝对地址

返回值：成功TRUE，失败FALSE

例：Yaf_View_Simple::setScriptPath

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function indexAction()
    {
        $this->getView()->setScriptPath("/tmp/views/");
    }
}
```

###### Yaf_View_Simple::getScriptPath

```php
public string Yaf_View_Simple::getScriptPath(void);
```

获取当前的模板目录

参数：void

返回值：成功则返回当前的视图模板文件根目录，失败返回NULL

例：Yaf_View_Simple::getScriptPath

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function indexAction()
    {
        echo $this->getView()->getScriptPath();
    }
}
```

###### Yaf_View_Simple::__set

```php
public boolean Yaf_View_Simple::__set(mixed $name, mixed $value = NULL);
```

为视图引擎分配一个模板变量，在视图模板中可直接通过 ${ $name } 获取模板变量值

参数：$name:  字符串或关联数组，如果为字符串，则 $value 不能为空，此字符串代表要分配的变量名。如果为数组则 $value 须为空，此参数为变量名和值的关联数组。

$value:  分配的模板变量值

返回值：成功返回 Yaf_View_Simple，失败返回 FALSE

例：Yaf_View_Simple::__set

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function init()
    {
        $this->getView()->name = "value";
    }
}
```

###### Yaf_View_Simple::__get

```php
public string Yaf_View_Simple::__get(string $name);
```

获取视图引擎的一个模板变量值

参数：$name:  模板变量名

返回值：成功则返回变量值，失败返回 NULL

例：Yaf_View_Simple::__get

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function init()
    {
        $this->initView();
    }
    public function indexAction()
    {
        // 通过 __get 直接获取变量值
        echo $this->_view->name;
    }
}
```

###### Yaf_View_Simple::get

```php
public string Yaf_View_Simple::get(string $name);
```

获取视图引擎的一个模板变量值

参数：$name:  模板变量名

返回值：成功返回变量值，失败返回NULL

例：Yaf_View_Simple::get

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function init()
    {
        $this->initView();
    }
    public function indexAction()
    {
        echo $this->_view->get("name");
    }
}
```



#### 11_12.Yaf_Request_Abstract

###### 简介

代表了一个实际请求，一般地我们不用主动实例化它，Yaf_Application 在 run 以后会自动根据当前请求实例化它。

在PHP5.3以后，打开 yaf.use_namespace 的情况下，也可以使用 Yaf\Request_Abstract

```php
abstract Yaf_Request_Abstract
{
    protected string _method;
    protected string _module;
    protected string _controller;
    protected string _action;
    protected array _params;
    protected string _language;
    protected string _base_uri;
    protected string _request_uri;
    protected boolean _dispatched;
    protected boolean _routed;
    public string getModuleName();
    public string getControllerName();
    public string getActionName();
    public boolean setModuleName(string $name);
    public boolean setControllerName(string $name);
    public boolean setActionName(string $name);
    public Exception getException();
    public mixed getParams();
    public mixed getParam(string $name, mixed $default = NULL);
    public mixed setParam(string $name, mixed $value);
    public mixed getMethod();
    abstract public mixed getLanguage();
    abstract public mixed getQuery(string $name = NULL);
    abstract public mixed getPost(string $name = NULL);
    abstract public mixed getEnv(string $name = NULL);
    abstract public mixed getServer(string $name = NULL);
    abstract public mixed getCookie(string $name = NULL);
    abstract public mixed getFiles(string $name = NULL);
    abstract public bool isGet();
    abstract public bool isPost();
    abstract public bool isHead();
    abstract public bool isXmlHttpRequest();
    abstract public bool isPut();
    abstract public bool isDelete();
    abstract public bool isOption();
    abstract public bool isCli();
    public bool isDispatched();
    public bool setDispatched();
    public bool isRouted();
    public bool setRouted();
}
```

###### 属性说明

_method:  当前请求的 Method。对于命令行来说，Method 为 “CLI”

_language:  当前请求希望接受的语言。对于 Http 请求而言，这个值来自分析请求头 Accept-Language。对于不能鉴别的情况，此值为 NULL.

_module:  在路由完成之后，请求被分配到的模块名

_controller:  在路由完成之后，请求被分配到的控制器名

_action:  在路由完成之后，请求被分配到的动作名

_params:  当前请求的附加参数

_routed:  表示当前请求是否已经完成路由

_dispatched:  表示当前请求是否已经完成分发

_requeset_uri:  当前请求的 Request URI

_base_uri:  当前请求 Request URI 要忽略的前缀，一般不需要手动配置，Yaf会自己分析。当 Yaf 分析出错时，可通过 application.baseUri 来手动设置。

###### The Yaf_Request_Http class

简介：代表了一个实际的 Http 请求。一般我们不用主动实例化它，Yaf_Application 会在 run 之后自动根据当前请求实例化它。

在PHP5.3后，打开 yaf.use_namespace 的请求下，也可以使用 Yaf\Request_Http

```php
final Yaf_Request_Http extends Yaf_Request_Abstract
{
    public void __construct(string $request_uri = NULL, string $base_uri = NULL);
    public mixed getLanguage();
    public mixed getQuery(string $name = NULL);
    public mixed getPost(string $name = NULL);
    public mixed getEnv(string $name = NULL);
    public mixed getServer(string $name = NULL);
    public mixed getCookie(string $name = NULL);
    public mixed getFiles(string $name = NULL);
    public boolean isGet();
    public boolean isPost();
    public boolean isHead();
    public boolean isXmlHttpRequest();
    public boolean isPut();
    public boolean isDelete();
    public boolean isOption();
    public boolean isCli();
    public boolean isDispatched();
    public boolean setDispatched();
    public boolean isRouted();
    public boolean setRouted();
    public string getBaseUri();
    public boolean setBaseUri(string $base_uri);
    public string getRequestUri();
}
```

###### The Yaf_Request_Simple class

简介：代表了一个实际的请求，一般的不用自己实例化它，Yaf_Application 在run以后会自动根据当前请求来实例化它。

在PHP5.3以后，打开 yaf.use_namespace 的情况下，也可以使用 Yaf\Request_Simple

```php
final Yaf_Request_Simple extends Yaf_Request_Abstract
{
    public void __construct(string $module, string $controller, string $action, string $method, array $params = NULL);
    public mixed getLanguage();
    public mixed getQuery(string $name = NULL);
    public mixed getPost(string $name = NULL);
    public mixed getEnv(string $name = NULL);
    public mixed getServer(string $name = NULL);
    public mixed getCookie(string $name = NULL);
    public mixed getFiles(string $name = NULL);
    public bool isGet();
    public bool isPost();
    public bool isHead();
    public bool isXmlHttpRequest();
    public bool isPut();
    public bool isDelete();
    public bool isOption();
    public bool isSimple();
    public bool isDispatched();
    public bool setDispatched();
    public bool isRouted();
    public bool setRouted();
}
```

###### Yaf_Request_Abstract::getException

```php
public Exception Yaf_Request_Abstract::getException();
```

此方法适用于异常捕获下，在异常发生的时候，流程进入 Error 控制器的 error 动作时，获取当前发生的异常对象。

参数：void

返回值：在有异常的情况下，返回当前异常对象。没有异常的情况下返回 NULL

例：Yaf_Request_Abstract::getException

```php
<?php
class ErrorController extends Yaf_Controller_Abstract
{
    public function errorAction()
    {
        $exception = $this->getRequest()->getException;
    }
}
```

###### Yaf_Request_Abstract::getModuleName

```php
public string Yaf_Request_Abstract::getModuleName();
```

获取当前请求被 路由 到的模块名

参数：void

返回值：路由成功以后，返回当前被分发处理此请求的模块名。路由失败，返回NULL

例：Yaf_Request_Abstract::getModuleName

```php
<?php
class ErrorController extends Yaf_Controller_Abstract
{
    public function errorAction()
    {
        echo "current Module:" . $this->getRequest()->getModuleName();
    }
}
```

###### Yaf_Request_Abstract::getControllerName

```php
public string Yaf_Request_Abstract::getControllerName();
```

获取当前请求被路由到的控制器名。

参数：void

返回值：路由成功以后，返回当前被分发来处理此次请求的控制器名。路由失败返回NULL

例：Yaf_Request_Abstract::getControllerName

```php
<?php
class ErrorController extends Yaf_Controller_Abstract
{
    public function errorAction()
    {
        echo "current Controller:" . $this->getRequest()->getControllerName();
    }
}
```

###### Yaf_Request_Abstract::getActionName

```php
public string Yaf_Request_Abstract::getActionName();
```

获取当前请求被路由到的 action 名

参数：void

返回值：路由成功以后，返回当前被分发处理此次请求的动作名。路由失败返回 NULL

例：Yaf_Request_Abstract::getActionName

```php
<?php
class ErrorController extends Yaf_Controller_Abstract
{
    public function errorAction()
    {
        echo "current Action:" . $this->getRequest()->getActionName();
    }
}
```

###### Yaf_Request_Abstract::getParams

```php
public array Yaf_Request_Abstract::getParams();
```

获取当前请求中的所有路由参数，路由参数不是指 $_GET 或者 $_POST，而是指在路由过程中，路由协议根据 Request Uri 分析出的请求参数。

比如对于默认的路由协议 Yaf_Route_Static，路由 一个如下请求：

> http://www.example.com/module/controller/action/name1/value1/name2/value2.

路由结束后，将得到两个路由参数：name1, name2，值对应分别为 value1, value2

> 注意：路由参数和 $_GET, $_POST 一样，是来自用户的输入，都是不可信的。使用时要自行处理

参数：void

返回值：当前所有的路由参数

例：Yaf_Request_Abstract::getParams

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function indexAction()
    {
        $this->getRequest()->getParams();
    }
}
```

###### Yaf_Request_Abstract::getParam

```php
public string Yaf_Request_Abstract::getParam(string $name, mixed $default_value = NULL);
```

获取当前请求中的路由参数，路由参数不是指 $_GET 或 $_POST , 而是在路由过程中，路由协议根据 Request Uri 分析出的请求参数。

比如对于默认的路由协议 Yaf_Route_Static，路由请求如下：

```json
http://{hostname}/module/controller/action/name1/value1/name2/value2
```

路由结束后将会得到两个路由参数，name1，name2，值分别为  value1，value2、

> 注意：路由参数和 $_GET，$_POST 一样，是来自用户的输入，都是不可信的，使用前需要做完全过滤

参数：$name: 要获取的路由参数名

$default_value: 如果设定此参数，如果没有找到 $name, 路由参数，则返回此参数值。

返回值：

找到返回对应的路由参数值，如果没有找到并且设置了 $default_value, 则返回 default_value, 否则返回 NULL

例：Yaf_Request_Abstract::getParam

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function indexAction()
    {
        echo "user id: " . $this->getRequest()->getParam("userid", 0);
    }
}
```

###### Yaf_Request_Abstract::setParam

```php
public booleam Yaf_Request_Abstract::setParam(string $name, mixed $value);
```

为当前的请求设置路由参数

参数：$name:  参数名

$value:  值

返回值：成功返回 Yaf_Request_Abstract 实例，失败返回 FALSE

例：Yaf_Request_Abstract::setParam

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function indexAction()
    {
        $this->getRequest()->setParam("userid", 0);
    }
}
```

###### Yaf_Request_Abstract::getMethod

```php
public string Yaf_Request_Abstract::getMethod();
```

获取当前请求的类型，可能的返回值为 GET, POST, PUT, CLI, HEAD等等

参数：void

返回值：当前的请求类型

例：Yaf_Request_Abstract::getMethod

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function indexAction()
    {
        if($this->getRequest()->getMethod() == "CLI") {
            echo "Running in CLI mode.";
        }
    }
}
```

###### Yaf_Request_Abstract::isCli

```php
public string Yaf_Request_Abstract::isCli();
```

获取当前请求是否为 CLI 请求

参数：void

返回值：是 CLI 请求返回  true， 不是返回 FALSE

例：Yaf_Request_Abstract::isCli

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function indexAction()
    {
        if($this->getRequest()->isCli()) {
            echo "Running in Cli mode.";
        }
    }
}
```

###### Yaf_Request_Abstract::isGet

```php
public string Yaf_Request_Abstract::isGet();
```

获取当前请求是否是 GET 请求。

参数： void

返回值：是GET 请求返回 TRUE，不是返回 FALSE

例：Yaf_Request_Abstract：：isGet

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function indexAction()\
    {
        if($this->getRequest()->isGet()) {
            echo "Running in Get mode.";
        }
    }
}
    
```

#### 11_13.Yaf_Response_Abstract

###### 简介

响应对象和请求对象相对应，是发送给请求端的响应载体。

在PHP5.3之后，打开 yaf.use_namespace 的情况下，也可以使用 Yaf\Response_Abstract

```php
abstract Yaf_Response_Abstract
{
    protected array _body;
    protected array _header;
    public boolean setBody(string $body, string $name = NULL);
    public boolean prependBody(string $body, string $name = NULL);
    public boolean appendBody(string $body, string $name = NULL);
    public boolean clearBody();
    public string getBody();
    public boolean response();
    public boolean setRedirect(string $url);
    public string __toString();
}
```

###### 属性说明：

_body：  响应给请求的 Body 内容

_header:  响应给请求的 Header，目前是保留属性

###### The Yaf_Response_Http class

简介：Yaf_Response_Http 是在 Yaf 作为 Web 应用时默认的响应载体。

```php
final Yaf_Response_Http extends Yaf_Response_Abstract
{
    protected array _code = 200;
}
```

属性说明：_code:  响应给请求端的 HTTP 状态码

###### The Yaf_Response_Cli class

简介：Yaf_Response_Cli 是在 Yaf 作为命令行应用时默认的响应载体。

```php
final Yaf_Response_Cli extends Yaf_Response_Abstract
{
    //TODO
}
```

###### Yaf_Response_Abstract::setBody

```php
public boolean Yaf_Response_Abstract::setBody(string $body, string $name = NULL);
```

设置响应的 Body，$name 参数是保留参数，目前没有特殊效果，留空即可。

参数：$body:  要响应的字符串，一般是一段 HTML 或者是一段 JSON

$name:  要响应的字符串 key，一般的我们可以通过指定不同的 key，给一个 response 对象设置很多响应字符串，可以在所有的请求结束后做 layout, 如果我们不做特殊处理，交给 Yaf 去发送响应的话，所有我们设置的响应字符串会按照设置的先后顺序被输出给客户端。

返回值：成功返回 Yaf_Response_Abstract 实例，失败返回 FALSE

例：Yaf_Response_Abstract::setBody

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function init()
    {
        $this->getResponse()->setBody('hello world');
    }
}
```

###### Yaf_Response_Abstract::appendBody

```php
public boolean Yaf_Response_Abstract::appendBody(string $body, string $name = NULL);
```

往已有的响应 body 后附加新的内容，$name 参数是保留参数，目前没有特殊效果，留空即可。

参数：$body:  要附加的字符串，一般是 HTML，或是一段 JSON（返回给 Ajax 请求）

$name:  保留参数，目前没有特殊效果

返回值：成功返回 Yaf_Response_Abstract, 失败返回 FALSE

例：Yaf_Response_Abstract::appendBody

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function init()
    {
        $this->getResponse()->appendBody("hello world");
    }
}
```

###### Yaf_Response_Abstract::prependBody

```php
public boolean Yaf_Response_Abstract::prependBody(string $body, string $name = NULL);
```

往已有的响应 body 前插入新的内容，$name 参数是保留参数，目前没有特殊效果，留空即可。

参数：$body:  要附加的字符串，一般是 HTML，或是一段 JSON（返回给 Ajax 请求）

$name:  保留参数，目前没有特殊效果

返回值：成功返回 Yaf_Response_Abstract, 失败返回 FALSE

例：Yaf_Response_Abstract::appendBody

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function init()
    {
        $this->getResponse()->preprendBody("hello world");
    }
}
```

###### Yaf_Response_Abstract::getBody

```php
public string Yaf_Response_Abstract::getBody();
```

参数：void

返回值：成功则返回已设置的 body 值，失败返回 NULL

例：Yaf_Response_Abstract::getBody

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function init()
    {
        echo $this->getResponse()->getBody();
    }
}
```

###### Yaf_Response_Abstract::clearBody

```php
public boolean Yaf_Response_Abstract::clearBody(); 
```

参数：void

返回值：成功返回 Yaf_Response_Abstract，失败返回 FALSE

例：Yaf_Response_Abstract::clearBody

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function init()
    {
        $this->getResponse()->clearBody();
    }
}
```

###### Yaf_Response_Abstract::response

```php
public boolean Yaf_Response_Abstract::response();
```

发送响应给请求端

参数：void

返回值：成功返回TRUE，失败返回 FALSE

例：Yaf_Response_Abstract::response

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function init()
    {
        $this->getResponse()->response();
    }
}
```

###### Yaf_Response_Abstract::setRedirect

```php
public boolean Yaf_Response_Abstract::setRedirect(string $url);
```

重定向请求到 URL

> 注意：和 Yaf_Controller_Abstract::forward 不同的是，这个重定向是 HTTP 3.1 重定向

参数：$url:  要重定向到的 URL

返回值：成功返回 Yaf_Response_Abstract,  失败返回 FALSE

例：Yaf_Response_Abstract::setRedirect

```php
<?php
class IndexController extends Yaf_Controller_Abstract
{
    public function init()
    {
        $this->getResponse()->setRedirect("http://{hostname}");
    }
}
```

###### Yaf_Response_Abstract::__toString

```php
public string Yaf_Response_Abstract::__toString();
```

魔术方法

参数: void

返回值：Yaf_Response_Abstract 中的 body 值

#### 11_14.Yaf_Router

###### 简介

Yaf 的路由器，负责分析请求中的 request uri，得出目标模板，控制器，动作。

在 PHP5.3 之后打开 yaf.use_namespace 的情况下，也可以使用 Yaf\Router

```php
final Yaf_Router
{
    protected array _routes;
    protected string _current_route;
    public Yaf_Router addRoute(string $name, Yaf_Route_Interface $route);
    public Yaf_Config addConfig(Yaf_Config_Abstract $routes_config);
    public array getRoutes();
    public array getRoute(string $name);
    public string getCurrentRoute();
    public boolean route(Yaf_Request_Abstract $request);
    public boolean isModuleName(string $name);
}
```

###### 属性说明

_routes:  路由器已有的 路由协议栈，默认的栈底总是名为 ”default“ 的Yaf_Route_Static 路由协议的实例。

_current_route:  在路由成功之后，路由生效的路由协议名。

###### Yaf_Router::addRoute

```php
public boolean Yaf_Router::addRoute(string $name, Yaf_Route_Interface $route);
```

给路由器增加一个名为 $name 的路由协议。

参数：$name:  要增加的路由协议名

$route:  要增加的路由协议，Yaf_Route_Interface 的一个实例

返回值：成功返回 Yaf_Router 的实例，失败返回 FALSE，并抛出异常（或直接触发错误）。

例：Yaf_Router::addRoute

```php
<?php
class Bootstrap extends Yaf_Bootstrap_Abstract
{
    public function _initRoute(Yaf_Dispatcher $dispatcher)
    {
        // 添加一个路由
        $route = new Yaf_Route_Rewrite(
        	"/product/list/:id/",
            array(
            	"controller" => "product",
                "action" => "info",
            ),
        );
        $router->addRoute('dummy', $route);
    }
}
```

###### Yaf_Config::addConfig

```php
public boolean Yaf_Config::addConfig(Yaf_Config_Abstract $routes_config);
```

给路由器通过配置增加一簇路由协议。

参数：$routers_config:  一个Yaf_Config_Abstract 的实例，它包含了一簇路由协议的定义，一个例子是：

```ini
;ini 配置文件
[product]
routes.regex.type="regex"
routes.regex.route="#^list/([^/]*)/([^/]*)#"
routes.regex.default.controller=Index
routes.regex.default.action=action
routes.regex.map.1=name
routes.regex.map.2=value

routes.simple.type="simple"
routes.simple.controller=c
routes.simple.module=m
routes.simple.action=a

routes.supervar.type="supervar"
routes.supervar.varname=r

routes.rewrite.type="rewrite"
routes.rewrite.route="/product/:name/:value"
routes.rewrite.default.controller=product
routes.rewrite.default.action=info
```

返回值：成功返回 Yaf_Config 实例，失败返回 FALSE，并抛出异常（或直接触发错误）

例：Yaf_Config::addConfig

```php
<?php
class Bootstrap extends Yaf_Bootstrap_Abstract
{
    public function _initRoute(Yaf_Dispatcher $dispatcher)
    {
        $router=Yaf_Dispatcher::getInstance()->getRouter();
        $router->addConfig(Yaf_Registry::get("config")->routes);
    }
}
```

###### Yaf_Router::getRoutes

```php
public array Yaf_Router::getRoutes();
```

获取当前路由器中的所有路由协议

参数:void

返回值：成功返回当前路由器的路由协议栈内容，失败返回 FALSE

例：Yaf_Router::getRoutes

```php
<?php
$routes = Yaf_Dispatcher::getInstance()->getRouter()->getRoutes();    
```

###### Yaf_Router::getRoute

```php
public Yaf_Route_Interface Yaf_Router::getRoute(string $name);
```

获取当前路由器的路由协议栈的名为 $name  的协议。

参数：void

返回值：成功返回目标路由协议，失败返回 NULL

例：Yaf_Router::getRoute

```php
<?php
// 路由器中永远一个名为 default 的 Yaf_Route_Static 路由协议实例
$routes = Yaf_Dispatcher::getInstance()->getRouter()->getRouter('default');    
```

###### Yaf_Router::getCurrentRoute

```php
public string Yaf_Router::getCurrentRoute();
```

在路由结束以后，获取匹配路由成功且路由生效的路由协议名

参数:void

返回值：成功则返回生效的 路由协议名，失败则返回 NULL

例：Yaf_Router::getCurrentRoute

```php
<?php
class UserPlugin extends Yaf_Plugin_Abstract
{
    public function routerShutdown(Yaf_Request_Abstract $request, Yaf_Response_Abstract $response)
    {
        echo "生效的路由协议是：" . Yaf_Dispatcher::getInstance()->getRouter()->getCurrentRoute();
    }
}
```

###### Yaf_Router::isModuleName

```php
public boolean Yaf_Router::isModuleName(string $name);
```

判断一个 Module 名，是否是已被声明存在的 Module

> 注意：通过 ap.modules 在配置文件中声明 加载的模块名列表
>
> 参数：$name:  Module 名
>
> 返回值：是返回 TRUE，不是返回 FALSE
>
> 例：Yaf_Router::isModuleName
>
> ```php
> <?php
> $routes = Yaf_Dispatcher::getInstance()->isModuleName("Index");    
> ```
>
> ###### Yaf_Router::route
>
> ```php
> public boolean Yaf_Router::route(Yaf_Request_Abstract $request);
> ```
>
> 路由一个请求，本方法不需要主动调用，Yaf_Dispatcher::dispatcher 会自动调用此方法。
>
> 参数：$request:  一个 Yaf_Request_Abstract 实例
>
> 返回值：成功返回TRUE，失败返回 FALSE

#### 11_15.Yaf_Route_Interface

#### 11_16.Yaf_Exception

