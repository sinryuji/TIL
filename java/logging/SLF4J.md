# SLF4J(Simple Logging Facade for Java)란?

Java에는 `java.util.logging`, `logback`, `log4j`와 같은 다양한 `Logging Framework`들이 존재하는데, `SLF4J(Simple Logging Facade for Java)`는 `추상화(인터페이스) 프레임워크`로 이 다양한 로깅 프레임워크들에 대해 `Facade 패턴`을 통해 획일화된 사용 방법을 제공함으로써 사용자로 하여금 어떤 로깅 프레임워크를 사용하든 그 실제 구현을 신경쓰지 않고 통일된 방법으로 로깅 프레임워크를 사용할 수 있게 해준다.
그래서 만약 로깅 프레임워크가 변경되더라도 SLF4J를 통해 작성된 코드는 수정할 필요가 없게 되므로 로깅 프레임워크 .jar 파일을 교체하는 것 만으로도 매우 간단하게 로깅 프레임워크를 변경 할 수 있다.

# @Slf4J

`@Slf4`는 `Lombok`에서 제공하는 어노테이션으로, `Lombok`에서 제공하는 다른 어노테이션들과 마찬가지로 개발자가 `SLF4J`를 사용하기 위해 작성해야하는 코드들을 생략할 수 있게끔 해준다.
```java
Example:
  @Slf4j
  public class LogExample {
  }
  
will generate:
  public class LogExample {
	  private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExample.class);
  }
```

실제 `@Slf4J` 어노테이션을 보면 위와 같은 주석이 달려있음을 확인할 수 있다. `@Slf4j` 어노테이션을 붙이면 `private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExample.class);` 필드를 자동으로 생성해준다는 뜻이다.

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class TeamService {

    private final UserRepository userRepository;

    private final TeamRepository teamRepository;

    private final TeamUserRepository teamUserRepository;

    @Transactional
    public List<Team> getTeamList(Long userId) {
        User user = this.userRepository.findById(userId)
            .orElseThrow(() -> new NotFoundException("존재하지 않는 유저 아이디 입니다."));
        log.info("hello");
        log.error("error");

        return user.getTeamUsers().stream().map(TeamUser::getTeam)
            .collect(Collectors.toList());
    }

```
실제 코드에 `Slf4j`를 사용해 본 모습이다. 로그가 찍히는 모습이 다음과 같다.
![[Pasted image 20230828145656.png]]
