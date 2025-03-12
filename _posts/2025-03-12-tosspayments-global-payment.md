---
layout: post
title: 토스페이먼츠 해외결제(Paypal, 해외카드, Alipay) 연동 후기 🚀
categories: wiki
subtitle: 
tags: [API, 토스페이먼츠, 해외결제, ]
---
프로젝트 진행 중 토스페이먼츠를 통해 해외결제를 진행하게 되었는데, 제공해준 가이드가 깔끔하고 구체적이었지만 개인적으로 헤맨 부분이 많아 정리해 볼 겸 작성하게 되었습니다. 
해외결제 중 Paypal, 해외카드, Alipay 세 가지를 작업하며 겪은 트러블슈팅까지 작성해두었으니 누군가에게 도움이 되었으면 좋겠습니다.
<br/>
<br/>


### 시작 전 주의사항 ⚠️
- 해외결제는 모든 계약이 선행되어야 합니다.
- 국내결제, 브랜드페이, 해외결제를 모두 사용하기 위해 결제위젯를 웹뷰(Thymeleaf)로 구현하였습니다.
- 국내결제, 브랜드페이를 이미 구현하였으므로, successUrl과 failUrl이 개발되어 있다는 전제하에 글을 작성하였습니다.
<br/>

### 요구사항 
- Paypal와 Alipay는 달러로, 해외카드는 원화로 결제합니다.
- 최근 해외카드도 달러로 결제하는 시스템이 도입되었습니다.
<br/>
<br/>


## Paypal
### 결제위젯 웹뷰
‘[해외결제 연동](https://docs.tosspayments.com/guides/v2/learn/foreign-payment)’ 가이드에 해외카드와 함께 있는 주문서(javascript) 코드는 구체적이지 않아 결제위젯 > ‘[Paypal 연동하기](https://docs.tosspayments.com/guides/v2/payment-widget/integration-paypal)’ 가이드에 있는 주문서(javascript) 코드를 참고하였습니다.
Paypal 연동하기 가이드에서 테스트용 페이팔 계정도 제공하고 있으니 참고해서 개발하시면 좋겠습니다.

### Trouble Shooting 💫
![Image](https://github.com/user-attachments/assets/46b260fc-ab90-41d1-b792-92ae94e332b5)
테스트를 진행하면서 결제 승인의 결과로 Payment 객체를 받습니다. 그런데 이미 국내 결제와 브랜드페이를 작업하면서 easyPay라는 필드를 **객체**로 받고 있었는데, Paypal의 경우 easyPay 필드에 객체가 아닌 “페이팔”이나 ”paypal”이라는 **String**을 전달하고 있었습니다.  
그래서 JsonDeserializer를 사용해서 객체일 때와 String일 때를 구분해서 연결해주었습니다.

```java
public class EasyPayDtoDeserializer extends JsonDeserializer<TossPaymentDto.EasyPayDto> {

    @Override
    public TossPaymentDto.EasyPayDto deserialize(JsonParser parser, DeserializationContext context) {
        try {
            JsonNode node = parser.getCodec().readTree(parser);
            if (node.isTextual()) { // String일 때
                return new TossPaymentDto.EasyPayDto(node.asText(), 0, 0);
            } else if (node.isObject()) { // 객체일 때
                String provider = node.has("provider") ? node.get("provider").asText() : null;
                int amount = node.has("amount") ? node.get("amount").asInt() : 0;
                int discountAmount = node.has("discountAmount") ? node.get("discountAmount").asInt() : 0;
                return new TossPaymentDto.EasyPayDto(provider, amount, discountAmount);
            }
            return null;
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

```java
@JsonDeserialize(using = EasyPayDtoDeserializer.class) // 커스텀 변환 적용
public static class EasyPayDto { // 간편 결제
	private String provider; // 간편결제 제공
	private double amount; // 계좌 혹은 현금성 포인트 금액 (충전식 결제 수단)
	private double discountAmount; // 적립 포인트, 쿠폰, 즉시 할인된 금액 (적립식 결제수단)
}
```
먼저 EasyPayDtoDeserializer 클래스를 구현하고, Payment 객체 안의 EasyPayDto 객체에 `@JsonDeserialize` 애노테이션을 붙여 구현한 클래스와 연결해주면 됩니다.  
<br/>

#### JsonDeserializer란?
JsonDeserializer는 Jackson 라이브러리에서 제공하는 기능으로, JSON 데이터를 특정 Java 객체로 변환하는 과정에서 커스텀 변환 로직을 적용할 수 있도록 도와주는 클래스입니다.  
기본적으로 Jackson은 `@JsonProperty`나 `@JsonDeserialize` 등의 어노테이션을 사용하여 JSON을 Java 객체로 자동 매핑하지만, JSON 데이터의 구조가 예상과 다르거나, 특별한 변환 로직이 필요할 때 JsonDeserializer를 사용합니다.
<br/>
<br/>
<br/>


## 해외 카드
### 참고
해외 카드 계약을 완료하면 기능 > 카드사 목록에서 해외 카드를 확인할 수 있지만, 저는 해외 결제일 때와 국내 결제일 때를 구분하여 위젯을 노출시키기 때문에 [해외결제 연동 API 가이드](https://docs.tosspayments.com/guides/v2/learn/foreign-payment)를 참고했습니다.  
해당 가이드 또한 테스트용 해외 카드 번호를 제공하니 참고해서 개발하시면 좋겠습니다.  
최근 토스페이먼츠에 해외카드 달러 결제 기능이 추가되었다고 합니다. 하지만 저희 요구사항은 해외카드를 원화 결제이기 때문에 원화로 결제한 코드라는 점 참고해주시기 바랍니다.

### 준비
페이팔을 설정해줄 때처럼 결제 UI에서 설정하면 되지만, 다른 점은 커스텀 결제수단 추가에서 이름은 ‘Card’, key는 ‘CARD’로 설정해야 한다는 점입니다. 여기서 설정한 key가 결제위젯 (웹뷰)에서 사용할 키값이라는 것을 기억해두어야 합니다.

### Trouble Shooting 💫
```jsx
const selectedPaymentMethod = await paymentMethodWidget.getSelectedPaymentMethod();
	if (selectedPaymentMethod.code === "CARD") { // 커스텀 결제 수단에서 추가한 키 값
		// 해외 카드 결제창 호출
		const clientKeyCard = ""; // API 개별 연동 클라이언트 키(해외 카드 MID)
		const tossPaymentsCard = TossPayments(clientKeyCard);
		const customerKey = "T_NuvKOyhjmLdxNz5SouY";
		const payment = tossPaymentsCard.payment({ customerKey });
		payment.requestPayment({
			method: "CARD", // 커스텀 결제 수단에서 추가한 키 값
			amount: {
				currency: "KRW",
				value: 50000, // KRW의 경우 반드시 정수형 타입
			},
			orderId: "mhob2uEy33o616FNnwVpD",
			orderName: "토스 티셔츠 외 2건",
			successUrl: baseUrl + "/api/reservation/widget/success", // 추가
			failUrl: baseUrl + "/api/reservation/widget/fail", // 추가
			customerEmail: "customer123@gmail.com",
			customerName: "김토스",
			customerMobilePhone: "01012341234",
			card: {
				useInternationalCardOnly: true,
			},
		});
	return;
}
```

```jsx
Uncaught (in promise) Error: 모바일 화면에서는 Promise 방식을 지원하지 않습니다.
```  
토스에서 제공해준 코드대로 작성해서 시도했더니 위처럼 Promise 방식을 지원하지 않는다는 에러를 마주치게 되었습니다.
그런데 현재 Promise 방식을 사용하지 않았기 때문에, 혹시 몰라서 토스페이먼츠 에러 코드를 찾아보니 뒤에  successUrl, failUrl을 추가하라는 멘트가 있길래 추가해봤습니다.
    
![Image](https://github.com/user-attachments/assets/4121c00e-684f-4a4b-8408-3ed2b47f8068)
    
하지만 이렇게 해도 `COMMON_ERROR 처리 중 오류가 발생했습니다`라는 에러가 발생했습니다. 이 부분에 대해 토스페이먼츠 디스코드에서 질문하던 중 방법을 찾아냈습니다. 저는 해외결제라고 해서 해외결제 MID를 넣고 있었는데요. 그런데 해외카드에서는 KRW로 결제 처리를 하기 때문에 KRW로 결제하기 위한 MID를 사용해야 했던 것입니다. 당연히 해외카드는 해외결제 관련 키를 사용해야 한다고 생각했는데, 이 부분 때문에 조금 헤맸으니 이걸 보시는 분들은 시간을 아끼시길 바랍니다.
<br/>
<br/>


## AliPay
알리페이 또한 해외결제에 해당하지만, Paypal과 달리 비동기로 진행되어 로직을 추가해야 합니다. Paypal의 결제 방식부터 한 번 살펴보겠습니다.

![Image](https://github.com/user-attachments/assets/8f8f72a7-c20b-4d8b-ae46-2966394fa54b)
Paypal은 동기 결제로 우리가 생각하는 흐름처럼 진행됩니다. 결제 위젯에서 결제 정보를 입력하고 ‘결제하기’ 버튼을 누르면 연결된 successUrl/failUrl로 결제 정보가 전달됩니다. successUrl로 전달되었다면 결제 승인 API를 호출하여 결제를 처리하고, 이때 200 응답을 받으면 결제가 성공하게 되는 것입니다.

![Image](https://github.com/user-attachments/assets/154c847a-e6c3-4b43-a144-28dd0a0852cd)
알리페이의 비동기 결제는 무엇이 다를까요? 결제 위젯에서 결제 정보를 입력하고 결제를 요청하는 것까지는 동일하지만 pendingUrl이라는 부분이 생겼습니다. 또한 결제 승인의 결과를 웹훅으로 받아야 합니다.  
위에서 중요한 부분이 있습니다. 바로 웹훅이 최대 10분이 걸릴 수 있다는 점입니다. 이를 위해 pendingUrl로 넘어오면 앱에서는 결제 대기 상태라는 것을 알리고, 추후 결제 완료가 되면 다시 알려주겠다는 액션을 취해야 합니다.

### 구현 팁

#### 1. pendingUrl  
제 상황처럼 앱을 구현하신다면 pendingUrl에서 딥링크를 호출해서 앱으로 돌아갈 수 있도록 구현해주시면 됩니다.  

**pendingUrl 페이지 예시**
        
        ```html
        <!DOCTYPE html>
        <html lang="ko">
        <head>
            <meta charset="utf-8"/>
            <link rel="icon" href="https://static.toss.im/icons/png/4x/icon-toss-logo.png"/>
            <link rel="stylesheet" href="/css/style.css" type="text/css">
            <meta http-equiv="X-UA-Compatible" content="IE=Chrome"/>
            <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
            <title>토스페이먼츠 결제 위젯 대기</title>
        </head>
        
        <body>
        
        <script>
            async function confirm() {
                location.href = ''; // app scheme 호출
            }
            confirm();
        </script>
        
        </body>
        </html>
        
        ```


#### 2. 웹훅 등록 및 결제 승인 웹훅 API 추가  
웹훅으로 전달받는 결제 승인 결과는 Payment 객체만 전달 받는 게 아니기 때문에 별도로 구현이 필요합니다.
알리페이 또한 간편결제이므로 easyPay라는 필드에 String으로 "알리페이" 혹은 "APLIPAY"라는 값으로 전달되며, status 필드에 따라 결제 성공/실패 분기를 처리해주시면 됩니다.  

**웹훅 분기 구현 예시**  
➕ 결제 상태(PaymentStatus)를 미리 enum 클래스로 구현해두었습니다.
        
        ```java
        public void confirmOrderWebhook(OrderWebhookDto webhook) {
                TossPaymentDto payment = webhook.getData();
                
                // 결제 상태에 따른 분기 처리
                if (PaymentStatus.DONE.equals(status)) {
                    // 결제 성공 시 로직 구현
                } else if (PaymentStatus.ABORTED.equals(status) || 
        						        PaymentStatus.EXPIRED.equals(status)) {
                    // 결제 실패 시 로직 구현
                }
        }
        ```
        
### 주의
1. AWallet이라는 테스트앱이 존재하지만, AWallet에서 결제를 진행해도 pendingUrl로 보내주지 않습니다.
2. 알리페이 앱에서 pin 번호 인증으로 결제가 성공하면, 현재 서버의 운영에 상관없이 결제가 성공된 것으로 처리합니다.

### 결론
결과적으로 말하자면 알리페이는 현재 단계에서 적용하지 않게 되었습니다. 테스트 앱에서 결제를 진행(성공/실패)한 것을 앱에서 확인할 수 없고, 라이브 결제 또한 중국에서 생성한 계정만 사용할 수 있어 실제로 사용하기 어렵겠다는 결정이 있었습니다.
<br/>
<br/>


### 해외결제 추가 팁❗️
- KRW 결제는 소수점을 지원하지 않으므로, 반드시 정수형 데이터 타입으로 값을 전달해야 합니다.
- Paypal과 해외카드는 실결제 테스트를 할 수 없습니다.
    - Paypal은 국내 아이피, 국내 회원가입 계정으로 결제하는 것이 Paypal 정책 및 국내법상 불가능합니다.
    - 해외카드는 해외에서 발급된 카드를 사용해야 합니다. 국내에서 발급받은 카드 중 VISA, MASTER가 붙은 카드도 국내 카드로 인식됩니다.
<br/>
<br/>


### insight & thoughts
- 이번 해외결제 연동을 진행하면서 예상치 못한 문제들이 있었지만, 해결 과정을 통해 토스페이먼츠의 결제 시스템을 이해할 수 있었습니다.
- 특히 Paypal, 해외카드, Alipay의 차이점을 명확히 알게 되었고, 각 결제 방식에 맞는 구현 방식과 주의점을 익혔습니다.
- JsonDeserializer를 활용한 객체와 문자열 처리 방식의 차이를 직접 적용하면서 유연한 데이터 처리 방법을 배운 것도 큰 수확이었습니다.
- 해외카드 결제 시 KRW와 USD의 차이를 고려해야 했고, MID 설정 문제로 인해 에러가 발생했지만, 토스페이먼츠 디스코드와 공식 문서를 통해 해결할 수 있었습니다.
    - 토스페이먼츠 디스코드에서 개발자분들이 친절하게 잘 알려주십니다. 다시 한번 감사드립니다 🙏
- Alipay의 경우 비동기 결제 및 웹훅 처리 방식이 Paypal과 다르다는 점이 새로웠고, 이를 구현하는 과정에서 pendingUrl을 활용한 사용자 경험 개선을 고민해야 했습니다.
- Paypal과 해외카드는 국내에서는 실제 테스트가 불가능하여 일부는 이론적인 접근으로 구현해야 했다는 점이 아쉬웠습니다.
- 특히 Alipay의 경우, 테스트 환경의 제한으로 인해 실사용이 어려웠다는 점이 아쉽지만, 향후 글로벌 환경에서 적용할 수 있는 기반을 마련한 점은 의미 있다고 생각합니다.
- 이번 프로젝트를 통해 해외 결제 시스템의 복잡성을 이해하고, 각 결제 수단의 차이를 명확히 정리할 수 있었던 점이 가장 큰 수확이었습니다. 앞으로 글로벌 서비스를 준비할 때 도움이 될 중요한 경험이 될 것 같습니다!
<br/>
<br/>


**참고 자료**
- [토스페이먼츠 해외결제 연동하기 가이드](https://docs.tosspayments.com/guides/v2/learn/foreign-payment)
- [토스페이먼츠 해외 간편결제 연동하기 가이드](https://docs.tosspayments.com/guides/v2/payment-widget/integration-foreignpay)
- [토스페이먼츠 웹훅 이벤트 가이드](https://docs.tosspayments.com/reference/using-api/webhook-events#payment_status_changed)
- [토스페이먼츠 결제 연동 백서 노션](https://www.notion.so/17e714bbfde780b68df9efa1a1667e4a?pvs=21)
	- 해외카드 달러 결제하실 분들 참고하시면 좋겠습니다.