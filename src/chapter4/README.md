# 源码目录结构
```bash
├── /dist/                    # 项目输出目录
├── /lib/                     # 项目源码目录
│ ├── /adapters/              # 定义请求的适配器xhr、http
│ │ ├── http.js               # 实现http适配器(包装http包) 
│ │ └── xhr.js                # 实现xhr适配器(包装xhr对象)
│ ├── /cancel/                # 定义取消功能
│ │ ├── Cancel.js             # Cancel类
│ │ ├── CancelToken.js        # CancelToken类
│ │ ├── isCancel.js           # 是否为Cancel实例的判断方法
│ ├── /core/                  # 一些核心功能
│ │ |── Axios.js              # axios的核心主类
│ │ |── dispatchRequest.js    # 用来调用http请求适配器方法发送请求的函数 
│ │ |── InterceptorManager.js # 拦截器的管理器
│ │ |── settle.js             # 根据http响应状态，改变Promise的状态 
│ ├── /helpers/               # 一些辅助方法
│ ├── axios.js                # 对外暴露接口 
│ ├── defaults.js             # axios的默认配置 
│ └── utils.js                # 公用工具
├── package.json              # 项目信息
├── index.d.ts                # 配置TypeScript的声明文件
└── index.js                  # 入口文件
```
# 源码分析

***axios与Axios的关系***

* 从语法上来说: axios不是Axios的实例
* 从功能上来说: axios是Axios的实例
* axios是Axios.prototype.request函数bind()返回的函数
* axios作为对象有Axios原型对象上的所有方法, 有Axios对象上所有属性

***instance与axios的区别***

* 相同:
  * 都是一个能发任意请求的函数: request(config)
  * 都有发特定请求的各种方法: get()/post()/put()/delete()
  * 都有默认配置和拦截器的属性: defaults/interceptors
* 不同:
  * 默认匹配的值很可能不一样
  * instance没有axios后面添加的一些方法: create()/CancelToken()/all()

***axios运行的整体流程***

* 整体流程:
  
  request(config) ==> dispatchRequest(config) ==> xhrAdapter(config)

* request(config):

  将请求拦截器 / dispatchRequest() / 响应拦截器 通过 promise 链串连起来, 返回 promise

* dispatchRequest(config):
  
  转换请求数据 ===> 调用 xhrAdapter()发请求 ===> 请求返回后转换响应数 据. 返回 promise
* xhrAdapter(config):
  
  创建 XHR 对象, 根据 config 进行相应设置, 发送特定请求, 并接收响应数据, 返回 promise

***axios的请求/响应拦截器是什么***

* 请求拦截器:
  
  在真正发送请求前执行的回调函数 可以对请求进行检查或配置进行特定处理 成功的回调函数, 传递的默认是 config(也必须是) 失败的回调函数, 传递的默认是 error
* 响应拦截器

  在请求得到响应后执行的回调函数 可以对响应数据进行特定处理 成功的回调函数, 传递的默认是 response 失败的回调函数, 传递的默认是 error

***axios 的请求/响应数据转换器是什么***

* 请求转换器: 对请求头和请求体数据进行特定处理的函数

  ```js
  if (utils.isObject(data)) {
    setContentTypeIfUnset(headers, 'application/json;charset=utf-8')
    return JSON.stringify(data)
  }
  ```
* 响应转换器: 将响应体 json 字符串解析为 js 对象或数组的函数

  ```js
  response.data = JSON.parse(response.data)
  ```

***response的整体结构***

```js
{
  data, status, statusText, headers, config, request
}
```

***error的整体结构***
```js
{
  message,
  response,
  request
}
```

***如何取消未完成的请求***
* 当配置了 cancelToken 对象时, 保存 cancel 函数
  * 创建一个用于将来中断请求的cancelPromise
  * 并定义了一个用于取消请求的cancel函数
  * 将cancel函数传递出来
* 调用 cancel()取消请求
  * 执行 cacel 函数, 传入错误信息 message
  * 内部会让 cancelPromise 变为成功, 且成功的值为一个 Cancel 对象
  * 在cancelPromise的成功回调中中断请求,并让发请求的proimse失败，失败的 reason 为 Cancel 对象
  