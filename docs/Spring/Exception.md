# 异常处理

在基于Spring框架的项目中，可以通过在ApplicationContext-MVC.xml，即SpringMVC配置文件中配置 

异常处理可以分为三种：

1. 第一种是进入 

1. 第二种是 

1. 第三种是 

以上三种情况都是restful的情况，结果会返回一个Json。

如果希望返回 

注意@ControllerAdvice需要搭配 

# @ExceptionHandler、HandlerExceptionResolver、@controlleradvice

把 @ExceptionHandler、HandlerExceptionResolver、@controlleradvice 三兄弟放在一起来写更有比较性。这三个东西都是用来处理异常的，但是它们使用的场景都不一样。看本文给你详细的讲解，再也不怕面试被问到了！

这三个注解都是来自于 SpringMVC 的，都能进行异常处理。

Java 程序员都非常的痛恨异常，很多人讨厌 Java 就是因为它的异常处理机制。到处的 try-catch-finally，再不是就是到处抛出异常。

所以 Spring 深知 Java 的疼点，推出了：@ExceptionHandler、HandlerExceptionResolver、@controlleradvice 来方便我们处理一些异常！

## @ExceptionHandler 注解

用于局部方法捕获，与抛出异常的方法处于同一个 Controller 类。源码如下：

```java
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ExceptionHandler {
    Class<? extends Throwable>[] value() default {};
}
```

从源码中可以看出，@ExceptionHandler 注解只能作用为对象的方法上，并且在运行时有效，value() 可以指定异常类。由该注解注释的方法可以具有灵活的输入参数。

异常参数可以包括一般的异常或特定的异常（即自定义异常），如果注解没有指定异常类，会默认进行映射。

用法代码如下：

```java
@Controller
public class XttblogController {
    @ExceptionHandler({NullPointerException.class})
    public String exception(NullPointerException e) {
        System.out.println(e.getMessage());
        e.printStackTrace();
        return "null pointer exception";
    }
    @RequestMapping("test")
    public void test() {
        throw new NullPointerException("出错了！");
    }
}
```

上面这段代码只会捕获 XttblogController 类中的 NullPointerException 异常。

## HandlerExceptionResolver 接口

HandlerExceptionResolver 是 Spring 提供的一个接口。它可以用来处理全局异常！

```java
@Component
public class GlobalExceptionResolver implements HandlerExceptionResolver{
    private ObjectMapper objectMapper;
    public CustomMvcExceptionHandler() {
        objectMapper = new ObjectMapper();
    }
    @Override
    public ModelAndView resolveException(HttpServletRequest request, 
        HttpServletResponse response,Object o, Exception ex) {
        response.setStatus(200);
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setCharacterEncoding("UTF-8");
        response.setHeader("Cache-Control", "no-cache, must-revalidate");
        Map<String, Object> map = new HashMap<>();
        if (ex instanceof NullPointerException) {
            map.put("code", ResponseCode.NP_EXCEPTION);
        } else if (ex instanceof IndexOutOfBoundsException) {
            map.put("code", ResponseCode.INDEX_OUT_OF_BOUNDS_EXCEPTION);
        } else {
            map.put("code", ResponseCode.CATCH_EXCEPTION);
        }
        try {
            map.put("data", ex.getMessage());
            response.getWriter().write(objectMapper.writeValueAsString(map));
        } catch (Exception e) {
            e.printStackTrace();
        }
        return new ModelAndView();
    }
}
```

在 Spring 源码中，我们可以看出它会获取一个实现了 HandlerExceptionResolver 接口的列表 List<HandlerExceptionResolver> resolvers; 如果这个列表不为空，则循环处理其中的异常。

HandlerExceptionResolve 虽然能够处理全局异常，但是 Spring 官方不推荐使用它。

## @controlleradvice 注解

另外一个能够处理全局异常的就是 @controlleradvice 注解了。

@controlleradvice 注解根据它的源码，我们知道它也只能作用在类上，并且作用于运行时。下面我们来看一个例子。

```java
@ControllerAdvice
public class ExceptionController {
    @ExceptionHandler(RuntimeException.class)
    public ModelAndView handlerRuntimeException(RuntimeException ex) {
        if (ex instanceof MaxUploadSizeExceededException) {
            return new ModelAndView("error").addObject("msg", "文件太大！");
        }
        return new ModelAndView("error").addObject("msg", "未知错误：" + ex);
    }
    @ExceptionHandler(Exception.class)
    public ModelAndView handlerMaxUploadSizeExceededException(Exception ex) {
        if (ex != null) {
            return new ModelAndView("error").addObject("msg", ex);
        }
        return new ModelAndView("error").addObject("msg", "未知错误：" + ex);
    }
}
```

需要注意的是，@ControllerAdvice 一般是和 @ExceptionHandler 组合在一起使用的。官方也推荐用这种方式处理统一全局异常。