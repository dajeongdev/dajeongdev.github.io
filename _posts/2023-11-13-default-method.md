---
layout: post
title: default method
subtitle: 
categories: wiki
tags: [Java,Java8]
---

# default method

## 기존의 인터페이스
기존의 인터페이스는 추상 메서드만 존재할 수 있고, 이를 상속 받는 구현체에서 직접 해당 추상 메소드를 구현해야 한다. 
이는 인터페이스에 메서드가 추가되면 그 인터페이스를 상속 받는 모든 구현체 클래스에 추가 메서드를 구현해주어야 하는 문제가 발생한다.
이 경우 **확장(open)은 할 수 있지만, 변경에 대한 폐쇄(close)는 위반했다**고 할 수 있다. (객체 지향의 원칙 중 OCP) 
**새로운 메소드가 인터페이스에 추가된다는 이유로 새로운 메소드를 사용하지 않는 구현체에도 추가적으로 구현을 해야 하기 때문**이다. 즉, 기존의 코드가 변경되어버린다.
<br>

## default method
JAVA 8부터 추가된 default method를 통해 인터페이스에서도 메소드를 구현할 수 있게 되었다.
인터페이스에 default 메소드를 추가해도 구현체에 추가적인 변경을 하지 않기 때문에 **기존 구현체의 코드 변경 없이 확장할 수 있게 된 것** 이다.
<br>

```java
public interface TestInterface {
		void testAbstractMethod1();
		void testAbstractMethod2();

		default String defaultMethod1() {
				// 구현 코드
		}
}
```
<br>

### 추가
자바 8부터는 이전 버전과는 다르게 추상 클래스를 그렇게 많이 쓰지 않고, 추상 클래스가 하던 일을 많은 부분 인터페이스가 하게 되었기 때문에 정말 필요한 경우가 아니라면 인터페이스에 있는 default 메소드를 사용해서 시도해보는 것이 좋다.
<br>
<br>

### 참고
- [https://velog.io/@heoseungyeon/디폴트-메서드Default-Method](https://velog.io/@heoseungyeon%EB%94%94%ED%8F%B4%ED%8A%B8-%EB%A9%94%EC%84%9C%EB%93%9CDefault-Method)
- [https://www.inflearn.com/course/디자인-패턴/dashboard](https://www.inflearn.com/course/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4/dashboard)
```
