# 열거형 ENUM

<aside>
💡 실무에서 다국어 개발을 하면서 DB에 저장되어있는 값이 한글이였기때문에 다국어처리가 어렵게 되어서 이때 enum으로 해결한적이있는데 enum에 대해 정리해보려고한다.

</aside>

## enum이란?

- 상수 데이터들의 집합
- 완전한 형태의 클래스로 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다
- 외부에서 접근할수있는 생성자를 제공하지 않으므로 사실상 final이다.

---

### 1. enum 선언

1. 기본적인 사용방법

```java
public enum NoticeStatusType {

    /**
     * 완료
     */
    DONE("notice.done"),

    /**
     * 진행
     */
    PROGRESS("notice.progress"),

    /**
     * 대기
     */
    STANDBY("notice.standby");
    
}
```

- 완전한 클래스 형태이기때문에 클래스 반환타입에 `enum` 으로 작성한다
- 관례적으로 열거상수는 모두 대문자로 작성해준다
- 바뀌지않는 값들을 enum으로 작성해주면 좋다

```java
//쿼리 조회
<select id="findAllWithNo" resultType="Notice">
        SELECT //쿼리에서 구분값으로 조회한 결과값을 열거형으로 선언한 값으로 조회한다
            CASE
                WHEN
                    FORMAT(GETDATE(), 'yyyyMMdd') <![CDATA[>]]> edate
                    THEN 'DONE'
                WHEN
                    FORMAT(GETDATE(), 'yyyyMMdd') <![CDATA[>=]]> sdate AND FORMAT(GETDATE(), 'yyyyMMdd') <![CDATA[<=]]> edate
                    THEN 'PROGRESS'
                WHEN
                    FORMAT(GETDATE(), 'yyyyMMdd') <![CDATA[<]]> sdate
                    THEN 'STANDBY'
                ELSE NULL
            END AS status
        FROM
            tbl_notice
    </select>
```

```java
			//type에 enum으로 작성해놓은 값들을 사용할수있다
			if (afterDate) {
                        // DONE
                        type = NoticeStatusType.DONE; 
                    } else if(BeforeDate) {
                        // STANDBY
                        type = NoticeStatusType.STANDBY;
                    } else if(betweenDate) {
                        // PROGRESS
                        type = NoticeStatusType.PROGRESS;
                    }
                    final var inputDate = DateUtils.dateTime(
                        onmNoticeMap.get("inputdate"),
                        "yyyyMMddHHmm"
                    ).toLocalDate();
```

---

## 2. enum 계산식적용 (상태와 행위를 한번에 관리)

<aside>
✏️ 화면에서 계산식이 각각 다를때 if or switch문을 통해서 분기하여 로직을 작성하는것보다 코드구성은 값(상태)과 메소드(행위)가 어떤 관계가있는지 한눈에 파악할수있는 enum으로 로직작성을 할수있다

</aside>

<aside>
💡 enum으로 상태만 다뤄봤지 계산식까지 한곳에서 관리할수있는것이 너무 좋았다~
그래서 어떤 결제수단을 선택하는지에 따라서 총가격 금액이 변경되도록 코드를 작성해봤다.

</aside>

**2-1. PayTypeEnum으로 열거형 상수 정의**

```java
package com.TIL.enumTest;

import lombok.RequiredArgsConstructor;

@RequiredArgsConstructor
public enum PayTypeEnum {

		//람다식으로 작성
    CASH(value -> value), //현금결제
    CARD(value -> (int)(value +(value * 0.005))), //카드결제
    PAY(value -> (int)(value +(value * 0.011))); //페이결제

    private final PayCalculator payCalculator;

    public int calculation (int value) {
        return payCalculator.calculate(value);
    }

    interface PayCalculator {//계산방식에 따른 계산담당 인터페이스
        int calculate(int value);
    }
}

```

- `현금결제` : CASH(value -> value)
   `카드결제` : CARD(value -> (int)(value +(value * 0.005)))
   `페이결제` :PAY(value -> (int)(value +(value * 0.011)))

**2-2. 가격과 결제방식 선택 화면생성**

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
</head>
<body>
<h1>ENUM TEST</h1>
<h2>결제방식을 선택해주세요</h2>
<form action="/pay/calculate" method="post">
    <input type="text" name="value" class="form-text" placeholder="가격을 입력하세요">
    <input type="radio" name="payment" id="cash" value="CASH">
    <label for="cash">현금결제</label>
    <input type="radio" name="payment" id="card" value="CARD">
    <label for="card">카드결제</label>
    <input type="radio" name="payment" id="pay" value="PAY">
    <label for="pay">페이결제</label>
    <button type="submit">결제하기</button>
</form>
</body>
</html>
```

![스크린샷 2024-05-02 오후 4.21.51.png](%E1%84%8B%E1%85%A7%E1%86%AF%E1%84%80%E1%85%A5%E1%84%92%E1%85%A7%E1%86%BC%20ENUM%20f809591950fe4f4d90b319abcabff04d/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2024-05-02_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_4.21.51.png)

**2-3.  계산값 확인**

```java
package com.TIL.enumTest.controller;

import com.TIL.enumTest.PayTypeEnum;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PostMapping;

@Slf4j
@Controller
public class PayController {

    PayTypeEnum payType;

    @PostMapping("/pay/calculate")
    public void pay(int value , String payment) {
        int total = 0; //결제 총금액
        payType = PayTypeEnum.valueOf(payment); //상태값
        total = payType.calculation(value); //계산된값

        log.info("결제금액은 {} 원입니다", total); //출력
    }
}
```

 

![스크린샷 2024-05-02 오후 4.24.43.png](%E1%84%8B%E1%85%A7%E1%86%AF%E1%84%80%E1%85%A5%E1%84%92%E1%85%A7%E1%86%BC%20ENUM%20f809591950fe4f4d90b319abcabff04d/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2024-05-02_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_4.24.43.png)

![Untitled](%E1%84%8B%E1%85%A7%E1%86%AF%E1%84%80%E1%85%A5%E1%84%92%E1%85%A7%E1%86%BC%20ENUM%20f809591950fe4f4d90b319abcabff04d/Untitled.png)

---

### 3. enum 메소드

- 여러가지 메소드들중 쓰면 좋을꺼같은것들 몇가지 작성 해보려고 한다

**enumMethod테스트용**

```java
package com.TIL.enumTest;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@RequiredArgsConstructor
@Getter
public enum EnumMethod {

    //상태 코드값들 enum으로 정리
    OK("200"),
    NOT_FOUND("404"),
    INTERNAL_SERVER_ERROR("500");

    private final String status;
}

```

1. `ordinal()` : 전체 enum중 현재값이 몇번째인지 확인하는 메소드

```java
@Slf4j
@Controller
public class EnumMethodController {

    @GetMapping("/ordinalTest")
    public void OrdinalTest() {
        EnumMethod m = EnumMethod.NOT_FOUND;
        log.info("NOT_FOUND는 {}번째 값입니다.", m.ordinal()); //몇번째인지 출력
    }
}
```

![Untitled](%E1%84%8B%E1%85%A7%E1%86%AF%E1%84%80%E1%85%A5%E1%84%92%E1%85%A7%E1%86%BC%20ENUM%20f809591950fe4f4d90b319abcabff04d/Untitled%201.png)

1. `valueOf()` : enum값을 가져오는 메소드

```java
    //enum값을 가져오는 메소드
    @GetMapping("/ValueOfTest")
    public void ValueOfTest() {
        EnumMethod m = EnumMethod.valueOf("INTERNAL_SERVER_ERROR");
        log.info("INTERNAL_SERVER_ERROR는 상태코드가 {} 입니다.", m.getStatus());
    }
```

![Untitled](%E1%84%8B%E1%85%A7%E1%86%AF%E1%84%80%E1%85%A5%E1%84%92%E1%85%A7%E1%86%BC%20ENUM%20f809591950fe4f4d90b319abcabff04d/Untitled%202.png)

1. `compareTo()`: 두개의 enum값을 비교하는 메소드

```java
@GetMapping("/compareToTest")
    public void compareToTest() {
        EnumMethod enum1 = EnumMethod.OK;
        EnumMethod enum2 = EnumMethod.NOT_FOUND;
        EnumMethod enum3 = EnumMethod.INTERNAL_SERVER_ERROR;

        //결과값보다 앞에 있다면 : 음수
        //결과값과 같다면 : 0
        //결과값보다 뒤에 있다면 : 양수
        log.info("enum1과 enum2은 : {}", enum1.compareTo(EnumMethod.NOT_FOUND));
        log.info("enum2와 enum2은 : {}", enum2.compareTo(EnumMethod.NOT_FOUND));
        log.info("enum3과 enum2은 : {}", enum3.compareTo(EnumMethod.NOT_FOUND));
    }
```

![Untitled](%E1%84%8B%E1%85%A7%E1%86%AF%E1%84%80%E1%85%A5%E1%84%92%E1%85%A7%E1%86%BC%20ENUM%20f809591950fe4f4d90b319abcabff04d/Untitled%203.png)