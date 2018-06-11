#  深入浅出 axios 源码

## 前言

axios 是目前最常用的 http 请求库，可以用于浏览器和 node.js , 在 github 上已有 43k 左右之多。

Axios 的主要特性包括：

- 基于 Promise
- 支持浏览器和 node.js
- 可添加拦截器和转换请求和响应数据
- 请求可以取消
- 自动转换 JSON 数据
- 客户端支持防范 XSRF
- 支持各主流浏览器及 IE8+

本文  将  带大家一起阅读 axios 的源码， 解析当中的一些封装技巧、具体的功能实现、以及阅读源码的一些思路。

## 环境搭建

阅读源码并不是只是一味单纯的‘读’，很多时候面对复杂的前后文依赖关系以及输入输出， 经常会如同乱麻一般理不出思绪。这时候往往加上一些 log 远胜于凭空的想象。而且可以忽略一些不那么重要的环节，使得更容易抓住主干。

工欲善其事必先利器，我们需要打造一个 playground ，用来 watch 变化 观察我们的 log 输出。在 axios 中已经包含了  一些 examples ，但是项目构建工具基于 grunt 并  没有 watch 变化，我们需要自己去添加。
在 `./GruntFile.js` 中添加`grunt.loadNpmTasks('grunt-contrib-watch');` 即可。 或者我们可以使用  构建工具来做到。然后启动 examples 服务器和 watch 构建任务即可。

```bash
  npm run examples
  npm run dev

  # open localhost:3000
```

## 项目目录结构

```
├── /lib/                      # 项目源码目
│ ├── /adapters/               # 定义发送请求的适配器
│ │ ├── http.js                # node环境http对象
│ │ └── xhr.js                 # 浏览器环境XML对象
│ ├── /cancel/                 # 定义取消功能
│ ├── /helpers/                # 一些辅助方法
│ ├── /core/                   # 一些核心功能
│ │ ├── Axios.js               # axios实例构造函数
│ │ ├── createError.js         # 抛出错误
│ │ ├── dispatchRequest.js     # 用来调用http请求适配器方法发送请求
│ │ ├── InterceptorManager.js  # 拦截器管理器
│ │ ├── mergeConfig.js         # 合并参数
│ │ ├── settle.js              # 根据http响应状态，改变Promise的状态
│ │ └── transformData.js       # 改变数据格式
│ ├── axios.js                 # 入口，创建构造函数
│ ├── defaults.js              # 默认配置
│ └── utils.js                 # 公用工具
```

## 从 API 入手

分析源码的时候，我们需要先要从 API 入手，尝试着猜想下内部的结构、带着问题再去  看源码会更加有效。
我们来大致把 axios 的 API  进行  归纳分类：

| API                                                                      | 类型                                               |
| ------------------------------------------------------------------------ | -------------------------------------------------- |
| axios(config)                                                            | 发送请求                                           |
| axios.create(config)                                                     | 创建新的请求实例，独立的上下文环境,实例继承请      |
| axios.request(get post delete...) / instance.request(get post delete...) | axios 以及  创建实例的请求别名(包含多种 http 方法) |
| axios.default / instance.default                                         | axios 以及  创建实例的  默认 config 设置           |
| axios.interceptors / instance                                            | axios 以及  创建实例拦截器的设置                   |
| axios.all() / axios.spread                                               | 处理并行请求的静态方法                             |
| axios.Cancel() / axios.CancelToken() /axios.isCancel()                   | 取消请求相关方法                                   |

以上大致就是 axios 中 api 的大致分类。我们可以看出  暴露给我们的 axios 对象下挂载的除了 all 和 spread 以及取消请求的方法以外都被创建的实例给继承，可以单独使用并保持自己的上下文环境。让我们从入口开始看看是怎么实现的:

```javascript
"use strict";

var utils = require("./utils");
var bind = require("./helpers/bind");
var Axios = require("./core/Axios");
var mergeConfig = require("./core/mergeConfig");
var defaults = require("./defaults");

/**
 * 创建Axios实例
 */
function createInstance(defaultConfig) {
  // new Axios 得到一个上下文环境 包含defatults配置以及拦截器
  var context = new Axios(defaultConfig);

  // instance实例为bind返回的一个函数(即是request发送请求方法)，此时this绑定到context上下文环境
  var instance = bind(Axios.prototype.request, context);
  // 将Axios构造函数中的原型方法绑定到instance上并且指定this作用域为context上下文环境
  utils.extend(instance, Axios.prototype, context);
  // 把上下文环境中的defaults 以及拦截器绑定到instance实例中
  utils.extend(instance, context);

  return instance;
}

// axios入口其实就是一个创建好的实例
var axios = createInstance(defaults);
// 这句没太理解，根据作者的注释是：暴露Axios类去让类去继承
axios.Axios = Axios;

// 工厂函数 根据配置创建新的实例
axios.create = function create(instanceConfig) {
  return createInstance(mergeConfig(axios.defaults, instanceConfig));
};

// 绑定取消请求相关方法到入口对象
axios.Cancel = require("./cancel/Cancel");
axios.CancelToken = require("./cancel/CancelToken");
axios.isCancel = require("./cancel/isCancel");

// all 和 spread 两个处理并行的静态方法
axios.all = function all(promises) {
  return Promise.all(promises);
};
axios.spread = require("./helpers/spread");

module.exports = axios;

// 允许使用Ts 中的 default import 语法
module.exports.default = axios;
```

通过以上入口方面分析我们可以看出端倪, axios 入口其实就是通过 createInstance 创建出的实例和 axios.create() 创建出的实例一样。而  源码入口中的重中之中就是 createInstance 这个方法。createInstance 流程大致为:

1.  使用 Axios 函数创建上下文 context ，包含自己的 defaults config 和 管理拦截器的数组
2.  利用 Axios.prototype.request 和 上下文 创建实例 instance，实例为一个 request 发送请求的函数 this 指向上下文 context
3.  绑定 Axios.prototype 的其他方法到  instance 实例，this 指向上下文 context
4.  把上下文 context 中的 defaults 和拦截器绑定到 instance 实例

## 请求别名

在 axios 中 axios.get 、axios.delete 、axios.head 等别名请求方法其实都是指向同一个方法 axios.request 只是把 default config 中的 请求 methods 进行了修改而已。 具体代码在 Axios  这个构造函数的原型上，让我们来看下源码的实现：

```javascript
utils.forEach(
  ["delete", "get", "head", "options"],
  function forEachMethodNoData(method) {
    Axios.prototype[method] = function(url, config) {
      return this.request(
        utils.merge(config || {}, {
          method: method,
          url: url
        })
      );
    };
  }
);

utils.forEach(["post", "put", "patch"], function forEachMethodWithData(method) {
  Axios.prototype[method] = function(url, data, config) {
    return this.request(
      utils.merge(config || {}, {
        method: method,
        url: url,
        data: data
      })
    );
  };
});
```

因为 post 、 put 、 patch 有请求体,所以要  分开进行处理。请求别名方便用户快速使用各种不同 API 进行请求。

## 拦截器的实现

首先在我们创建实例中，会去创建上下文实例 也就是 new Axios ,会得到 interceptors 这个属性,这个属性分别又有 request 和 responese 两个属性 ， 它们的值分别是 new InterceptorManager 构造函数返回的数组。这个构造函数同样负责拦截器数组的添加和移除。让我们看下源码:

```javascript
"use strict";

var utils = require("./../utils");

function InterceptorManager() {
  this.handlers = [];
}

// axio或实例上调用 interceptors.request.use 或者 interceptors.resopnse.use
// 传入的resolve, reject 将被添加入数组尾部
InterceptorManager.prototype.use = function use(fulfilled, rejected) {
  this.handlers.push({
    fulfilled: fulfilled,
    rejected: rejected
  });
  return this.handlers.length - 1;
};
// 移除拦截器，将该项在数组中置成null
InterceptorManager.prototype.eject = function eject(id) {
  if (this.handlers[id]) {
    this.handlers[id] = null;
  }
};

// 辅助方法，帮助便利拦截器数组，跳过被eject置成null的项
InterceptorManager.prototype.forEach = function forEach(fn) {
  utils.forEach(this.handlers, function forEachHandler(h) {
    if (h !== null) {
      fn(h);
    }
  });
};

module.exports = InterceptorManager;
```

上下文环境有了拦截器的数组， 又如何去  做到多个拦截器请求到响应的顺序处理以及实现呢？为了了解这点我们还需要进一步往下看 Axios.protoType.request 方法。
![拦截器请求链](https://user-images.githubusercontent.com/16076364/41226155-3ff10a60-6da3-11e8-99ce-3187adf41062.png)

## Axios.protoType.request

Axios.protoType.request 方法是请求开始的入口，分别处理了请求的 config，以及链式处理请求拦截器 、请求、响应拦截器,并返回 Proimse 的格式方便我们处理回调。让我们来看下源码部分：

```javascript
Axios.prototype.request = function request(config) {
  //判断参数类型，支持axios('url',{})以及axios(config)两种形式
  if (typeof config === "string") {
    config = arguments[1] || {};
    config.url = arguments[0];
  } else {
    config = config || {};
  }
  //传入参数与axios或实例下的defaults属性合并
  config = mergeConfig(this.defaults, config);
  config.method = config.method ? config.method.toLowerCase() : "get";

  // 创造一个请求序列数组，第一位是发送请求的方法,第二位是空
  var chain = [dispatchRequest, undefined];
  var promise = Promise.resolve(config);
  //把实例中的拦请求截器数组依从加入头部
  this.interceptors.request.forEach(function unshiftRequestInterceptors(
    interceptor
  ) {
    chain.unshift(interceptor.fulfilled, interceptor.rejected);
  });
  //把实例中的拦截器数组依从加入尾部
  this.interceptors.response.forEach(function pushResponseInterceptors(
    interceptor
  ) {
    chain.push(interceptor.fulfilled, interceptor.rejected);
  });
  //遍历请求序列数组形成prmise链依次处理并且处理完弹出请求序列数组
  while (chain.length) {
    promise = promise.then(chain.shift(), chain.shift());
  }
  //返回最终promise对象
  return promise;
};
```

我们可以看到 Axios.protoType.request 中使用了精妙的封装方法，形成 promise 链 去依次挂载在 then 方法顺序处理。为了更清晰的认识我们可以画个图去方便认识这一过程。

## 取消请求

Axios.prototype.request 调用 dispatchRequest 是最终处理 axios 发起请求的函数，他的执行  过程流程包括了：

1.  取消请求的处理和判断
2.  处理  参数和默认参数
3.  使用相对应的环境 adapter 发送请求(浏览器环境使用 XMLRequest 对象、Node 使用 http 对象)
4.  返回后抛出取消请求 message，根据配置 transformData 转换  响应数据

这一过程除了取消请求的处理， 其余的流程都相对十分的简单，所以我们要对取消请求进行详细的分析。我们还是先看调用方式:

```javascript
const CancelToken = axios.CancelToken;
const source = CancelToken.source();

axios
  .get("/user/12345", {
    cancelToken: source.token
  })
  .catch(function(thrown) {
    if (axios.isCancel(thrown)) {
      console.log("Request canceled", thrown.message);
    } else {
      // handle error
    }
  });

source.cancel("Operation canceled by the user.");
```

从调用方式我们可以看到，我们需要从 config 传入 axios.CancelToken.source().token ,  并且可以用 axios.CancelToken.source().cancel() 执行取消请求。我们还可以从  看出 canel 函数不仅是取消了请求，并且  使得整个请求走入了 rejected 。从整个 API 设计我们就可以看出这块的  功能可能有点复杂， 让我们一点点来分析,从 CancelToken.source 这个方法看实现过程 ：

```javascript
CancelToken.source = function source() {
  var cancel;
  var token = new CancelToken(function executor(c) {
    cancel = c;
  });
  return {
    token: token,
    cancel: cancel
  };
};
```

axios.CancelToken.source().token 返回的是一个 new CancelToken 的实例，axios.CancelToken.source().cancel,是 new CancelToken 是传入 new CancelToken 中的方法的一个参数。再看下 CancelToken 这个构造函数：

```javascript
function CancelToken(executor) {
  if (typeof executor !== "function") {
    throw new TypeError("executor must be a function.");
  }

  var resolvePromise;
  this.promise = new Promise(function promiseExecutor(resolve) {
    resolvePromise = resolve;
  });

  var token = this;
  executor(function cancel(message) {
    if (token.reason) {
      return;
    }

    token.reason = new Cancel(message);
    resolvePromise(token.reason);
  });
}
```

我们根据构造函数可以知道 axios.CancelToken.source().token 最终拿到的实例下挂载了 promise 和 reason 两个属性，promise 属性是一个处于 pending 状态的 promise 实例，reason 是执行 cancel 方法后传入的 message。而 axios.CancelToken.source().cancel 是一个函数方法，负责判断是否执行，若未执行拿到 axios.CancelToken.source().token.promise 中 executor 的 resolve 参数，作为触发器，触发处于处于 pending 状态中的 promise 并且  传入的 message 挂载在 xios.CancelToken.source().token.reason 下。若有  已经挂载在 reason 下则返回防止反复触发。而这个 pending 状态的 promise 在 cancel 后又是怎么进入 axios 总体 promise 的 rejected 中呢。我们需要看看 adpater 中的处理：

```javascript
//如果有cancelToken
if (config.cancelToken) {
  config.cancelToken.promise.then(function onCanceled(cancel) {
    if (!request) {
      return;
    }
    //取消请求
    request.abort();
    //axios的promise进入rejected
    reject(cancel);
    // 清楚request请求对象
    request = null;
  });
}
```

取消请求的总体逻辑大体如此，可能理解起来比较困难，需要反复看源码感受内部的流程，让我们大致在屡一下大致流程：

1.  axios.CancelToken.source()返回一个对象，tokens 属性 CancelToken 类的实例，cancel 是 tokens 内部 promise 的 resove 触发器
2.  axios 的 config 接受了 CancelToken 类的实例
3.  当 cancel 触发处于 pending 中的 tokens.promise，取消请求，把 axios 的 promise 走向 rejected 状态

## 总结

axios 的大体流程  如上述般大体介绍完了，我们可以画个图更加直观的梳理一下

![拦截器请求链](https://user-images.githubusercontent.com/16076364/41226163-48a2b122-6da3-11e8-999d-98fa65f63df5.png)

Axios 的源码分析就到这里,如果有错请多多指教
