
## 前端架构的前世今生

### 架构是如何产生的

初始无架构：

* 前端代码嵌入到后端代码中

后端mvc架构：

* 将视图层、数据层、控制层分离

* 缺点：重度依赖开发环境，代码混淆严重

前后端分离架构：

* 将前后端代码从后端环境中提炼出来（ajax促进了前后端分类架构的发展）多页面架构

* 前端缺乏独立部署能力，整体流程依赖后端环境的部署



node.js的广泛使用促进了前端技术的飞速发展

* 各种打包工具、构建工具营运而生

* 诞生了多元化前端开发方式，使得前端开发可以脱离后端环境



单页面架构

打包: gulp、roullup、webpack、vite

框架：vue/react/angular

ui库：antd/iview/material

优势：

* 切换页面无需刷新浏览器，用户体验过号
* 组件化开发方式，极大提升了代码复用率

劣势：

* 不利于seo，首次渲染会出现较长的白屏（可解决）

大前端时代

* 后端框架：express、koa
* 包管理工具：npm、yarn
* node版本管理：nvm



总结

* 过于灵活的实现也导致了前端应用拆分过多，维护困难

* 往往一个功能需求会跨两三个项目进行开发

### 微型前端架构

优势

* 技术栈无关
* 主框架不限制接入应用的技术栈，微应用具备完全自主权
* 独立部开发、独立部署
* 增量升级
* 是一种非常好的实施渐进式重构的手段和策略
* 微应用仓库独立，前后端可独立开发，主框架自动完成同步更新
* 独立运行时

劣势

* 接入难度高
* 应用场景-移动端少、管理端多



## 软件设计原则与分层

* 单一职责原则
  * 永远不应该有多个原因修改同一个
* 开放封闭原则
* 里式替换原则
  * 父类一定能够被子类替换

       * 最少知识原则
       * 。。。

## 选择架构

架构师分类

* 系统架构
  * 从系统的角度，负责整体系统的架构设计
  * 主要是基础服务和各系统间协调，着眼全局
  * 比如关注负载，可靠性，伸缩，扩展，整体项目拆分，缓存应用等方面的基础架构设计

* 应用架构
  * 从应用程度的维度，负责某个应用的技术架构，主要片业务系统。
  * 关注理解业务，梳理模型，设计模式，接口，数据交互等方面。

* 业务架构
  * 从业务流程的维度，关注某个行业、业务的领域分析，获取领域模型，最终获得系统的模型。
  * 也可以叫做业务领域专家、行业专家、产品咨询师、资深顾问。

技术前期准备

* 技术选型：社区氛围、发展规模、未来发展趋势、与当前团队的契合度、执行成本、维护和迁移成本、执行效率等内容的调研和报告。
* 充分调研每一项技术可能带来的利弊
* 最大程度上预测架构设计中的缺陷，以防止问题的发生

技术优化

* 在架构发展过程中，可能会存在一些有悖与当前机构设计的实现，造成了架构发展阻塞，所以需要进行架构优化，使架构设计的适应性更高。

架构优化

* 架构不是一蹴而就的，在业务发展过程中，架构也在不断演进。
* 对架构设计进行实时调优，是架构优化成为常态。
* 通过不断的调整架构实现，改进初始架构中设计的不足，补足短板。

## 微前端实现方式

1. iframe

   优势

   * 技术成熟
   * 支持页面嵌入
   * 天然支持运行筛箱隔离、独立运行

   劣势

   * 页面之间可以是不同的域名，
   * 需要对应用的设计一套应用通讯机制，如何监听、传参格式等内容
   * 应用加载、渲染、缓存等体系的实现。

2. web component

   优势

   * 支持自定义元素
   * 支持shadow dom，等通过关联进行控制
   * 支持模板tmplate和插槽slot，引入自定义组件

   劣势

   * 接入微前端需要重写当前项目
   * 生态系统不完善，技术过新容易出现兼容性问题
   * 整体架构设计复杂，组件与组件之间拆分过细时，容易造成通讯和控制繁琐

3. 自研框架

   优势

   * 高度定制化，满足需要做兼容的一切场景
   * 独立的通讯机制和沙箱运行环境，可以解决应用之间相互影响的问题
   * 可以不同技术栈子应用，可无缝实现无刷新页面

   劣势

   * 技术实现难度高
   * 需要实现一套定制的通讯机制
   * 首次加载会出现资源过大的情况

4. 最终实现方式-自研
   * 路由分发式
   * 主应用控制路由匹配和子应用加载，共享依赖加载。
   * 子应用做功能，并接入主应用实现主子控制和联动

## 微前端项目架构

1.主应用

	* 注册子应用
	* 加载渲染子应用
	* 路由匹配
	* 获取数据
	* 通信

2.子应用的功能

	* 渲染
	* 监听通信

3.微前端框架

	* 子应用的注册
	* 有内容
	* 路由更新判断
	* 匹配对应的子应用
	* 加载子应用的内容
	* 完成所有依赖项的执行
	* 将子应用渲染在固定容器内
	* 公共事件的管理
	* 异常的捕获和报错
	* 全局的状态管理内容
	* 沙箱的隔离
	* 通信的机制

![image-20211107171620577](/Users/wangshuya/Library/Application Support/typora-user-images/image-20211107171620577.png)

## 微前端实现

### 子应用注册事件

react16注册

```react
import React from "react";
import "./index.scss";
import ReactDOM from "react-dom";
import BasicMap from "./src/router";

const render = () => {
  ReactDOM.render(<BasicMap />, document.getElementById("app-react"));
};

if (!window.__MICRO_WEB__) {
  render();
}
export const bootstrap = () => {
  console.log("react16 bootstrap");
};

export const mount = () => {
  render();
  console.log("react16 mount");
};

export const unmount = () => {
  console.log("react16 unmount");
};

```



vue3注册

```vue
import { createApp } from "vue";
import App from "./App.vue";
import router from "./router";

let instance = null;

function render() {
  instance = createApp(App);
  instance.use(router).mount("#app");
}

if (!window.__MICRO_WEB__) {
  render();
}
export const bootstrap = () => {
  console.log("vue3 bootstrap");
};

export const mount = () => {
  render();
  console.log("vue3 mount");
};

export const unmount = () => {
  console.log("vue3 unmount");
};

```

所有的子应用的webpack配置都需要添加额外的配置

react相关的

```javascript
module.exports = {
  ...
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'react15.js',
    library: 'react15',//这个子应用对外暴露的名称
    libraryTarget: 'umd',//打包出来的模块需要统一
    umdNamedDefine: true,
    publicPath: 'http://localhost:9002/'//
  },
  devServer: {
    // 配置允许跨域
    headers: { 'Access-Control-Allow-Origin': '*' },//允许跨域
    contentBase: path.join(__dirname, 'dist'),
    compress: true,
    port: 9002,
    historyApiFallback: true,
    hot: true,
  }
}
```



vue相关的

```javascript
module.exports = {
  outputDir: "dist", // 打包的目录
  assetsDir: "static", // 打包的静态资源
  filenameHashing: true, // 打包出来的文件，会带有hash信息
  publicPath: "http://localhost:9004",
  devServer: {
    contentBase: path.join(__dirname, "dist"), //必须配置
    hot: false,
    disableHostCheck: true,
    port,
    headers: {
      "Access-Control-Allow-Origin": "*", // 本地服务的跨域内容
    },
  },
  // 自定义webpack配置
  configureWebpack: {
    resolve: {
      alias: {
        "@": resolve("src"),
      },
    },
    output: {
      // 把子应用打包成 umd 库格式 commonjs 浏览器，node环境
      library: `${packageName}`, //子应用的名称
      libraryTarget: "umd",
    },
  },
};
```



### 主应用注册子应用列表

主应用采用vue3框架

```javascript
//subNavList
export const subNavList = [
  {
    name: "react15", // 唯一
    entry: "//localhost:9002/",
    loading,
    container: "#micro-container", //子应用挂载的容器
    activeRule: "/react15",
    appInfo,
  },
  {
    name: "react16",
    entry: "//localhost:9003/",
    loading,
    container: "#micro-container",
    activeRule: "/react16",
    appInfo,
  },
  {
    name: "vue2",
    entry: "//localhost:9004/",
    loading,
    container: "#micro-container",
    activeRule: "/vue2",
    appInfo,
  },
  {
    name: "vue3",
    entry: "//localhost:9005/",
    loading,
    container: "#micro-container",
    activeRule: "/vue3",
    appInfo,
  },
];

```

```javascript
export const registerMicroApps = (appList) => {
  setList(appList);
};

export const registerApp = (list) => {
  // 注册到微前端框架里面
  registerMicroApps(list);
};

```



```javascript
//main.js
import { createApp } from "vue";
import App from "./App.vue";
import router from "./router";
import { subNavList } from "./store/sub";
import { registerApp } from "./util";

registerApp(subNavList);

createApp(App).use(router()).mount("#micro_web_main_app");

```

### 主应用进行路由拦截

```javascript
export const turnApp = async (event) => {
  if (isTurnChild()) {
    console.log("切换");
  }
};
// 子应用是否做了切换
export const isTurnChild = () => {

  const { pathname, hash } = window.location;
  const url = pathname + hash;

  // 当前路由无改变。
  const currentPrefix = url.match(/(\/\w+)/g);

  if (
    currentPrefix &&
    currentPrefix[0] === window.__CURRENT_SUB_APP__ &&
    hash === window.__CURRENT_HASH__
  ) {
    return false;
  }

  window.__ORIGIN_APP__ = window.__CURRENT_SUB_APP__;

  const currentSubApp = window.location.pathname.match(/(\/\w+)/);

  if (!currentSubApp) {
    return false;
  }
  // 当前路由以改变，修改当前路由
  window.__CURRENT_SUB_APP__ = currentSubApp[0];

  // 判断当前hash值是否改变
  window.__CURRENT_HASH__ = hash;

  return true;
};

// 给当前的路由跳转打补丁
export const patchRouter = (globalEvent, eventName) => {
  return function () {
    const event = new Event(eventName);
    globalEvent.apply(this, arguments);
    window.dispatchEvent(event);
  };
};

// 重写window的路由跳转
export const rewriteRouter = () => {
  window.history.pushState = patchRouter(
    window.history.pushState,
    "micro_push"
  );
  window.history.replaceState = patchRouter(
    window.history.pushState,
    "micro_replace"
  );
  window.addEventListener("micro_push", turnApp);
  window.addEventListener("micro_replace", turnApp);
  window.onpopstate = function () {
    turnApp();
  };
};

```

### 主应用获取子应用

```javascript
export const currentApp = () => {
  const currentUrl = window.location.pathname;
  filterApp("activeRule", currentUrl);
};

export const filterApp = (key, value) => {
  const currentApp = getList().filter((item) => item[key] === value);
  return currentApp || [];
};

// 启动微前端框架
export const start = () => {
  const apps = getList();
  
  if (!apps.length) {
    // 子应用列表为空
    throw Error("子应用列表为空， 请正确注册");
  }

  // 有子应用的内容， 查找到符合当前路由的子应用
  const app = currentApp();

  const { pathname, hash } = window.location;

  if (!hash) {
    // 当前没有在使用的子应用
    // 1. 抛出一个错误，请访问正确的连接
    // 2. 访问一个默认的路由，通常为首页或登录页面
    window.history.pushState(null, null, "/vue3#/index");
  }

  if (app && hash) {
    const url = pathname + hash;

    window.__CURRENT_SUB_APP__ = app.activeRule;
    console.log("pushstate");
    window.history.pushState("", "", url);
  }
};


```

### 主子应用生命周期

```javascript
export const lifeCycle = async () => {
  // 获取到上一个子应用
  // const prevApp = null;
  const prevApp = findAppByRoute(window.__ORIGIN_APP__);
  // 获取到要跳转的子应用
  const nextApp = findAppByRoute(window.__CURRENT_SUB_APP__);
  console.log(prevApp, nextApp);
  if (!nextApp) {
    return;
  }
  if (prevApp?.unmount) {
    if (prevApp.proxy) {
      prevApp.proxy.inactive(); //将上一个沙箱销毁
    }
    //如果没有之前的app，不执行主应用和子应用的卸载
    await destroyed(prevApp);
  }
  const app = await beforeLoad(nextApp);
  await mounted(app);
};

export const beforeLoad = async (app) => {
  await runMainLifeCycle("beforeLoad");
  app?.beforeLoad?.();
  const subApp = await loadHtml(app);
  subApp?.bootstrap?.();
  return subApp;
};

export const mounted = async (app) => {
  app?.mount?.({
    appInfo: app.appInfo,
    entry: app.entry,
  });
  await runMainLifeCycle("mounted");
};

export const destroyed = async (app) => {
  app?.unmount?.();
  // 对应的执行主应用的生命周期
  await runMainLifeCycle("destroyed");
};

export const runMainLifeCycle = async (type) => {
  const mainLife = getMainLifeCycle();
  await Promise.all(mainLife[type].map(async (item) => await item()));
};

```



### 加载和解析html、js文件

```javascript
export const fetchResource = (url) => fetch(url).then((res) => res.text());
// 加载html
export const loadHtml = async (app) => {
  // 子应用显示
  let container = app.container;
  // 子应用入口
  let entry = app.entry;

  const [dom, scripts] = await parseHtml(entry, app.name);

  const ele = document.querySelector(container);
  ele.innerHTML = dom;
  scripts.forEach((item) => {
    // performScriptForEval(item);
    sandBox(item, app);
  });
  return app;
};
// 根据子应用的name来做缓存
const cache = {};
export const parseHtml = async (entry, name) => {
  if (cache[name]) {
    return cache[name];
  }
  const html = await fetchResource(entry);
  // 标签 link script
  const div = document.createElement("div");
  div.innerHTML = html;
  const [dom, scriptUrl, script] = await getResource(div, entry);
  const fetchedScripts = await Promise.all(
    scriptUrl.map((item) => fetchResource(item))
  );
  cache[name] = [dom, script.concat(fetchedScripts)];
  return cache[name];
};

export const getResource = async (root, entry) => {
  const scriptUrl = [];
  const script = [];
  const dom = root.outerHTML;
  function deepParse(element) {
    const children = element.children;
    const parent = element.parent;
    // 第一步解析位于script中的内容
    if (element.nodeName.toLowerCase() === "script") {
      const src = element.getAttribute("src") || "";
      if (!src) {
        script.push(element.outerHTML);
      } else {
        if (src.startsWith("http")) {
          scriptUrl.push(src);
        } else {
          // webpack配置不带publicPath时是相对路径
          scriptUrl.push(`http:${entry}${src}`);
        }
      }
      if (parent) {
        parent.replaceChild(
          document.createComment("此js文件已经被解析"),
          element
        );
      }
    }
    if (element.nodeName.toLowerCase() === "link") {
      const href = element.getAttribute("href") || "";
      if (href.endsWith(".js")) {
        if (href.startsWith("http")) {
          scriptUrl.push(href);
        } else {
          // webpack配置不带publicPath时是相对路径
          scriptUrl.push(`http:${entry}${href}`);
        }
      }
    }
    for (let i = 0; i < children.length; i++) {
      deepParse(children[i]);
    }
  }
  deepParse(root);
  return [dom, scriptUrl, script];
};

```



### 执行js脚本

new Function

```javascript
//global参数进行隔离
export const performScriptForFunction = (script, appName, global) => {
  window.proxy = global;
  const scriptText = `
  return ((window) => {
    ${script}
    return window['${appName}']
  })(window.proxy)
  `;
  return new Function(scriptText)();
};
```

eval

```javascript
export const performScriptForEval = (script, appName, global) => {
  window.proxy = global;
  const scriptText = `((window) => {
    ${script}
    return window['${appName}']
  })(window.proxy)`;
  return eval(scriptText);
};
```



### 沙箱环境

快照沙箱

```javascript
// 快照沙箱
// 应用场景比较老的浏览器
export class ProxySandBox {
  constructor() {
    //代理对象
    this.proxy = window;
    this.active();
  }
  //沙箱激活
  active() {
    // 创建一个沙箱快照
    this.snapShot = new Map();
    // 遍历全局环境
    for (const key in window) {
      this.snapShot[key] = window[key];
    }
  }
  //沙箱销毁
  inactive() {
    for (const key in window) {
      if (window[key] !== this.snapShot[key]) {
        // 还原操作
        window[key] = this.snapShot[key];
      }
    }
  }
}

```

代理沙箱

```javascript
// 代理沙箱
let defaultValue = {}; //子应用的沙箱容器
export class ProxySandBox {
  constructor() {
    //代理对象
    this.proxy = null;
    this.active();
  }
  //沙箱激活
  active() {
    this.proxy = new Proxy(window, {
      get(target, key) {
        if (typeof target[key] === "function") {
          return target[key].bind(target);
        }
        return defaultValue[key] || target[key];
      },
      set(target, key, value) {
        defaultValue[key] = value;
        return true;
      },
    });
  }
  //沙箱销毁
  inactive() {
    console.log("销毁");
    defaultValue = {};
  }
}
```



### css隔离

module css

shadow css

mini css

### 主子通信

props 

CustomEvent

```javascript
//react16//src//utils/main.js
let main = null

export const setMain = (data) => {
  main = data
}

export const getMain = () => {
  return main
}
```

子应用的生命周期里面接收主应用传来的数据，存储在全局变量里面

```javascript
//react16/src/index.js
export const mount = async (app) => {
  setMain(app);
  render();
  console.log("react16 mount");
};


```

子应用在页面里面使用存储的全局数据

```javascript
//react16/src/pages/login/index.jsx
import { getMain } from "../../utils/main";

const Login = () => {
  useEffect(() => {
    // 登录页面隐藏header、nav
    const main = getMain();
    main.appInfo.header.changeHeader(false);
    main.appInfo.nav.changeNav(false);
  }, []);

  return (
   ...
  );
};

export default Login;


```



2.CustomEvent

微前端声明一个发布订阅相关的事件

```javascript
//micro/customeEvent/index.js
export class Custom {
  // 事件绑定
  on(name, cb) {
    window.addEventListener(name, (e) => {
      cb(e.detail);
    });
  }
  // 事件触发
  emit(name, data) {
    const event = new CustomEvent(name, { detail: data });
    window.dispatchEvent(event);
  }
}

```

主应用在注册子应用的时候，添加全局事件

```javascript
//主应用订阅
const custom = new Custom();
custom.on("test", (data) => {
  console.log("test事件触发");
  console.log(data);
});
window.custom = custom;
```

```javascript
//子应用触发主应用事件
window.custom.emit("test", {
  a: 1,
});
```

### 子应用间通信

1.props： 子应用 - 父应用 - 子应用

子应用定义一个全局变量

### 全局状态

```javascript
export const createStore = (initData) =>
  (() => {
    let store = initData;
    const observers = [];
    const getStore = () => store;
    const update = (value) => {
      if (value !== store) {
        const oldValue = store;
        store = value;
        observers.forEach(async (item) => await item(store, oldValue));
      }
    };
    const subscribe = (fn) => {
      observers.push(fn);
    };
    return {
      getStore,
      update,
      subscribe,
    };
  })();
```



### 缓存子应用

```javascript
// 根据子应用的name来做缓存
const cache = {};
export const parseHtml = async (entry, name) => {
  if (cache[name]) {
    return cache[name];
  }
  const html = await fetchResource(entry);
  // 标签 link script
  const div = document.createElement("div");
  div.innerHTML = html;
  const [dom, scriptUrl, script] = await getResource(div, entry);
  const fetchedScripts = await Promise.all(
    scriptUrl.map((item) => fetchResource(item))
  );
  cache[name] = [dom, script.concat(fetchedScripts)];
  return cache[name];
};
```



### 预加载子应用

```javascript
export const preFetch = async () => {
  // 获取所有的子应用列表-不包括当前正在显示的
  const leftList = getList().filter(
    (item) => !window.location.pathname.startsWith(item.activeRule)
  );
  // 预加载剩下所有的子应用

  await Promise.all(leftList.map((item) => parseHtml(item.entry, item.name)));
};

```

## 源码分析

### 相关目地址

https://github.com/diandiandidi519/micro-frontend-monorepo

### qiankun源码分析

https://github.com/umijs/qiankun

### single-spa源码分析

https://github.com/single-spa/single-spa

### garfish源码分析

