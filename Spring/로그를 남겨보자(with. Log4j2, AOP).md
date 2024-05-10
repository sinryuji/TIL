# Why Log4j2?

Spring Boot에서는 별도로 로거를 설정해주지 않아도 간단하게 로깅을 할 수 있도록 `logback`이라는 로깅 프레임워크를 기본적으로 사용하고 있습니다. 하지만..!

![](https://i.imgur.com/kuyMxpe.png)

`log4j2`는 자바 진영의 대표적인 로깅 프레임워크 3인방 `logback`, `log4j1`, `log4j2` 셋 중 가장 최신에 나온 프레임워크이며 압도적으로 빠릅니다(물론 다중 쓰레드 환경에서)!

그렇기에 우리는 굳이 간단하게 사용할 수 있는 `logback`을 걷어내고 직접 `log4j2`를 세팅해 사용하기로 했습니다.
# Log4j2 Configuration

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration>

  <Properties>
    <Property name="log_dir">./logs</Property>
    <!-- @formatter:off -->
    <Property name="console_pattern">%style{%d{UTF-8}}{white} %highlight{%-5level} %style{%pid}{magenta} --- [%style{%t}{bright,blue}] %style{%c{1.}}{cyan}: %msg%n%throwable</Property>
    <Property name="file_pattern">%d %p [%t] %c{1.} %m%n</Property>
    <!-- @formatter:on -->
  </Properties>

  <Appenders>
    <Console name="console_appender" target="SYSTEM_OUT">
      <PatternLayout pattern="${console_pattern}"/>
    </Console>

    <RollingFile name="info_appender"
      fileName="${log_dir}/info.log" filePattern="${log_dir}/%d{yyyy}/%d{MM}/%d{dd}/info_%i.log">
      <PatternLayout pattern="${file_pattern}"/>
      <LevelRangeFilter minLevel="INFO" maxLevel="INFO" onMatch="ACCEPT" onMismatch="DENY"/>
      <Policies>
        <SizeBasedTriggeringPolicy size="10MB"/>
        <TimeBasedTriggeringPolicy interval="1"/>
        <TimeBasedTriggeringPolicy/>
      </Policies>
      <DefaultRolloverStrategy max="10" fileIndex="min">
        <!--        <Delete basePath="${log_dir}" maxDepth="3">-->
        <!--          <IfLastModified age="30d"/>-->
        <!--        </Delete>-->
      </DefaultRolloverStrategy>
    </RollingFile>

    <RollingFile name="error_appender"
      fileName="${log_dir}/error.log" filePattern="${log_dir}/%d{yyyy}/%d{MM}/%d{dd}/error_%i.log">
      <PatternLayout pattern="${file_pattern}"/>
      <LevelRangeFilter minLevel="WARN" maxLevel="ERROR" onMatch="ACCEPT" onMismatch="DENY"/>
      <Policies>
        <SizeBasedTriggeringPolicy size="10MB"/>
        <TimeBasedTriggeringPolicy interval="1"/>
        <TimeBasedTriggeringPolicy/>
      </Policies>
      <DefaultRolloverStrategy max="10" fileIndex="min">
        <!--        <Delete basePath="${log_dir}" maxDepth="3">-->
        <!--          <IfLastModified age="30d"/>-->
        <!--        </Delete>-->
      </DefaultRolloverStrategy>
    </RollingFile>
  </Appenders>

  <Loggers>
    <!-- spring logger  -->
    <Logger name="org.springframework" additivity="false" level="INFO">
      <AppenderRef ref="console_appender"/>
      <AppenderRef ref="info_appender"/>
      <AppenderRef ref="error_appender"/>
    </Logger>

    <!-- project logger -->
    <logger name="peer" additivity="false" level="INFO">
      <AppenderRef ref="console_appender"/>
      <AppenderRef ref="info_appender"/>
      <AppenderRef ref="error_appender"/>
    </logger>

    <!-- default logger -->
    <root additivity="false" level="OFF">
      <AppenderRef ref="console_appender"/>
    </root>
  </Loggers>
</Configuration>
```

현재 저희 메인에 올라가있는 Log4j2 설정 파일입니다.
설정 파일에 대한 자세한 설명은 [여기](https://github.com/sinryuji/TIL/blob/main/java/logging/Log4j2%20Configuration.md)를 참고해주세요!

# AOP로 로그를 찍어봅시다.

로그인이 될때마다 그 인증 내역을 로그로 찍고 싶다! 혹은 S3로 파일을 업로드 할 때 마다 그 로그를 찍고 싶다! 할 때마다 해당 비즈니스 로직 내에서 직접 `@Slf4j`로 로그를 찍으셔도 좋지만, 로깅 같은 부차적인 코드는 스프링 [AOP](https://github.com/sinryuji/TIL/blob/main/Spring/AOP.md)를 이용하여 관심사를 완전히 분리하는 것이 좋습니다! 왜냐하면?

1. 메소드나 기능이 추가 될 때 마다 그 모든 곳에 로깅 코드를 작성하는게 귀찮다
2. 핵심 로직과 아예 상관이 없는 로깅 코드가 중간 중간 삽입되어 있는게 가독성에 좋지 않다
3. 로그의 포맷을 수정하고 싶은데... 이걸 다 일일히 찾아가서 수정해야 돼?

이는 `SOLID`에서 `SRP`에 해당하는 부분이며, `Controller`, `Service`, `Repository`등과 같이 관심사 별로 계층을 나누어 객체를 관리하는 Spring의 기본 골조와 이어지는 맥락입니다!

### 그래서 어떻게 하는데?

```java
@Component
@Aspect
@Slf4j
public class RequestLoggingAspect {

    private String paramMapToString(Map<String, String[]> paramMap) {
        return paramMap.entrySet().stream()
            .map(entry -> String.format("%s -> (%s)",
                entry.getKey(), String.join(",", entry.getValue())))
            .collect(Collectors.joining(", "));
    }

    private String getRequestInfo(ProceedingJoinPoint pjp, HttpServletRequest request) {
        StringBuilder builder = new StringBuilder();

        Map<String, String[]> paramMap = request.getParameterMap();
        String params = "";
        if (!paramMap.isEmpty()) {
            params = " [" + paramMapToString(paramMap) + "]";
        }

        return builder
            .append(pjp.getSignature())
            .append(" ")
            .append(request.getMethod())
            .append(" ")
            .append(request.getRequestURI())
            .append(" ")
            .append(params)
            .toString();
    }

    @Pointcut("execution(public * peer.backend.controller..*(..))")
    public void onRequest() {
    }

    @Around("peer.backend.aspect.RequestLoggingAspect.onRequest()")
    public Object loggingApi(ProceedingJoinPoint pjp) throws Throwable {
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()).getRequest();
        ObjectMapper mapper = new ObjectMapper();
        String requestInfo = this.getRequestInfo(pjp, request);

        try {
            long start = System.currentTimeMillis();
            log.info("[Request] {}", requestInfo);
            Object result = pjp.proceed();
            long end = System.currentTimeMillis();
            log.info("[Response] {}: {} < ({}ms)", requestInfo,
                mapper.writeValueAsString(result), end - start);
            return result;
        } catch (Exception e) {
            log.error("[Error] {} {}", requestInfo, e);
            throw e;
        }
    }
}

```

`Aspect`를 구현할 땐 `@Aspect`로 `Aspect`임을 명시해주고(당연...) `@Component`로 컨테이너에 빈으로 등록해주셔야 합니다.
Aspect는 Advice + Pointcut이라고 생각하시면 됩니다!
- `Advice`: 부가적인 기능을 담은 구현체, 위 코드를 예시로 들면 loggingApi 메소드로 등록되는 빈 그자체를 생각하시면 됩니다.
- `Pointcut`: 여러 JointPoint의 집합체로 필터링된 JoinPoint를 의미합니다. 표현식을 통해 필터링을 수행함!
- `Joinpoint`: 인스턴의 생성시점, 메소드를 호출하는 시점, Exception이 발생하는 시점과 같이 어플리케이션이 실행될 때 특정작업이 실행되는 시점을 의미합니다.
즉, 어플리케이션이 실행되며 쏟아지는 `Joinpoint`에서 내가 원하는 `Joinpoint`만을 `Pointcut`으로 낚아 챈 뒤, 그 포인트에 `Advice`에 구현해 놓은 내용이 동작을 한다! 라고 생각하시면 됩니다.

위 코드를 예시로 들자면, peer.backend.controller 패키지 내부에 있는 모든 public 메소드 들이 실행(execution)되는 시점을 포인트 컷으로 잡고 그 내부에서 로그를 찍고 있는 것입니다.
@Around의 경우 해당 조인 포인트의 앞, 뒤, 에러 발생 시점 등 모든 조인 포인트에서 동작을 하는 어노테이션이고 외에도 @Before, @After, @After-returning, @After-throwing이 있습니다.
이 중 @Arroud를 사용한 이유는 API 처리에 걸린 시간을 측정하기 위해서였고, @Before로 리퀘스트 로그를, @After-returning으로 리스폰스 로그를, @After-throwing으로 에러 로그를 찍어도 무방했습니다.

`pjp.proceed()`에서 메소드가 실행되고, 저 pjp 객체에서 `getArgs()`를 통해 아규먼트들을 뽑아와 활용을 할 수도 있습니다.