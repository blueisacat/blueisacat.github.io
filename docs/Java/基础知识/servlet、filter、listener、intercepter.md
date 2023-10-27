---
layout: default
title: servlet、filter、listener、intercepter
parent: 基础知识
grand_parent: Java
---

- servlet:是一种运行于服务器端的java应用程序，具有独立于平台和协议的特性，并且可以动态生成web页面;它工作在客户端请求和服务器响应的中间层。servlet的主要功能在于交互式地浏览和修改数据，生成动态的web内容。


- filter:是一个可以复用的代码片段，可以用来转换http请求,响应和头信息。随web的启动而启动，只初始化一次，以后就可以拦截相关的请求，只有web应用停止或重新部署的时候才销毁。filter与servlet的区别在于:不能直接向用户生成响应，只能修改对某一资源的请求或响应。


- listener:监听器，通过listener可以监听web服务器中某一执行动作，并根据其要求做出相应的响应。也就是说，在application,session,request三个对象创建, 消亡或往其中添加修改删除属性时自动执行代码的功能组件。随web的启动而启动，只初始化一次，随web的停止而销毁。


- intercepter:面向切面编程，在servlet或某个方法前或后调用一个方法，基于java的反射机制，比如动态代理就是拦截器的简单实现。
servlet,filter,listener配置在web.xml中，执行顺序依次为: context-param -> listener ->filter ->servlet
