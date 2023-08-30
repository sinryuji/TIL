```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration>

  <!-- File Information -->
  <Properties>
    <Property name="file_name">./logs/${date:yyyy-MM-dd}.log</Property>
    <!-- @formatter:off -->
    <Property name="console_log_pattern">%style{%d{UTF-8}}{white} %highlight{%-5level} %style{%pid}{magenta} --- [%style{%t}{bright,blue}] %style{%c{1.}}{cyan}: %msg%n%throwable</Property>
    <Property name="file_log_pattern">%d %p [%t] %c{1.} %m%n</Property>
    <!-- @formatter:on -->
  </Properties>

  <Appenders>
    <Console name="console_appender" target="SYSTEM_OUT">
      <PatternLayout pattern="${console_log_pattern}"/>
    </Console>

    <File name="file_appender">
      <fileName>${file_name}</fileName>
      <PatternLayout>
        <pattern>${file_log_pattern}</pattern>
      </PatternLayout>
    </File>
  </Appenders>

  <Loggers>
    <!-- Spring log  -->
    <Logger name="org.springframework" additivity="false" level="INFO">
      <AppenderRef ref="console_appender"/>
      <AppenderRef ref="file_appender"/>
    </Logger>

    <!-- project log -->
    <logger name="peer" additivity="false" level="DEBUG">
      <AppenderRef ref="console_appender"/>
      <AppenderRef ref="file_appender"/>
    </logger>

    <!-- auth log  -->
    <!--    <logger name="com.pacakge.projectname.common.filter" additivity="false" level="INFO">-->
    <!--      <AppenderRef ref="Console_Appender"/>-->
    <!--      <AppenderRef ref="File_Info_Appender"/>-->
    <!--      <AppenderRef ref="File_Error_Appender"/>-->
    <!--    </logger>-->

    <!-- default setting -->
    <root additivity="false" level="OFF">
      <AppenderRef ref="console_appender"/>
    </root>
  </Loggers>
</Configuration>

```

# Configuration

```xml
<Configuration status="INFO">
```

`Configuration`은 로그 설정의 최상위 요소이다. 일반적으로 `Configuration`은 `Properties`, `Appenders`, `Loggers` 들을 자식으로 가진다.
`status` 속성은 log4j의 내부 이벤트에 대한 로그 레벨을 의미한다.

# Properties

```xml
  <Properties>
    <Property name="file_name">./logs/${date:yyyy-MM-dd}.log</Property>
    <!-- @formatter:off -->
    <Property name="console_log_pattern">%style{%d{UTF-8}}{white} %highlight{%-5level} %style{%pid}{magenta} --- [%style{%t}{bright,blue}] %style{%c{1.}}{cyan}: %msg%n%throwable</Property>
    <Property name="file_log_pattern">%d %p [%t] %c{1.} %m%n</Property>
    <!-- @formatter:on -->
  </Properties>
```

`Properties`는 해당 Configuration 파일에서 사용할 프로퍼티를 의미한다. 변수라고 생가하면 편하다!

# Layout

`Appender`는 로그 이벤트를 `Layout`을 통해 포맷팅한다. `Appender`가 받은 로그 이벤트를 `Layout`에게 전달하면, Byte 배열을 반환한다. Charset을 지정해주면 확실한 값을 얻을 수 있다.

## PatternLayout

```xml
%style{%d{UTF-8}}{white} %highlight{%-5level} %style{%pid}{magenta} --- [%style{%t}{bright,blue}] %style{%c{1.}}{cyan}: %msg%n%throwable
```

![](https://i.imgur.com/LuOHYXu.png)


`Layout`을 설정한대로 로그를 출력한 결과이다. 날짜 포맷 , 색깔, 출력 요소등을 정할 수 있다. C언어의 printf 처럼 Format 문자를 사용한다. 문법은 다음과 같다.

- **%c, %logger** : 해당 로그를 쓰는 로거의 이름.
- **%C, %class** : 해당 로그를 요청한 클래스 이름
- **%d, %date** : 해당 로그가 발생한 시간
- **%enc, %encode** : 특정 언어에서의 출력을 위한 문자 인코딩
- **%ex, %exception, %throwable** : 예외 로그. 길이를 설정할 수 있음.
- **%F, %file** : 해당 로그가 발생한 클래스 파일명
- **%l, %location** : 해당 로그가 발생한 클래스명.메소드명(파일:라인)
- **%L, %line** : 해당 로그가 발생한 라인 번호
- **%m, %msg, %message** : 로그문에 전달된 메시지
- **%n** : 줄바꿈
- **%p, %level** : 로그 레벨
- **%r, %relative** : 로그 처리시간
- **%t, %thread** : 해당 로그가 발생한 스레드명
- **%style{pattern}{ANSI style}** : ANSI를 사용해 특정 패턴을 스타일링함
- **%highlight{pattern}{style}** : 로그 레벨명을 ANSI 색깔로 하이라이트

# Appenders

`Appender`는 로그 메세지를 특정 위치에 전달해주는 녀석이다. `Appender`는 `Layout`을 통해 로그를 포맷팅하고 어떤 방식으로 로그를 제공할지 결정한다. `Appender`에는 여러 종류가 있다.

- ConsoleAppender
- RollingFileAppender
- AsyncAppender
- FileAppender
- JDBCAppender
- SMTP
- ...

## ConsoleAppender

```xml
<Console name="console_appender" target="SYSTEM_OUT">
  <PatternLayout pattern="${console_log_pattern}"/>
</Console>
```

`ConsoleAppender`는 말 그대로 콘솔에 system.out으로 로그를 출력하는 `Appender`이다. 로그 출력 포맷팅은 `PatternLayout` 방식으로 위에서 설정한 Pattern 대로 로그를 출력한다.
`ConsoleAppender`가 갖는 속성은 다음과 같다.

- **layout** : 출력 레이아웃 양식
- **name** : Appender 이름
- **target** : 출력 방식. 기본값은 SYSTEM_OUT, SYSTEM_ERR로 출력가능

## RollingFileAppender

```xml
<RollingFile name="file_appender" fileName="logs/${logNm}.log" filePattern="logs/${logNm}_%d{yyyy-MM-dd}_%i.log.gz">
    <PatternLayout pattern="${layoutPattern}"/>
    <Policies>
         <SizeBasedTriggeringPolicy size="200KB"/>
         <TimeBasedTriggeringPolicy interval="1"/>
    </Policies>
    <DefaultRolloverStrategy max="10" fileIndex="min"/>
</RollingFile>
```

로그가 한 파일에 계속 저장된다면, 파일이 너무 커져서 실행이 불가능할 수도 있고, 파일에 문제가 생기면 로그 전부를 날려버릴 수도 있다. `RollingFileAppender`는 이런 문제를 해결해준다. 파일에 로그를 기록하고, 특정 기준에 따라 압축하여 저장하는 방식의 `Appender`이다. `Console Appender`와 대표적으로 다른 부분은 File과 관련된 속성, `Policy`와 관련된 속성, `Strategy`와 관련된 속성이다.

### 1. Policy

`Policy`는 `File Rolling Up`의 기준이다. 하나의 `RollingFileAppender`에는 여러 `Policy`를 적용할 수 있다.

- **OnStartupTriggeringPolicy** : jvm start시 trigger
- **TimeBasedTriggeringPolicy** : time에 따른 trigger
- **SizeBasedTriggeringPolicy** : file size에 따른 trigger
- **CronTriggeringPolicy** : Cron Expression(시간에 관한 표현)에 따른 trigger

이름처럼 각각의 `Policy`들은 특정 조건에 따라 trigger된다. 주어진 예제는 로그파일이 200KB가 되는 시점과 하루 마다 `logs/${logNm}_%d{yyyy-MM-dd}_%i.log.gz` 패턴으로 압축되어 저장된다.
여기서 재밌는 점은 `TimeBasedTriggeringPolicy`의 `interval`은 파일 패턴의 날짜 단위의 최소값으로 계산된다는 점이다. 만약 파일 패턴의 날짜 포맷이 초단위까지 있다면, `interval`을 1로하면 1초에 한번씩 압축이 되는 셈이다.

### 2. DefaultRolloverStrategy

`DefaultRolloverStrategy`는 `datetime` 패턴과 파일 패턴의 int값을 받아서 결정됩니다. `datetime`은 현재 시간으로 대체되고, 파일 패턴 숫자는 DB의 `autoincrement`처럼 `rollover` 마다 1씩 증가합니다. 예제의 filePattern을 보시면 `%d{yyyy-MM-dd}` 가 날짜 패턴, `%i`가 파일패턴의 int입니다. 각각 `rollover` 이후에는 다음과 같이 저장됩니다.

![](https://i.imgur.com/7vm7gOV.png)

해당 `Strategy`에는 다음 속성을 부여할 수 있다.

- **fileIndex** : max로 설정 시 높은 index가 더 최신 파일이 됩니다. min으로 설정 시 작은 index가 최신 파일이 됩니다. 기존의 파일들을 rename하는 방식으로 동작합니다.
- **min** : counter 최소값. 기본값은 1입니다.
- **max** : counter 최대값. 만약 최대값에 도달하면 오래된 파일을 삭제합니다. 기본값은 7입니다.
- **compressionLevel** : 0~9까지 정수값. 0은 압축하지 않고, 1은 최고 속도, 9는 최고 압축입니다. ZIP파일만 가능합니다.
- **tempCompressedFilePattern** : 압축하는 동안의 파일 이름 패턴.

# Logger

```xml
<Logger name="org.springframework" additivity="false" level="INFO">
  <AppenderRef ref="console_appender"/>
  <AppenderRef ref="file_appender"/>
</Logger>
```

로거는 로깅을 직접하는 요소이다. 로거는 `name`을 통해 패키지별로 설정할 수 있다.
`AppenderRef`를 통해 앞서 설정했던 `Appender`들을 등록할 수 있고, `level`에 설정해놓은 레벨 이상의 로그를 찍게 되고. `additivity`의 경우 중복된 로그를 남길지에 대한 속성이다.