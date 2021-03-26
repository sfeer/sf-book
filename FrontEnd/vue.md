# Vue.js

## Vue CLI

https://cli.vuejs.org/zh/

## Vue Router

https://router.vuejs.org/zh/

## WAR包发布

在tomcat下发布单页应用

```
// 确定发布名称，例如test
1. 配置vue.config.js
publicPath: '/test',

2. 配置路由文件
new Router({
    mode: 'history',
    base: '/test/'
}
3. 新建WEB-INF文件夹
4. 新建web.xml文件
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <display-name>test</display-name>
	<description>test</description>
	<error-page>
		<error-code>404</error-code>
		<location>/</location>
	</error-page>
</web-app>
```
