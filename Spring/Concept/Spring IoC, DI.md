# IoC란?
**제어의 역전(Inversion of Control, IoC)**이란 소프트웨어 디자인인의 중요한 원칙 중 하나로, **프로그램의 제어 흐름을 개발자가 직접 관리하지 않고, 외부 프레임워크나 컨테이너가 관리하도록 위임하는 설계 원칙**이다.

프레임워크를 사용하지 않는 전통적인 개발 방식은 개발자가 프로그램의 실행 흐름을 결정한다. 그 흐름에 따라 객체를 직접 생성하고 의존성을 연결하기도 하며 외부 라이브러리를 호출하기도 한다. 그런데 Spring과 같이 IoC 원칙을 따르는 프레임워크를 사용하여 개발을 할 때는 **이미 설계되어 있는 프레임워크의 프로그램 실행 흐름에 따라 필요한 부분들을 개발하는 방식**으로 진행한다. 이를 통해 객체의 생명 주기, 의존성 주입을 모두 외부 컨테이너가 담당을 하게 되고 개발자들은 오직 비즈니스 로직을 구현하는데만 집중 할 수 있게 해준다.

# DI란?
**의존성 주입(Dependency Injection, DI)**은 디자인 패턴의 일종으로 IoC를 구현하는 방법 중 하나이다. **DI는 객체의 의존성을 내부가 아니라 외부에서 주입하여 해결을 하는 방법**이다.

객체지향 언어에서 의존성을 해결하는 방법 중 하나는 의존성을 객체 내부에서 해결하는 것이다. 다음 코드를 보자.

```java
public class Person {

	Dog dog;
    
    public Person() {
    	this.dog = new Dog();
    }

    void playWithPet() {
        this.dog.sound();
    }

    public static void main(String[] args) {
        Person person = new Person();
        person.playWithPet();
    }
}

class Dog {
    public void sound() {
        System.out.println("Bow-wow");
    }
}
```

`Person` 객체가 생성자를 통해 내부적으로 `Dog` 객체를 생성하고 있다. 위와 같은 방식은 가장 직관적으로 떠오를 수 있는 구현 방법이지만 너무 강한 결합도를 가지고 있다는 단점이 존재한다. 

만약 **애완동물을 개가 아니라 고양이로 바꾸고 싶다면 `Person` 클래스의 생성자부터 필드, `playWithPet()` 함수까지 수정을 해야한다.** 이는 확장에 대해서는 개방되어 있어야 하지만 수정에 대해서는 폐쇄되어 있어야 한다는 **`SOLID`의 OCP(Open Close Principle, 개방 폐쇄 원칙)에 위배**된다.

물론 DI가 없이도 객체지향의 다형성을 이용하여 OCP 원칙을 지킬 수 있다. 다음 코드를 보자.

```java
public class Person {

    Animal animal;

    void adoptDog() {
        this.animal = new Dog();
    }

    void adoptCat() {
        this.animal = new Cat();
    }

    void playWithPet() {
        if (this.animal != null) {
            this.animal.sound();
        } else {
            System.out.println("I don't have a pet");
        }
    }

    public static void main(String[] args) {
        Person person1 = new Person();
        person1.adoptDog();
        person1.playWithPet();

        Person person2 = new Person();
        person2.adoptCat();
        person2.playWithPet();
    }
}

interface Animal {
    void sound();
}

class Dog implements Animal {
    public void sound() {
        System.out.println("Bow-wow");
    }
}

class Cat implements Animal {
    public void sound() {
        System.out.println("Mew");
    }
}
```

`Dog`와`Cat` 클래스를 `Animal` 인터페이스의 구현체로 구현함으로써 애완동물이 변경되더라도 기존의 코드를 수정할 필요가 없어졌다. 다른 애완동물이 추가되더라도 `Animal`의 구현체를 하나 더 구현하면 된다. 다만 해당 구현체를 할당하는 함수를 하나 더 추가해야 하고, **여전히 강한 결합도를 가지고 있다는 점은 유효**하다. 그로 인해 `playWithDog()`를 호출하기 전에 `adoptDog()`나 `adoptCat()`을 반드시 호출해야 하고 이 때문에 null-check 로직 또한 추가되어야만 했다. 

만약 이 코드를 직접 구현한 개발자라면 `Person` 클래스를 사용하는데 있어서 아무런 문제가 없을 것이다. 하지만 해당 개발자가 퇴사를 하거나, 협업 과정에서 다른 개발자가 이미 구현된 `Person` 클래스를 이용해야 하는 경우는 비일비재하다. 이런 경우에 코드를 자세히 뜯어보지 않는 이상 **`plathWithPet()`을 호출 한 뒤 예상과 동작이 달라 런타임에 디버깅을 하는 과정을 반드시 거칠 것**이다.

이를 다음과 같이 **DI 패턴을 통해 의존성을 내부에서 해결하는 것이 아닌, 외부에서 주입을 받아 해결**할 수 있다.

```java
public class Person {

    Animal animal;

    public Person(Animal animal) {
        this.animal = animal;
    }

    void playWithPet() {
        this.animal.sound();
    }

    public static void main(String[] args) {
        Person person1 = new Person(new Dog());
        person1.playWithPet();

        Person person2 = new Person(new Cat());
        person2.playWithPet();
    }
}

interface Animal {
    void sound();
}

class Dog implements Animal {
    public void sound() {
        System.out.println("Bow-wow");
    }
}

class Cat implements Animal {
    public void sound() {
        System.out.println("Mew");
    }
}
```

앞선 코드와 달리 `Dog`나 `Cat`을 생성자 파라미터를 통해 **주입 받아 생성**하고 있다. 이렇게 되면 **`playWithPet()`을 호출하기 전에 `Animal`을 할당해주는 코드를 호출해야할 필요가 없어졌고** `Person`의 생성자 파라미터에 `Animal`이 존재하므로 **할당의 필요성을 컴파일 과정에서 인지**할 수 있다. 이를 컴파일 과정에서 인지하는 것과 런타임 과정에서 인지하는 것은 하늘과 땅 차이이다!!

# Spring의 IoC/DI

앞서 DI는 IoC를 구현하는 방법 중 하나라고 했다. [Spring 공식 문서](https://docs.spring.io/spring-framework/reference/core/beans/introduction.html)에서도 **IoC Container에 대한 설명으로 "DI 패턴을 사용하여 IoC 설계 원칙을 구현하고 있다"라고 서술**하고 있다.

이를 풀어 설명하면, Spring은 IoC 설계 원칙을 지켜 객체의 의존성을 프레임워크에서 해결하기 위해 **개발자들이 구현한 객체들의 의존성을 IoC Container가 외부에서 주입하는 DI 패턴으로 구현**하고 있다고 이해를 하면 된다.

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/api/v1/board")
public class BoardController {

    private final BoardService boardService;
    
    ...
}
```

그 덕에 우리는 위와 같이 Spring을 이용하여 개발을 할 때 객체 초기화나 의존성 같은 걸 고민할 필요 없이 필드를 선언하고 생성자를 만들어주는 것만으로도 충분한 것이다!