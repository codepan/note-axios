# MDN文档
https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Overview
# HTTP请求报文
***请求行***

```http
Method URL
GET /user/1
POST /login
```

***请求头***

```http
HOST:www.codepan.cc
Cookie:CODEPANID=ASD4GHD2442;CODEPANPSID=AD4R5Y76G;
Content-Type:application/x-www-form-urlencoded 或者 application/json
```

***请求体***

```http
username=tom&password=123456
{"username":"tom","password":"123456"}
```
# HTTP响应报文
***响应行***

```http
status statusText
```

***响应头***

```http
Content-Type:text/html;charset=utf-8
Set-Cookie:CP_SUM=1;path=/
```

***响应体***

html 文本/json 文本/js/css/图片/...

# post请求体参数格式
* Content-type:application/x-www-form-urlencoded;charset-utf-8

  用于键值对参数，参数的键值用"="连接，参数之间用"&"连接
  eg：name=codepan&age=12
* Content-type:application/json;charset-utf-8

  用于json字符串参数
  eg：{"name":"codepan","age":12}
* Content-type:multipart/form-data

  用于文件上传请求

# 常见的响应状态码
status|statusText|message
---|---|---
200|OK|请求成功，一般用于GET和POST请求
201|Created|已创建，成功请求并创建了新的资源
401|Unauthorized|未授权/请求要求用户的身份认证
404|Not Found|服务器无法根据客户端的请求找到资源
500|Internal Server Error|服务器内部错误，无法完成请求
# 不同类型的请求及作用
类型|作用
---|---
GET|从服务器端读取数据-查询
POST|向服务器端添加新数据-新增
PUT|更新服务器端已经存在的数据-更新
DELETE|删除服务器端数据-删除
# API的分类
* REST API：restful
  * 发送请求进行CRUP哪个操作，由请求方式来决定
  * 同一个请求路径可以实现多个操作
  * 请求方式会用到GET/POST/PUT/DELETE
* 非REST API：restless
  * 请求方式不决定请求的CRUD操作
  * 一个请求路径只对应一个操作
  * 一般只有GET/POST

# 使用json-server搭建REST API
### json-server是什么？
用来快速搭建REST API的工具包
### 使用json-server
* 在线文档：https://github.com/typicode/json-server
* 下载：`npm install -g json-server`或者`yarn global add json-server`
* 目标根目录下创建数据库json文件：db.json

  ```json
  {
    "posts": [
      {"id": 1, "title": "json-server", "author": "typicode"}
    ],
    "comments": [
      {"id": 1, "body": "some comment", "postId": 1}
    ],
    "profile": {"name": "typicode"}
  }
  ```
* 启动服务器执行命令: `json-server --watch db.json`
* 测试
  * 使用浏览器访问测试
  
    ```
    http://localhost:3000/posts
    http://localhost:3000/posts/1
    http://localhost:3000/comments
    http://localhost:3000/profile
    ```
  * 使用axios访问测试

    ```html
    <body>
      <div>
        <button onclick="testGet()">GET 请求</button>
        <button onclick="testPost()">POST 请求</button>
        <button onclick="testPut()">PUT 请求</button>
        <button onclick="testDelete()">DELETE 请求</button>
      </div>
    </body>
    <script src="https://cdn.bootcss.com/axios/0.19.0/axios.js"></script>
    <script>
      /* 1. GET 请求: 从服务器端获取数据*/
      function testGet() {
        // axios.get('http://localhost:3000/posts')
        // axios.get('http://localhost:3000/posts/1') // 获取 id 为 1 的对象
        // axios.get('http://localhost:3000/posts?id=1&id=2') // 获取 id 为1或2的数组
        axios.get('http://localhost:3000/posts?title=json-server&author=typicode')
      }
      /* 2. POST 请求: 向服务器端添加新数据*/
      function testPost() {
        axios.post('http://localhost:3000/posts', {title: 'xxx', author: 'yyyy'}) // 保存数据
      }
      /* 3. PUT 请求: 更新服务器端已经数据 */
      function testPut() {
        axios.put('http://localhost:3000/comments/2', {body: 'yyy', postId: 2})
      }
      /* 4. DELETE 请求: 删除服务器端数据 */
      function testDelete() {
        axios.delete('http://localhost:3000/comments/2')
      }
    </script>
    ```