---
layout: post
title: List에서 원소 삭제하기 remove()
subtitle: 
categories: wiki
tags: [Java,List,Method]
---
### 배경
코딩테스트로 문제를 풀다가 list에서 원소를 삭제하는 remove() 메소드를 사용할 때 값이 일치하면 삭제하고 싶은데 그냥 숫자를 넣으면 인덱스로 인식하는 것을 발견하였다.   
<br/>

### Integer remove(int index)
-   파라미터를 int로 전달하면 해당 index의 값을 삭제한다.   
<br/>   

### boolean remove(Object o)
-   파라미터로 Object 객체를 전달하면, List에서 해당 객체를 찾아서 **첫번째**로 나오는 값을 삭제한 뒤 값을 삭제하는 데 성공하면 **true**, 값이 없으면 **false**를 리턴한다.
-   **예시** (프로그래머스 level 0 배열에서 원소 삭제하기)
    -   [https://school.programmers.co.kr/learn/courses/30/lessons/181844](https://school.programmers.co.kr/learn/courses/30/lessons/181844)
```java
public int[] solution(int[] arr, int[] delete_list) {
	List<Integer> answer = Arrays.stream(arr).boxed().collect(Collectors.toList());
	for (int i = 0; i < delete_list.length; i++)
		if (answer.contains(delete_list[i]))
				answer.remove(Integer.valueOf(delete_list[i])); // (Integer)으로 형변환도 가능
	return answer.stream().mapToInt(i -> i).toArray();
}
```
-   그래서 위처럼 Object로 인식할 수 있도록 **Wrapper 클래스로(여기선 Integer) 형변환해주는 방법**이 있다.