```toc
```

# 스프링 배치(Spring Batch)란?

`Spring Batch`는 대용량 일괄처리를 위한 배치 프레임워크이다. Spring의 특성을 그대로 가져왔기 때문에 DI, AOP, 서비스 추상화 등 Spring 프레임워크의 3대 요소를 모두 사용할 수 있다.

보통 다음과 같은 경우 많이 사용한다.
- 대용량 비즈니스 데이터를 복잡한 작업으로 처리해야 하는 경우
- 특정한 시점에 스케쥴러를 통해 자동화된 작업이 필요한 경우
	- ex) 푸시알림, 월 별 리포트
- 대용량 데이터의 포맷을 변경, 유효성 검사 등의 작업을 트랜잭션 안에서 처리 후 기록해야 하는 경우

배치 어플리케이션은 다음의 조건을 만족해야만 한다.
- 대용량 데이터: 대량의 데이터를 가져오거나, 전달하거나, 계산하는 등의 처리를 할 수 있어야 한다.
- 자동화: 심각한 문제 해결을 제외하고는 사용자 개입 없이 실행되어야 한다.
- 견고성: 잘못된 데이터를 충돌/중단 없이 처리할 수 있어야 한다.
- 신뢰성: 무엇이 잘못되었는지를 추적할 수 있어야 한다. (로깅, 알림)
- 성능: 지정한 시간안에 처리를 완료하거나 동시에 실행되는 다른 어플리케이션을 방해하지 않도록 수행되어야 한다.

```text
간혹 Spring Batch와 Spring Quartz를 비교하곤 하는데 Quartz는 스케줄러의 역할이지, Batch와 같이 대용량 데이터 배치 처리에 대한 기능을 지원하지 않는다. 반대로 Batch 역시 Quartz와 같은 다양한 스케줄 기능을 지원하지 않아 보통 Batch + Quartz를 조합해 사용한다.
```

# 프로세스

1. Job Launcher로 Job을 실행 (Job Launcher -> Job)
2. Job은 Step을 통해 실제 배치처리를 수행 (Job -> Step)
3. Step에서는 읽어오고(Item Reader) -> 처리하고(Item Processor) -> 저장(Item Writer)을 수행 (Step -> Item Reader, Processor, Writer)

![](https://i.imgur.com/xbJg0D1.png)

## Job

> [!question] Job이란?
>  - 하나 이상의 Step으로 구성이 되며 `배치 처리의 최상위 단위`를 의미한다.
>  - 실행 시점에 파라미터를 전달 받을 수 있으며 실행 결과를 반환할 수 있다.

![](https://i.imgur.com/UcLYGTc.png)

- `Job` : 배치 처리의 단위 작업을 의미
- `JobInstance` : 하나의 Job 실행을 나타내는 인스턴스를 의미
- `JobParameters` : Job 실행 시 필요한 파라미터를 의미
- `JobRepository` : Job 실행 정보를 저장하고 관리하는 저장소를 의미
- `JobLauncher` : Job을 실행하는 인터페이스를 의미
- `JobExecution` : Job 실행 정보를 나타내는 인스턴스를 의미
- `JobExecutionListener` : Job 실행 전후로 수행할 작업을 정의하는 인터페이스를 의미
## Step

> [!question] Step이란?
> - 실제로 배치 처리를 '수행'하는 단위를 의미한다. Spring Batch Job 안에는 한 개 이상의 Step으로 구성되어 있다.
> - Step에서는 하나의 작업만 처리를 수행하는 'Tasklet' 방식 또는 Reader-Processor-Writer 묶음으로 여러 작업을 처리를 하는 'Chunk' 방식이 있다.

![](https://i.imgur.com/jNkt8Ku.png)

- `Step` : Job을 구성하는 작은 단위 작업을 의미한다.
- `ItemReader` : 데이터를 읽어오는 인터페이스를 의미한다.
- `ItemProcessor` : 읽어온 데이터를 처리하는 인터페이스를 의미한다.
- `ItemWriter` : 처리한 데이터를 출력하는 인터페이스를 의미한다.
- `ExecutionContext` : Step 실행 중 필요한 컨테스트 정보를 의미한다.
- `StepExecution` : Step 실행 정보를 나타내는 인스턴스를 의미한다.
- `StepExecutionListener` : Step 실행 전후로 수행할 작업을 정의하는 인터페이스를 의미한다.

# Spring Boot Batch의 종류

## 1. Tasklet 방식

> [!question] Tasklet을 이용한 Task 기반 처리 방식
> 
> - Batch의 Step 단계에서 '단일한 레코드(row)'나 '파일'을 하나의 작업만 처리하는 방식을 방식을 의미한다. 해당 모델에서의 처리는 각각의 처리를 하나의 트랜잭션에서 처리한다.
> - 일반적으로 파일을 읽고 처리한 다음 결과를 데이터베이스에 쓰는 등의 작업을 수행한다.
> - 단일 작업을 처리하기 때문에 작업이 끝날 때 까지 대기해야 한다. 그렇기에 대용량 데이터 처리에 적합하지 않다.

## 2. Chunk 방식

> [!question] Chunk를 이용한 Chunk 기반 처리 방식
> 
> - Batch의 Step 단계에서 ‘단일한 레코드(row)를 묶어서’ 여러 작업을 처리하는 방식을 의미한다. 해당 방식에서는 ‘묶인 레코드를 하나의 트랜잭션으로 처리’하며 실패를 하는 경우 롤백을 수행한다.
> - 해당 방식은 병렬 처리를 위해 Chunk를 사용하되 ‘순차적으로 처리하는 방식’이며 Parallel Chunk 방식은 Chunk를 독립적으로 처리하여 병렬 처리를 수행한다.
> - 대용량 데이터를 처리할 때, 성능이 향상되고 중복 레코드 처리나 실패한 레코드 처리 등 예외 상황에 대한 대처가 용이하다.  
> - 예를 들어, 파일을 읽어들여 데이터를 처리하는 작업이나 DB에서 데이터를 조회하여 처리하는 작업 등이 있다.

- `Chunk`
	- 데이터를 일정한 크기로 나눈 데이터 셋을 의미
	- Chunk 단위로 나누면 전체 데이터를 한 번에 처리하지 않기에 메모리 부하를 줄이고 성능을 향상할 수 있다.
- `Chunk` 기반 STEP의 Reader-Processor-Writer 처리 방식
	1. 데이터 베이스에서 데이터를 읽어온다,(Item Reader)
	2. 읽어온 데이터를 처리한다.(Item Processor)
	3. 데이터를 저장한다.(Item Writer)

![](https://i.imgur.com/nQBl2Mj.png)
![](https://i.imgur.com/znbRU2Q.png)


|**구분**|**Tasklet**|**Chunk**|
|---|---|---|
|**실행 시점**|STEP 실행 중|STEP 실행 전|
|**실행 방식**|‘작은 단위’로 분할하여 실행|‘하나의 큰 덩어리’로 실행|
|**커밋 방식**|‘Tasklet’ 단위로 처리 후 커밋|‘Chunk’ 단위로 처리 후 커밋|
|**배치 성능**|‘작은 단위’의 데이터 처리 시 유리|‘대량의 데이터’ 처리 시 유리|
|**재시작**|실패한 태스크릿만 다시 실행|실패 시 청크 전체 다시 실행|
|**가독성**|코드가 분할되어 가독성 향상|코드가 길어져 가독성 저하|
|**유지보수**|수정이 용이|수정이 어려움|

## 3. Parallel Chunk

> [!question] Parallel Chunk란?
> - Chunk 방식의 처리에서 더욱 빠른 처리 속도를 위해 Chunk를 독립적으로 처리하여 여러 개의 'Chunk를 병렬로 처리'한다. 병렬 처리를 통해 처리 속도를 높일 수 있다는 장점이 있다.

- `Parallel Chunk` 수행 과정
	1. 데이터를 적절한 크기로 분할한다.
	2. 분할된 각각의 부분을 병렬 처리할 수 있도록 작업을 분배한다.
	3. 각각의 부분을 병렬 처리한다.
	4. 처리된 결과를 다시 합친다.


# 출처
https://adjh54.tistory.com/169