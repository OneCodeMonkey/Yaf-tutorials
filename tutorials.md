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

