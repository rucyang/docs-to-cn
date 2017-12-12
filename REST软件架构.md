# 相关概念
## REST（Representational State Transfer）
翻译成中文就是“表现成状态转换”

* 资源: 每个URI代表一种资源，表示网络上的一个实体
* 表现层：资源的表现信息等以元数据的形式存在于HTTP请求头中，如Accept字段和Content-Type字段
* HTTP协议：HTTP协议是无状态协议，所有的状态都保存再服务器端，客户端用户每次通过发送HTTP请求来改变服务器端的状态信息。
  1. GET用来获取资源
  2. POST用来新建资源（或者更新资源）
  3. PUT用来更新资源
  4. DELETE用来删除资源

## 架构思想
1. 每个URI代表一种资源
2. 客户端与服务器之间，传递这种资源的某种表现形式
3. 客户端通过四个HTTP动词，对服务器端的资源进行操作，实现“表现层状态转化”
4. 所有的状态信息，操作信息都应当通过HTTP协议来实现，而URI只表示一个网络资源或一种网络服务

## 常见的设计误区
1. URI包含动词：URI表示资源实体，不应该包含动词，动词应该放在HTTP请求中。如果有些动作是HTTP请求表示不了的，应该将该动作表示为一种资源（或服务），使用该动词的名词形式。
2. URI中包含版本号：不同版本号对应的都是同一资源，应该使用相同的URI，而其版本信息是该资源的不同状态，应该放到请求的字段中去。


# RESTful API设计指南

## 协议
API与用户的通信协议总是使用HTTP协议

## 域名
API尽量部署在专用的域名之下，如`https://api.example.com`；如果API比较简单，不会再拓展，可以考虑放在主域名下，如`https://example.com/api/`

## 版本
应将API的版本放在URL中，如`https://api.exmaple.com/v1/`

## 路径（Endpoint）
路径又称终点，表示api的具体网址。

在RESTful架构中，每个网址代表一种资源（resource），所以网址中不能有动词，只能是名词，而且所用的名词往往与数据库的表格名相对应。一般来说，数据库中表都是同种记录的集合（collection），所以API中的名词也应该使用复数。如：
* `https://api.example.com/v1/zoos`
* `https://apu.example.com/v1/employees`

## HTTP动词

* GET（SELECT）： 从服务器取出资源（一项或者多项）
* POST（CRAETE）： 在服务器新建一个资源
* PUT（UPDATE）： 在服务器更新资源（客户端改变后的完整资源）
* PATCH（UPDATE）： 在服务器更新资源（客户端提供改变的属性）
* DELETE（DELETE）： 从服务器删除资源
* HEAD： 获取资源的元数据
* OPTIONS： 获取信息，关于资源的那些属性是客户端可以改变的。

* `GET /zoos`: 列出所有动物园
* `GET /zoos/ID`: 获取指定动物园信息
* `POST /zoos`: 新建一个动物园
* `PUT /zoos/ID`: 更新某个指定动物园的信息（提供该动物园的全部信息）
* `PATCH /zoos/ID`: 更新指定动物园的信息（提供该动物园的部分信息）
* `DELETE /zoos/ID`: 删除指定动物园
* `GET /zoos/ID/animals`: 获取指定动物园的所有动物

## 过滤信息
如果记录数量过多，服务器不可能将它们全部返回给用户。API应提供参数，过滤返回结果。
* `?limit=10`: 指定返回记录的数量
* `?offset=10`: 指定返回记录开始的位置
* `?page=2&per_page=100`: 指定第几页，以及每页记录数
* `?sortby=name&order=asc`: 指定返回结果按照哪个属性排序，以及排序顺序
* `animal_type_id=1`：指定筛选条件

参数的设计允许存在冗余，即允许API路径和URL参数偶尔有重复。

## 状态码
服务器向用户返回的状态吗和提示信息，常见如下：
* `200 OK - [GET]`: 服务器成功返回用户请求的数据，该操作是等幂的（Idempotent）
* `201 CREATED - [POST/PUT/PATCH]`: 用户新建或修改数据成功
* `202 Accepted - [*]`: 表示一个请求已经进入后台排队（异步任务）
* `204 NO CONTENT - [DELETE]`: 用户删除数据成功
* `400 INVALID REQUEST - [POST/PUT/PATCH]`: 用户发出的请求有错误，服务器没有进行新建或修改数据，该操作是等幂的
* `401 Unauthorized - [*]`: 表示用户没有权限（令牌，用户名，密码错误等）
* `403 Forbidden - [*]`: 表示用户得到授权，但是访问被禁止
* `404 NOT FOUND - [*]`: 用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是等幂的
* `406 NOT Acceptable - [GET]`: 用户请求的格式不可得，（比如用户请求的是JSON格式，但是只有XML格式）
* `410 Gone - [GET]`: 用户请求的资源已被永久删除，且不会再得到
* `422 Unprocesable entity - [POST/PUT/PATCH]`: 当创建一个对象时，发生一个验证错误
* `500 INTERNAL SERVER ERROR - [*]`: 服务器发生错误，用户将无法判断发出的请求是否成功

## 错误信息
如果状态码是4xx，就应该向用户返回出错信息。一般来说，返回的信息中将error作为键名，出错信息作为键值即可
```
{
  error: "Invalid API key"
}
```

## 返回结果
针对不同操作，服务器向用户返回的结果应该符合以下规范。
* `GET /collection`: 返回资源对象的列表（数组）
* `GET /collection/resource`: 返回单个资源对象
* `POST /collection`: 返回生成的新的资源对象
* `PUT /collection`: 返回完整的资源对象
* `PATCH /collection/resource`: 返回完整的资源对象
* `DELETE /collection/resource`: 返回一个空文档

## Hypermedia API
RESTful API最好做到Hypermedia， 即返回结果中提供链接，连向其他API方法，使得用户不查文档，也知道下一步要做什么。

## 其他
* API的身份认证应使用OAuth 2.0框架。
* 服务器返回的数据格式，应该尽量使用JSON，避免使用XML


参考资料：
1. 阮一峰，[理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful.html)
2. 阮一峰，[RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)
