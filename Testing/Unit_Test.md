# 단위 테스트란?

> 작은 코드 단위를 독립적으로 검증하는 테스트
> 
- 작은 코드 단위 := 클래스 or 메서드
- 독립적 := 외부 네트워크를 이용하거나 외부 상황에 의존하지 않는 것

⇒ 검증 속도가 빠르고, 안정적이다.

# JUnit 5

> 단위 테스트를 위한 테스트 **프레임워크**
> 

# AssertJ

> 테스트 코드 작성을 원활하게 돕는 테스트 **라이브러리**
> 
- 풍부한 API
- 메서드 체이닝 지원

```java
import static org.assertj.core.api.Assertions.assertThat;
...

@Test
void getNameTest() {
	Americano americano = new Americano();
	
	assertEquals(americano.getName(), "아메리카노");
	assertThat(americano.getName()).isEqualTo("아메리카노"):
}
```

이와 같은 방식으로 사용할 수 있다. 

<aside>
💡

`assertEquals` 는 JUnit의 API이다.

앞으로는 AssertJ를 이용할 것이기 때문에, `assertThat`을 사용하자.

</aside>

## List 관련 메서드

### `hasSize()`

> List의 크기를 확인하는 메서드
> 

```java
...
assertThat(cafekiosk.getBeverages()).hasSize(1);
...
```

### `isEmpty()`

> List가 비었는지 확인하는 메서드
> 

```java
...
assertThat(cafekiosk.getBeverages()).isEmpty();
...
```

---

# 테스트 케이스 세분화하기

### 1. 해피 케이스

> 요구사항을 그대로 만족하는 테스트 케이스
> 

### 2. 예외 케이스

> 예외가 발생하는 테스트 케이스
> 

⇒ **경계값 테스트** : 범위(이상, 이하, 초과, 미만), 구간, 날짜 등

경계값이 존재하는 경우에는 경계값에서 해피 케이스와 예외 케이스를 만들어 테스트하기

## 예외에 대한 테스트

### `assertThatThrownBy()`

```java
...
assertThatThrownBy(() -> cafeKiosk.add(americano, 0))
			.isInstanceOf(IllegalArgumentException.class)
			.hasMessage("음료는 1장 이상 주문하실 수 있습니다.");
...
```

> 어떤 상황에서 어떤 예외가 발생할 것인지 명시
> 

---

# 테스트하기 어려운 영역을 분리하기

다음과 같은 요구사항이 추가되었다고 생각해보자.

***가게 운영 시간(10:00 ~ 22:00) 외에는 주문을 생성할 수 없다.***

그냥 테스트를 작성하게 되면, 테스트를 하는 시점마다 결과값이 달라지기에 테스트를 보장할 수 없게 된다. 

**외부로 분리할수록 테스트 가능한 코드가 많아진다.**  

⇒ 테스트가 어려운 영역을 구분하고, 분리해라!

## 테스트하기 어려운 영역

### 1. 관측할 때마다 다른 값에 의존하는 코드

> 현재 날짜/시간, 랜덤 값, 전역 변수/함수, 사용자 입력 등
> 

### 2. 외부 세계에 영향을 주는 코드

> 표준 출력, 메시지 발송, DB 기록 등
> 

## 테스트하기 쉬운 영역 ⇒ 순수함수(pure functions)

### 1. 같은 입력에는 항상 같은 결과

### 2. 외부 세상과 단절된 형태

### 3. 테스트하기 쉬운 코드