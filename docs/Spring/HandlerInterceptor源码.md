首先，我们看一下org.springframework.web.servlet.DispatcherServlet doDispatch方法：

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;
    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
    try {
        ModelAndView mv = null;
        Exception dispatchException = null;
        try {
            processedRequest = checkMultipart(request);
            multipartRequestParsed = (processedRequest != request);
            // Determine handler for the current request.
            mappedHandler = getHandler(processedRequest);
            if (mappedHandler == null || mappedHandler.getHandler() == null) {
                noHandlerFound(processedRequest, response);
                return;
            }
            // Determine handler adapter for the current request.
            HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
            // Process last-modified header, if supported by the handler.
            String method = request.getMethod();
            boolean isGet = "GET".equals(method);
            if (isGet || "HEAD".equals(method)) {
                long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                if (logger.isDebugEnabled()) {
                    logger.debug("Last-Modified value for [" + getRequestUri(request) + "] is: " + lastModified);
                }
                if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                    return;
                }
            }
            //调用已注册HandlerInterceptor的preHandle()方法
            if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                return;
            }
            // 真正执行Controller对应的方法
            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
            if (asyncManager.isConcurrentHandlingStarted()) {
                return;
            }
            applyDefaultViewName(processedRequest, mv);
            //调用已注册HandlerInterceptor的postHandle()方法
            mappedHandler.applyPostHandle(processedRequest, response, mv);
        }
        catch (Exception ex) {
            dispatchException = ex;
        }
        catch (Throwable err) {
            // As of 4.3, we're processing Errors thrown from handler methods as well,
            // making them available for @ExceptionHandler methods and other scenarios.
            dispatchException = new NestedServletException("Handler dispatch failed", err);
        }
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }
    catch (Exception ex) {
        //调用已注册HandlerInterceptor的afterCompletion()方法
        triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
    }
    catch (Throwable err) {
        //调用已注册HandlerInterceptor的afterCompletion()方法
        triggerAfterCompletion(processedRequest, response, mappedHandler,
                new NestedServletException("Handler processing failed", err));
    }
    finally {
        if (asyncManager.isConcurrentHandlingStarted()) {
            // Instead of postHandle and afterCompletion
            if (mappedHandler != null) {
                //调用已注册HandlerInterceptor的afterConcurrentHandlingStarted()方法
                mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
            }
        }
        else {
            // Clean up any resources used by a multipart request.
            if (multipartRequestParsed) {
                cleanupMultipart(processedRequest);
            }
        }
    }
}
/**
 * Handle the result of handler selection and handler invocation, which is
 * either a ModelAndView or an Exception to be resolved to a ModelAndView.
 */
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
        HandlerExecutionChain mappedHandler, ModelAndView mv, Exception exception) throws Exception {
    boolean errorView = false;
    if (exception != null) {
        if (exception instanceof ModelAndViewDefiningException) {
            logger.debug("ModelAndViewDefiningException encountered", exception);
            mv = ((ModelAndViewDefiningException) exception).getModelAndView();
        }
        else {
            Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
            mv = processHandlerException(request, response, handler, exception);
            errorView = (mv != null);
        }
    }
    // Did the handler return a view to render?
    if (mv != null && !mv.wasCleared()) {
        render(mv, request, response);
        if (errorView) {
            WebUtils.clearErrorRequestAttributes(request);
        }
    }
    else {
        if (logger.isDebugEnabled()) {
            logger.debug("Null ModelAndView returned to DispatcherServlet with name '" + getServletName() +
                    "': assuming HandlerAdapter completed request handling");
        }
    }
    if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
        // Concurrent handling started during a forward
        return;
    }
    if (mappedHandler != null) {
        //调用已注册HandlerInterceptor的afterCompletion()方法
        mappedHandler.triggerAfterCompletion(request, response, null);
    }
}
```

接着，我们来看看 HandlerExecutionChain的applyPreHandle方法实现：

```java
 /**
 * 执行注册到该请求上的所有HandlerInterceptor的 preHandle 方法
 */
boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HandlerInterceptor[] interceptors = getInterceptors();
    if (!ObjectUtils.isEmpty(interceptors)) {
        for (int i = 0; i < interceptors.length; i++) {
            HandlerInterceptor interceptor = interceptors[i];
            if (!interceptor.preHandle(request, response, this.handler)) {
                triggerAfterCompletion(request, response, null);
                return false;
            }
            this.interceptorIndex = i;
        }
    }
    return true;
}
public HandlerInterceptor[] getInterceptors() {
    if (this.interceptors == null && this.interceptorList != null) {
        this.interceptors = this.interceptorList.toArray(new HandlerInterceptor[this.interceptorList.size()]);
    }
    return this.interceptors;
}
```

HandlerExecutionChain的applyPostHandle方法实现：

```java
 /**
 * 执行注册到该请求上的所有HandlerInterceptor的 postHandle 方法
 */
void applyPostHandle(HttpServletRequest request, HttpServletResponse response, ModelAndView mv) throws Exception {
    HandlerInterceptor[] interceptors = getInterceptors();
    if (!ObjectUtils.isEmpty(interceptors)) {
        for (int i = interceptors.length - 1; i >= 0; i--) {
            HandlerInterceptor interceptor = interceptors[i];
            interceptor.postHandle(request, response, this.handler, mv);
        }
    }
}
```

HandlerExecutionChain的triggerAfterCompletion方法实现：

```java
/**
 * 执行注册到该请求上的所有HandlerInterceptor的 afterCompletion 方法
 */
void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, Exception ex)
        throws Exception {
    HandlerInterceptor[] interceptors = getInterceptors();
    if (!ObjectUtils.isEmpty(interceptors)) {
        for (int i = this.interceptorIndex; i >= 0; i--) {
            HandlerInterceptor interceptor = interceptors[i];
            try {
                interceptor.afterCompletion(request, response, this.handler, ex);
            }
            catch (Throwable ex2) {
                logger.error("HandlerInterceptor.afterCompletion threw exception", ex2);
            }
        }
    }
}
```

最后，HandlerExecutionChain的applyAfterConcurrentHandlingStarted方法实现：

```java
 /**
 * 执行注册到该请求上的所有AsyncHandlerInterceptor的 afterConcurrentHandlingStarted 方法
 */
void applyAfterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response) {
    HandlerInterceptor[] interceptors = getInterceptors();
    if (!ObjectUtils.isEmpty(interceptors)) {
        for (int i = interceptors.length - 1; i >= 0; i--) {
            if (interceptors[i] instanceof AsyncHandlerInterceptor) {
                try {
                    AsyncHandlerInterceptor asyncInterceptor = (AsyncHandlerInterceptor) interceptors[i];
                    asyncInterceptor.afterConcurrentHandlingStarted(request, response, this.handler);
                }
                catch (Throwable ex) {
                    logger.error("Interceptor [" + interceptors[i] + "] failed in afterConcurrentHandlingStarted", ex);
                }
            }
        }
    }
}
```

接下来，HandlerExecutionChain 内部的Interceptor数组是在什么时候初始化的呢？

来看看 HandlerExecutionChain 类的定义，源码如下：

```java
package org.springframework.web.servlet;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.springframework.util.CollectionUtils;
import org.springframework.util.ObjectUtils;
public class HandlerExecutionChain {
    private static final Log logger = LogFactory.getLog(HandlerExecutionChain.class);
    private final Object handler;
    private HandlerInterceptor[] interceptors;
    private List<HandlerInterceptor> interceptorList;
    private int interceptorIndex = -1;
    public HandlerExecutionChain(Object handler) {
        this(handler, (HandlerInterceptor[]) null);
    }
    public HandlerExecutionChain(Object handler, HandlerInterceptor... interceptors) {
        if (handler instanceof HandlerExecutionChain) {
            HandlerExecutionChain originalChain = (HandlerExecutionChain) handler;
            this.handler = originalChain.getHandler();
            this.interceptorList = new ArrayList<HandlerInterceptor>();
            CollectionUtils.mergeArrayIntoCollection(originalChain.getInterceptors(), this.interceptorList);
            CollectionUtils.mergeArrayIntoCollection(interceptors, this.interceptorList);
        }
        else {
            this.handler = handler;
            this.interceptors = interceptors;
        }
    }
    /**
     * 向handler中添加interceptor
     */
    public void addInterceptor(HandlerInterceptor interceptor) {
        initInterceptorList().add(interceptor);
    }
    /**
     * 向handler中添加多个interceptor
     */
    public void addInterceptors(HandlerInterceptor... interceptors) {
        if (!ObjectUtils.isEmpty(interceptors)) {
            initInterceptorList().addAll(Arrays.asList(interceptors));
        }
    }
    private List<HandlerInterceptor> initInterceptorList() {
        if (this.interceptorList == null) {
            this.interceptorList = new ArrayList<HandlerInterceptor>();
            if (this.interceptors != null) {
                // An interceptor array specified through the constructor
                this.interceptorList.addAll(Arrays.asList(this.interceptors));
            }
        }
        this.interceptors = null;
        return this.interceptorList;
    }
    /**
     * 按顺序返回handler上的HandlerInterceptor列表
     */
    public HandlerInterceptor[] getInterceptors() {
        if (this.interceptors == null && this.interceptorList != null) {
            this.interceptors = this.interceptorList.toArray(new HandlerInterceptor[this.interceptorList.size()]);
        }
        return this.interceptors;
    }
}
```

HandlerExecutionChain 提供了addInterceptors()方法来添加HandlerInterceptor，那addInterceptors是在哪里被调用的呢？

我们回到DispatcherServlet 的 doDispatch方法中，如下：

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;
    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
    try {
        ModelAndView mv = null;
        Exception dispatchException = null;
        try {
            processedRequest = checkMultipart(request);
            multipartRequestParsed = (processedRequest != request);
            // 获取当前请求对应的handler
            mappedHandler = getHandler(processedRequest);
            if (mappedHandler == null || mappedHandler.getHandler() == null) {
                noHandlerFound(processedRequest, response);
                return;
            }
            ...
        }
        catch (Exception ex) {
            dispatchException = ex;
        }
        catch (Throwable err) {
            ...
        }
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }
    catch (Exception ex) {
        triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
    }
    catch (Throwable err) {
        triggerAfterCompletion(processedRequest, response, mappedHandler,
                new NestedServletException("Handler processing failed", err));
    }
    finally {
        ...
    }
}
/**
 * 获取当前请求对应的HandlerExecutionChain
 */
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    for (HandlerMapping hm : this.handlerMappings) {
        if (logger.isTraceEnabled()) {
            logger.trace(
                    "Testing handler map [" + hm + "] in DispatcherServlet with name '" + getServletName() + "'");
        }
        HandlerExecutionChain handler = hm.getHandler(request);
        if (handler != null) {
            return handler;
        }
    }
    return null;
}
```

接着来看看HandlerMapping 的getHandler(request)方法，HandlerMapping是一个接口，getHandler(request)方法是在AbstractHandlerMapping中实现的，如下：

```java
 @Override
public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    //获取请求对应的handler
    Object handler = getHandlerInternal(request);
    if (handler == null) {
        handler = getDefaultHandler();
    }
    if (handler == null) {
        return null;
    }
    // Bean name or resolved handler?
    if (handler instanceof String) {
        String handlerName = (String) handler;
        handler = getApplicationContext().getBean(handlerName);
    }
    //根据handler构造HandlerExecutionChain
    HandlerExecutionChain executionChain = getHandlerExecutionChain(handler, request);
    if (CorsUtils.isCorsRequest(request)) {
        CorsConfiguration globalConfig = this.corsConfigSource.getCorsConfiguration(request);
        CorsConfiguration handlerConfig = getCorsConfiguration(handler, request);
        CorsConfiguration config = (globalConfig != null ? globalConfig.combine(handlerConfig) : handlerConfig);
        executionChain = getCorsHandlerExecutionChain(request, executionChain, config);
    }
    return executionChain;
}
/**
 * 构造HandlerExecutionChain, 并初始化HandlerInterceptor
 */
protected HandlerExecutionChain getHandlerExecutionChain(Object handler, HttpServletRequest request) {
    HandlerExecutionChain chain = (handler instanceof HandlerExecutionChain ?
            (HandlerExecutionChain) handler : new HandlerExecutionChain(handler));
    String lookupPath = this.urlPathHelper.getLookupPathForRequest(request);
    for (HandlerInterceptor interceptor : this.adaptedInterceptors) {
        if (interceptor instanceof MappedInterceptor) {
            MappedInterceptor mappedInterceptor = (MappedInterceptor) interceptor;
            if (mappedInterceptor.matches(lookupPath, this.pathMatcher)) {
                chain.addInterceptor(mappedInterceptor.getInterceptor());
            }
        }
        else {
            chain.addInterceptor(interceptor);
        }
    }
    return chain;
}
```