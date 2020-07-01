### 출처
해당 내용은 https://jaehun2841.github.io/2018/08/30/2018-08-25-spring-mvc-handle-exception/ 에서 가져왔습니다.



## 예외(Exception) 처리는 어떻게 하는가
예외처리에 대해서는 Spring에서 강력하게 지원을 해준다
 - **Controller** level에서 처리
 - **Global** level에서 처리
 - **HandlerExceptionResolver**를 이용한 처리
 
### Controller 레벨에서의 처리
Spring에서는 Controller에서 발생한 예외에 대해 Common하게 처리 할 수 있는 기능을 제공한다. `@ExceptionHandler` 어노테이션을 통해 Controller의 메소드에서 throw된 Exception에 대한 공통적인 처리를 할 수 있도록 지원하고 있다.

#### 예제코드
```java
package com.example.springstudy.demo2.controller;

import com.example.springstudy.demo2.exception.DemoException;
import com.example.springstudy.demo2.exception.FilterException;
import com.example.springstudy.demo2.exception.InterceptorException;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.GetMapping;

@Slf4j
@Controller
public class DemoController {

    @GetMapping(path="/exception/demo")
    public String occurDemoException() {
        //강제로 DemoException을 발생 시켜 보았다.
        throw new DemoException(); //occur DemoException (RuntimeException)
    }
    
    @GetMapping(path="/exception/demo2")
    public String occurDemoException2() {
        //강제로 DemoException을 발생 시켜 보았다.
        throw new DemoException(); //occur DemoException (RuntimeException)
    }

    @ExceptionHandler(value=DemoException.class)
    public String handleDemoException(DemoException e) {
        log.error(e.getMessage());
        return "/error/404";
    }
}
```



DemoController내에서 발생한 DemoException에 대해서는 handleDemoException 메소드에서 모두 처리를 해준다.

 * Controller 메소드 내의 하위 서비스에서 Checked Exception이 발생하더라도, Controller 메소드 상위까지 예외를 throw 시키면'@ExceptionHandler' 어노테이션을 사용하여 Controller 전역적으로 예외처리가 가능하다.
 * Controller 메소드 내의 하위 서비스에서 Runtime Exception이 발생하면, 서비스를 호출한 최상위 Controller에서 해당 예외를 처리해준다.
 
Spring MVC 모델에서 예외처리를 미루고 미루면 이런식으로 Controller 레벨에서 예외처리를 Common하게 해줄 수 있다.
