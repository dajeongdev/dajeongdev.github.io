---
layout: post
title: 배열 오름차순/내림차순 정렬
subtitle: 
categories: wiki
tags: [Java, Arrays, Sort]
---
### 배열 오름차순/내림차순 정렬
#### 배경
코딩테스트 문제로 자주 등장하는 배열의 오름차순/내림차순 정렬에 대해 정리해두겠습니다.
<br/>
<br/>


#### <1> 오름차순 정렬
<img src="https://dajeongdev.github.io/assets/images/posts/arrays-sort.png">
- 오름차순 정렬은 아주 간단하다. Arrays 클래스의 sort() 메소드를 사용하면 된다.

``` java
int[] A = {4, 3, 3};
Arrays.sort(A); // 결과: A = {3, 3, 4}
```
<br/>


#### <2> 내림차순 정렬
<img src="https://dajeongdev.github.io/assets/images/posts/arrays-sort-collections.png">
- 내림차순 정렬도 어렵진 않지만, 한 가지 주의할 점은 Comparator<?>로 받는 T가 제네릭 클래스로 받는 객체이기 때문에 int 같은 Primitive 타입은 사용할 수 없다는 것이다. 
- 그래서 만약 int 배열을 내림차순 정렬하기 위해 Arrays.sort()를 사용한다면 반드시 Integer 타입의 배열로 변환하는 과정이 필요하다.
	- 하지만 변환하는 과정이 생각보다 효율적이진 않으니 간단한 알고리즘이라면 다른 방법을 사용해보는 것도 좋다. (실제로 코딩테스트 문제에서 시도했다가 효율성 점수가 정말 낮게 나온 적이 있다..)

``` java
Integer[] A = {4, 5, 3};
Arrays.sort(A, Collections.reverseOrder()); // 결과: A = {5, 4, 3};
```
<br/>


#### 참고
- int[]에서 Integer[]로 변환

``` java
int[] A = {4, 3, 3};
Integer[] arr = Arrays.stream(A).boxed().toArray(Integer[]::new);
```

- Integer[]에서 int[]로 변환

``` java
Integer[] A = {4, 3, 3};
int[] arr = Arrays.stream(A).mapToInt(Integer::intValue).toArray();
```