---
layout: post
title: Spring MVC拦截器
categories: [Springboot, Interceptor]
description: Springboot Interceptor
keywords: Springboot, Interceptor
---

## 拦截器

### 1.简介

Spring MVC 中的拦截器（Interceptor）类似于 Servlet  开发中的过滤器 Filter，它主要用于拦截用户请求并作相应的处理,它也是 AOP 编程思想的体现,底层通过动态代理模式完成。

### 2.定义实现类

拦截器有两种实现方式： 1.实现 HandlerInterceptor 接口 2.继承 HandlerInterceptorAdapter 抽象类（看源码最底层也是通过 HandlerInterceptor 接口 实现）

### 3.HandlerInterceptor方法介绍

```java
	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		
        //进行逻辑判断，如果ok就返回true，不行就返回false，返回false就不会处理请求
		return true;
	}
	
	@Override
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			ModelAndView modelAndView) throws Exception {
	}

	@Override
	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
			throws Exception {
	}

```

preHandle：在业务处理器处理请求之前被调用。预处理，可以进行编码、安全控制、权限校验等处理； postHandle：在业务处理器处理请求执行完成后，生成视图之前执行。 afterCompletion：在 DispatcherServlet 完全处理完请求后被调用，可用于清理资源等。

### 4.应用场景

> 1.日志记录：记录请求信息的日志，以便进行信息监控、信息统计、计算PV（Page View）等； 2.登录鉴权：如登录检测，进入处理器检测检测是否登录； 3.性能监控：检测方法的执行时间； 4.其他通用行为。

### 5.与 Filter 过滤器的区别

> 1.拦截器是基于java的反射机制的，而过滤器是基于函数回调。 2.拦截器不依赖于servlet容器，而过滤器依赖于servlet容器。 3.拦截器只能对Controller请求起作用，而过滤器则可以对几乎所有的请求起作用。 4.拦截器可以访问action上下文、值栈里的对象，而过滤器不能访问。 5.在Controller的生命周期中，拦截器可以多次被调用，而过滤器只能在容器初始化时被调用一次。 6.拦截器可以获取IOC容器中的各个bean，而过滤器不行。这点很重要，在拦截器里注入一个service，可以调用业务逻辑。

## 具体实现

### 单个拦截器

#### 1.新建拦截器

```java
	public class Test1Interceptor implements HandlerInterceptor{

	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		System.out.println("执行preHandle方法-->01");
		return true;
	}
	
	@Override
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			ModelAndView modelAndView) throws Exception {
		System.out.println("执行postHandle方法-->02");
	}

	@Override
	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
			throws Exception {
		System.out.println("执行afterCompletion方法-->03");
	}
}

```

#### 2.配置拦截器

```java
@Configuration
public class WebMvcConfig extends WebMvcConfigurationSupport {
	/*
	 * 拦截器配置
	 */
	@Override
	public void addInterceptors(InterceptorRegistry registry) {
		// 注册自定义拦截器，添加拦截路径和排除拦截路径
		registry.addInterceptor(new Test1Interceptor()) // 添加拦截器
				.addPathPatterns("/**") // 添加拦截路径
				.excludePathPatterns(// 添加排除拦截路径
						"/hello").order(0);//执行顺序
		super.addInterceptors(registry);
	}

}

```

#### 3.测试拦截器

```java
@RestController
public class TestController {

	@RequestMapping("/hello")
	public String getHello() {
		System.out.println("这里是Hello");
		return "hello world";
	}
	
	
	@RequestMapping("/test1")
	public String getTest1() {
		System.out.println("这里是Test1");
		return "test1 content";
	}
	
	@RequestMapping("/test2")
	public String getTest2() {
		System.out.println("这里是Test2");
		return "test2 content";
	}
}

```

#### 4.单个拦截器的执行流程

通过浏览器测试： http://127.0.0.1:8080/hello 结果：

```
这里是Hello

```

http://127.0.0.1:8080/test1 、http://127.0.0.1:8080/test2 结果：

```
执行preHandle方法-->01
这里是Test1
执行postHandle方法-->02
执行afterCompletion方法-->03

```

### 多个拦截器

#### 1.新建两个拦截器

Test1Interceptor

```java
public class Test1Interceptor implements HandlerInterceptor{
	
	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		System.out.println("执行Test1Interceptor preHandle方法-->01");
		return true;
	}
	
	@Override
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			ModelAndView modelAndView) throws Exception {
		System.out.println("执行Test1Interceptor postHandle方法-->02");
	}

	@Override
	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
			throws Exception {
		System.out.println("执行Test1Interceptor afterCompletion方法-->03");
	}
}

```

Test2Interceptor

```java
public class Test2Interceptor extends HandlerInterceptorAdapter{

	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		System.out.println("执行Test2Interceptor preHandle方法-->01");
		return true;
	}
	
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			ModelAndView modelAndView) throws Exception {
		System.out.println("执行Test2Interceptor postHandle方法-->02");
	}

	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
			throws Exception {
		System.out.println("执行Test2Interceptor afterCompletion方法-->03");
	}
}


```

#### 2.配置拦截器

```java
@Configuration
public class WebMvcConfig extends WebMvcConfigurationSupport {
	/*
	 * 拦截器配置
	 */
	@Override
	public void addInterceptors(InterceptorRegistry registry) {
		// 注册自定义拦截器，添加拦截路径和排除拦截路径
		registry.addInterceptor(new Test1Interceptor()) // 添加拦截器1
				.addPathPatterns("/**") // 添加拦截路径
				.excludePathPatterns(// 添加排除拦截路径
						"/hello")
				.order(0);
		registry.addInterceptor(new Test2Interceptor()) // 添加拦截器2
				.addPathPatterns("/**") // 添加拦截路径
				.excludePathPatterns(// 添加排除拦截路径
						"/test1")
				.order(1);
		super.addInterceptors(registry);
	}

}

```

#### 3.测试拦截器

```java
@RestController
public class TestController {

	@RequestMapping("/hello")
	public String getHello() {
		System.out.println("这里是Hello");
		return "hello world";
	}
	
	
	@RequestMapping("/test1")
	public String getTest1() {
		System.out.println("这里是Test1");
		return "test1 content";
	}
	
	@RequestMapping("/test2")
	public String getTest2() {
		System.out.println("这里是Test2");
		return "test2 content";
	}
}

```

#### 4.多个拦截器的执行流程

通过浏览器测试： http://127.0.0.1:8080/test2 结果：

```
执行Test1Interceptor preHandle方法-->01
执行Test2Interceptor preHandle方法-->01
这里是Test2
执行Test2Interceptor postHandle方法-->02
执行Test1Interceptor postHandle方法-->02
执行Test2Interceptor afterCompletion方法-->03
执行Test1Interceptor afterCompletion方法-->03

```

通过示例，简单的说多个拦截器执行流程就是**先进后出**。

### 简单的 token 判断示例

#### 1.拦截器

```java
@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		System.out.println("执行Test1Interceptor preHandle方法-->01");
		
		String token = request.getParameter("token");
		if (StringUtils.isEmpty(token)) {			
			response.setContentType("text/html");
			response.setCharacterEncoding("UTF-8");
			response.getWriter().println("token不存在");
			return false;
		}
		return true;
	}

```

#### 2.测试及结果

未传token：

```
执行Test1Interceptor preHandle方法-->01

```

传token：

```
执行Test1Interceptor preHandle方法-->01
页码：1
页码大小：10
执行Test1Interceptor postHandle方法-->02
执行Test1Interceptor afterCompletion方法-->03
```

转载[掘金](https://juejin.im/post/5dae915de51d4524ce22276c#heading-3)