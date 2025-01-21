# Dispatcher-Servlet이란?

Spring 공식 문서에서 Spring MVC에 대한 설명으로 "DispatcherServlet이 중앙에서 HTTP 요청을 처리해주는데 이는 Front Controller 패턴으로 설계되어있다" 라고 설명하고 있다.
즉, **Dispatcher-Servlet은 HTTP로 오는 모든 요청을 가장 먼저 받아 적합한 컨트롤러에 위임을 해주는 프론트 컨트롤러(Front Controller)**라고 이해하면 된다.

우리가 흔히 Spring으로 다음과 같이 Controller를 작성한다.
```kotlin
@RestController
@RequestMapping("/api/user")
@Tag(name = "User", description = "유저 관련 API")
class UserController(
    private val userService: UserService,
) {}
```
이 때 `@RequestMapping`을 통해 통해 `"/api/user"`와 같이 컨트롤러가 처리를 할 특정 url을 지정을 해주게 된다. 이 때 요청이 온 url에 따라 요청을 처리할 적절한 컨트롤러를 찾아주는 역할을 하는 것이다. 이를 통해는 우리는 매번 HTTP Message를 파싱을 한다던가 무한 if 숲으로 적절한 작업을 배정을 한다던가 할 필요가 없는 것이다.

또 **Dispatcher-Servlet은 모든 컨트롤러에서 처리해야 할 공통 작업을 처리**하기도 한다. 예외 처리, 인터셉터 처리, 파일 업로드 처리, JSON/XML 직렬화/역직렬화 등의 작업을 처리한다.

# Dispatcher-Servlet의 동작 과정

![](https://velog.velcdn.com/images/sinryuji/post/e3e1f59d-7b5c-467c-8173-543376ccd448/image.png)

1. Client(브라우저)에서 HTTP 요청이 들어오면 DispatcherServlet 객체가 요청을 분석한다.
2. DispatcherServlet 객체는 분석한 데이터를 토대로 Handler mapping을 통해 Controller를 찾아 요청을 전달한다.
> 💡 [Example]
>
GET /api/hello → HelloController 의 hello() 함수
GET /api/user/login → UserController 의 login() 함수
GET /api/user/signup → UserController 의 signup() 함수
POST /api/user/signup → UserController 의 registerUser() 함수
> 
**Handler mapping 에는 API path 와 Controller 메서드가 매칭되어 있다.**

3. 해당 Controller는 요청에 대한 처리를 완료 후 처리에 대한 결과 즉, 데이터('Model')와 'View' 정보를 전달한다.

4. ViewResolver 통해 View에 Model을 적용하여 View를 Client에게 응답으로 전달한다.

Rest API라면 3, 4번 과정이 Response Body를 구성하여 Client에게 전달하는 것이 될 것이다.

이렇듯 Dispatcher-Servlet은 최전방(?)에서 가장 먼저 요청을 처리하고 가장 나중에 응답을 구성하는 역할을 한다!