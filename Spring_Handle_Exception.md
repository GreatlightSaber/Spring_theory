### 출처
해당 내용은 https://jaehun2841.github.io/2018/08/30/2018-08-25-spring-mvc-handle-exception/ 에서 가져왔습니다.



## 예외(Exception) 처리는 어떻게 하는가
예외처리에 대해서는 Spring에서 강력하게 지원을 해준다
 - **Controller** level에서 처리
 - **Global** level에서 처리
 - **HandlerExceptionResolver**를 이용한 처리
 
### Controller level에서의 처리
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

 * Controller 메소드 내의 하위 서비스에서 Checked Exception이 발생하더라도, Controller 메소드 상위까지 예외를 throw 시키면`@ExceptionHandler` 어노테이션을 사용하여 Controller 전역적으로 예외처리가 가능하다.
 * Controller 메소드 내의 하위 서비스에서 Runtime Exception이 발생하면, 서비스를 호출한 최상위 Controller에서 해당 예외를 처리해준다.
 
Spring MVC 모델에서 예외처리를 미루고 미루면 이런식으로 Controller 레벨에서 예외처리를 Common하게 해줄 수 있다.


### Global level에서의 처리
만약 여러 Controller에서 같은 Exception이 발생하는 경우엔 어떻게 해야 할까? 위의 방식 처럼 Controller 별로 `@ExceptionHandler` 어노테이션이 붙은 메소드를 만들게 된다면, 중복코드가 양산 될 것이고 결국 유지보수의 비용이 증가하게 될 것이다. Spring에서는 이런 상황을 위해 Web Application 전역적으로 `@ExceptionHandler`를 사용할수 있도록 지원한다.
 * `@ControllerAdvice` - Exception 처리 후 Error Page 등을 통해 처리가 가능하다.
 * `@RestControllerAdvice`
   * REST API에 대한 Exception 처리 등에 용이.(Default 데이터를 리턴 해 줄 수 있다.)
   * @RestControllerAdvice = @ControllerAdvice + @ResponseBody

위 2개의 어노테이션을 이용하여 Web Application 전역적으로 `@ExceptionHandler`를 사용할 수 있다.

**`@ControllerAdvice`로 케어 가능한 범위는 Dispatcher Servlet 내에서 이루어집니다.**

### 예제코드
```java
import com.example.springstudy.demo2.exception.DemoException;
import com.example.springstudy.demo2.exception.FilterException;
import com.example.springstudy.demo2.exception.InterceptorException;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

@Slf4j
@ControllerAdvice
public class DemoControllerAdvisor {

    //모든 Controller에서 일어나는 DemoException에 대해 전역적으로 예외처리
    @ExceptionHandler(value = DemoException.class)
    public String handleDemoExceptionForGlobal(DemoException e) {
        log.error(e.getMessage());
        return "/error/404";
    }
}
```

#### `@ExceptionHandler`와 `@ControllerAdvice` 클래스 내의 `@ExceptionHandler`중 실행 우선 순위
해당 Controller에 `@ExceptionHandler`가 있으면 해당 컨트롤러에서 Exception 처리 후 `@ControllerAdvice`의 `@ExceptionHandler` 영역까지는 실행하지 않는다.


### HandlerExceptionResolver를 이용한 처리
HandlerExceptonResolver는 Controller의 작업 중에 발생한 예외를 어떻게 처리 할 지에 대한 전략이다. **DispatcherServlet 이외의 영역에서 발생한 에러는 Servelt Container 내부에서 처리 될 것**이다. 

HandlerExceptionResolver는 아래와 같은 인터페이스를 제공한다.
```java
package org.springframework.web.servlet;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.lang.Nullable;

public interface HandlerExceptionResolver {
    @Nullable
    ModelAndView resolveException(HttpServletRequest var1, HttpServletResponse var2, @Nullable Object var3, Exception var4);
}
```
DispatcherServlet내에서 예외 발생 시 ,resolverExceptino 메소드를 구현한 HandlerExceptionResolver들이 실행 계획에 따라  처리되며 예외를 처리하게 된다. 위에서 본 2가지 방식의 예외처리도 HandlerExceptionResolver를 이용한 예외 처리 방법이다. Spring에서 기본적으로 설정되어 있는 HandlerExceptionResolver에 대해 알아보도록 하겠다.

Dispatcher Servlet에 기본적으로 3개의 HandlerExceptionResolver가 등록 되어있다.
 * ExceptionHandlerExceptionResolver
 * ResponseStatusExceptionResolver
 * DefaultHandlerExceptionResolver
순서로 Resolver가 실행된다.

### ExceptionHandlerExceptionResolver
Spring 3.2때 AnnotationMethodHandlerExceptionResolver라는 이름으로 등장하였다. 현재는 Deprecated처리 되어 ExceptionHandlerExceptionResolver 클래스를 사용하고 있다. 위에서 사용한 `@ExceptionHandler` 어노테이션에 대한 Resolver 클래스이다.

### ResponseStatusExceptionResolver
ResponseStatusExceptionResolver는 예외에 대한 Http 응답을 설정해 줄 수 있다. 특정 예외가 발생하였을 때 , 단순히 500 (internal-server-error) 대신 더 구체적인 응답 상태값을 전달 해 줄 수 있다.

#### 사용예제(`@ExceptionHandler`와 함께 사용)
```java
//@ExceptionHandler 어노테이션과 함께 사용할 수 있다.
//구체적인 응답 코드를 줄 뿐 아니라, 간단한 사유도 전달 할 수 있다.
@ResponseStatus(value = HttpStatus.FORBIDDEN, reason = "Permission Denied")
@ExceptionHandler(value=DemoException.class)
public String handleDemoException(DemoException e) {
    log.error(e.getMessage());
    return "/error/403";
}
```
### DefaultHandlerExceptionResolver
DispatcherServlet에 디폴트로 등록 된 3가지 HandlerExceptionResolver에서 예외처리를 하지 못하는 경우, 마지막으로 `DefaultHandlerExceptionResolver`에서 예외처리를 해준다.

DefaultHandlerExceptionResolver에서는 내부적으로 Spring 표준 예외처리를 해준다. 각 상황에 걸맞는 응답 코드를 리턴해 주는 역할을 한다.
 * Request URL에 맞는 Controller를 못찾는 경우 --> 404 Not Found
 * Controller 메소드 실행 중 예외가 발생한 경우 --> 500 Internal Server Error
 * Countoller의 파리머터 형식이 잘못된 경우 --> 400 Bad Request

### SimpleMappingExceptionResolver
SimpleMappingExceptionResolver는 web.xml에 error-page를 지정하는것과 비슷한 처리를 할 수 있도록 해준다. Exception별로 error-page를 매핑 할 수 있는 기능을 제공한다.

#### 예시(java config)
```java
@Configuration
@EnableWebMvc
public WebMvcConfig extends WebMvcConfigurerAdapter {
    @Bean(name=“customMappingExceptionResolver”)
    public SimpleMappingExceptionResolver customMappingExceptionResolver() {
    	SimpleMappingExceptionResolver r = new SimpleMappingExceptionResolver();

        Properties mappings = new Properties();
        mappings.setProperty("DatabaseException", "databaseError");
        mappings.setProperty("DemoException", "demoError");

        r.setExceptionMappings(mappings);  
        r.setDefaultErrorView("default-error-page");    
        r.setExceptionAttribute("ex");     
        return r;
    }
}
```

#### 예시(xml)
```xml
<bean id="simpleMappingExceptionResolver" class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
    <property name="exceptionMappings"> 
        <map> 
            <entry key="DatabaseException" value="databaseError"/> 
            <entry key="DemoException" value="demoError"/> 
        </map> 
    </property> 
    <property name="defaultErrorView" value="error"/> 
    <property name="exceptionAttribute" value="ex"/>
</bean>
```

Spring MVC의 대한 처리는 99프로가 `Dispatcher Servlet`에서 일어난다. 그렇기 때문에 Dispatcher Servlet 내부에서 발생한 Exception은 Dispatcher Servlet 내부에서 자체적으로 해결이 가능하다. 하지만, Dispatcher Servlet이전에 Filter에서 Exception이 발생 할 경우 Dispatcher Servlet 내의 `HandlerExceptionResolver` 처리를 받을 수 없다.

### Filter에서의 예외 처리
Filter에서 예외가 발생하면 Web Application 레벨에서 처리해줘야한다.
 * web.xml에 error-page를 잘 등록해줘서 에러를 사용자에게 표현
 * Filter 내부에서 try-catch 구문을 통해 예외 방생 시, `request.getRequestDispatcher(String)`를 통해 어떻게든 Controller까지 예외를 보내서 처리하게 한다. (filter보다는 interceptor에서 로직을 처리하는것이 예외처리가 쉽습니다. - interceptor은 DispatherServlet 내부에서 실행되기 때문)



