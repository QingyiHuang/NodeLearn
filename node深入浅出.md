###node的http服务器###
    如果我们使用PHP来编写后端的代码时，需要Apache 或者 Nginx 的HTTP 服务器，
    并配上 mod_php5 模块和php-cgi。
    从这个角度看，整个"接收 HTTP 请求并提供 Web 页面"的需求根本不需 要 PHP 来处理。
    不过对 Node.js 来说，概念完全不一样了。使用 Node.js 时，我们不仅仅 在实现一个应用，
    同时还实现了整个 HTTP 服务器。事实上，我们的 Web 应用以及对应的 Web 服务器基本上是一样的
###简易的http服务器###
    var http = require('http');
    
    http.createServer(function (request, response) {
    
    	// 发送 HTTP 头部 
    	// HTTP 状态值: 200 : OK
    	// 内容类型: text/plain
    	response.writeHead(200, {'Content-Type': 'text/plain'});
    
    	// 发送响应数据 "Hello World"
    	response.end('Hello World\n');
    }).listen(8888);
    
    // 终端打印如下信息
    console.log('Server running at http://127.0.0.1:8888/');

//我们向 createServer 函数传递了一个匿名函数。匿名函数可拆分
###node的模块机制###
引入require模块
例如：

    var http = require("http");
> 在模块中，上下文提供require()方法引入外部模块，对应引入功能
> 上下文提供的export对象用于导出当前模块的方法或者变量，并且它是唯一的导出的出口。
> 在模块中还存在一个module对象，他代表模块自身，且export是module的属性，在node中，一个文件就是一个模块，将方法挂载到export对象上作为属性即可定义导出的方式
> !

    在node中引入模块，需要经历三个步骤：
    1路径分析
    2文件定位
    3编译执行
require会优先从缓存中加载缓存文件

###模块系统：1创建模块###
在 Node.js 中，创建一个模块非常简单，如下我们创建一个 'main.js' 文件，代码如下:

	var hello = require('./hello');
	hello.world();

> 以上实例中，代码 require('./hello') 引入了当前目录下的hello.js文件（./ 为当前目录，node.js默认后缀为js）。
> Node.js 提供了exports 和 require 两个对象，其中 exports 是模块公开的接口，require 用于从外部获取一个模块的接口，即所获取模块的 exports 对象。

接下来我们就来创建hello.js文件，代码如下：

	exports.world = function() {
	  console.log('Hello World');
	}

> 在以上示例中，hello.js 通过 exports 对象把 world 作为模块的访 问接口，在 main.js 中通过 require('./hello') 加载这个模块，然后就可以直接访 问main.js 中 exports 对象的成员函数了。

有时候我们只是想把一个对象封装到模块中，格式如下：

	module.exports = function() {
	  // ...
	}

例如:

	//hello.js 
	function Hello() { 
	 var name; 
	    this.setName = function(thyName) { 
	       name = thyName; 
	  }; 
	   this.sayHello = function() { 
	     console.log('Hello ' + name); 
	  }; 
	}; 
	module.exports = Hello;

这样就可以直接获得这个对象了：

	//main.js 
	var Hello = require('./hello'); 
	hello = new Hello(); 
	hello.setName('BYVoid'); 
	hello.sayHello(); 

模块接口的唯一变化是使用 module.exports = Hello 代替了exports.world = function(){}。 在外部引用该模块时，其接口对象就是要输出的 Hello 对象本身，而不是原先的 exports。
####node提供的模块叫做核心模块，用户编写的模块称作文件模块####
	核心模块部分在node进程启动的时候就已经引入，
	所以文件定位和编译执行两个步骤可以省略
###路径分析与文件定位###
模块标志符

	以/开始的绝对路径文件模块//项目的根文件
	以./或../开始的相对路径文件模块[
	非路径形式的文件模块，比如自定义的connect模块
	核心模块，如http,fs，paht；
###目录分析和包###
	在分析标识符的过程中，require()通过分析文件扩展名之后，可能没查找到文件，却得到一个目录，

###NPM的三种场景###
下载第三方包在node_modules目录中，无需指示路径
可以通过项目文件里的package.json， npm命令直接 npm install 下载指定文件

    我们可以使用以下命令来下载第三方包。
    $ npm install argv
    ...
    argv@0.0.2 node_modules\argv
    下载好之后，argv包就放在了工程目录下的node_modules目录中，因此在代码中只需要通过require('argv')的方式就好，无需指定第三方包路径。
    以上命令默认下载最新版第三方包，如果想要下载指定版本的话，可以在包名后边加上@<version>，例如通过以下命令可下载0.0.1版的argv。
    $ npm install argv@0.0.1
    ...
    argv@0.0.1 node_modules\argv
    NPM对package.json的字段做了扩展，允许在其中申明第三方包依赖。因此，上边例子中的package.json可以改写如下：
    {
    "name": "node-echo",
    "main": "./lib/echo.js",
    "dependencies": {
    "argv": "0.0.2"
    }
    }
    这样处理后，在工程目录下就可以使用npm install命令批量安装第三方包了。
    更重要的是，当以后node-echo也上传到了NPM服务器，别人下载这个包时，NPM会根据包中申明的第三方包依赖自动下载进一步依赖的第三方包。
    例如，使用npm install node-echo命令时，NPM会自动创建以下目录结构。
    - project/
    - node_modules/
    - node-echo/
    - node_modules/
    + argv/
    ...
    ...
    如此一来，用户只需关心自己直接使用的第三方包，不需要自己去解决所有包的依赖关系。
    安装命令行程序
    
    从NPM服务上下载安装一个命令行程序的方法与第三方包类似。
    例如上例中的node-echo提供了命令行使用方式，只要node-echo自己配置好了相关的package.json字段，对于用户而言，只需要使用以下命令安装程序。
    $ npm install node-echo -g
    参数中的-g表示全局安装，因此node-echo会默认安装到以下位置，并且NPM会自动创建好Linux系统下需要的软链文件或Windows系统下需要的.cmd文件。
    - /usr/local/   # Linux系统下
    - lib/node_modules/
    + node-echo/
    ...
    - bin/
    node-echo
    ...
    ...
    
    - %APPDATA%\npm\# Windows系统下
    - node_modules\
    + node-echo\
    ...
    node-echo.cmd
    ...
    发布代码
    
    第一次使用NPM发布代码前需要注册一个账号。终端下运行npm adduser，之后按照提示做即可。
    账号注册完成后，接着我们需要编辑package.json文件，加入NPM必需的字段。接着上边node-echo的例子，package.json里必要的字段如下。
    {
    "name": "node-echo",   # 包名，在NPM服务器上须要保持唯一
    "version": "1.0.0",# 当前版本号
    "dependencies": {  # 第三方包依赖，需要指定包名和版本号
    "argv": "0.0.2"
      },
    "main": "./lib/echo.js",   # 入口模块位置
    "bin" : {
    "node-echo": "./bin/node-echo"  # 命令行程序名和主模块位置
    }
    }
    之后，我们就可以在package.json所在目录下运行npm publish发布代码了。
###Nodejs回调函数###
> Node.js 异步编程的直接体现就是回调。
> 异步编程依托于回调来实现，但不能说使用了回调后程序就异步化了。
> 回调函数在完成任务后就会被调用，Node 使用了大量的回调函数，Node 所有 API 都支持回调函数。
> 例如，我们可以一边读取文件，一边执行其他命令，在文件读取完成后，我们将文件内容作为回调函数的参数返回。这样在执行代码时就没有阻塞或等待文件 I/O 操作。这就大大提高了 Node.js 的性能，可以处理大量的并发请求。
实例：

input.txt:

	hello my name is huang;
阻塞例子
main.js:

    var fs = require("fs");
    
    var data = fs.readFileSync('input.txt');
    
    console.log(data.toString());
    console.log("程序执行结束!");

    //$ node main.js
    hello my name is huang;

非阻塞例子：
main.js

    var fs = require("fs");
    fs.readFile('input.txt', function (err, data) {
    if (err) return console.error(err);
    console.log(data.toString());
    });
    
    console.log("程序执行结束!");
    
       // $ node main.js
    	hello my name is huang;
    
###nodejs的事件循环###
> Node.js 是单进程单线程应用程序，但是通过事件和回调支持并发，所以性能非常高。
> Node.js 的每一个 API 都是异步的，并作为一个独立线程运行，使用异步函数调用，并处理并发。
> Node.js 基本上所有的事件机制都是用设计模式中观察者模式实现。
> Node.js 单线程类似进入一个while(true)的事件循环，直到没有事件观察者退出，每个异步事件都生成一个事件观察者，如果有事件发生就调用该回调函数.

###Buffer 缓冲区###
[buffer](http://www.w3cschool.cn/nodejs/nodejs-buffer.html)
##NPM包管理工具##
###包描述文件package.json###
	name:包名/包名必须唯一/不要加上node或者js重复标识符
	description：包的简介
	vertion:版本号
	keywords:关键词数组用来做分类搜索
	maintainers:包维护者列表
	"maintainers":[{"name":"HuangQingyi","email":"297947067@qq.com","web":"http://huangqingyi.com"}]
	contributors:贡献者列表
	bugs:一个可以反馈bug的网页地址
	licenses:当前包所使用的许可证列表
	repositories:托管源代码的位置列表，表明可以通过那些方式和地址访问包的源代码
	dependencies：使用当前包所需要的以来的包列表，npm会根据这个属性帮助自动加载依赖的包
	homepage：当前包的网站地址
	os：操作系统支持列表，如果设置为空则没有任何假设
	engine：支持的javascript引擎列表
	builtin：标志当前包是否内建在底层系统的标准组件
	directories：包目录说明
	implements：实现规范的列表
	script：脚本说明对象，用来被包管理器用来安装编译测试和卸载包，
###包的常用功能###
####安装依赖包####
执行语句是 npm install express。
执行完毕npm将会在当前目录下创建node_modules目录 然后在该目录下创建express目录，接着将包解压到这个目录下。
然后，直接在代码中调用require('express')就可以引入这个包。
require()方法在做路径分析的时候会通过模块路径查找到express所在位置
###npm ls分析包 可以展示当前路径下可以引用的所有包###
###CommonJs总结###
规范简单，意义强大，node 通过模块规范，组织了自身的原生模块，弥补了javascript弱结构性的问题，形成了稳定的结构，并且向外提供服务
，让Node 有序发展着
###Nodejs 函数###
	function say(word) {
	  console.log(word);
	}
	
	function execute(someFunction, value) {
	  someFunction(value);
	}
	
	execute(say, "Hello");
	以上代码中，我们把 say 函数作为execute函数的第一个变量进行了传递。这里返回的不是 say 的返回值，而是 say 本身！
	这样一来， say 就变成了execute 中的本地变量 someFunction ，execute可以通过调用 someFunction() （带括号的形式）来使用 say 函数。
	当然，因为 say 有一个变量， execute 在调用 someFunction 时可以传递这样一个变量。
###匿名函数###
    我们可以把一个函数作为变量传递。但是我们不一定要绕这个"先定义，再传递"的圈子，我们可以直接在另一个函数的括号中定义和传递这个函数：
    function execute(someFunction, value) {
      someFunction(value);
    }
    
    execute(function(word){ console.log(word) }, "Hello");
    我们在 execute 接受第一个参数的地方直接定义了我们准备传递给 execute 的函数。
    用这种方式，我们甚至不用给这个函数起名字，这也是为什么它被叫做匿名函数 。
###node 路由###
> nodejs 路由，我们腰围路由提供请求的URL和其他需要GET以及POST参数
> 随后路由根据这些数据执行相应的代码
> 我们需要查看HTTP请求，从中提取出请求的URL以及GET/POST参数。这一功能我们暂定为HTTP的功能

> 我们所需要的数据都会包含在rquest对象当众，该对象当作
>  http.createServer里匿名函数的一个参数传递(作为onRequest()回调函数的第一个参数传递，但是为了解析这些数据，我们需要额外的nodeJS模块，它们分别是url和querystring模块)

querystring模块来解析POST请求体中的参数
现在我们先给onRequest()函数加上一些逻辑用来找出浏览器请求的URL路径
	var http = require("http");
	var url = require("url");
	
	function start() {
	  function onRequest(request, response) {
	    var pathname = url.parse(request.url).pathname;
	    console.log("Request for " + pathname + " received.");
	    response.writeHead(200, {"Content-Type": "text/plain"});
	    response.write("Hello World");
	    response.end();
	  }
	
	  http.createServer(onRequest).listen(8888);
	  console.log("Server has started.");
	}
	
	exports.start = start;

> 我们的应用现在可以通过请求的URL路径来区别不同请求了--这使我们得以使用路由（还未完成）来将请求以URL路径为基准映射到处理程序上。
> 在我们所要构建的应用中，这意味着来自/start和/upload的请求可以使用不同的代码来处理。稍后我们将看到这些内容是如何整合到一起的。
> 现在我们可以来编写路由了，建立一个名为router.js的文件，添加以下内容：
	
	function route(pathname) {
	  console.log("About to route a request for " + pathname);
	}
	
	exports.route = route;

> 如你所见，这段代码什么也没干，不过对于现在来说这是应该的。在添加更多的逻辑以前，我们先来看看如何把路由和服务器整合起来。
> 我们的服务器应当知道路由的存在并加以有效利用。我们当然可以通过硬编码的方式将这一依赖项绑定到服务器上，但是其它语言的编程经验告诉我们这会是一件非常痛苦的事，因此我们将使用依赖注入的方式较松散地添加路由模块。
> 首先，我们来扩展一下服务器的start()函数，以便将路由函数作为参数传递过去：
	
	var http = require("http");
	var url = require("url");
	
	function start(route) {
	  function onRequest(request, response) {
	    var pathname = url.parse(request.url).pathname;
	    console.log("Request for " + pathname + " received.");
	
	    route(pathname);
	
	    response.writeHead(200, {"Content-Type": "text/plain"});
	    response.write("Hello World");
	    response.end();
	  }
	
	  http.createServer(onRequest).listen(8888);
	  console.log("Server has started.");
	}
	
	exports.start = start;

同时，我们会相应扩展index.js，使得路由函数可以被注入到服务器中：

	var server = require("./server");
	var router = require("./router");
	
	server.start(router.route);
在这里，我们传递的函数依旧什么也没做。
如果现在启动应用（node index.js，始终记得这个命令行），随后请求一个URL，你将会看到应用输出相应的信息，这表明我们的HTTP服务器已经在使用路由模块了，并会将请求的路径传递给路由：

    bash$ node index.js
    Request for /foo received.
    About to route a request for /foo
##异步IO##
###[全局对象](http://www.w3cschool.cn/nodejs/nodejs-global-object.html)###