---
title: Django的CSRF保护机制
date: 2017-11-03 11:32:39
tags:
- Django
- CSRF
categories:
- Web框架
- Django
---

## 什么是CSRF

CSRF， Cross Site Request Forgery，跨站点伪造请求。

举例来说，某个恶意网站上有一个指向你的网站的链接，如果某个用户已经登录到你的网站上了，难得当用户点击这个链接时，就会向你的网站发来一个请求，你的网站会以为这个请求是用户自己发来的，其实是那个恶意网站伪造的。

## Django提供的CSRF保护机制

Django第一次响应来自某个客户端的请求时，会在服务器端随机生成一个`token`，把这个`token`放在`cookie`里。然后每次POST请求都会带上这个`token`，这样就能避免被CSRF攻击。

1. 在返回的HTTP响应的`cookie`里，Django会为你添加一个`csrftoken`字段，其值为一个自动生成的`token`
2. 在所有的POST表单时，必须包含一个`csrfmiddlewaretoken`字段（只需要在模板里加一个tag，Django就会自动帮你生成，见下面）
3. 在处理POST请求之前，Django会验证这个请求的cookie里的`csrftoken`字段的值和提交的表单里的`csrfmiddlewaretoken`字段的值是否一样。如果一样，则表明这是一个合法的请求，否则，这个请求可能是来自于别人的`csrf`攻击，返回403Forbidden。
4. 在所有`ajax` POST请求里，添加一个X-CSRFTOKEN header，其值为`cookie`里的`csrftoken`的值。

## Django里如何使用CSRF防护

1. 首先，最基本的原则是：GET请求不要有副作用。也就是说任何处理GET请求的代码对资源的访问都一定要是“只读”的。
2. 要启用`django.middleware.csrf.CsrfViewMiddleware`这个中间件
3. 再次，在所有的POST表单元素时，需要加上一个`\{% csrf_token %}`tag
4. 在渲染模块时，使用`RequestContext`。`RequestContext`会处理`csrf_token`这个tag，从而自动为表单添加一个名为`csrfmiddlewaretoken`的input。