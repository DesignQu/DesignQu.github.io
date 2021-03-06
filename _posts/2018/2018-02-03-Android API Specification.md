---
layout: post
title: "APP客户端与服务器通信接口规范化设计手册"
excerpt: "整理一些APP客户端与API规范化的内容"
date: 2018-2-3
categories:
  - 笔记
tags:
  - API
  - 规范化
  - 版本
  - 安全
---

-------------------

## 0x00 接口版本设计

> 版本控制的意义在于方便接口的维护与日常更新。当应用已经上线，接口发生逻辑更新，为不影响历史版本的使用应增加接口版本管理。

**常用方法**  
1、如业务改动较小。根据App调用接口传入接口版本号，服务器业务逻辑做相应区分处理。  
2、如业务改动较大，应启用新接口，在BaseAPI后端加入版本号例v1、v2、v3
![1](/assets/image/2018-02-03/2018-02-03-Android API Specification 1.png)  

-------------------

## 0x01 数据传输

> 接口的数据一般都采用Json格式进行传输，Json的值只有六种数据类型

| :------:| :------------------  |
| Number  |  整数或浮点数        |
| String  |  字符串              |
| Boolean | true 或 false        |
| Array   | 数组包含在方括号[]中 |
| Object  | 对象包含在大括号{}中 |
| Null    | 空类型               |


传输的数据类型不能超过这六种数据类型。如果强制性传输Date类型字段，在不同平台不同解析库的解析方式可能不同，有的可能会转乱，有的可能直接发生异常。解决方案是用时间戳。
  
如果出现空的字段，该字段应不必包含在数据包里，或者赋值为“”或null，不可以出现字符串的"true"、"false"、"null"等，比如 String 用””，int 用 0，Object 用 {}，Array 用 []。
  
#### 数据结构类型
```
{
    "code": 1,
    "message": "no message",
    "data": {}
}
```
以上为常用字段，根据实际需求进行修改。

1、状态码  
应该全局定义统一状态码（code），而不应该每个接口单独去定义。如code=1时表示登录成功，那么其他接口code=1的状态就不能再使用，如此定义后，前端可以进行全局的统一处理。
常见错误状态码类型
- 普通异常
- token不合法，需要重新登录
- 重复登录
- 需要完善个人信息
- 第三方账号登陆，需要绑定官方账号
- 数据解密错误

2、data字段  
Data字段可包含JsonArray或JsonObject，这里只关心想要获取的数据，如果出现空的字段，该字段应不必包含在数据包里，或者赋值为“”或null。

-------------------

## 0x02 接口安全机制
> 主要介绍这三种Token、Timestamp和Sign，保证接口的数据不会被篡改和重复调用

### 1、Token授权机制  
用户使用用户名密码登录后服务器给客户端返回一个唯一的Token值，并将Token-UserId以键值对的形式存放在缓存服务器中。服务端接收到请求后进行Token验证，如果Token不存在，说明请求无效。Token是客户端访问服务端的凭证。

### 2、时间戳超时机制  
用户每次请求都带上当前时间的时间戳timestamp，服务端接收到timestamp后跟当前时间进行比对，如果时间差大于一定时间（比如5分钟），则认为该请求失效。时间戳超时机制是防御DOS攻击的有效手段。

### 3、签名机制  
将 Token 和 Timestamp 或其他请求参数再用MD5或其他加密算法加密，加密后的数据就是本次请求的签名sign，服务端接收到请求后以同样的算法得到签名，并跟当前的签名进行比对，如果不一样，说明参数被更改过，直接返回错误标识。签名机制保证了数据不会被篡改。

### 4、拒绝重复调用（非必须）  
客户端第一次访问时，将签名sign存放到缓存服务器中，超时时间设定为跟时间戳的超时时间一致，二者时间一致可以保证无论在timestamp限定时间内还是外 URL都只能访问一次。如果有人使用同一个URL再次访问，如果发现缓存服务器中已经存在了本次签名，则拒绝服务。如果在缓存中的签名失效的情况下，有人使用同一个URL再次访问，则会被时间戳超时机制拦截。这就是为什么要求时间戳的超时时间要设定为跟时间戳的超时时间一致。拒绝重复调用机制确保URL被别人截获了也无法使用（如抓取数据）。

#### 整个流程
1、客户端通过用户名密码登录服务器并获取Token  
2、客户端生成时间戳timestamp，并将timestamp作为其中一个参数  
3、客户端将所有的参数，包括Token和timestamp按照自己的算法进行排序加密得到签名sign  
4、将token、timestamp和sign作为请求时必须携带的参数加在每个请求的URL后边（http://url/request?token=123&timestamp=123&sign=123123123）
5、服务端写一个过滤器对token、timestamp和sign进行验证，只有在token有效、timestamp未超时、缓存服务器中不存在sign三种情况同时满足，本次请求才有效
  
在以上三中机制的保护下，如果有人劫持了你的请求，并对请求中的参数进行了修改，签名就无法通过；
如果有人使用已经劫持的URL进行DOS攻击，服务器则会因为缓存服务器中已经存在签名或时间戳超时而拒绝服务，所以DOS攻击也是不可能的；

### 5、HTTPS
HTTPS能够有效防止中间人攻击，有效保证接口不被劫持，对数据窃取篡改做了安全防范。但HTTP升级HTTPS会带来更多的握手，而握手中的运算会带来更多的性能消耗。这也是不得不考虑的问题。
总得来说，我们非常有必要在设计接口的同时考虑安全性的问题，根据业务特点，采用的安全策略也不全相同。当然大多数安全策略更多的都是提高安全门槛，并不能保证100%的安全，但该做的还是不能少。

参考[API接口安全性设计](https://www.jianshu.com/p/c6518a8f4040)  
参考[Android 密钥保护和 C/S 网络传输安全理论指南](http://drakeet.me/android-security-guide/)

## 0x03 其他
### 1、接口应该按模块化划分还是页面划分？  
按页面的接口尽可能让前端一个页面只请求一次，一次返回所需要的全部信息；按模块的接口在后端定义自己的业务模块如用户、标签、搜索等，并尽量避免模块间的耦合。

从后端角度来说，按模块当然是更好的（只需要划分地够细就好），到时候需求有什么变更，让前端自己去改变接口的组合就好。但从前端的角度来说，接口的组合涉及到异步之间的关系，尽管RxJava这样的响应式编程框架让异步简单了很多，但仍然希望可以避免，更严重的是，多次接口请求会让前端的体验变差，并行接口的影响稍小，而一些有前置后置关系的接口则麻烦比较大，一个接着一个请求，会让用户等很久。即便是并行接口，有时候页面的渲染仍然需要所有接口数据返回后才可以进行。
但如果让后端按照页面去套，这样在后端其实一样有性能的损耗，需要一个页面接口去单独调用各个模块的接口，然后进行组合。
那么折中办法是在服务器性能足够的前提下，后端应该尽量减少页面请求次数，尤其是有依赖关系的串行请求。
另外，一个好的设计师，也应该会贯彻一个地方只应该以一样东西为主体，而不应该去把乱七八糟的东西拼凑在一起。

### 2、过滤逻辑应该写到前端还是后端？
客户端设备用户可能root、越狱，甚至可以反编译客户端或者直接模拟请求。
那么良好的配置检查应该前后端都需要，后端收到请求自行检查过滤，如果出错则返回错误信息给前端显示。
  
### 3、空字段  
空字段，如果没有，服务端在返回的在数据集里应该不包含此字段，或返回一个空的默认字段 比如 String用””，int用0，Object用{}，Array用[]。

### 4、RESTful API 遵循原则
- 使用https协议
- 版本号放入URL
- 只提供json返回格式
- post,put上使用json作为输入
- 使用http状态码作为错误提示
- Path（路径）尽量使用名词，不使用动词，把每个URL看成一个资源
- 使用HTTP动词（GET,POST,PUT,DELETE）作为action操作URL资源
- 过滤信息
  - limit：指定返回记录数量
  - offset：记录开始位置
  - direction：请求数据的方向，取值prev-上一页数据；next-下一页数据
  - page：第几页
  - per_page：每页条数
  - total_count：总记录数
  - total_pages：总页数，等于page时，表示当前是最后一页
  - sort：column1,column2排序字段
  - orderby：排序规则，desc或asc
  - q：搜索关键字（uri encode之后的）
- 返回结果
  - GET：返回资源对象
  - POST：返回新生成的资源对象
  - PUT：返回完整的资源对象
  - DELETE：返回一个空文档
  
-------------------

## 0x04 相关链接
- [Android 密钥保护和 C/S 网络传输安全理论指南](http://drakeet.me/android-security-guide/) 这篇写的非常好，讲解了整套关于 Android 密钥、Http 安全方面的原理及思想，干货