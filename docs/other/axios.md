### axios项目结构
当前的版本0.20.4

```
lib
└─ adapters
   ├─ http.js // node 环境下利用 http 模块发起请求
   ├─ xhr.js // 浏览器环境下利用 xhr 发起请求
└─ cancel
   ├─ Cancel.js
   ├─ CancelToken.js
   ├─ isCancel.js
└─ core
    ├─ Axios.js // 生成 Axios 实例
    ├─ InterceptorManager.js // 拦截器
    ├─ dispatchRequest.js  // 调用适配器发起请求
    ...
└─ helpers
    ├─ mergeConfig.js // 合并配置
    ├─ ...
├─ axios.js  // 入口文件
├─ defaults.js  // axios 默认配置项
├─ utils.js
```

### axios介绍

`Axios` 是一个基于 `Promise` 网络请求库，作用于 `node.js` 和浏览器中。在服务端它使用原生 node.js`http`模块, 而在客户端 (浏览端) 则使用 `XMLHttpRequests`。特性：

- 从浏览器创建XMLHttpRequests

- 从 node.js 创建http请求

- 支持PromiseAPI

- 拦截请求和响应

- 转换请求和响应数据

- 取消请求

- 自动转换 JSON 数据

- 客户端支持防御XSRF

### **Axios 内部运作流程**

![img](https://bytedance.feishu.cn/space/api/box/stream/download/asynccode/?code=ZWUyNTdkMDBjY2U0NTc5N2IyZWQ5M2IxNzU2MzE0MDVfeG9RNkVQWXNObVJnZ3dPNlBiYWpzOVF5UnpkTk5kcXZfVG9rZW46Ym94Y25WdTh3STEzMVFpUXJxSkxLbWRyZHVmXzE2MzYxOTEzMTc6MTYzNjE5NDkxN19WNA)

### axios.js文件

```javascript
//返回axios实例
function createInstance(defaultConfig) {
  //创建一个axios实例
  var context = new Axios(defaultConfig);
  // 将request请求的this绑定到新建的axios实例上 支持axios({})这种模式
  var instance = bind(Axios.prototype.request, context);
  // 将 axios.prototype 上的方法 传递给 instance，并且绑定this为conetxt
  utils.extend(instance, Axios.prototype, context);
  // 将context属性传递给instance
  utils.extend(instance, context);
  // Factory for creating new instances
  instance.create = function create(instanceConfig) {
    return createInstance(mergeConfig(defaultConfig, instanceConfig));
  };
  return instance;
}

// Create the default instance to be exported
var axios = createInstance(defaults);
// Expose Axios class to allow class inheritance
axios.Axios = Axios;

// Expose Cancel & CancelToken
axios.Cancel = require('./cancel/Cancel');
axios.CancelToken = require('./cancel/CancelToken');
axios.isCancel = require('./cancel/isCancel');
axios.VERSION = require('./env/data').version;

// Expose all/spread
axios.all = function all(promises) {
  return Promise.all(promises);
};
axios.spread = require('./helpers/spread');

// Expose isAxiosError
axios.isAxiosError = require('./helpers/isAxiosError');
```

`axios`是通过createInstance创建出实例对象。

其中`var instance = bind(Axios.prototype.request, context);`返回的是个函数，函数的`this`绑定为`conext`，也就是通过`new Axios()`出来的实例。当我们调用`axios({})`也就是调用`Axios.prototype.request`方法。

`utils.extend(instance, context)`这句话把`context`实例相关的属性拷贝到`instance`这个函数上。

`instance.create = function create(instanceConfig) {return ... };`可以通过工厂模式让用户自定义一些配置。例如

```javascript
const instance = axios.create({
  baseURL: 'https://some-domain.com/api/',
  timeout: 1000,
  headers: {'X-Custom-Header': 'foobar'}
});
```

所以最终的`axios`既是函数也是对象。

最关键的点是Axios构造函数，接下来分析一下这个构造函数

### Axios构造函数

```javascript
function Axios(instanceConfig) {
  this.defaults = instanceConfig;
  this.interceptors = {
    request: new InterceptorManager(),
    response: new InterceptorManager()
  };
}

//真正发送请求的函数
Axios.prototype.request = function request(config) {
  ...
};

//
Axios.prototype.getUri = function getUri(config) {
  config = mergeConfig(this.defaults, config);
  return buildURL(config.url, config.params, config.paramsSerializer).replace(/^\?/, '');
};

// 下面的两个forEach提供一些请求方法的别名 最终还是调通request方法
//axios.delete/get/head/options()
utils.forEach(['delete', 'get', 'head', 'options'], function forEachMethodNoData(method) {
  Axios.prototype[method] = function(url, config) {
    return this.request(utils.merge(config || {}, {
      method: method,
      url: url
    }));
  };
});

//axios.post/put/patch()
utils.forEach(['post', 'put', 'patch'], function forEachMethodWithData(method) {
  Axios.prototype[method] = function(url, data, config) {
    return this.request(utils.merge(config || {}, {
      method: method,
      url: url,
      data: data
    }));
  };
});
```

`Axios`构造函数内部有默认配置属性`defaults`和`interceptors`拦截器属性。`interceptors`对象上有`request`和`response`两个属性。其中拦截器是通过`InterceptorManager`构造函数`new`出来的实例。Axios原型上有`get/detele/head/options/post/put/patch/request/getUri `这些方法。其中`get/detele/head/options/post/put/patch `默认也是调用`request`方法。例如当`axios.get()`时。

最终的`Axios`构造函数如下

![img](https://bytedance.feishu.cn/space/api/box/stream/download/asynccode/?code=MGZlY2M0ZDQ3YjY5NjZkN2VkZGI1ODBjNDZkZWMwYTlfUjRIYVM2MEJMUlN2cUpta1g3bUZGbzF3YkkxbzFSQTRfVG9rZW46Ym94Y25SUDAwOFdBbGw2ZWtzN09uMFpISXdjXzE2MzYxOTEzMTc6MTYzNjE5NDkxN19WNA)



#### `defauls`默认配置

```javascript
var DEFAULT_CONTENT_TYPE = {
  'Content-Type': 'application/x-www-form-urlencoded'
};
var defaults = {
  transitional: {
    silentJSONParsing: true,
    forcedJSONParsing: true,
    clarifyTimeoutError: false
  },
  adapter: getDefaultAdapter(),//适配器
  transformRequest: [function transformRequest(data, headers) {
    ...
  }],
  transformResponse: [function transformResponse(data) {
    ...
  }],
  /**
   * A timeout in milliseconds to abort a request. If set to 0 (default) a
   * timeout is not created.
   */
  timeout: 0,

  xsrfCookieName: 'XSRF-TOKEN',
  xsrfHeaderName: 'X-XSRF-TOKEN',

  maxContentLength: -1,
  maxBodyLength: -1,

  validateStatus: function validateStatus(status) {
    return status >= 200 && status < 300;
  },

  headers: {
    common: {
      'Accept': 'application/json, text/plain, */*'
    }
  }
};

utils.forEach(['delete', 'get', 'head'], function forEachMethodNoData(method) {
  defaults.headers[method] = {};
});

//post put patch 请求'Content-Type': 'application/x-www-form-urlencoded'
utils.forEach(['post', 'put', 'patch'], function forEachMethodWithData(method) {
  defaults.headers[method] = utils.merge(DEFAULT_CONTENT_TYPE);

});
```

默认配置里面有请求头、超时、防止csrf攻击等相关属性。

### `InterceptorManager`构造函数

```javascript
function InterceptorManager() {
  this.handlers = [];
}

InterceptorManager.prototype.use = function use(fulfilled, rejected, options) {
  this.handlers.push({
    fulfilled: fulfilled,
    rejected: rejected,
    synchronous: options ? options.synchronous : false,//默认false 标志这个拦截器需要同步
    runWhen: options ? options.runWhen : null//当满足条件的时候才调用这个拦截器
  });
  return this.handlers.length - 1;
};

InterceptorManager.prototype.eject = function eject(id) {
  if (this.handlers[id]) {
    this.handlers[id] = null;
  }
};

InterceptorManager.prototype.forEach = function forEach(fn) {
  utils.forEach(this.handlers, function forEachHandler(h) {
    if (h !== null) {
      fn(h);
    }
  });
};
```

`InterceptorManager`构造函数有`handlers`数组属性用来存储拦截器。

`InterceptorManager.prototype.use`用来添加拦截器。

`InterceptorManager.prototype.eject用`来删除拦截器。

### `Axios.prototype.request`

当发送请求的时候真正调用的`request`方法。接下来分析下这个函数。

先看下源码

```javascript
Axios.prototype.request = function request(config) {
  //省略一部分配置
  ...
  // filter out skipped interceptors
  var requestInterceptorChain = [];//存储请求拦截器
  var synchronousRequestInterceptors = true;//标志同步触发还是异步触发
  //依次遍历请求拦截器
  this.interceptors.request.forEach(function unshiftRequestInterceptors(interceptor) {
    //只有当满足条件的拦截器，才会加入到请求拦截器里面
    if (typeof interceptor.runWhen === 'function' && interceptor.runWhen(config) === false) {
      return;
    }
    //
    synchronousRequestInterceptors = synchronousRequestInterceptors && interceptor.synchronous;
    //将这个请求拦截器放到数组前面
    requestInterceptorChain.unshift(interceptor.fulfilled, interceptor.rejected);
  });
  var responseInterceptorChain = [];//存储响应拦截器
  this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {
    responseInterceptorChain.push(interceptor.fulfilled, interceptor.rejected);
  });

  var promise;
  //默认都是异步触发
  if (!synchronousRequestInterceptors) {
    //触发请求
    var chain = [dispatchRequest, undefined];
    //请求拦截器放到chain的前面
    Array.prototype.unshift.apply(chain, requestInterceptorChain);
    //响应拦截器放到chain的后面
    chain = chain.concat(responseInterceptorChain);
    //生成promise,把config参数传递给后面的then
    promise = Promise.resolve(config);
    //形成链式调用
    while (chain.length) {
      promise = promise.then(chain.shift(), chain.shift());
    }
    return promise;
  }

  //同步触发请求拦截器
  var newConfig = config;
  while (requestInterceptorChain.length) {
    var onFulfilled = requestInterceptorChain.shift();
    var onRejected = requestInterceptorChain.shift();
    try {
      newConfig = onFulfilled(newConfig);
    } catch (error) {
      onRejected(error);
      break;
    }
  }
  try {
    promise = dispatchRequest(newConfig);
  } catch (error) {
    return Promise.reject(error);
  }

  while (responseInterceptorChain.length) {
    promise = promise.then(responseInterceptorChain.shift(), responseInterceptorChain.shift());
  }
  return promise;
};
```

如果使用了拦截器，请求拦截器执行的顺序是先加入后执行，响应拦截器是先加入先执行。

结合例子看一下

```javascript
axios.interceptors.request.use(function requestSuccess1(config) {
  // Do something before request is sent
  console.log('------request------success------1');
  return config;
}, function requestError1(error) {
  // Do something with request error
  console.log('------response------error------1');
  return Promise.reject(error);
});


// Add a response interceptor
axios.interceptors.response.use(function responseSuccess1(response) {
  // Any status code that lie within the range of 2xx cause this function to trigger
  // Do something with response data
  console.log('------response------success------1');
  return response;
}, function responseError1(error) {
  console.log('------response------error------1');
  // Any status codes that falls outside the range of 2xx cause this function to trigger
  // Do something with response error
  return Promise.reject(error);
});


axios.interceptors.request.use(function requestSuccess2(config) {
  // Do something before request is sent
  console.log('------request------success------2');
  return config;
}, function requestError2(error) {
  // Do something with request error
  console.log('------response------error------2');
  return Promise.reject(error);
});


// Add a response interceptor
axios.interceptors.response.use(function responseSuccess2(response) {
  // Any status code that lie within the range of 2xx cause this function to trigger
  // Do something with response data
  console.log('------response------success------2');
  return response;
}, function responseError2(error) {
  console.log('------response------error------2');
  // Any status codes that falls outside the range of 2xx cause this function to trigger
  // Do something with response error
  return Promise.reject(error);
});
```

![img](https://bytedance.feishu.cn/space/api/box/stream/download/asynccode/?code=NmE2MWVmY2U0YWFkYTE0MGNlMjBiZmVhZmJmNzIwN2ZfSzIxVlhEaDlqWDAzYlRXMlZSaU1wUWZMc2NOQzhKTGRfVG9rZW46Ym94Y25wY3FqODFQaFpqNHY2SU03cVBOY2tjXzE2MzYxOTEzMTc6MTYzNjE5NDkxN19WNA)

`chain`数组就是这样的结构。

```
var chain = [
  '请求成功拦截2', '请求失败拦截2',  
  '请求成功拦截1', '请求失败拦截1',  
  dispatch,  undefined,
  '响应成功拦截1', '响应失败拦截1',
  '响应成功拦截2', '响应失败拦截2',
]
```

 这段代码相对比较绕。也就是会生成如下类似的代码，中间会调用`dispatchRequest`方法。

```
// config 是 用户配置和默认配置合并的
var promise = Promise.resolve(config);
promise.then('请求成功拦截2', '请求失败拦截2')
.then('请求成功拦截1', '请求失败拦截1')
.then(dispatchRequest, undefined)
.then('响应成功拦截1', '响应失败拦截1')
.then('响应成功拦截2', '响应失败拦截2')

.then('用户写的业务处理函数')
.catch('用户写的报错业务处理函数');

```


```javascript
var promise = Promise.resolve(config);
// promise.then('请求成功拦截2', '请求失败拦截2')
promise.then(function requestSuccess2(config) {
  console.log('------request------success------2');
  return config;
}, function requestError2(error) {
  console.log('------response------error------2');
  return Promise.reject(error);
})

// .then('请求成功拦截1', '请求失败拦截1')
.then(function requestSuccess1(config) {
  console.log('------request------success------1');
  return config;
}, function requestError1(error) {
  console.log('------response------error------1');
  return Promise.reject(error);
})

// .then(dispatchRequest, undefined)
.then( function dispatchRequest(config) {
  /**
    * 适配器返回的也是Promise 实例
      adapter = function xhrAdapter(config) {
            return new Promise(function dispatchXhrRequest(resolve, reject) {})
      }
    **/
  return adapter(config).then(function onAdapterResolution(response) {
    // 省略代码 ...
    return response;
  }, function onAdapterRejection(reason) {
    // 省略代码 ...
    return Promise.reject(reason);
  });
}, undefined)

// .then('响应成功拦截1', '响应失败拦截1')
.then(function responseSuccess1(response) {
  console.log('------response------success------1');
  return response;
}, function responseError1(error) {
  console.log('------response------error------1');
  return Promise.reject(error);
})

// .then('响应成功拦截2', '响应失败拦截2')
.then(function responseSuccess2(response) {
  console.log('------response------success------2');
  return response;
}, function responseError2(error) {
  console.log('------response------error------2');
  return Promise.reject(error);
})

// .then('用户写的业务处理函数')
// .catch('用户写的报错业务处理函数');
.then(function (response) {
  console.log('哈哈哈，终于获取到数据了', response);
})
.catch(function (err) {
  console.log('哎呀，怎么报错了', err);
});
```

 仔细看这段`Promise`链式调用，代码都类似。`then`方法最后返回的参数，就是下一个`then`方法第一个参数。

 `catch`错误捕获，都返回`Promise.reject(error)`，这是为了便于用户`catch`时能捕获到错误。

用一张图片更直观

![img](https://bytedance.feishu.cn/space/api/box/stream/download/asynccode/?code=MjZhMmEyZDE2M2M2OGZlN2ViNTZkZGMwNDBiN2IzNDNfcUlFYW1MbktjWjA3ajZrSnRWR044U2ZDSUkyUmtxWGxfVG9rZW46Ym94Y25HdTZPNDhJaEVmM2tTWXQ5Rmw1TW9jXzE2MzYxOTEzMTc6MTYzNjE5NDkxN19WNA)

### dispatchRequest

```javascript
module.exports = function dispatchRequest(config) {
  throwIfCancellationRequested(config);
  // Ensure headers exist
  config.headers = config.headers || {};
  // Transform request data
  config.data = transformData.call(
    config,
    config.data,
    config.headers,
    config.transformRequest
  );
  // Flatten headers
  config.headers = utils.merge(
    config.headers.common || {},
    config.headers[config.method] || {},
    config.headers
  );
  
  utils.forEach(
    ['delete', 'get', 'head', 'post', 'put', 'patch', 'common'],
    function cleanHeaderConfig(method) {
      delete config.headers[method];
    }
  );

  var adapter = config.adapter || defaults.adapter;//发送请求
  
  return adapter(config).then(function onAdapterResolution(response) {
    throwIfCancellationRequested(config);

    // Transform response data
    response.data = transformData.call(
      config,
      response.data,
      response.headers,
      config.transformResponse
    );

    return response;
  }, function onAdapterRejection(reason) {
    if (!isCancel(reason)) {
      throwIfCancellationRequested(config);
      // Transform response data
      if (reason && reason.response) {
        reason.response.data = transformData.call(
          config,
          reason.response.data,
          reason.response.headers,
          config.transformResponse
        );
      }
    }


    return Promise.reject(reason);
  });
};
```

这个函数主要做了如下几件事情：

>  1.如果已经取消，则 `throw` 原因报错，使`Promise`走向`rejected`。
>  2.确保 `config.header` 存在。
>  3.利用用户设置的和默认的请求转换器转换数据。
>  4.拍平 `config.header`。
>  5.删除一些 `config.header`。
>  6.返回适配器`adapter`（`Promise`实例）执行后 `then`执行后的 `Promise`实例。返回结果传递给响应拦截器处理。

### adapter适配器之xhr

```javascript
function getDefaultAdapter() {
  var adapter;
  if (typeof XMLHttpRequest !== 'undefined') {
    // For browsers use XHR adapter
    adapter = require('./adapters/xhr');
  } else if (typeof process !== 'undefined' && Object.prototype.toString.call(process) === '[object process]') {
    // For node use HTTP adapter
    adapter = require('./adapters/http');
  }
  return adapter;
}
```

在最开始的默认配置里面，便会判断当前执行的环境，浏览器使用`xhr`，`node`环境使用`http`。

最熟悉的`XMLHTTRequest`终于要来了。

```javascript
module.exports = function xhrAdapter(config) {
  return new Promise(function dispatchXhrRequest(resolve, reject) {
    var requestData = config.data;
    var requestHeaders = config.headers;
    var responseType = config.responseType;
    var onCanceled;
    //请求完成了释放相关的取消请求的相关信息
    function done() {
      if (config.cancelToken) {
        config.cancelToken.unsubscribe(onCanceled);
      }
      if (config.signal) {
        config.signal.removeEventListener('abort', onCanceled);
      }
    }

    if (utils.isFormData(requestData)) {
      delete requestHeaders['Content-Type']; // Let the browser set it
    }
    
    var request = new XMLHttpRequest();
    // HTTP basic authentication
    if (config.auth) {
      var username = config.auth.username || '';
      var password = config.auth.password ? unescape(encodeURIComponent(config.auth.password)) : '';
      requestHeaders.Authorization = 'Basic ' + btoa(username + ':' + password);
    }

    var fullPath = buildFullPath(config.baseURL, config.url);
    request.open(config.method.toUpperCase(), buildURL(fullPath, config.params, config.paramsSerializer), true);
    // Set the request timeout in MS
    request.timeout = config.timeout;
    //请求成功以后
    function onloadend() {
      if (!request) {
        return;
      }
      // Prepare the response
      var responseHeaders = 'getAllResponseHeaders' in request ? parseHeaders(request.getAllResponseHeaders()) : null;
      var responseData = !responseType || responseType === 'text' ||  responseType === 'json' ?
        request.responseText : request.response;
      var response = {
        data: responseData,
        status: request.status,
        statusText: request.statusText,
        headers: responseHeaders,
        config: config,
        request: request
      };
      //不论请求成功还是失败都会 处理 取消请求相关
      settle(function _resolve(value) {
        resolve(value);
        done();
      }, function _reject(err) {
        reject(err);
        done();
      }, response);

      // Clean up request
      request = null;
    }


    if ('onloadend' in request) {
      // Use onloadend if available
      request.onloadend = onloadend;
    } else {
      // Listen for ready state to emulate onloadend
      request.onreadystatechange = function handleLoad() {
        if (!request || request.readyState !== 4) {
          return;
        }
        // The request errored out and we didn't get a response, this will be
        // handled by onerror instead
        // With one exception: request that using file: protocol, most browsers
        // will return status as 0 even though it's a successful request
        if (request.status === 0 && !(request.responseURL && request.responseURL.indexOf('file:') === 0)) {
          return;
        }
        // readystate handler is calling before onerror or ontimeout handlers,
        // so we should call onloadend on the next 'tick'
        setTimeout(onloadend);
      };
    }
    //请求被取消
    // Handle browser request cancellation (as opposed to a manual cancellation)
    request.onabort = function handleAbort() {
      if (!request) {
        return;
      }
      reject(createError('Request aborted', config, 'ECONNABORTED', request));

      // Clean up request
      request = null;
    };

    // Handle low level network errors
    request.onerror = function handleError() {
      // Real errors are hidden from us by the browser
      // onerror should only fire if it's a network error
      reject(createError('Network Error', config, null, request));
      // Clean up request
      request = null;
    };

    // Handle timeout
    request.ontimeout = function handleTimeout() {
      var timeoutErrorMessage = config.timeout ? 'timeout of ' + config.timeout + 'ms exceeded' : 'timeout exceeded';
      var transitional = config.transitional || defaults.transitional;
      if (config.timeoutErrorMessage) {
        timeoutErrorMessage = config.timeoutErrorMessage;
      }
      reject(createError(
        timeoutErrorMessage,
        config,
        transitional.clarifyTimeoutError ? 'ETIMEDOUT' : 'ECONNABORTED',
        request));

      // Clean up request
      request = null;
    };


    // Add xsrf header
    // This is only done if running in a standard browser environment.
    // Specifically not if we're in a web worker, or react-native.
    if (utils.isStandardBrowserEnv()) {
      // Add xsrf header
      var xsrfValue = (config.withCredentials || isURLSameOrigin(fullPath)) && config.xsrfCookieName ?
        cookies.read(config.xsrfCookieName) :
        undefined;

      if (xsrfValue) {
        requestHeaders[config.xsrfHeaderName] = xsrfValue;
      }
    }

    // Add headers to the request
    if ('setRequestHeader' in request) {
      utils.forEach(requestHeaders, function setRequestHeader(val, key) {
        if (typeof requestData === 'undefined' && key.toLowerCase() === 'content-type') {
          // Remove Content-Type if data is undefined
          delete requestHeaders[key];
        } else {
          // Otherwise add header to the request
          request.setRequestHeader(key, val);
        }
      });
    }


    // Add withCredentials to request if needed
    if (!utils.isUndefined(config.withCredentials)) {
      request.withCredentials = !!config.withCredentials;
    }


    // Add responseType to request if needed
    if (responseType && responseType !== 'json') {
      request.responseType = config.responseType;
    }


    // Handle progress if needed
    if (typeof config.onDownloadProgress === 'function') {
      request.addEventListener('progress', config.onDownloadProgress);
    }


    // Not all browsers support upload events
    if (typeof config.onUploadProgress === 'function' && request.upload) {
      request.upload.addEventListener('progress', config.onUploadProgress);
    }

    //取消请求相关
    if (config.cancelToken || config.signal) {
      // Handle cancellation
      // eslint-disable-next-line func-names
      onCanceled = function(cancel) {
        if (!request) {
          return;
        }
        reject(!cancel || (cancel && cancel.type) ? new Cancel('canceled') : cancel);
        request.abort();
        request = null;
      };


      config.cancelToken && config.cancelToken.subscribe(onCanceled);
      if (config.signal) {
        config.signal.aborted ? onCanceled() : config.signal.addEventListener('abort', onCanceled);
      }
    }

    if (!requestData) {
      requestData = null;
    }

    // Send the request
    request.send(requestData);
  });
};
```

这个函数就是通过`Promise`封装的`ajax`请求，正好可以通过`promise`形成链式调用。

函数内部通过`XMLHttpRequest`发起请求，里面都是一些常规的属性配置。

最难理解的是取消请求了。接下来看一下。

### 取消请求

首先取消请求可以通过如下方式进行调用

```javascript
const CancelToken = axios.CancelToken;
const source = CancelToken.source();
axios.get('/get/server', {
  cancelToken: source.token,
})
source.cancel('取消请求')
```

`CancelToken.source`做了什么呢？

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

`CancelToken.source`内部通过`new CancelToken`实例化了一个`token`对象，`source`返回了一个包含`token`和`cancel`的对象。

`CancelToken`构造函数做了什么呢？

```javascript
ffunction CancelToken(executor) {
  var resolvePromise;
  this.promise = new Promise(function promiseExecutor(resolve) {
    resolvePromise = resolve;
  });
  var token = this;
  this.promise.then(function(cancel) {
    if (!token._listeners) return;
    var i;
    var l = token._listeners.length;
    for (i = 0; i < l; i++) {
      token._listeners[i](cancel);
    }
    token._listeners = null;
  });
  this.promise.then = function(onfulfilled) {
    var _resolve;
    // eslint-disable-next-line func-names
    var promise = new Promise(function(resolve) {
      token.subscribe(resolve);
      _resolve = resolve;
    }).then(onfulfilled);
    promise.cancel = function reject() {
      token.unsubscribe(_resolve);
    };
    return promise;
  };
executor(function cancel(message) {
  if (token.reason) {
    // Cancellation has already been requested
    return;
  }
  token.reason = new Cancel(message);
  resolvePromise(token.reason);
});
}
CancelToken.prototype.throwIfRequested = function throwIfRequested() {
  if (this.reason) {
    throw this.reason;
  }
};

CancelToken.prototype.subscribe = function subscribe(listener) {
  if (this.reason) {
    listener(this.reason);
    return;
  }
  if (this._listeners) {
    this._listeners.push(listener);
  } else {
    this._listeners = [listener];
  }
};

CancelToken.prototype.unsubscribe = function unsubscribe(listener) {
  if (!this._listeners) {
    return;
  }
  var index = this._listeners.indexOf(listener);
  if (index !== -1) {
    this._listeners.splice(index, 1);
  }
};
```

函数接收一个回调函数作为参数，会在`new`实例的时候调用这个回调函数。

函数内部通过`new Promise`声明了一个`promise`属性，将`promise`的`resolve`函数赋值给了外部的变量`resolvePromise`，这一步将内部的`resolve`暴露了出去。此时的`promise`的状态为`pending`。

接着给`promise`添加了`.then`回调函数。

接着重写了`this.promise.then`，没太看懂

最后执行了传进来的回调函数，又给回调函数传递了一个`cancel`函数参数。

这个`cancel`函数内部`new Cancel` 一个实例，调用了`resolvePromise`，将`promise`的状态由`pending`变为`fulfilled`。

那当我们调用取消请求`source.cancel`的时候，其实调用的就是这个`cancel`函数。

在未调用`source.cancel`此时的source实例如下。

![img](https://bytedance.feishu.cn/space/api/box/stream/download/asynccode/?code=Yjk3YjMzZmQxMTE2YTcwMGYwNjg2Mzc1NGUxODY5M2RfWDVKOUNWWTE2d1ZIa0VDNUpKazNZZDZEc1dBRW41TzNfVG9rZW46Ym94Y25aS2d1SXhTV0ZnNnhmVU9COXlIOWxnXzE2MzYxOTEzMTc6MTYzNjE5NDkxN19WNA)

那发起请求的时候，是如何知道当前对应的请求，有取消请求的标志呢？

```javascript
var onCanceled;
function done() {
  if (config.cancelToken) {
    config.cancelToken.unsubscribe(onCanceled);
  }


  if (config.signal) {
    config.signal.removeEventListener('abort', onCanceled);
  }
}
...
if (config.cancelToken || config.signal) {
  onCanceled = function(cancel) {
    if (!request) {
      return;
    }
    reject(!cancel || (cancel && cancel.type) ? new Cancel('canceled') : cancel);
    request.abort();
    request = null;
  };


  config.cancelToken && config.cancelToken.subscribe(onCanceled);
  if (config.signal) {
    config.signal.aborted ? onCanceled() : config.signal.addEventListener('abort', onCanceled);
  }
}
```

实际上，在发起 `xhr`请求的时候已经做了判断，如何当前请求配置里面含有了`cancelToken`这个信息，然后就会往之前的`source.token`实例上订阅一个取消请求的回调函数`cancelToken.subscribe(onCanceled)`

其中`onCanceled`这个函数里面对当前的`xhr`请求进行了`abort()`操作。

此时的`source.token`属性上多了一个`_listeners`数组，存放了`onCanceled`这个回调函数。

![img](https://bytedance.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTYxYThjZjA2NTQ2ZDJlOGM2ZTJiZjFjNmMyZTJlMzlfajNrNUNpYmFld1RhRmV1emw2YXltbnhBOEJ4aVVLY2FfVG9rZW46Ym94Y25PN2Z5QXhqZHBJUzQ4V3lxRTZoWjZjXzE2MzYxOTEzMTc6MTYzNjE5NDkxN19WNA)



具体再分析下这个`onCanceled`函数什么时候会被调用。

当`source.cancel()`被调用的时候，是怎么触发这个`onCanceled`函数的，再看下`CancelToken`构造函数内部，

```javascript
this.promise.then(function(cancel) {
  if (!token._listeners) return;
  var i;
  var l = token._listeners.length;
  for (i = 0; i < l; i++) {
    token._listeners[i](cancel);
  }
  token._listeners = null;
});
this.promise.then = function(onfulfilled) {
  var _resolve;
  var promise = new Promise(function(resolve) {
    token.subscribe(resolve);
    _resolve = resolve;
  }).then(onfulfilled);
  promise.cancel = function reject() {
    token.unsubscribe(_resolve);
  };
  return promise;
};
executor(function cancel(message) {
  if (token.reason) {
    // Cancellation has already been requested
    return;
  }
  token.reason = new Cancel(message);
  resolvePromise(token.reason);
});
```

当调用`source.cancel`的时候就是调用`cancel`函数。

`cancel`函数内部给`source.token`添加了`Cancel`实例化的对象，调用了`source.token`内部的`promise.resolve`方法。

这时候的`source.token`内部的`promise`变成了`fullfilled`状态

![img](https://bytedance.feishu.cn/space/api/box/stream/download/asynccode/?code=YzNkNTY5YmI1MGRiYzExYjAxMTc3ZGQ3ZGIwN2Q2MDRfSlBZa2NDSHg0TFhzMk1CQ1pCM1BpcHV0N1lpaEVUSE1fVG9rZW46Ym94Y25GazVoRGl6YnRWeWlxcnkzc1BtbHNnXzE2MzYxOTEzMTc6MTYzNjE5NDkxN19WNA)

然后执行了`this.promise.then`回调，在`.then`回调函数内部，依次执行了实例上绑定的订阅的方法，就是取消`xhr`请求的`onCanceled`回调函数。

在`onCanceled`内部将当前的`xhr`请求进行了`abort()`操作。

对外`reject(!cancel || (cancel && cancel.type) ? new Cancel('canceled') : cancel);`

然后触发了`xhr.onabort`事件。

### 转换请求和响应数据

### 自动转换JSON数据

### csrf防御


### 参考文章
[学习 axios 源码整体架构，打造属于自己的请求库](https://juejin.cn/post/6844904019987529735)

[77.9K Star 的 Axios 项目有哪些值得借鉴的地方](https://juejin.cn/post/6885471967714115597)

[https://github.com/axios/axios](https://github.com/axios/axios)
