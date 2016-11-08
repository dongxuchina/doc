### 一、协议 ###
```
1. API与用户的通信协议，使用HTTP/HTTPs协议。
```
### 二、域名 ###
```
1. 将API部署在专用域名之下
```
### 三、版本（Versioning）###
应该将API的版本号放入URL。

```
    https://api.xiaoyun.com/v1/
```
### 四、路径（Endpoint）###

路径又称"终点"（endpoint），表示API的具体网址。 在RESTful架构中，每个网址代表一种资源（resource），所以网址中不能有动词，只能有名词，而且所用的名词往往与数据库的表格名对应。一般来说，数据库中的表都是同种记录的"集合"（collection），所以API中的名词也应该使用复数。 举例来说，有一个API提供动物园（zoo）的信息，还包括各种动物和雇员的信息，则它的路径应该设计成下面这样。

```
    https://api.example.com/v1/zoos
    https://api.example.com/v1/animals
    https://api.example.com/v1/employees
```
### 五、HTTP动词 ###
对于资源的具体操作类型，由HTTP动词表示。 常用的HTTP动词有下面五个（括号里是对应的SQL命令）。

```
    GET（SELECT）：从服务器取出资源（一项或多项）。
    POST（CREATE）：在服务器新建一个资源。
    PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。
    PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）。
    DELETE（DELETE）：从服务器删除资源。
```
还有两个不常用的HTTP动词。

```
    HEAD：获取资源的元数据。
    OPTIONS：获取信息，关于资源的哪些属性是客户端可以改变的。
```
下面是一些例子。

```
    GET /zoos：列出所有动物园
    POST /zoos：新建一个动物园
    GET /zoos/ID：获取某个指定动物园的信息
    PUT /zoos/ID：更新某个指定动物园的信息（提供该动物园的全部信息）
    PATCH /zoos/ID：更新某个指定动物园的信息（提供该动物园的部分信息）
    DELETE /zoos/ID：删除某个动物园
    GET /zoos/ID/animals：列出某个指定动物园的所有动物
    DELETE /zoos/ID/animals/ID：删除某个指定动物园的指定动物
```
### 六、过滤信息（Filtering）###
如果记录数量很多，服务器不可能都将它们返回给用户。API应该提供参数，过滤返回结果。 下面是一些常见的参数。

```
        ?page=2&per_page=100：指定第几页，以及每页的记录数。
        ?animal_type_id=1：指定筛选条件
```
### 七、状态码（Status Codes）###
服务器向用户返回的状态码和提示信息，常见的有以下一些（方括号中是该状态码对应的HTTP动词）。

```
200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
204 NO CONTENT - [DELETE]：用户删除数据成功。
400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。
```
### 八、错误处理（Error handling）###
如果状态码是4xx，就应该向用户返回出错信息。

```
{
    code: "10101110",
    message: "Invalid API key",
    toUser: true,// 是否是展现给用户的错误；
    ext:{}
}
```
code: 错误码 message：错误描述 toUser: boolean //是否显示给用户（考虑是否增加） ext: {}, 扩展信息（考虑是否增加）
### 九、返回结果 ###
针对不同操作，服务器向用户返回的结果应该符合以下规范。 服务器返回的数据格式，使用JSON。

```
    GET /collection：返回资源对象的列表（数组）
    GET /collection/resource：返回单个资源对象
    POST /collection：返回新生成的资源对象
    PUT /collection/resource：返回完整的资源对象
    PATCH /collection/resource：返回完整的资源对象
    DELETE /collection/resource：返回一个空文档
```
### 十、其他 ###

```
1. API的身份认证应该使用Kong框架。
2.  服务器返回的数据格式，使用JSON。
```
### 业务错误码定义 ###
* 业务代码为全局定义，总计8位；
* 8位数字分为四组，前面两位，代表系统级别
* 一组数组代表一个级别；
* 例如：10101010

```
    10****** 中心数据
    11****** DZ站  
    12****** ios客户端
    13****** android 客户端
    14****** 服务市场
    15****** 商城
    16****** 微站
    17****** 钱包
    18****** 开发者开放平台
    19****** 直播
    20****** UI
    21****** Oauth2.0
```

### 国际化支持 ###
```
headers 里面添加 Accept-Language 参数
```