---
title: Uploadify插件的使用及兼容性
date: 2017-11-11 19:02:59
tags:
- Uploadify
- Debug
categories:
- 错误、调试和测试
- 调试
---

## 情境

要把一个使用Java编写的免费开源的轻量级CMS系统部署到服务器上，主要包括后台权限系统、CMS栏目系统、内容发布系统、简约风格换肤系统。这个系统选用的技术如下：

| 后端     |                                       |
| ------ | ------------------------------------- |
| 核心框架   | Spring Framework 4.2.5                |
| 安全框架   | Apache Shiro 1.3.2                    |
| 视图框架   | Spring MVC 4.2.5                      |
| 缓存框架   | Ehcache                               |
| 数据库连接池 | Tomcat JDBC                           |
| ORM框架  | Spring Data JPA、Hibernate 4.3.5       |
| 日志管理   | SLF4J 1.7.21、Log4j                    |
| 编辑器    | ueditor                               |
| 工具类    | Apache Commons、Jackson 2.8.5、POI 3.15 |
| view层  | JSP                                   |
| 数据库    | MySQL、Oracle等关系型数据库                   |

| 前端       |                          |
| -------- | ------------------------ |
| DOM      | jquery                   |
| 分页       | jquery                   |
| UI管理     | common                   |
| UI集成     | uiExtend                 |
| 滚动条      | jquery.nicescroll.min.js |
| 图表       | highcharts               |
| 3D图表     | highcharts-more          |
| 轮播图      | jquery-swipe             |
| 表单提交     | jquery.form              |
| 文件上传     | jquery.uploadify         |
| 表单验证     | jquery.validator         |
| 展现树      | jquery.ztree             |
| html模版引擎 | template                 |

在系统后台发布文章的时候，可以为文章添加一个封面`CoverImageUrl`。然而，当我选中本地的图片，点击上传之后，却发现图片并没有上传。点击保存之后，前台页面也没有出现文章的封面。

## 思路

1. 搞明白自己遇到的问题是什么？

   + 在系统的后台管理界面发布文章时，给文章添加封面。实质上就是 一个上传图片（图片从本地移动到系统指定的路径下），并把图片的相关数据存进数据库的一个过程。
   + 当浏览器访问文章时，再从数据库中取出图片的相关数据，渲染出来就可以了。

2. 搞明白是哪里出了问题？

   + 从数据流来看，图片数据就是经过前端到达后台再进入数据库的。那么是这中间的哪一环出了问题？

   + 首先检查数据库，文章封面对应的字段`CoverImageUrl`的值没有变化，说明没有改变数据库的数据。

   + 然后调试后台的Java代码`UploadController.java`，发现数据根本就没有传到后台。这就说明是在前台出问题的。

   + 那么检查前台的代码，盯着看了很久还是没有发现问题，调试的时候发现，数据甚至没有传到前台的代码里，百思不得其解，在这个地方卡了很久。

     ```html
     <tr>
       <td class="l_title w200">图片：</td>
       <td>
         <div class="J_toolsBar fl">
           <div class=" w200 ml10">
             <label> 
               <input type="file" id="attachImage" /> 
             </label>
           </div>
         </div>
       </td>
     </tr>
     ```

     ```javascript
     $("#attachImage").uploadify(
     		{
     			'swf' : '${pageContext.request.contextPath}/static/js/uploadify/uploadify.swf',
     			'uploader' : '${pageContext.request.contextPath}/upload/uploadAttach',
     			'cancelImg' : '${pageContext.request.contextPath}/static/js/uploadify/uploadify-cancel.png',
     			'queueID' : 'fileQueue',
     			'auto' : true,
     			'multi' : false,
     			'simUploadLimit' : 1,
     			'buttonText' : '上传图片',
     			'fileObjName' : 'fileData',
     			'width' : 70,
     			'height' : 20,
     			'uploadLimit':3,
     			'onUploadSuccess' : function(file, data, response) {
     					if(data != null){
     						var attachUrl = '${pageContext.request.contextPath}' + data;								
     						$("#attachURL").attr('src',attachUrl); 
     						$("#coverImageUrl").val(data);
     					}
     		 }
     	   });
     ```

   + 后来灵机一动，直接搜索`uploadify`，了解了一下这个控件。发现有很多使用这个控件无法上传图片的情况，一个一个点进去看，有关于jQuery包的导入顺序的，有基于cookie信息验证身份的等等，最后确定是flash插件没有更新，终于把这个问题解决了。

3. 总结一下检查的步骤

   1. **检查是否安装Flash插件**
      + 如果检查完毕，发现没有安装，需要根据你浏览器类别，浏览器版本来进行下载对应的Flash版本，这也就是该插件的一大坑，Flash插件强关联操作系统，强关联浏览器类别，强关联具体某一类浏览器的版本。如，实际兼容Firefox4.0过程中，Flash版本对应Flash for Firefox 10，其他的版本安装上也不能使用。即IE，Firefox，chrome需要下载不同的Flash安装。
   2. **Firefox兼容问题**
      + 在项目中，处于安全性考虑，每次请求必然会做合法性校验，即请求经过相关过滤器，而登录过滤器会根据`sessionid`获取用户session登录信息。而Firefox使用Flash上传文件时，Flash本身的bug导致提交时不会带上`session cookie`，而HTTP是无状态请求，session的存在以客户端和服务端的交换标志而延续。而session的`sessionid`是存储在cookie中的，导致请求不合法。
      + 解决方案一
        + 采用插件自身的`formData `属性实现
      + 解决方案二
        + 在原有请求后加上`;jsessionid=${pageContext.session.id}`

## 总结

**最后总结一下Debug之道**

+ 首先要摸清楚数据流，搞明白问题出在哪里。
+ 要科学地利用搜索工具，你遇到的问题，别人有可能已经遇到过了，并早已经把解决方案发布在互联网上了。