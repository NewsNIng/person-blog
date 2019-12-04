---
title: Taro 小程序实战技术摘要
categories: 技术
date: 2019-05-26 23:44:23
tags: 前端
---

使用 Taro 实际小程序项目开发中，分享一些心得和项目的构成。

<!-- more -->

### React
一个用于构建用户界面的 JAVASCRIPT 库。采用声明范式，可以轻松描述应用。通过对DOM的模拟，最大限度地减少与DOM的交互。组件复用，能够很好的应用在大项目的开发中。单向响应的数据流，使得开发变得清晰。
### Taro
Taro 是一套遵循 React 语法规范的 多端开发 解决方案。现如今市面上端的形态多种多样，Web、React-Native、微信小程序等各种端大行其道，当业务要求同时在不同的端都要求有所表现的时候，针对不同的端去编写多套代码的成本显然非常高，这时候只编写一套代码就能够适配到多端的能力就显得极为需要。

使用 Taro，我们可以只书写一套代码，再通过 Taro 的编译工具，将源代码分别编译出可以在不同端（微信/百度/支付宝/字节跳动小程序、H5、React-Native 等）运行的代码。

### TypeScript
TypeScript 是 JavaScript 的一个超集，主要提供了类型系统和对 ES6 的支持，它由 Microsoft 开发，代码开源于 GitHub 上。

### GitFlow
git-flow 是一个 git 扩展集，按 Vincent Driessen 的分支模型提供高层次的库操作
![](https://axd-doc.oss-cn-hangzhou.aliyuncs.com/share/20190106140713.png)
### VsCode + 插件
Visual Studio Code（简称VsCode）是一个轻量且强大的代码编辑器，支持Windows，OS X和Linux。内置JavaScript、TypeScript和Node.js支持，而且拥有丰富的插件生态系统，可通过安装插件来支持C++、C#、Python、PHP等其他语言。以及更多丰富便捷的插件库。
![](https://axd-doc.oss-cn-hangzhou.aliyuncs.com/share/20190106142936.png)
### 代码检查
代码检查主要是用来发现代码错误、统一代码风格。

在 JavaScript 项目中，我们一般使用 ESLint 来进行代码检查。它通过插件化的特性极大的丰富了适用范围，搭配 typescript-eslint-parser 之后，甚至可以用来检查 TypeScript 代码。

TSLint 与 ESLint 类似，不过除了能检查常规的 js 代码风格之外，TSLint 还能够通过 TypeScript 的语法解析，利用类型系统做一些 ESLint 做不到的检查。

#### 为什么需要代码检查
有人会觉得，JavaScript 非常灵活，所以需要代码检查。而 TypeScript 已经能够在编译阶段检查出很多问题了，为什么还需要代码检查呢？

因为 TypeScript 关注的重心是类型的匹配，而不是代码风格。当团队的人员越来越多时，同样的逻辑不同的人写出来可能会有很大的区别：

- 缩进应该是四个空格还是两个空格？
- 是否应该禁用 var？
- 接口名是否应该以 I 开头？
- 是否应该强制使用 === 而不是 ==？

这些问题 TypeScript 不会关注，但是却影响到多人协作开发时的效率、代码的可理解性以及可维护性。

下面来看一个具体的例子：

```typescript
let myName = 'Tom';

console.log(`My name is ${myNane}`);
console.log(`My name is ${myName.toStrng()}`);
console.log(`My name is ${myName}`)

// tsc 报错信息：
//
// index.ts(3,27): error TS2552: Cannot find name 'myNane'. Did you mean 'myName'?
// index.ts(4,34): error TS2551: Property 'toStrng' does not exist on type 'string'. Did you mean 'toString'?
//
//
//
// eslint 报错信息：
//
// /path/to/index.ts
//   3:27  error  'myNane' is not defined         no-undef
//   5:38  error  Missing semicolon               semi
//
// ✖ 2 problems (2 errors, 0 warnings)
//   1 errors, 0 warnings potentially fixable with the `--fix` option.
//
//
//
// tslint 报错信息：
//
// ERROR: /path/to/index.ts[5, 36]: Missing semicolon
```
![](https://axd-doc.oss-cn-hangzhou.aliyuncs.com/share/20190106141452.png)

下图表示了 tsc, eslint 和 tslint 能覆盖的检查：
![](https://axd-doc.oss-cn-hangzhou.aliyuncs.com/share/20190106141722.png)

上图中，**tsc**, **eslint** 和 **tslint** 之间互相都有重叠的部分，也有各自独立的部分。

虽然发现代码错误比统一的代码风格更重要，但是**当一个项目越来越庞大，开发人员也越来越多的时候，代码风格的约束还是必不可少的**。

### UI设计协作
蓝湖是一款设计图共享平台,帮助互联网团队管理设计图。蓝湖可以自动生成标注,与团队共享设计图,展示页面之间的跳转关系。支持从Sketch一键分享、在线讨论、自动为设计图生成标注，而且只需简单几步就能将设计图变成一个可以点击的演示原型，支持分享给同事，让他也可以在手机中查看设计效果。蓝湖已经成为新一代产品设计的工作方式。

![](https://axd-doc.oss-cn-hangzhou.aliyuncs.com/share/20190106142142.png)

直接在线编辑和预览的方式，让产品与设计师的交流反馈得到极大的提升，同时开发者可以得到详细的标注，以及不同平台的支持提示，使得开发效率变快。

![](https://axd-doc.oss-cn-hangzhou.aliyuncs.com/share/20190106142615.png)



### 框架搭建

##### 全局配置文件
**（ps: 截图中的秘钥配置和配置都是测试，大家看一下大概配置就好）**
通过 Config 文件，区分开发、生产环境，动态的编译代码后，使用相应正确的配置，还有小程序不同的标识，同时保存了微信、支付宝、神策等第三方平台的密钥。统一的配置文件能让后期更好维护。
![](https://axd-doc.oss-cn-hangzhou.aliyuncs.com/share/20190106143222.png)

##### 通过Swagger文档生成API文件
在需求评审后，前后端人员将会在一起制定接口结构，这点体现在Swagger文档之中，随后前端使用脚手架工具把Swagger文档yaml文件生成可调用的fetch请求文件集成到项目，避免了手动编写接口代码的错误风险，这点前提是对Swagger有严格的要求，基于接口规范编码。

1. 放入Swagger文件

![](https://axd-doc.oss-cn-hangzhou.aliyuncs.com/share/20190106145120.png)

2. 执行转化命令
```shell
sudo api-cli MINI js
```
3. 得到api请求配置文件  

![](https://axd-doc.oss-cn-hangzhou.aliyuncs.com/share/222222.png)

4. 代码中使用

![](https://axd-doc.oss-cn-hangzhou.aliyuncs.com/share/20190106145633.png)


##### 开发适配
在自动生成api请求文件后，需要更改一些配置才能在于小程序之中运行,所以我们做了以下适配：
- 小程序请求
    - Taro.request 适配 微信、支付宝小程序请求
- 全局请求头添加
    - userToken 用来识别用户身份信息
    - branch 后端服务路径分支名称
    - clientType 小程序平台名称
    - version 当前程序版本
- 测试调试工具
    - baseURL 更改全局请求地址
    - branch 更改请求头分支名
    - clear 清除缓存数据

以上适配让小程序的可拓展、可维护性得到大大提高。

##### 资源优化
由于官方的限制，一个小程序的代码包括资源文件大小不能超过2M，在真实迭代情况下，随着业务的增加，功能的改变，2M大小对我们来说可能很快就要超出，我们采用以下方案：

1. 代码分包  

此方式是官方推荐的方法，通过把不同业务代码资源分离出来，主要的功能先行下载运行，如：主页商铺列表、商铺详情、订单列表等，而相对低频的功能可以后续异步下载运行，如：退款列表/详情、会员权益说明等，但是官方还是限制了最多为4个分离包，每个大小限制也为2M。

2. 图片资源远端保存  

这是一种常用的包大小减小体积的方案，超过30k大小的文件放入阿里云OSS资源仓库保存，代码中使用远程路径方式引入，但同时要考虑首屏渲染的平衡

3. 代码压缩  

代码运行终究是机器做的事，只是顺便让我们程序员看看，所以在我们编译代码的同时对其js、css等文件内容压缩，去除运行中不必要的注释、打印、多余空格等，同时用编译环境控制是否开启压缩功能，这样平时开发调试的时候可以关闭压缩功能，更好的进行调试、测试、排错。

##### 线上代码报错日志 fundebug
在真实线上的环境，难免会遇到很隐性的BUG，通过集成fundebug的方式。来检测到线上程序报错的信息。  

fundebug 可以快速复现出错场景，记录出错前点击、页面跳转、网络请求，控制台打印等信息。通过 Source Map 还原生产环境中的压缩代码，提供完整的堆栈信息，准确定位到出错源码，帮助我们快速修复Bug。
![](https://axd-doc.oss-cn-hangzhou.aliyuncs.com/share/20190106153110.png)  

一键还原出错代码  

![](https://axd-doc.oss-cn-hangzhou.aliyuncs.com/share/20190106153156.png)

##### 数据采集

Sensors（神策分析）针对企业级客户的自助式用户行为分析平台。实时数据采集、建模、分析，驱动市场营销、产品优化、用户运营、管理监控。

![](https://axd-doc.oss-cn-hangzhou.aliyuncs.com/share/20190106153329.png)

神策官方提供了小程序的SDK，但是我们需要配合产品提供的事件和逻辑，自己编写埋点。

- 埋点事件模型和抽象类
```typescript

// 点击优惠券列表
export interface IClickCouponMerchantList extends IClickCoupon {}

// 点击兑换优惠券
export interface IClickToExChange {}

// 使用商品
export interface IConsume extends IOrder, IMerchant, ICommodity {}

// 点击小程序首页banner
export interface IClickMiniBanner {}

// ... 

export interface IStatisEvents {
  ClickCouponMerchantList?: (params: IClickCouponMerchantList) => any
  ClickToExChange?: (params: IClickToExChange) => any
  Consume?: (params: IConsume) => any
  ClickMiniBanner?: (params: IClickMiniBanner) => any
  // ...
}
```

- 数据采集封装
```typescript

interface IStatis extends IStatisEvents {
  sensors: any;
  init: () => any;
  defaultTrack: (eventName: string, eventParams: any) => any;
}
const loadStatisEvent = (statis: Statis): IStatis => {
  return new Proxy(statis, {
    get(obj, prop) {
      const target = obj[prop];
      if (obj.hasOwnProperty(prop)) {
        return target;
      }
      if (!SaConfig.enable) {
        return noop;
      }
      if (typeof target === "function") {
        return target.bind(obj);
      }
      // 默认采集
      return obj.defaultTrack.bind(obj, prop);
    }
  });
};

class Statis implements IStatis {
  public sensors: any;

  constructor(sensors) {
    this.sensors = sensors;
  }

  init() {
    this.sensors.init();
  }

  defaultTrack(eventName: string, eventParams: any) {
    this.sensors.track(eventName, eventParams);
  }

  Login(id: string) {
    this.sensors.login(id);
  }

  RegisterApp(params: IRegisterApp) {
    const param = Object.assign(
      {
        platform: AppConfig.CLENT_TYPE
      },
      params
    );
    this.sensors.registerApp(param);
  }
}

const sa = loadStatisEvent(new Statis(sensors_));
```
- 事件采集

![](https://axd-doc.oss-cn-hangzhou.aliyuncs.com/share/20190106154340.png)


### 结语

微信/支付宝 双端发布，虽说是一套代码，但是做了满多兼容，主要在两端授权用户信息不太一样，需要封装统一，再一个是样式写的不规范，两端表现的也不一致。  
swagger API文件生成 可调用的 fetch 文件、统一请求封装和自定义请求头、处理多端授权用户信息等，还有很多技术和流程的细节，容我后续再跟大家慢慢讲解。