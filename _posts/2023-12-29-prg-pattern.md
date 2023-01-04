---
layout: post
title: PRG 패턴과 RedirectAttributes
subtitle: redirect 시 GET, POST
categories: wiki
tags: [Java,HTTP,Spring]
---

### PRG 패턴과 RedirectAttributes


#### **배경 및 문제점**
- POST로 가입하는 로직을 작성하다가 해당 페이지에서 새로고침을 하면 중복 가입이 되거나 성공 후 이동하는 로직에서 처리가 되지 않는 버그를 발견했다.
<br/>


#### 해결 방안 (=PRG 패턴)
```java
@PostMapping("/add")
public String addItemV5(Item item) {
      itemRepository.save(item);
      return "redirect:/basic/items/" + item.getId();
}
```
- 회원 가입 메소드(POST)에서 새로고침을 하거나 성공 로직으로 넘어갈 때 forward를 사용했더니 url이 바뀌지 않는다는 것을 확인했다. 그래서 forward가 아닌 redirect를 사용하여 다시 로그인 view로 보내는 방식으로 해결했다.
- forward는 서버 내부에서 일어나는 호출이기 때문에 URL이 바뀌지 않기 때문에 클라이언트가 인식하지 못한다. 반면에, redirect의 경우 클라이언트에 응답을 보냈다가, 클라이언트가 redirect path로 재요청하기 때문에 URL 경로도 변경된다. 
- 즉, 마지막에 호출한 내용이 로그인 화면이 되는 것이다. 이후 새로 고침을 해도 로그인 화면으로 이동하게 되므로 문제를 해결할 수 있다.
- 이러한 해결 방식을 **PRG(Post-Redirect-Get) 패턴**이라고 한다.
<br/>


#### RedirectAttributes
```java
@PostMapping("/add")
public String addItemV6(Item item, RedirectAttributes redirectAttributes) {
      Item saveItem = itemRepository.save(item);
      redirectAttributes.addAttribute("itemId", saveItem.getId());
      redirectAttributes.addAttribute("status", true);
      return "redirect:/basic/items/{itemId}";
}
```
- URL 인코딩 및 @PathVariable과 쿼리 파라미터를 처리해주는 객체
- 추가된 속성이 @PathVariable에 포함되면 바인딩해주고, 나머지 값은 쿼리 파라미터로 처리한다.