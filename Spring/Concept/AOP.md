# AOP(Aspect Oriented Programming)란?

`AOP`는 `Aspect Oriented Programming`의 약자로 `관점 지향 프로그래밍`이라 불린다. 관점 지향은 모듈화를 할 때 어떤 로직을 기준으로 핵심적인 관점, 부가적인 관점으로 나누어 모듈화를 하겠다는 것을 의미한다. 
예를 들어, **핵심적인 관점은 서비스의 핵심 비즈니스 로직**이 된다. 그리고 **부가적인 관점은 핵심적인 로직을 실행하기 위한 데이터베이스 연결, 로깅, 파일 입출력 등**이 된다.
`AOP`는 이 부가적인 관점에 해당하는 코드들의 중복을 최소한으로 하기 위해 `횡단 관심사(cross-cutting-concern)`의 분리를 허용함으로써 모듈화를 증가시키는 것이 목적인 프로그래밍 패러다임이다.
간단하게 말해, 아래 그림과 같이 중복되는 부가적인 코드들을 따로 모듈화를 하는 것이라고 생각하면 된다.
![](https://i.imgur.com/0d98UAM.png)

# AOP 주요 개념

- **Aspect**: 위에서 설명한 횡단 관심사들을 모듈화 한 것.
- **Target**: Aspect를 적용하는 곳(클래스, 메서드 등)
- **Advice**: 실질적인 부가 기능을 담은 구현체
- **JointPoint**: **Advice**가 적용될 위치, 끼어들 수 있는 지점. 예를 들어 메서드 진입 지점, 생성자 호출 시점, 필드에서 값을 꺼내올 때 등 다양한 시점에 적용 가능
- **PointCut**: JointPoint의 상세한 스펙을 정의한 것. 'A란 메서드의 진입 시점에 호출할 것'과 같이 더욱 구체적으로 Advice가 실행될 지점을 정할 수 있음

# 스프링 AOP 특징

- 프록시 패턴 기반의 **AOP** 구현체, 프록시 객체를 쓰는 이유는 접근 제어 및 부가 기능을 추가하기 위해서임
- 스프링 빈에만 **AOP**를 적용 가능
- 모든 **AOP**기능을 제공하는 것이 아닌 스프링 **IoC**와 연동하여 엔터프라이즈 애플리케이션에서 가장 흔한 문제(중복 코드, 플고