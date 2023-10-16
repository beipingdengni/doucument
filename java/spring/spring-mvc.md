### 这个文档中放置一些关于spring mvc 文档



### 全局异常

@ControllerAdvice



@ResponseBody
@ExceptionHandler(value = Exception.class)



### 使用RequestBodyAdvice 和 ResponseBodyAdvice增强器

RequestBodyAdvice 和 ResponseBodyAdvice都需要搭配`@RestControllerAdvice`或`@ControllerAdvice`使
