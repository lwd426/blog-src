title: 10个有关RESTful API良好设计的最佳实践（转）
date: 2014-04-05 10:09:03
tags:
- RESTful
categories: RESTful

---

Web API已经在最近几年变成重要的话题，一个干净的API设计对于后端系统是非常重要的。

通常我们为Web API使用RESTful设计，`REST概念分离了API结构和逻辑资源，通过Http方法GET, DELETE, POST 和 PUT来操作资源。`

下面是进行RESTful Web API十个最佳实践，能为你提供一个良好的API设计风格。
<!-- more -->
 
#### 使用名词而不是动词

| Resource | GET  | POST  | PUT | DELETE |
| ------ |:------:| -----:| -----:| -----:|
| 资源      | 读  | 创建  | 删除 | 删除 |
| /cars      | 返回 cars集合  | 创建新的资源  | 批量更新cars | 删除所有cars |
| /cars/711      | 返回特定的car  | 该方法不允许(405)  | 更新一个指定的资源 | 擅长指定资源 |

不要使用：

* /getAllCars
* /createNewCar
* /deleteAllRedCars

 

#### Get方法和查询参数不应该涉及状态改变

使用PUT, POST 和DELETE 方法 而不是 GET 方法来改变状态，不要使用GET 进行状态改变:

* GET /users/711?activate 
* GET /users/711/activate


#### 使用复数名词

不要混淆名词单数和复数，为了保持简单，只对所有资源使用复数。

* /cars 而不是 /car
* /users 而不是 /user
* /products 而不是 /product
* /settings 而部署 /setting

 

#### 使用子资源表达关系

如果一个资源与另外一个资源有关系，使用子资源：

* GET /cars/711/drivers/ 返回 car 711的所有司机
* GET /cars/711/drivers/4 返回 car 711的4号司机

#### 使用Http头声明序列化格式

在客户端和服务端，双方都要知道通讯的格式，格式在HTTP-Header中指定

* Content-Type 定义请求格式
* Accept 定义系列可接受的响应格式

#### 使用HATEOAS

Hypermedia as the Engine of Application State 超媒体作为应用状态的引擎，超文本链接可以建立更好的文本浏览：

```javascript
{
  "id": 711,
  "manufacturer": "bmw",
  "model": "X5",
  "seats": 5,
  "drivers": [
   {
    "id": "23",
    "name": "Stefan Jauker",
    "links": [
     {
     "rel": "self",
     "href": "/api/v1/drivers/23"
    }
   ]
  }
 ]
}
```

注意href指向下一个URL

#### 为集合提供过滤 排序 选择和分页等功能

1. Filtering过滤:

   使用唯一的查询参数进行过滤：

  * GET /cars?color=red 返回红色的cars
  * GET /cars?seats<=2 返回小于两座位的cars集合

2. Sorting排序:
 
  允许针对多个字段排序

 * GET /cars?sort=-manufactorer,+model

  这是返回根据生产者降序和模型升序排列的car集合

3. Field selection

  移动端能够显示其中一些字段，它们其实不需要一个资源的所有字段，给API消费者一个选择字段的能力，这会降低网络流量，提高API可用性。

 * GET /cars?fields=manufacturer,model,id,color

4. Paging分页

  使用 limit 和offset.实现分页，缺省limit=20 和offset=0；

  * GET /cars?offset=10&limit=5

  为了将总数发给客户端，使用订制的HTTP头： X-Total-Count.

  链接到下一页或上一页可以在HTTP头的link规定，遵循Link规定:

  * Link: <https://blog.mwaysolutions.com/sample/api/v1/cars?offset=15&limit=5>; rel="next",
  * <https://blog.mwaysolutions.com/sample/api/v1/cars?offset=50&limit=3>; rel="last",
  * <https://blog.mwaysolutions.com/sample/api/v1/cars?* offset=0&limit=5>; rel="first",
  * <https://blog.mwaysolutions.com/sample/api/v1/cars?offset=5&limit=5>; rel="prev",

 

#### 版本化你的API

使得API版本变得强制性，不要发布无版本的API，使用简单数字，避免小数点如2.5.

一般在Url后面使用?v

* /blog/api/v1

 

#### 使用Http状态码处理错误

如果你的API没有错误处理是很难的，只是返回500和出错堆栈不一定有用

Http状态码提供70个出错，我们只要使用10个左右：

* 200 – OK – 一切正常
* 201 – OK – 新的资源已经成功创建
* 204 – OK – 资源已经成功擅长

* 304 – Not Modified – 客户端使用缓存数据

* 400 – Bad Request – 请求无效，需要附加细节解释如 "JSON无效"
* 401 – Unauthorized – 请求需要用户验证
* 403 – Forbidden – 服务器已经理解了请求，但是拒绝服务或这种请求的访问是不允许的。
* 404 – Not found – 没有发现该资源
* 422 – Unprocessable Entity – 只有服务器不能处理实体时使用，比如图像不能被格式化，或者重要字段丢失。

* 500 – Internal Server Error – API开发者应该避免这种错误。

使用详细的错误包装错误：

```javascript
{
  "errors": [
   {
    "userMessage": "Sorry, the requested resource does not exist",
    "internalMessage": "No car found in the database",
    "code": 34,
    "more info": "http://dev.mwaysolutions.com/blog/api/v1/errors/12345"
   }
  ]
}
```
#### 允许覆盖http方法

一些代理只支持POST 和 GET方法， 为了使用这些有限方法支持RESTful API，需要一种办法覆盖http原来的方法。

使用订制的HTTP头 X-HTTP-Method-Override 来覆盖POST 方法.

*注释：此文是转的，但是转载的地方没有写他转自哪里，所以没有找到连接。*