---
layout: post
title: Map - getOrDefault(Object key, V defaultValue)
subtitle: 
categories: wiki
tags: [Java,Map,Method]
---

### Map - getOrDefault(Object key, V defaultValue)


```java
public V getOrDefault(Object key, V defaultValue) {
		Node<K,V> e;
		return (e = getNode(key)) == null ? defaultValue : e.value;
}
```
- key가 존재하면 key의 value를 반환하고, 없으면 설정한 디폴트 값을 반환한다.
<br/>


**사용 예시 (프로그래머스 level2 위장)**
```java
String[][] clothes1 = {{"yellow_hat", "headgear"}, 
				{"blue_sunglasses", "eyewear"}, 
				{"green_turban", "headgear"}
		};
HashMap<String, Integer> map = new HashMap<>();
for (int i = 0; i < clothes.length; i++) {
		map.put(clothes[i][1], map.getOrDefault(clothes[i][1], 1) + 1);
}
```
- HashMap은 동일한 key에 value를 추가할 경우 value가 덮어쓰기 되기 때문에 getOrDefault() 메소드를 사용하여 예시처럼 카운트를 추가하는 등의 방법을 사용할 수 있다.