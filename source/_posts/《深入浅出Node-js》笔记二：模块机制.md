---
title: 《深入浅出Node.js》笔记二：模块机制
categories: 技术
date: 2019-12-15 15:02:38
tags: Node
---

Node的模块机制，从中可以了解Node是如何实现CommonJS模块和包规范。详细解释了模块在饮用过程中的编译、加载规则。另外，还能读到更深度的关于Node自身源代码的组织架构。

<!-- more -->


#### Node模块机制

JavaScript先天就缺乏的一项功能：模块。在其他高级语言中，Java有类文件，Python有import机制，Ruby有require，PHP有include和require。而JavaScript使用*script*标签引入代码的方式显得杂乱无章，语言自身毫无组织和约束能力。

但是经历过十多年的发展，社区也为JavaScript制定了相应的规范，其中CommonJS规范的提出算是最为重要的里程碑。

#### CommonJS规范

对于JavaScript自身而言，它的规范依然是薄弱的，还有以下缺陷：

- 没有模块系统
- 标准版较少。（ECMAScript仅定义了部分核心库，对于文件系统，I/O流等常见API没有标准。W3C标准在推进这个过程，但仅限于浏览器端。）
- 没有标准接口。（几乎没有定义过Web服务器或者数据库之类的标准统一接口。）
- 缺乏包管理机制。（导致应用中基本没有自动加载和安装依赖的能力。）

CommonJS的提出，JavaScript多出了很多应用场景：

- 服务端JavaScript应用程序。
- 命令行工具。
- 桌面图形界面应用程序。
- 混合应用。

---

Node与浏览器以及W3C组织、CommonJS组织、ECMAScript之间的关系，共同构成了一个繁荣的生态系统。


![Nodejs](/img/深入浅出Nodejs/017E63F2-9A64-4676-BA46-986C6E68A19C.png)


- ECMAScript：制定 JavaScript 语法标准。
- W3C：制定 BOM、DOM 标准。
- CommonJS：制定 FS、TCP、Buffer、Stream 等标准。
- 浏览器：实现 ECMAScript、W3C 标准。
- Node：实现 ECMAScript、CommonJS 标准。

--- 

CommonJS对模块的定义十分简单，主要分 **模块引用**、**模块定义** 和 **模块标识** 3个部分。

1. **模块引用**： 通过require引入一个模块的API到上下文中。
2. **模块定义**：在模块上下文中，提供exports对象用于导出当前模块方法或变量，同时module对象代表模块自身，exprots是module的属性。
3. **模块标识**：实际为require的参数，必须是小写驼峰字符串，或者.、..开头的相对路径，/开头的绝对路径。

CommonJS这套导入导出机制让用户完全不必考虑变量污染，相比命名空间等方案与之相比相形见拙。

#### Node的模块实现

Node并未完全按照规范实现，而是做了一定的取舍，也加了不少自身需要的特性。

在Node引入模块，需要经历以下3个步骤：

1. **路径分析**
2. **文件定位**
3. **编译执行**

模块分为两类，一类是Node提供的模块，称为**核心模块**；另一类是用户编写的模块，成为**文件模块**。

- **核心模块**：部分核心模块在Node源代码编译过程中，编译进了二进制执行文件。在Node启动时，部分核心模块就直接加载进了内存，直接**省略了文件定位、编译执行**两个步骤，并且在require时优先判断，所以它的加载速度是最快的。
- **文件模块**：运行时动态加载，需要**完整的 路径分析、文件定位、编译执行**3个步骤，速度比核心模块慢。

##### 优先从缓存加载

像前端浏览器一样，Node对引入的模块都会进行缓存，以减少二次引入的开销。不同在于浏览器仅仅缓存文件、而**Node缓存的是编译和执行之后的对象**。

不论是核心模块还是文件模块，require方法对相同的模块二次加载都一律采用缓存优先的方式。**核心模块的缓存检查先于文件模块的缓存检查**。

##### 路径分析和文件定位

模块标识符分析主要分为以下几类：

- 核心模块，如http、fs、path等。
- .或者..开始的相对文件模块。
- /开头的绝对路径文件模块。
- 非路径形式的文件模块，如require('express')

以下从 加载速度 快->慢 一一介绍。

> 核心模块 例子：require('http')

**核心模块** 的优先级仅次于缓存加载，在Node编译过程中已经是可执行的二进制代码，其加载速度最快。如果试图加载一个与核心模块相同的自定义模块标识符，比如你发布一个npm包，名为 http ，当引入require('http')时，Node还是加载内置的核心模块。

> 路径文件模块 例子：require('../common/xx.js'')  require('/src/xx.js'')

**路径形式的文件模块**，.、..、/开头在引入时会转换为真实的路径，比如 User/xxx/project/src/common/xx.js ，编译执行后结果放入缓存中，使得二次加载更快。

> 自定义模块 例子：require('express')

**自定义模块** 是一种特殊的文件模块，可能是一个文件（当前文件夹下的express.js文件），也可能是一个包（node_modules/express）。所以查找这类模块是最费时的。

*自定义模块会向上级目录逐层在node_modules目录查找，当文件路径越深，加载速度会越慢。*

---

文件定位有一些细节需要注意：

> 假设你这样引入 require('./common/xx')

1. **Node依次按照拓展名.js .json .node尝试分析你不带拓展名的引入**，本质上会通过fs模块同步阻塞判断文件是否存在，所以**小诀窍**是：如果是.json和.node文件，请补齐拓展名。


2. **拓展名分析却还没找到文件，Node则还认为有可能你的引入是一个包**，如果你xx是一个目录，由于CommonJS规范，**Node会先定位你xx目录下的package.json文件，使用JSON.parse解析内容，找到main属性指定的路径作为入口**，当然不写拓展名它还是会自动分析，再假设你**main属性文件不存在或者package.json文件不存在，则Node会去xx目录下依次寻找 index.js index.json index.node文件**。到这步还找不到入口文件，OK，向上逐层寻找，到根目录再找不到就抛错咯。

![Nodejs](/img/深入浅出Nodejs/03957F11-1D25-4C72-8B60-49471281E084.png)

对于不同的文件，其载入方式也不同：

- **.js文件**：通过fs模块同步获取读取文件后编译执行。
- **.node文件**：这是通过C/C++编写的拓展文件，通过dlopen()方法加载最后编译生成的文件。
- **.json文件**：通过fs模块同步读取文件后，用JSON.parse()解析返回结果。
- **其余拓展文件**：它们都当作.js文件载入。

如果想对自定的拓展名进行特殊的加载，可以通过 require.extensions['.ext']的方式实现。

每一个编译成功的模块都会将其文件路径作为索引缓存存在Module_cache对象上，以提高二次引入的性能。

其中一个正常的JavaScript文件会被包装成如下：

```javascript
(function(exports, require, module, __filename, __dirname){
    var math = require('math');
    exports.area = function(radius){
        return Math.PI * radius * radius;
    };
});
```

这样每个模块文件之间都进行了作用域隔离。

#### 包与 NPM

Node模块规范的实现，一定程度的解决了变量依赖、依赖关系等代码组织性问题。包的出现，则是在模块的基础上进一步组织JavaScript代码。

##### 包结构

包实际上为一个完全符合CommonJS规范的目录，应该包含以下文件：

- package.json： 包描述文件。
- bin：用于存放可执行二进制文件的目录。
- lib：用于存放JavaScript代码的目录。
- doc：用于存放文档的目录。
- test：用于存放单元测试用力的代码目录。

##### 包常用功能

> npm help <command>

可以查看到命令具体帮助

**安装依赖包**，npm install express，执行命令后，NPM在当前目录下创建node_modules目录，然后在node_modules目录下创建express目录，接着将包解压到这个目录下。

安装完后，require('express') 通过路径分析找到express所在位置。模块引入和包的安装这两个步骤是相辅相成的。

**全局安装**，其实，-g 是将一个包安装为全局可用的可执行命令。根据包描述文件的bin属性，将实际脚本链接到与Node可执行文件相同的路径下：

```javascript
    "bin": {
        "express": "./bin/express"
    }
```

**其它源安装**，可以从本地安装：它可以是一个包含package.json文件夹，也可以是一个URL地址：

> npm install <traball file>
> npm install <tarball url>
> npm ionstall <folder>

如果不能从官方源安装，可以指定安装源

> npm install underscore --registry=http://registry.url

以上是单次安装指定安装源，你也可以指定全局默认安装源：

> npm config set registry http://registru.url

**NPM钩子命令**，你可能在某一些包描述中看到这样的属性：

```javascript
    "scripts": {
        "preinstall": "preinstall.js",
        "install": "install.js",
        "uninstall": "uninstall.js",
        "test": "test.js",
    }
```

这是包在安装或者卸载等过程中提供钩子机制，比如C/C++模块实际上要编译后才能使用，安装包前先执行preinstall钩子编译。
npm install <package>，preinstall 脚本将会加载运行，然后install脚本执行。在npm uninstall <package>时，uninstall脚本将会执行，比如一些清理工作等。


