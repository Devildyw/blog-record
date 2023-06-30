## SpringMVC 之 RequestContextHolder

## 简介

在业务编写中，常常会出现将 `request` 和 `response` 传来传去的场景，正常来说在 `service` 层是没有 `request` 的，然而直接从 `Controller` 传过来的话方法太粗暴了，而 `SpringMVC` 提供的 `RequestContextHolder` **可以在一个请求线程的中获取到 `Request` 并将其存储在底层的 `ThreadLocal` 中**，避免了 `Request` 从头传到尾的情况。一般项目中，会对这个类进行再次封装，便于获取请求的相关信息，常见的比如用户信息。

## 原理剖析

`RequestContextHolder` 基于 `ThreadLocal` 实现。

```JAVA
public abstract class RequestContextHolder  {

   private static final boolean jsfPresent =
         ClassUtils.isPresent("javax.faces.context.FacesContext", RequestContextHolder.class.getClassLoader());

   private static final ThreadLocal<RequestAttributes> requestAttributesHolder =
         new NamedThreadLocal<>("Request attributes");

   //用于子线程的
   private static final ThreadLocal<RequestAttributes> inheritableRequestAttributesHolder =
         new NamedInheritableThreadLocal<>("Request context");
}
```

从 `SpringMVC` 源码入手，在 `FrameworkServlet#processRequest` 中，会在进入处理请求前，将 Request 封装为 `RequestAttributes`，放到 `RequestContextHolder` 中。

```JAVA
protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
      throws ServletException, IOException {

	......

   LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
   LocaleContext localeContext = buildLocaleContext(request);

   RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
   ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

   WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
   asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());

   initContextHolders(request, localeContext, requestAttributes);
    
    ......
        
}

private void initContextHolders(HttpServletRequest request,
		@Nullable LocaleContext localeContext, @Nullable RequestAttributes requestAttributes) {
	if (localeContext != null) {
		LocaleContextHolder.setLocaleContext(localeContext, this.threadContextInheritable);
	}
	if (requestAttributes != null) {
		RequestContextHolder.setRequestAttributes(requestAttributes, this.threadContextInheritable);
	}
}
```

`RequestContextHolder` 会根据 `threadContextInheritable` 选择将 `RequestAttributes` 放入 `inheritableRequestAttributesHolder` 或者 `inheritableRequestAttributesHolder` 中。

```JAVA
public static void setRequestAttributes(@Nullable RequestAttributes attributes, boolean inheritable) {
   if (attributes == null) {
      resetRequestAttributes();
   }
   else {
      if (inheritable) {
         inheritableRequestAttributesHolder.set(attributes);
         requestAttributesHolder.remove();
      }
      else {
         requestAttributesHolder.set(attributes);
         inheritableRequestAttributesHolder.remove();
      }
   }
}
```

取出 `RequestAttributes` 时会先从 `requestAttributes` 中取，取不到再到 `inheritableRequestAttributesHolder` 中取。

```JAVA
@Nullable
public static RequestAttributes getRequestAttributes() {
   RequestAttributes attributes = requestAttributesHolder.get();
   if (attributes == null) {
      attributes = inheritableRequestAttributesHolder.get();
   }
   return attributes;
}
```

## `inheritableRequestAttributesHolder` 与 `requestAttributesHolder`

`RequestContextHolder` 底层由 `ThreadLocal` 实现。

通过源码剖析我们可以看到，在 `RequestContextHolder` 中有两个 `ThreadLocal` 变量，分别为 `inheritableRequestAttributesHolder` 与 `requestAttributesHolder` 。这两个变量有什么区别吗？

`RequestContextHolder` 默认从 `requestAttributesHolder` 存取，但是在**多线程的情况下，子线程无法访问父线程中的数据**，即 `RequestContextHolder#getRequestAttributes` 返回 null，此时就需要用到 `inheritableRequestAttributesHolder`。`inheritableRequestAttributesHolder` 是 `NamedInheritableThreadLocal` 类型，`NamedInheritableThreadLocal` 继承于 `InheritableThreadLocal`，`InheritableThreadLocal` **实现了子线程从父线程继承数据**，这样在**子线程也可以访问父线程中 `InheritableThreadLocal` 的数据。**

> [ThreadLocal | Devil的个人博客 (devildyw.github.io)](https://devildyw.github.io/2022/02/28/ThreadLocal/)

要使用 `inheritableRequestAttributesHolder` 替代 `requestAttributesHolder` ，关键在于 `FrameworkServlet` 中的 `threadContextInheritable`，该值为 false，即默认使用 `requestAttributesHolder`，将其设置为 true，则会使用 `inheritableRequestAttributesHolder`。通常 `requestAttributesHolder` 已经够用了。

```JAVA
private boolean threadContextInheritable = false;

RequestContextHolder.setRequestAttributes(requestAttributes, this.threadContextInheritable);

public static void setRequestAttributes(@Nullable RequestAttributes attributes, boolean inheritable) {
	if (attributes == null) {
		resetRequestAttributes();
	}
	else {
		if (inheritable) {
			inheritableRequestAttributesHolder.set(attributes);
			requestAttributesHolder.remove();
		}
		else {
			requestAttributesHolder.set(attributes);
			inheritableRequestAttributesHolder.remove();
		}
	}
}
```

`InheritableThreadLocal` 解决了父线程向子线程传递数据的问题，但**传递数据发生在创建 Thread 阶段**，如果**使用了线程池，线程被复用，子线程的数据仍然是创建时传递的数据，而不是执行任务时父线程的数据**。这种情况下，就需要重写 `RequestContextHolder`，使用 `TransmittableThreadLocal` 代替 `ThreadLocal`。`TransmittableThreadLocal` 用于解决使用线程池时，父线程向子线程传递数据的问题，详见 [解决ThreadLocal在开启子线程时，父线程向子线程值传递问题，源码分析](https://blog.csdn.net/qq_26012495/article/details/104379137)