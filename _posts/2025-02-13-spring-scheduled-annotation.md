---
layout: post
title: Spring의 @Scheduled로 간단하게 원하는 시간에 스케줄을 걸어보자
categories: wiki
subtitle: 
tags: [Spring, Scheduled, TaskScheduler, Annotation]
---
오늘은 실무에서 자주 사용하는 `@Scheduled`에 대해서 정리해두려고 한다. 이번 프로젝트에서도 어김없이 `@Scheduled`를 사용하게 되었는데, 마침 이번에 알게 된 부분까지 추가해서 `@Scheduled`를 사용하는 방법과 `@Scheduled`에서 원하는 시간에 스케줄러를 돌릴 때 사용할 수 있는 Cron식까지 알아보자.
<br/>
<br/>

## @Scheduled
- `@Scheduled`는 Spring에서 제공하는 스케줄링 어노테이션으로, 특정 작업을 일정한 주기로 자동 실행할 수 있도록 도와준다.
- 알림 전송, 배치 작업, 데이터 백업 등의 주기적인 작업을 수행해야 할 때 사용할 수 있다.
<br/>

### @Scheduled 설정
```java
@EnableScheduling
@SpringBootApplication
public class SchedulerApplication { 
	public static void main(String[] args) {        
		SpringApplication.run(DemoApplication.class, args);    
	} 
}

```
- 먼저 Application 클래스 main() 메소드에 `@EnableScheduling` 어노테이션을 설정해준다. (Config 클래스를 만들어서 지정하는 것도 가능하다.)
<br/>

### @Scheduled 사용법
```java
@Component
public class SchedulerService {
	@Scheduled(fixedDelay = 1000) // 1초마다 실행
	public void run() {
		System.out.println("Hello Cron!");
	}
}
```
- 그러면 이제 메소드에는 `@Scheduled` 어노테이션을 붙여 바로 사용할 수 있는데, 주의해야 할 점은 `@Scheduled`를 사용할 클래스가 스프링 빈에 등록된 클래스여야 한다는 것이다.
    - `@Component`, `@Service` 등
<br/>

### @Scheduled 메소드 규칙
- `@Scheduled`를 사용하는 메소드는 아래 두 규칙을 지켜야 한다.
    1. void 리턴 타입
    2. 매개변수 사용 불가
<br/>
<br/>


## @Scheduled 속성

### fixedDelay
- 이전 Task의 종료 시점부터 정의된 시간만큼 지난 후 Task 실행 (단위: milliseconds)
- (작업 수행 시간을 포함하여) **작업을 마친 후부터** 일정 주기마다 메소드 호출

```java
@Scheduled(fixedDelay = 5000) // 5초마다 실행
public void fixedDelaySchedule() {
    log.info("FixedDelay Task = {}", System.currentTimeMillis());
}
```
```java
0초 -> 실행 (3초 걸림, 5초 대기)
8초 -> 실행 (3초 걸림, 5초 대기)
13초 -> 실행
```

### fixedRate
- 이전 Task의 시작 시점으로부터 정의된 시간만큼 지난 후 Task 실행 (단위: milliseconds)
- **작업 수행시간과 상관없이** 일정 주기마다 메소드 호출

```java
@Scheduled(fixedRate = 5000) // 5초마다 실행
public void fixedRateSchedule() {
    log.info("FixedRate Task = {}", System.currentTimeMillis());
}
```
```java
0초 -> 실행 (3초 걸림)
5초 -> 실행 (3초 걸림)
10초 -> 실행
```

### initialDelay
- 애플리케이션 시작 후 초기 지연시간 설정
- 서버가 시작되자마자 실행되는 것을 방지

```java
@Scheduled(fixedRate = 5000, initialDelay = 10000) // 5초마다 실행, 초기 10초 대기
public void initialDelaySchedule() {
    log.info("InitialDelay Task = {}", System.currentTimeMillis());
}
```
```java
앱 시작 -> 10초 대기
10초 -> 실행
15초 -> 실행
20초 -> 실행
```
<br/>
<br/>


## Cron
- 일정한 주기가 아니라 특정 시간에 작업을 실행하고 싶다면 cron 속성에 Cron식을 사용하면 된다.

```java
@Scheduled(cron = "0 0 10 * * *") // 매일 오전 10시 실행
public void cronSchedule() {
	log.info("Cron Task = {}", LocalDateTime.now());
}
```

### 순서
1.초(0-59)
2.분(0-59)
3.시간(0-23)
4.일(1-31)
5.월(1-12)
6.요일(0-6) (0:일, … 6: 토)

### 표현

|표현|설명|예시|
|---|---|---|
|*|모든 조건 (매시, 매일, 매주처럼 사용)|0 0 18 * * * -> 매일 18시|
|?|설정값 없음 (날짜, 요일에서만 사용 가능)|0 0 14 10,20 * ? -> 매달 10일, 20일 14시|
|-|범위 지정|0 0/5 9-18 * * * -> 매일 9시0분에서18시55분 사이에 5분 간격|
|,|여러 값 지정|0 0 14 10,20 * ? -> 매달 10일, 20일 14시|
|/|증분값, 즉 초기값과 증가치 설정에 사용|0 0 0/1 * * * -> 1시간마다|
|L|마지막 - 지정할 수 있는 범위의 마지막 값 설정 시 사용 (날짜, 요일에서만 사용 가능)|0 30 10 ? * 6L -> 매달 마지막 토요일 10시 30분마다|
|W|가장 가까운 평일(weekday) 설정|10W -> 10일이 평일이면 10일, 주말이면 가까운 평일에 실행|
|#|N번째 주 특정 요일 설정 (요일에서만 사용 가능)|4#2 → 목요일#둘째주에 실행|

**`*`와 `?`의 차이**
- `?`는 특정 값을 지정하지 않음을 의미하는데, `*`와 헷갈릴 수 있다.
- 예를 들어 `“0 0 12 * * ?”`는 매일 12시에 실행하지만, `“0 0 12 * * *”`는 매일 12시에 실행하면서 요일도 모든 값에 해당된다는 차이점이 있다.
<br/>
  
### zone
- cron 표현식에서 사용할 time zone (default: Local time zone)
- 서버의 기본 시간대에 의존하면 배포 환경마다 실행 시간이 다를 수 있으므로 zone으로 명확하게 지정해주는 것이 좋다.
```java
@Scheduled(cron = “0 0 12 * * ?”, zone = "Asia/Seoul")
public void seoulTimeZoneTask() {
	log.info("한국 시간 기준 12시 실행");
}    
```
<br/>
<br/>


## Trouble Shooting 💫

### 같은 시간에 두 개의 스케줄러를 설정했는데 왜 하나만 실행될까❓🤔
- `@Scheduled`는 기본적으로 **싱글 스레드**에서 실행되므로, 여러 개의 스케줄러를 동시에 실행하려면 application.yml이나 Config 클래스를 사용하여 스레드 풀을 2개 이상으로 설정해야 한다.

```java
  task:
    scheduling:
      pool:
        size: 5
```
```java
@Configuration
public class TaskSchedulerConfig {
    @Bean
    public TaskScheduler taskScheduler() {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.setPoolSize(5);
        scheduler.initialize();
        return scheduler;
    }
}
```
<br/>

### 시간마다 말고 한 번만 스케줄러를 실행시킬 수는 없나❓🤔
- 한 번만 실행되는 스케줄링 작업은 여러 가지가 있지만, 스프링을 사용하고 있다면 TaskScheduler 인터페이스를 구현한 ThreadPoolTaskScheduler를 사용해볼 수 있다.
- `@Scheduled`처럼 설정만 해주면 멀티 스레드가 가능하고, 대량의 작업을 처리할 수 있다.

```java
@Service
public class OneTimeTaskSchedulerService {

    private final ThreadPoolTaskScheduler taskScheduler;

    public OneTimeTaskSchedulerService() {
        this.taskScheduler = new ThreadPoolTaskScheduler();
        this.taskScheduler.initialize(); // 초기화 필수
    }

    public void scheduleOneTimeTask() {
        taskScheduler.schedule(() -> {
            System.out.println("10분 후 실행되는 작업 실행됨! " + System.currentTimeMillis());
            // 실행할 로직 추가
        }, Instant.now().plusSeconds(600)); // 현재 시간 기준 600초(10분) 후 실행
    }
}
```

- 위에서처럼 Config 클래스로 `@Bean` 등록을 해준다면 생성자에서 초기화할 필요 없어 코드도 간결하고, 여러 곳에서 사용할 수 있어 더 편리하다.

```java
@RequiredArgsConstructor
@Service
public class OneTimeTaskSchedulerService {

    private final TaskScheduler taskScheduler;

    public void scheduleOneTimeTask() {
        taskScheduler.schedule(() -> {
            System.out.println("예약된 작업이 실행되었습니다! " + System.currentTimeMillis());
            // 실행할 로직 추가
        }, Instant.now().plusSeconds(600)); // 현재 시간 기준 600초(10분) 후 실행
    }
}
```
<br/>
<br/>


**참고**
- [Spring 공식 가이드](https://spring.io/guides/gs/scheduling-tasks)
- [https://dev-coco.tistory.com/176](https://dev-coco.tistory.com/176)
- [https://data-make.tistory.com/699](https://data-make.tistory.com/699)
- [https://chatgpt.com/c/67ac8d2b-83d0-8003-98ec-3ffc372b8437](https://chatgpt.com/c/67ac8d2b-83d0-8003-98ec-3ffc372b8437)