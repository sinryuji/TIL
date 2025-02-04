---
title: "JPA 영속성 컨텍스트 총 정리"
description: "JPA의 영속성 컨테스트에 대해 알아보자!"
date: 2025-02-03T16:30:49.461Z
tags: ["JPA"]
---
# 영속성 컨텍스트란?
![](https://velog.velcdn.com/images/sinryuji/post/b627a72e-6b69-4dcb-a3c8-2c6f7d7d63aa/image.png)

**Persistence Context**에서 Persistence는 영속성, 지속성이라는 뜻을 가지고 있다. 즉, 이를 객체의 관점에서 보자면 객체의 생명과 성질이 지속되는 공간으로 볼 수 있다. JPA에서의 영속성 컨텍스트는 **Entity 들의 변경 사항이 바로 DB에 반영되지 않고 임시로 그 상태를 저장하는 환경**이 된다.

# EntityManager
![](https://velog.velcdn.com/images/sinryuji/post/33615276-56d6-4f87-9d9b-ca1f06796bfb/image.png)

이러한 영속성 컨텍스트에 접근하여 **Entity 객체들을 조작하기 위해서는 EntityManager가 필요**하고 이는 EntityManagerFactory를 통해 생성하여 사용할 수 있다.

EntityManagerFactory는 일반적으로 DB 하나에 하나만 생성되어 애플리케이션이 동작하는 동안 사용된다. EntityManagerFactory를 만들기 위해서는 DB에 대한 정보를 전달해야한다.

정보를 전달하기 위해서는 /resources/META-INF/  위치에 persistence.xml 파일을 만들어 정보를 넣어두면 된다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="memo">
        <class>com.sparta.entity.Memo</class>
        <properties>
            <property name="jakarta.persistence.jdbc.driver" value="com.mysql.cj.jdbc.Driver"/>
            <property name="jakarta.persistence.jdbc.user" value="root"/>
            <property name="jakarta.persistence.jdbc.password" value="{비밀번호}"/>
            <property name="jakarta.persistence.jdbc.url" value="jdbc:mysql://localhost:3306/memo"/>

            <property name="hibernate.hbm2ddl.auto" value="create" />

            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
        </properties>
    </persistence-unit>
</persistence>
```

다음 코드를 보자.

```java
import jakarta.persistence.EntityManager;
import jakarta.persistence.EntityManagerFactory;
import jakarta.persistence.Persistence;

EntityManagerFactory emf = Persistence.createEntityManagerFactory("memo");
EntityManager em = emf.createEntityManager();
```

`Persistence.createEntityManagerFactory("memo")`를 통해  persistence.xml의 정보를 토대로 EntityManagerFactory를 생성한다. 그 후 `emf.createEntityManager()`를 통해 EntityManager를 생성 할 수 있다.

# 영속성 컨텍스트의 기능

## 1차 캐싱
![](https://velog.velcdn.com/images/sinryuji/post/5c978297-1a37-4378-be0b-0e2f422871b6/image.png)

영속성 컨텍스트에는 유용한 기능들이 많지만 가장 중요한 기능이 바로 1차 캐싱이다. 앞서 영속성 컨텍스트를 **Entity 들의 변경 사항이 바로 DB에 반영되지 않고 임시로 그 상태를 저장하는 환경**이라 하였다.

만약 Entity들의 변경사항이 있을 때 마다 DB에 쿼리를 날리면 DB에 많은 부하가 발생할 것이다. 그래서 영속성 컨텍스트는 **마치 DB의 Transaction처럼 1차 캐시에 변경 사항들을 저장하고 있다가 마지막에 commit이 되는 순간 그 변경 사항들을 모두 적용하는 것이다.**

이 캐시 저장소는 Map 자료구조 형태로 되어있다. **Key에는 @Id로 매핑한 테이블의 Primary key에 해당하는 식별자**를 저장하고 **Value에는 Entity 객체**를 저장한다.

### EntityTransaction과 Entity 저장

다음 코드를 보자.

```java
    @Test
    @DisplayName("1차 캐시 : Entity 저장")
    void test1() {
        EntityTransaction et = em.getTransaction();

        et.begin();

        try {

            Memo memo = new Memo();
            memo.setId(1L);
            memo.setUsername("Robbie");
            memo.setContents("영속성 컨텍스트와 트랜잭션 이해하기");

            em.persist(memo);

            et.commit(); 
        } catch (Exception ex) {
            ex.printStackTrace();
            et.rollback();
        } finally {
            em.close();
        }

        emf.close();
    }
```

우선 `em.getTransaction()`을 통해 EntityTrasaction을 가져온다. 이후 `et.begin()`을 통해 트랜잭션을 시작한다.

![](https://velog.velcdn.com/images/sinryuji/post/ae72eb5d-b813-455f-87c7-a6aed7d1284e/image.png)


`em.persist(memo)`을 통해 Memo 객체를 `영속화`한다. **이 영속화 시점에서 객체를 1차 캐시에 저장**을 하게 되는 것이다.

이후 `et.commit()`를 호출하게 되면 이 때 1차 캐시에 저장된 변경 사항들이 DB에 반영이 된다.

> 💡 트랜잭션이라 하여서 JPA가 DB의 트랜잭션 기능을 사용하는 것이라고 착각할 수 있는데 실제로 DB의 트랜잭션을 사용하는 것은 아니고, DB의 트랜잭션 개념을 그대로 가져와 1차 캐싱을 구현하였으므로 트랜잭션이라고 칭하는 것이며 캐시 저장소를 통해 자체적으로 마치 트랜잭션과 동일하게 돌아가는 것과 같이 구현이 되어있는 것 뿐이다!

> 💡 그리고 또 주의해야할 점이 영속성 컨텍스트에 조회를 할 때는 필수가 아니지만 영속성 컨텍스트에 Entity를 저장하거나 수정을 할 때는 반드시 JPA 트랜잭션 내에서만 가능하다.

실제로 디버깅 과정을 통해 1차 캐시에 엔티티가 담기는 것을 눈으로 확인 할 수 있다.

![](https://velog.velcdn.com/images/sinryuji/post/73d13799-345a-45d4-838c-dcfd8d38cbbe/image.png)

`em.persist(memo)`만 호출하고 아직 `et.commit()`을 호출하지 않은 상태이다. `persistenctContext`의 `entitiesByKey`에 Key-Value로 Entity가 잘 저장되어 있는 것을 확인 할 수 있다.

### Entity 조회

![](https://velog.velcdn.com/images/sinryuji/post/3811843c-3dc7-4077-8c56-6f8283ad1683/image.png)

Entity를 조회할 때는 당연하게도 영속성 컨텍스트의 1차 캐시에 해당 Entity가 존재하는지 먼저 체크한다. **존재한다면 그 Entity를 반환하고 존재하지 않는다면 그 때 DB에 select 쿼리를 날린다.**

다음 코드를 보자.

```java
    @Test
    @DisplayName("Entity 조회 : 캐시 저장소에 해당하는 Id가 존재하지 않은 경우")
    void test2() {
        try {

            Memo memo = em.find(Memo.class, 1);
            System.out.println("memo.getId() = " + memo.getId());
            System.out.println("memo.getUsername() = " + memo.getUsername());
            System.out.println("memo.getContents() = " + memo.getContents());


        } catch (Exception ex) {
            ex.printStackTrace();
        } finally {
            em.close();
        }

        emf.close();
    }
```

위 코드에서 `em.find(Memo.class, 1)`를 통해 영속성 컨텍스트에 Memo 객체이면서 아이디가 1인 Entity가 존재하는지 확인한다. 그런데 최초 조회에서는 당연히 영속성 컨텍스트가 비어있으므로 Select 쿼리가 날아간 후 memo의 정보가 출력 될 것이다. 다음과 같이 말이다.

![](https://velog.velcdn.com/images/sinryuji/post/518ada3a-d3ef-4748-a873-ce42f1c3fec2/image.png)

하지만 다음 코드를 보자.

```java
    @Test
    @DisplayName("Entity 조회 : 캐시 저장소에 해당하는 Id가 존재하는 경우")
    void test3() {
        try {

            Memo memo1 = em.find(Memo.class, 1);
            System.out.println("memo1 조회 후 캐시 저장소에 저장\n");

            Memo memo2 = em.find(Memo.class, 1);
            System.out.println("memo2.getId() = " + memo2.getId());
            System.out.println("memo2.getUsername() = " + memo2.getUsername());
            System.out.println("memo2.getContents() = " + memo2.getContents());


        } catch (Exception ex) {
            ex.printStackTrace();
        } finally {
            em.close();
        }

        emf.close();
    }
```

첫 번째 `em.find(Memo.class, 1)`에서 select 쿼리가 수행되고 영속성 컨텍스트에 Entity가 담길 것이다. 그러니 두 번째 `em.find(Memo.class, 1)`에서는 select 쿼리가 수행되지 않는다. 다음과 같이 말이다.

![](https://velog.velcdn.com/images/sinryuji/post/2d6d7898-23dc-4547-b2b7-95ccc4459259/image.png)

select 쿼리가 한 번만 날아간 것을 확인할 수 있다.

또한 영속성 컨텍스트는 DB row당 오직 하나의 객체만이 사용되는 것, 즉 **객체 동일성을 보장**한다. 다음 코드를 보자.

```java
@Test
@DisplayName("객체 동일성 보장")
void test4() {
    EntityTransaction et = em.getTransaction();

    et.begin();
    
    try {
        Memo memo3 = new Memo();
        memo3.setId(2L);
        memo3.setUsername("Robbert");
        memo3.setContents("객체 동일성 보장");
        em.persist(memo3);

        Memo memo1 = em.find(Memo.class, 1);
        Memo memo2 = em.find(Memo.class, 1);
        Memo memo  = em.find(Memo.class, 2);

        System.out.println(memo1 == memo2);
        System.out.println(memo1 == memo);

        et.commit();
    } catch (Exception ex) {
        ex.printStackTrace();
        et.rollback();
    } finally {
        em.close();
    }

    emf.close();
}
```

새로운 Memo를 영속화 한 후 두 번 `em.find(Memo.class, 1)`를 해와서 각각 memo1과 memo2에 담았다. 이것이 일반적인 Java 객체라면 위와 같이 서로 다른 객체에 저장이 되면 그 값이 같더라도 서로 다른 메모리 주소에 저장이 되어 서로 다른 객체임을 알고 있을 것이다. 그런데 다음을 보자.

![](https://velog.velcdn.com/images/sinryuji/post/8433af68-c592-43f1-84eb-4bc55b134182/image.png)

첫 `em.find(Memo.class, 1)`에 select 쿼리가 날아가고 이후 memo1과 memo2를 비교한 코드에서 true가 출력이 되었다. 이게 바로 객체 동일성을 보장하는 것이다. 당연하게도 다른 Id를 가진 memo와 비교한 결과는 flase가 출력된다. 마지막 insert 쿼리는 `et.commit()`을 할 때 발생하며, 앞서 영속화를 했던 memo3에 대한 쿼리이다.

### Entity 삭제

![](https://velog.velcdn.com/images/sinryuji/post/a4fe10cc-6f2c-41ba-a8f1-2babe79a7c95/image.png)

삭제 역시 조회와 마찬가지로 먼저 1차 캐시에 해당 Entity가 존재하는지를 조회한다. 만약에 존재하지 않는다면 DB에 select 쿼리를 날려 1차 캐시에 저장을 한다. 그 후 **`em.remove(entity)`를 호출하면 1차 캐시의 Entity가 DELETED 상태가 되며 이후 트랜잭션 commit시 DB에 delete 쿼리가 날아가게 된다.**

```java
@Test
@DisplayName("Entity 삭제")
void test5() {
    EntityTransaction et = em.getTransaction();

    et.begin();

    try {

        Memo memo = em.find(Memo.class, 2);

        em.remove(memo);

        et.commit();

    } catch (Exception ex) {
        ex.printStackTrace();
        et.rollback();
    } finally {
        em.close();
    }

    emf.close();
}
```

그 과정 역시 디버깅을 통해 확인 할 수 있다. 위 코드에서 `em.find(Memo.class, 2)`가 호출된 시점을 보자. 이 때 em -> persistenceContext -> entityEntryContext -> nonEnhancedEntityXref를 보면 1차 캐시에 다음과 같이 Entitiy가 MANAGED 상태임을 볼 수 있다. 영속성 컨텍스트에 의해 관리되고 있는 상태를 의미한다.

![](https://velog.velcdn.com/images/sinryuji/post/56fe69bb-472e-4ee0-8f36-90ade3942f43/image.png)

이후 다시 `em.remove(memo)`가 호출 된 후 확인흘 해보면 DELETED 상태가 된 Entity를 확인 할 수 있다. 이 상태의 Entity는 commit이 되면 DB에 delete 쿼리가 날아가게 된다.

![](https://velog.velcdn.com/images/sinryuji/post/e20af748-6e51-489b-b196-4c7e1660f097/image.png)

## 쓰기 지연 저장소(ActionQueue)

![](https://velog.velcdn.com/images/sinryuji/post/bfdb469d-ca63-497e-a027-b7611509168b/image.png)

앞서 JPA는 트랜잭션에 의해 commit이 호출되기 전까지 변경 사항을 모아 한 번에 DB에 반영을 한다고 하였다. JPA를 이를 구현하기 위해 **쓰기 지연 저장소(ActionQueue)**를 만들어 쿼리를 모아두었다가 순서대로 적용을 시킨다.

다음 코드를 통해 이를 확인해보자.

```java
@Test
@DisplayName("쓰기 지연 저장소 확인")
void test6() {
    EntityTransaction et = em.getTransaction();

    et.begin();

    try {
        Memo memo = new Memo();
        memo.setId(2L);
        memo.setUsername("Robbert");
        memo.setContents("쓰기 지연 저장소");
        em.persist(memo);

        Memo memo2 = new Memo();
        memo2.setId(3L);
        memo2.setUsername("Bob");
        memo2.setContents("과연 저장을 잘 하고 있을까?");
        em.persist(memo2);

        System.out.println("트랜잭션 commit 전");
        et.commit();
        System.out.println("트랜잭션 commit 후");

    } catch (Exception ex) {
        ex.printStackTrace();
        et.rollback();
    } finally {
        em.close();
    }

    emf.close();
}
```

`em.persist(memo)`에 브레이크 포인트를 걸어 memo 객체가 영속화가 되는 시점을 살펴보자.

`em.persist(memo)`가 호출되면 ActionQueue에는 insert 쿼리가 하나 쌓이게 된다. 다음과 같이 말이다.

![](https://velog.velcdn.com/images/sinryuji/post/2def5476-bca0-45a3-af66-c46d670fdc9a/image.png)

em의 actionQueue의 insertions의 사이즈가 1로 늘어난 것을 확인할 수 있다.

![](https://velog.velcdn.com/images/sinryuji/post/db8fea43-c632-47d6-ae13-8364c8b954bd/image.png)

이후 두 번째 영속화가 된 후 확인을 해보면 사이즈가 2로 늘어난 것을 확인할 수 있다.

![](https://velog.velcdn.com/images/sinryuji/post/6aa2f4c8-50c0-4280-b3b7-b084ac54e3f6/image.png)

commit이 된 후 시점이다. commit이 되면서 ActionQueue에 저장된 쿼리들을 모두 순차적으로 실행을 시켰을테니 큐가 모두 비워진 것을 확인할 수 있다.

### flush()

지금까지 계속 JPA 트랜잭션이 commit된 시점에 DB에 쿼리가 요청된다고 설명을 하였는데 이는 이해를 쉽게 하기 위해 조금 생략된 설명이고, 더 정확히 설명을 하자면 **commit이 되면 EntityManager가 내부적으로 `flush()`란 함수를 호출하며, 이를 통해 쿼리가 쿼리가 요청이 되는 것**이다.

즉, 우리는 commit을 하지 않아도, 그러니까 트랜잭션을 마치지 않아도 중간에 `flush()`를 호출하여 변경 사항을 DB에 적용을 시킬 수 있는 것이다. 물론 실제 개발 환경에서는 거의 사용되지 않고 테스트 환경에서만 많이 사용되는 방법이지만 분명히 알아둬야하는 부분이다.

다음 코드를 보자.

```java
@Test
@DisplayName("flush() 메서드 확인")
void test7() {
    EntityTransaction et = em.getTransaction();

    et.begin();

    try {
        Memo memo = new Memo();
        memo.setId(4L);
        memo.setUsername("Flush");
        memo.setContents("Flush() 메서드 호출");
        em.persist(memo);

        System.out.println("flush() 전");
        em.flush(); // flush() 직접 호출
        System.out.println("flush() 후\n");
        

        System.out.println("트랜잭션 commit 전");
        et.commit();
        System.out.println("트랜잭션 commit 후");

    } catch (Exception ex) {
        ex.printStackTrace();
        et.rollback();
    } finally {
        em.close();
    }

    emf.close();
}
```

지금까지 우리가 배운대로라면 `em.persist(memo)`가 호출되는 시점에서 ActionQueue에 insert 쿼리가 하나 쌓일 것이고, 원래라면 commit이 되는 시점에 이 쿼리가 요청이 되어야 할 것이다. 그런데 중간에 `em.flush()`를 호출하여 commit이 되기전에 실제로 쿼리가 요청이 되는지를 확인해보자.

![](https://velog.velcdn.com/images/sinryuji/post/cff42c8f-43ab-436d-98a8-6ab0289dae78/image.png)

원래라면 트랜잭션 commit 전과 후 사이에 쿼리가 날아가야 하는데 그 전에 쿼리가 날아갔다. `flush()`가 호출되는 시점에 ActionQueue를 모두 비우며 쿼리를 요청하는 것을 확인할 수 있다.

## 변경 감지(Dirty Checking)

![](https://velog.velcdn.com/images/sinryuji/post/c6f81506-e4a1-4272-b163-44ac57923bb3/image.png)

DB의 값을 수정을 하려면 update 쿼리를 요청해야 한다. 그런데 만약에 **영속성 컨텍스트에 저장된 Entity의 값이 변할 때 마다 ActionQueue에 update 쿼리가 쌓인다면 불필요한 쿼리들이 발생하게 되므로 이럴 필요 없이 오직 최종 변경 사항만을 반영하여 update 쿼리를 요청**하면 된다.

JPA는 이를 구현하기 위해 **영속성 컨텍스트에 `LoadedState`로 초기 상태를 저장을 해놓고, `flush()`가 호출되어 최종적으로 DB에 쿼리가 날아가기전에 실제 저장된 Entity와 이 초기 상태를 비교하여 변경점이 있다면 update 쿼리를 요청**하게 된다.

이러한 과정을 변경 감지, Dirty Checking이라고 부른다. 다음 코드를 보자.

```java
@Test
@DisplayName("변경 감지 확인")
void test8() {
    EntityTransaction et = em.getTransaction();

    et.begin();

    try {
        System.out.println("변경할 데이터를 조회합니다.");
        Memo memo = em.find(Memo.class, 4);
        System.out.println("memo.getId() = " + memo.getId());
        System.out.println("memo.getUsername() = " + memo.getUsername());
        System.out.println("memo.getContents() = " + memo.getContents());

        System.out.println("\n수정을 진행합니다.");
        memo.setUsername("Update");
        memo.setContents("변경 감지 확인");

        System.out.println("트랜잭션 commit 전");
        et.commit();
        System.out.println("트랜잭션 commit 후");

    } catch (Exception ex) {
        ex.printStackTrace();
        et.rollback();
    } finally {
        em.close();
    }

    emf.close();
}
```

우리는 처음 `em.find(Memo.class, 4)`가 호출되었을때 Entity와 LoadedState를 비교해보고, Entity 객체의 값을 수정한 후 다시 비교를 해볼 것이다.

![](https://velog.velcdn.com/images/sinryuji/post/83c57927-55a2-443f-bdd5-59e1535059dd/image.png)

em -> persistenceContext -> entityEntryContext -> nonEnhancedEntityXref를 보면 `em.find(Memo.class, 4)`로 찾아온 Entity를 볼 수 있는데 entityInstance와 loadedState의 값이 같은 것을 확인할 수 있다.

그런데 값을 수정한 후에 다시 확인을 해보면,

![](https://velog.velcdn.com/images/sinryuji/post/7659725e-5b33-469d-bc9a-a1706f4babd9/image.png)

loadedState의 값을 그대로지만 entityInstance의 값이 바뀐 것을 확인할 수 있다. 이후 commit이 되면서 `flush()`가 호출되면 최종적으로 다음과 같이 update 쿼리가 날아가게 된다.

![](https://velog.velcdn.com/images/sinryuji/post/f34f21e6-ab7a-4f46-beb4-fdac5b77f6c6/image.png)

이로써 우리는 setter를 통해 Entity의 값을 수정할 때마다 update 쿼리를 날리는게 아니라, 단 한 번만의 update 쿼리로 DB의 값을 변경할 수 있다.

# Entity의 상태

![](https://velog.velcdn.com/images/sinryuji/post/57a036a3-b0d6-4673-b1e2-326e4ccefa90/image.png)


영속성 컨텍스트에서 Entity는 다음과 같이 총 4가지 상태로 존재할 수 있다.

- 비영속(Transient)
- 영속(Managed)
- 준영속(Detached)
- 삭제(Removed)

하나씩 알아보자.

## 비영속(Transient)
비영속이라는 뜻 그대로 영속성 컨텍스트에 영속되어 있지 않은 상태를 의미한다. 다음 코드를 보자.

```java
Memo memo = new Memo(); // 비영속 상태
memo.setId(1L);
memo.setUsername("Robbie");
memo.setContents("비영속과 영속 상태");
```

new 연산자를 통해 객체를 하나 생성했다. 이는 그냥 Heap 메모리에 존재하는 일반적인 Java 객체이다. 이와 같이 **영속성 컨텍스트에 저장된 적이 없는 객체들을 비영속 상태**라 칭한다.

## 영속(Managed)

```java
em.persist(memo);
```

우리는 앞서 계속해서 영속성 컨텍스트에 Entity를 넣기 위해 `persist()`를 호출해왔다. 이렇게 **Entity가 영속화가 되어 영속성 컨테스트 내부에서 관리가 되기 시작한 상태를 영속 상태**라 칭한다.

```java
@Test
@DisplayName("비영속과 영속 상태")
void test1() {
    EntityTransaction et = em.getTransaction();

    et.begin();

    try {

        Memo memo = new Memo(); // 비영속 상태
        memo.setId(1L);
        memo.setUsername("Robbie");
        memo.setContents("비영속과 영속 상태");

        em.persist(memo);

        et.commit();

    } catch (Exception ex) {
        ex.printStackTrace();
        et.rollback();
    } finally {
        em.close();
    }

    emf.close();
}
```

위 코드에서 `em.persist(memo)`가 호출되기 전 시점을 보자.

![](https://velog.velcdn.com/images/sinryuji/post/45b78fa9-7354-4cda-bc85-ca0a930b0567/image.png)

영속성 컨텍스트에서 아무런 Entity도 관리되고 있지 않은 상태이다. 즉, **memo 객체는 아직 비영속 상태**인 것이다. 그런데 `em.persist(memo)`가 호출된 후를 보자.

![](https://velog.velcdn.com/images/sinryuji/post/5e24ac16-0c3d-464b-b22c-4bbcebce1f8b/image.png)

영속성 컨텍스트에 Memo Entity 객체가 하나 추가되었고 맨아래에 MANAGED 상태인 것을 볼 수 있다. **`persist()`를 통해 영속화를 함으로써 영속성 컨텍스트에 들어와 영속 상태가 된 것**이다.


## 준영속(Detached)

영속성 컨텍스트 내부에서 영속 상태로 관리가 되다가, **영속성 컨텍스트로부터 분리되어 더 이상 관리를 받지 않는 상태**이다. 1차 캐시, 쓰기 지연, 변경 감지 등 영속성 컨텍스트의 어떠한 기능도 동작하지 않는다. 주의할 점은 **영속성 컨텍스트 내에서만 제거가 되었을 뿐, 실제 DB에서는 삭제가 되지 않았다**라는 것이다.

준영속 상태로 만드는 방법은 총 3가지가 존재한다.

### detach()

특정 Entity만 준영속 상태로 만든다.

```java
@Test
@DisplayName("준영속 상태 : detach()")
void test2() {
    EntityTransaction et = em.getTransaction();

    et.begin();

    try {

        Memo memo = em.find(Memo.class, 1);
        System.out.println("memo.getId() = " + memo.getId());
        System.out.println("memo.getUsername() = " + memo.getUsername());
        System.out.println("memo.getContents() = " + memo.getContents());

        // em.contains(entity) : Entity 객체가 현재 영속성 컨텍스트에 저장되어 관리되는 상태인지 확인하는 메서드
        System.out.println("em.contains(memo) = " + em.contains(memo));

        System.out.println("detach() 호출");
        em.detach(memo);
        System.out.println("em.contains(memo) = " + em.contains(memo));

        System.out.println("memo Entity 객체 수정 시도");
        memo.setUsername("Update");
        memo.setContents("memo Entity Update");

        System.out.println("트랜잭션 commit 전");
        et.commit();
        System.out.println("트랜잭션 commit 후");

    } catch (Exception ex) {
        ex.printStackTrace();
        et.rollback();
    } finally {
        em.close();
    }

    emf.close();
}
```

우선 `detach()`를 호출하기 전에 Entity 객체의 상태를 보자.

![](https://velog.velcdn.com/images/sinryuji/post/5aa999b9-d2c0-45da-a991-926544ee3e44/image.png)

`em.find(Memo.class, 1)`를 통해 MANAGED 상태로 잘 영속되어 있는 것을 확인할 수 있다. 그 후 `em.detach(memo)`를 호출하면?

![](https://velog.velcdn.com/images/sinryuji/post/2cac116f-5acb-4930-9c5c-652e681649ba/image.png)

영속 상태였던 Entity가 사라진 것을 확인할 수 있다. 이렇듯 `detach()`를 사용하면 특정 Entity만 지정하여 준영속 상태로 만들 수 있다.

![](https://velog.velcdn.com/images/sinryuji/post/a331c449-ae64-4a20-9309-e4acfe911f8d/image.png)

준영속 상태가 되었기 때문에 객체의 값을 변경하여도 변경 감지(Dirty Checking) 또한 동작하지 않는 것을 확인 할 수 있다. `contains()`는 현재 영속성 컨텍스트에 저장되어 관리되고 있는 상태인지를 체크하는 함수인데 해당 함수또한 `detach()` 호출 후에는 false를 반환하는 것을 확인 할 수 있다.

### clear(), close()

```java
em.clear();
em.close();
```

둘 다 영속성 컨텍스트의 모든 Entity를 준영속 상태로 전환하는 함수이다. 차이점은 **`clear()`는 영속성 컨텍스트 자체는 유지한채 내부 캐시만 비우는 함수**로 인스턴스화한 영속성 컨텍스트는 계속 사용이 가능하다. 그런데 **`close()`는 아예 영속성 컨텍스트를 종료하는 함수**로 해당 영속성 컨텍스트를 계속 사용 할 수 없다.

이렇게 총 3가지 함수가 존재하며 영속 -> 준영속 뿐만 아니라 준영속 -> 영속 또한 가능하다. 이에 사용되는 함수가 `merge()`이다.

### merge(entity)

```java
em.merge(memo);
```

전달 받은 Entity를 사용하여 새로운 영속 상태의 Entity를 반환한다. `merge()`는 파라미터로 전달 된 Entity의 식별자 값으로 영속성 컨텍스를 조회하는데 이 때 다음 두 가지 케이스로 동작한다.

- 해당 Entity가 영속성 컨텍스트에 없는 경우
  - DB에서 새롭게 조회를 해온다.
  - 조회한 Entity를 영속성 컨텍스트에 저장한다.
  - 전달 받은 Entity의 값을 사용하여 병합한다.
  - update 쿼리가 수행된다.
- DB에도 없는 경우
  - 전달 받은 Entity를 영속성 컨텍스트에 저장한다.
  - insert 쿼리가 수행된다.
  
즉, `merge()`는 준영속, 비영속 상태 모두 파라미터로 받을 수 있으며 이에 따라 수정, 혹은 저장의 동작을 수행한다.

```java
@Test
@DisplayName("merge() : 저장")
void test5() {
    EntityTransaction et = em.getTransaction();

    et.begin();

    try {

        Memo memo = new Memo();
        memo.setId(3L);
        memo.setUsername("merge()");
        memo.setContents("merge() 저장");

        System.out.println("merge() 호출");
        Memo mergedMemo = em.merge(memo);

        System.out.println("em.contains(memo) = " + em.contains(memo));
        System.out.println("em.contains(mergedMemo) = " + em.contains(mergedMemo));

        System.out.println("트랜잭션 commit 전");
        et.commit();
        System.out.println("트랜잭션 commit 후");

    } catch (Exception ex) {
        ex.printStackTrace();
        et.rollback();
    } finally {
        em.close();
    }

    emf.close();
}
```

`merge()`에게 비영속 상태의 Entity를 전달하는 코드이다. 위 코드의 출력은 다음과 같다.

![](https://velog.velcdn.com/images/sinryuji/post/229775c7-326c-4bde-9fe2-09ce6e9d79f1/image.png)


비영속 상태의 Entity를 전달했기에 당연히 영속성 컨텍스트에는 존재하지 않으니 먼저 select 쿼리가 수행된다. 그 후 `contains()` 호출 결과를 보면, 파라미터로 전달됐던 Entity는 false를 출력하고 `merge()`가 반환한 Entity는 true를 출력한다. **영속 상태인 완전히 새로운 Entity를 생성한 것**이다. 그 후 DB에 존재하지 않으니 inert 쿼리가 수행된다.

```java
@Test
@DisplayName("merge() : 수정")
void test6() {
    EntityTransaction et = em.getTransaction();

    et.begin();

    try {

        Memo memo = em.find(Memo.class, 3);
        System.out.println("memo.getId() = " + memo.getId());
        System.out.println("memo.getUsername() = " + memo.getUsername());
        System.out.println("memo.getContents() = " + memo.getContents());

        System.out.println("em.contains(memo) = " + em.contains(memo));

        System.out.println("detach() 호출");
        em.detach(memo); // 준영속 상태로 전환
        System.out.println("em.contains(memo) = " + em.contains(memo));

        System.out.println("준영속 memo 값 수정");
        memo.setContents("merge() 수정");

        System.out.println("\n merge() 호출");
        Memo mergedMemo = em.merge(memo);
        System.out.println("mergedMemo.getContents() = " + mergedMemo.getContents());

        System.out.println("em.contains(memo) = " + em.contains(memo));
        System.out.println("em.contains(mergedMemo) = " + em.contains(mergedMemo));

        System.out.println("트랜잭션 commit 전");
        et.commit();
        System.out.println("트랜잭션 commit 후");

    } catch (Exception ex) {
        ex.printStackTrace();
        et.rollback();
    } finally {
        em.close();
    }

    emf.close();
}
```

이번엔 준영속 상태의 Entity를 파라미터로 넘겨보자. 앞서 저장한 Id 3의 객체를 조회해와서 영속성 컨텍스트에 저장한 후 `detach()`를 통해 준영속 상태로 만든다. 그 후 `merge()`를 호출하면 영속성 컨텍스트에 존재하지 않으니 다시 한 번 select 쿼리가 수행 될 것이고 **준영속 상태, 즉 DB에는 저장이 된 상태이니 이번에는 update 쿼리가 수행 될 것**이다. 실제로 그런지 확인해보자.
 
 ![](https://velog.velcdn.com/images/sinryuji/post/bccce7ea-9569-4c0e-9af9-656d70bb9f22/image.png)

예상대로 두 번의 select 쿼리 후 update 쿼리가 수행되는 것을 확인할 수 있다. 중간에 `contains()`를 통해 영속 상태를 확인하는 부분도 살펴보면 좋을 것 같다.

## 삭제(Removed)

마지막으로 삭제 상태이다. 앞서 `em.remove(memo)`를 통해 Entity를 다음과 같이 DELETED 상태로 만든 적이 있다. 이는 아직 `flush()`가 호출 되지 않아 delete 쿼리가 수행되지는 않았지만, 곧 삭제될 상태의 Entity이다.

![](https://velog.velcdn.com/images/sinryuji/post/e20af748-6e51-489b-b196-4c7e1660f097/image.png)

# 마무리

사실 대부분의 개발자들이 Java로 백엔드 개발을 한다면 Spring을 사용하고 있고, 그 환경에서 JPA를 사용한다면 JPA를 추상화한 Spring Data JPA를 사용할 것이고, 그렇다면 영속성 컨텍스트고 나발이고 EntityManger의 E자도 볼 일이 없다.

그렇지만 우리가 **밥먹듯이 `@Transaction`을 붙인 서비스 함수들이 왜 조회를 해온 후 JpaRepository의 `save()`를 호출 하지 않아도 객체의 변경 사항이 저장되는 지를 이해하려면 반드시 영속성 컨텍스트를 알아야한다.** 그리고 프레임워크나 외부 라이브러리들을 사용하며 '알아서 다 해 주네 개꿀'이라는 마인드보다, 이게 왜 이렇게 동작하지?라고 파고 들어가며 그 근본 동작 원리에 대해 공부를 해나가는 것이 실력이 상승하는 길이라고 생각한다!

나도 영속성 컨텍스트에 대해 알고는 있었지만 뭔가 확실하게 정리를 하고 있지는 않았는데, 현재 수강하고 있는 부트 캠프 강의와 정리 자료가 너무 잘 정리되어 있는 것 같아서 이를 인용하여 글을 작성해 보았다. 항상 느끼는 거지만 블로그로 이렇게 장문의 글을 작성하는 게 생각보다 많은 시간을 투자해야 한다. 하지만 하고 나면 그만큼 뿌듯하고 머릿속에 확실히 각인이 된다. 앞으로도 정리가 필요한 내용들은 꼭 블로그에 글로 기록을 해야겠다!