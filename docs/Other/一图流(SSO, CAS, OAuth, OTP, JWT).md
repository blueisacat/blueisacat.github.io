---
layout: default
title: 一图流(SSO, CAS, OAuth, OTP, JWT)
parent: Other
---

# 一、概念

在目前很多应用中，内部存在着很多的系统，而每一个系统都需要用户实现登入、获取用户信息，用户可能需要在多个系统中进行重复登入

## 1. 单点登入（SSO）

是一种思想、一种架构

- 在所有的系统外部存在一个独立的认证中心，所有子系统的身份认证都可以统一使用该认证中心进行用户身份的认证与注册。

- 用户只需要登入一次，不需要在多个子系统中进行重复的身份认证

- 有多种实现方式


- 同域名的cookie建立全局的登入态

- CAS

## 2. 中心授权服务（CAS）

是一个开源协议，是SSO思想的一种实现

![](../../assets/images/Other/attachments/一图流(SSO,%20CAS,%20OAuth,%20OTP,%20JWT)_image_0.png)

## 3. OAuth2.0

是一种授权协议，而不是一种认证协议

- 授权强调应用是否获取用户的允许，是否能否访问资源

- 认证强调用户身份的鉴定

![](../../assets/images/Other/attachments/一图流(SSO,%20CAS,%20OAuth,%20OTP,%20JWT)_image_1.png)

## 4. 一次性口令（OTP）

一种双因素校验的手段，是一种增强验证的方式，同时是一次性的

双因素校验有：

- 短信验证码

- 图形验证码（可以通过OCR技术破解）

- OTP技术


- 服务器通过安全的方式将密钥告诉客户端

- TOTP（时间同步），运行的时候，每过一段时间口令发生变化


- 使用密钥以及时间作为参数生成OTP码

- 时间偏移可能会导致OTP码不一致

- HOTP（事件同步），每次运行的时候，口令都会发生变化


- 使用密钥以及计数器作为参数生成OTP码

- 由于次数并不多，同时OTP码的过期时间不算短，存在被暴力破解的可能性

- 客户端与服务器的计数匹配不上时，如何进行纠正


- 客户端计数器少于服务器时

- 服务器计数器少于客户端时

## 5. JSON Web Token（JWT）

也是一种token，用于传递用户信息，可以通过解码的方式获得JWT携带的用户信息，并且带有校验信息

![](../../assets/images/Other/attachments/一图流(SSO,%20CAS,%20OAuth,%20OTP,%20JWT)_image_2.png)

## 6. CSRF

是一种攻击手段，利用在该域名下已种下的cookie，发送敏感请求到对应网站，有许多方法可以防止

- 二次验证

- 验证refer字段

- 使用token，请求必须在指定header字段带上token信息，或者在参数中显式带上


- 将token作为用户信息的一部分，从session中获取用户信息后，进行token比对

- 把加密算法作为非对称信息，将密钥以及token都返回给用户，但是由于黑客不知道使用的加密算法，无法进行伪造


- 加密可以视为非对称信息使用

# 二、实现

## 1. 什么是单点登录？

单点登录的英文名叫做：Single Sign On（简称SSO）。

在初学/以前的时候，一般我们就单系统，所有的功能都在同一个系统上。

![](../../assets/images/Other/attachments/一图流(SSO,%20CAS,%20OAuth,%20OTP,%20JWT)_image_3.png)

后来，我们为了合理利用资源和降低耦合性，于是把单系统拆分成多个子系统。

![](../../assets/images/Other/attachments/一图流(SSO,%20CAS,%20OAuth,%20OTP,%20JWT)_image_4.png)

比如阿里系的淘宝和天猫，很明显地我们可以知道这是两个系统，但是你在使用的时候，登录了天猫，淘宝也会自动登录。

![](../../assets/images/Other/attachments/一图流(SSO,%20CAS,%20OAuth,%20OTP,%20JWT)_image_5.png)

简单来说，单点登录就是在多个系统中，用户只需一次登录，各个系统即可感知该用户已经登录。

## 2. 回顾单系统登录

在我初学JavaWeb的时候，登录和注册是我做得最多的一个功能了（初学Servlet的时候做过、学SpringMVC的时候做过、跟着做项目的时候做过…），反正我也数不清我做了多少次登录和注册的功能了...这里简单讲述一下我们初学时是怎么做登录功能的。

众所周知，HTTP是无状态的协议，这意味着服务器无法确认用户的信息。于是乎，W3C就提出了：给每一个用户都发一个通行证，无论谁访问的时候都需要携带通行证，这样服务器就可以从通行证上确认用户的信息。通行证就是Cookie。

如果说Cookie是检查用户身上的”通行证“来确认用户的身份，那么Session就是通过检查服务器上的”客户明细表“来确认用户的身份的。Session相当于在服务器中建立了一份“客户明细表”。

HTTP协议是无状态的，Session不能依据HTTP连接来判断是否为同一个用户。于是乎：服务器向用户浏览器发送了一个名为JESSIONID的Cookie，它的值是Session的id值。其实Session是依据Cookie来识别是否是同一个用户。

所以，一般我们单系统实现登录会这样做：

- 登录：将用户信息保存在Session对象中


- 如果在Session对象中能查到，说明已经登录

- 如果在Session对象中查不到，说明没登录（或者已经退出了登录）

- 注销（退出登录）：从Session中删除用户的信息

- 记住我（关闭掉浏览器后，重新打开浏览器还能保持登录状态）：配合Cookie来用

## 3. 多系统登录的问题与解决

### 3.1 Session不共享问题

单系统登录功能主要是用Session保存用户信息来实现的，但我们清楚的是：多系统即可能有多个Tomcat，而Session是依赖当前系统的Tomcat，所以系统A的Session和系统B的Session是不共享的。

![](../../assets/images/Other/attachments/一图流(SSO,%20CAS,%20OAuth,%20OTP,%20JWT)_image_6.png)

解决系统之间Session不共享问题有一下几种方案：

- Tomcat集群Session全局复制（集群内每个tomcat的session完全同步）【会影响集群的性能呢，不建议】

- 根据请求的IP进行Hash映射到对应的机器上（这就相当于请求的IP一直会访问同一个服务器）【如果服务器宕机了，会丢失了一大部分Session的数据，不建议】

- 把Session数据放在Redis中（使用Redis模拟Session）【建议】


- 如果还不了解Redis的同学，建议移步（Redis合集）

我们可以将登录功能单独抽取出来，做成一个子系统。

![](../../assets/images/Other/attachments/一图流(SSO,%20CAS,%20OAuth,%20OTP,%20JWT)_image_7.png)

总结：

- SSO系统生成一个token，并将用户信息存到Redis中，并设置过期时间

- 其他系统请求SSO系统进行登录，得到SSO返回的token，写到Cookie中

- 每次请求时，Cookie都会带上，拦截器得到token，判断是否已经登录

到这里，其实我们会发现其实就两个变化：

- 将登陆功能抽取为一个系统（SSO），其他系统请求SSO进行登录

- 本来将用户信息存到Session，现在将用户信息存到Redis

### 3.2 Cookie跨域的问题

上面我们解决了Session不能共享的问题，但其实还有另一个问题。Cookie是不能跨域的比如说，我们请求[https://www.google.com/](https://www.google.com/)时，浏览器会自动把google.com的Cookie带过去给google的服务器，而不会把[https://www.baidu.com/](https://www.baidu.com/)的Cookie带过去给google的服务器。

这就意味着，由于域名不同，用户向系统A登录后，系统A返回给浏览器的Cookie，用户再请求系统B的时候不会将系统A的Cookie带过去。

针对Cookie存在跨域问题，有几种解决方案：

- 服务端将Cookie写到客户端后，客户端对Cookie进行解析，将Token解析出来，此后请求都把这个Token带上就行了

- 多个域名共享Cookie，在写到客户端的时候设置Cookie的domain。

- 将Token保存在SessionStroage中（不依赖Cookie就没有跨域的问题了）

到这里，我们已经可以实现单点登录了。

### 3.3 CAS原理

说到单点登录，就肯定会见到这个名词：CAS （Central Authentication Service），下面说说CAS是怎么搞的。

如果已经将登录单独抽取成系统出来，我们还能这样玩。现在我们有两个系统，分别是www.java3y.com和www.java4y.com，一个SSOwww.sso.com

![](../../assets/images/Other/attachments/一图流(SSO,%20CAS,%20OAuth,%20OTP,%20JWT)_image_8.png)

首先，用户想要访问系统Awww.java3y.com受限的资源(比如说购物车功能，购物车功能需要登录后才能访问)，系统Awww.java3y.com发现用户并没有登录，于是重定向到sso认证中心，并将自己的地址作为参数。请求的地址如下：

- www.sso.com?service=www.java3y.com

sso认证中心发现用户未登录，将用户引导至登录页面，用户进行输入用户名和密码进行登录，用户与认证中心建立全局会话（生成一份Token，写到Cookie中，保存在浏览器上）

![](../../assets/images/Other/attachments/一图流(SSO,%20CAS,%20OAuth,%20OTP,%20JWT)_image_9.png)

随后，认证中心重定向回系统A，并把Token携带过去给系统A，重定向的地址如下：

- www.java3y.com?token=xxxxxxx

接着，系统A去sso认证中心验证这个Token是否正确，如果正确，则系统A和用户建立局部会话（创建Session）。到此，系统A和用户已经是登录状态了。

![](../../assets/images/Other/attachments/一图流(SSO,%20CAS,%20OAuth,%20OTP,%20JWT)_image_10.png)

此时，用户想要访问系统Bwww.java4y.com受限的资源(比如说订单功能，订单功能需要登录后才能访问)，系统Bwww.java4y.com发现用户并没有登录，于是重定向到sso认证中心，并将自己的地址作为参数。请求的地址如下：

- www.sso.com?service=www.java4y.com

注意，因为之前用户与认证中心www.sso.com已经建立了全局会话（当时已经把Cookie保存到浏览器上了），所以这次系统B重定向到认证中心www.sso.com是可以带上Cookie的。

认证中心根据带过来的Cookie发现已经与用户建立了全局会话了，认证中心重定向回系统B，并把Token携带过去给系统B，重定向的地址如下：

- www.java4y.com?token=xxxxxxx

接着，系统B去sso认证中心验证这个Token是否正确，如果正确，则系统B和用户建立局部会话（创建Session）。到此，系统B和用户已经是登录状态了。

![](../../assets/images/Other/attachments/一图流(SSO,%20CAS,%20OAuth,%20OTP,%20JWT)_image_11.png)

看到这里，其实SSO认证中心就类似一个中转站。

# 三、CAS认证和Oauth2.0认证对比

1. CAS 是支持单点登录的技术框架

1. OAuth2.0 是授权协议

1. CAS适合场景


多个应用系统，只需要登录一次就可以同样访问其他系统

![](../../assets/images/Other/attachments/一图流(SSO,%20CAS,%20OAuth,%20OTP,%20JWT)_image_12.png)

CAS它提供了CAS Server 和CAS Client , CAS Server 独立部署， CAS Client 是一个jar 包，导入到项目中，配合 CAS Server 实现多个项目的SSO 。 实现项目之间的SSO技术上有很强的关联性， 各系统的用户名也应该在CAS Server 认证中心存在 ，访问其中某一个项目时，重定向到统一登录页面，登录完成后，带上凭证信息重定向该项目，该项目通过cas client 拦截验证cas server 提供的认证信息，完成当前项目的登录 , CAS比较适合技术体系差不多的公司内部项目做单点登录技术方案。


1. Oauth2.0场景

希望能给外部系统颁发token，外部系统通过token访问资源服务器的接口


![](../../assets/images/Other/attachments/一图流(SSO,%20CAS,%20OAuth,%20OTP,%20JWT)_image_13.png)


	我想用知乎发文章，但不想再次注册，这个时候可以选择通过QQ登录，这个过程就使用到了Oauth2.0授权。


流程：


1. 当我点击使用QQ登录时，相当于我告诉知乎，你可以使用我在QQ的用户信息作为我在知乎的登录信息

1. 知乎这个时候去QQ要我的用户信息， QQ说好，没问题，但是需要我输入用户密码授权知乎获取我在QQ的用户信息

1. 这个时候QQ展现一个登录框给我，我输入用户名密码验证成功，代表允许了知乎从QQ那里获取我的用户信息， 知乎拿到用户信息，给我自动登录了知乎，这个时候我在没有注册知乎的情况下就可以登录使用知乎了。