## vue-resource 实现 get, post, jsonp请求

https://github.com/pagekit/vue-resource

除了 vue-resource 之外，还可以使用 `axios` 的第三方包实现实现数据的请求

1. 之前的学习中，如何发起数据请求？
2. 常见的数据请求类型？  get  post jsonp
3. 测试的URL请求资源地址：

- get请求地址： http://vue.studyit.io/api/getlunbo
- post请求地址：http://vue.studyit.io/api/post
- jsonp请求地址：http://vue.studyit.io/api/jsonp

4. JSONP的实现原理

- 由于浏览器的安全性限制，不允许AJAX访问 协议不同、域名不同、端口号不同的 数据接口，浏览器认为这种访问不安全；
- 可以通过动态创建script标签的形式，把script标签的src属性，指向数据接口的地址，因为script标签不存在跨域限制，这种数据获取方式，称作JSONP（注意：根据JSONP的实现原理，知晓，JSONP只支持Get请求）；
- 具体实现过程：

- 先在客户端定义一个回调方法，预定义对数据的操作；
- 再把这个回调方法的名称，通过URL传参的形式，提交到服务器的数据接口；
- 服务器数据接口组织好要发送给客户端的数据，再拿着客户端传递过来的回调方法名称，拼接出一个调用这个方法的字符串，发送给客户端去解析执行；
- 客户端拿到服务器返回的字符串之后，当作Script脚本去解析执行，这样就能够拿到JSONP的数据了；

- 带大家通过 Node.js ，来手动实现一个JSONP的请求例子；

```js
// global Vue object
Vue.http.get('/someUrl', [config]).then(successCallback, errorCallback);
Vue.http.post('/someUrl', [body], [config]).then(successCallback, errorCallback);

// in a Vue instance
this.$http.get('/someUrl', [config]).then(successCallback, errorCallback);
this.$http.post('/someUrl', [body], [config]).then(successCallback, errorCallback);
```

List of shortcut methods:

- `get(url, [config])`
- `head(url, [config])`
- `delete(url, [config])`
- `jsonp(url, [config])`
- `post(url, [body], [config])`
- `put(url, [body], [config])`
- `patch(url, [body], [config])`

## Config

| Parameter        | Type                           | Description                                                  |
| ---------------- | ------------------------------ | ------------------------------------------------------------ |
| url              | `string`                       | 请求地址                                                     |
| body             | `Object`, `FormData`, `string` | Data to be sent as the request body                          |
| headers          | `Object`                       | Headers object to be sent as HTTP request headers            |
| params           | `Object`                       | Parameters object to be sent as URL parameters               |
| method           | `string`                       | HTTP method (e.g. GET, POST, ...)                            |
| responseType     | `string`                       | Type of the response body (e.g. text, blob, json, ...)       |
| timeout          | `number`                       | Request timeout in milliseconds (`0` means no timeout)       |
| credentials      | `boolean`                      | Indicates whether or not cross-site Access-Control requests should be made using credentials |
| emulateHTTP      | `boolean`                      | Send PUT, PATCH and DELETE requests with a HTTP POST and set the `X-HTTP-Method-Override` header |
| emulateJSON      | `boolean`                      | Send request body as `application/x-www-form-urlencoded` content type |
| before           | `function(request)`            | Callback function to modify the request object before it is sent |
| uploadProgress   | `function(event)`              | Callback function to handle the [ProgressEvent](https://developer.mozilla.org/en-US/docs/Web/API/ProgressEvent) of uploads |
| downloadProgress | `function(event)`              | Callback function to handle the [ProgressEvent](https://developer.mozilla.org/en-US/docs/Web/API/ProgressEvent) of downloads |

## Response

A request resolves to a response object with the following properties and methods:

| Property   | Type                       | Description                             |
| ---------- | -------------------------- | --------------------------------------- |
| url        | `string`                   | Response URL origin                     |
| body       | `Object`, `Blob`, `string` | Response body                           |
| headers    | `Header`                   | Response Headers object                 |
| ok         | `boolean`                  | HTTP status code between 200 and 299    |
| status     | `number`                   | HTTP status code of the response        |
| statusText | `string`                   | HTTP status text of the response        |
| **Method** | **Type**                   | **Description**                         |
| text()     | `Promise`                  | Resolves the body as string             |
| json()     | `Promise`                  | Resolves the body as parsed JSON object |
| blob()     | `Promise`                  | Resolves the body as Blob object        |

## Example

> 示例1

```html
<div id="app">
        <input type="button" value="get请求" @click="getInfo">
        <input type="button" value="post请求" @click="postInfo">
        <input type="button" value="jsonP请求" @click="jsonpInfo">
    </div>
    <script>
        //创建Vue实例，得到ViewModel
        var vm = new Vue({
            el: '#app',
            data: {
                
            },
            methods: { //methods定义vue实例所有可用的方法
                getInfo(){
                    // get请求
                    this.$http.get('/ http://vue.studyit.io/api/getlunbo').then(function (result) {
                        console.log(result.body);
                    });
                },
                postInfo(){
                    // post请求  application/x-www-form-urlencoded
                    // 手动发起的post请求，默认没有表单格式，有的服务器不能处理
                    // 设置提交的类型为普通的表单
                    this.$http.post('http://vue.studyit.io/api/post', {},{emulateJSON:true}).then(function (result) {
                        console.log(result.body);
                    });
                },
                jsonpInfo(){
                    // jsonp请求
                    this.$http.jsonp('http://vue.studyit.io/api/jsonp').then(function (result) {
                        console.log(result.body);
                    });
                }
            },
        });
    </script>
```
> 示例2：全局配置数据的根接口

Set default values using the global configuration.

```js
Vue.http.options.root = 'http://vue.studyit.io';
Vue.http.headers.common['Authorization'] = 'Basic YXBpOnBhc3N3b3Jk';
```

Set default values inside your Vue component options.

全局配置：

```js
Vue.http.options.root='http://vue.studyit.io'
```

局部配置：

```js
new Vue({

  http: {
    root: '/root',
    headers: {
      Authorization: 'Basic YXBpOnBhc3N3b3Jk'
    }
  }
})
```
:warning: Note that for the root option to work, the path of the request must be relative(相对路径). This will use this the root option: Vue.http.get('someUrl') while this will not: Vue.http.get('/someUrl').

> 是使用路径的时候使用的是相对路径，绝对路径就会出错

使用

```js
 this.$http.get('api/delproduct/'+id).then(function (result) {})
```

全局配置POST请求的方式：(以表单的方式发送请求)

If your web server can't handle requests encoded as `application/json`, you can enable the `emulateJSON` option. This will send the request as `application/x-www-form-urlencoded` MIME type, as if from an normal HTML form.

```js
Vue.http.options.emulateJSON = true;
```

