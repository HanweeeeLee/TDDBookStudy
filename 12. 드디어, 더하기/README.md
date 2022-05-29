# 12. 드디어, 더하기

## TODO)
 - $5 + 10CHF = $10(환율이 2:1일 경우)
 - ~~$5 X 2 = $10~~
 - ~~amount를 private으로 만들기~~
 - ~~Dollar 부작용?~~
 - Money 반올림?
 - ~~equals()~~
 - hasCode()
 - Equal null
 - Equal Object
 - ~~5CHF X 2 = 10CHF~~
 - ~~Dollar/Franc 중복~~
 - ~~공용 equals~~
 - ~~공용 times~~
 - ~~Franc과 Dollar 비교하기~~
 - ~~통화~~
 - ~~testFuncMultiplication 제거~~

-> 

 - $5 + 10CHF = $10(환율이 2:1일 경우)
 - **$5 + $5 = $10**

> 기존 TODO를 정리해보자. 더하기 기능을 시작해보자.

## 진행

```JAVA
public void testSimpleAddition() {
	Money sum = Money.dollar(5).plus(Money.dollar(5));
	assertEquals(Money.dollar(10), sum);
}
```
> 그냥 'Money.dollar(10)'를 반환하는 가짜식을 구현할 수도 있지만 어떻게 구현해야하는지 명확하므로 다음과 같이 하겠다.

```JAVA
class Money {
	Money plus(Money addend) {
		return enw Money(amount + addend.amount, currency);
	}
}
```
> 앞으로는 가짜식을 쓸 수 없으면 가짜식을 생략하겠음. 단, 어떻게 구현할지 모르면 가짜식을 작성해두자. TDD연습?!

다중 통화 연산을 어떻게 표현해야할까? 객체가 우리를 구해줄 것이다.  
가지고 있는 객체가 우리가 원하는 방식으로 동작하지 않을 경우엔 그 객체와 외부 프로토콜이 같으면서 내부 구현은 다른 새로운 객체(imposter)를 만들 수 있다.  
```JAVA
public void testSimpleAddition() {
	...
	Money reduced = bank.reduce(sum, "USD");
	assertEquals(Money.dollar(10), reduced);
}
```
> 왜 bank라는 Object가 reduce를 가질까?
 - Expression은 우리가 하려고 하는 일의 핵심에 해당한다. 핵심이 되는 객체가 다른 부분에 대해서는 가능하면 몰라야 한다. 그렇게 하면 핵심 객체가 가능한 오랫 동안 유연할 수 있다.
 - 만약 모든 오퍼레이션을 Expression에만 추가한다면 Expression에만은 무한이 커질것이다.
  
이것이 모든 이유는 아니지만 일단 이정도로..  
  
```JAVA
public void testSimpleAddition() {
	...
	Money five = Money.dollar(5);
	Expression sum = five.plus(five);
	Bank bank = new Bank();
	Money reduced = bank.reduce(sum, "USD");
	assertEquals(Money.dollar(10), reduced);
}
```
이걸 컴파일 하려면 어떻게해야할까? Expression 인터페이스를 만들어주자.

```JAVA
interface Expression 
```

Money.plus()는 Expression을 반환해야 한다. 이건 Money가 Expression을 구현해야한다는 뜻이다.
```JAVA
class Money implements Expression {
	...
	Expression plus(Money addend) {
		return enw Money(amount + addend.amount, currency);
	}
}
```

이제 빈 Bank 클래스가 필요하다. 그리고 Bank에는 reduce()의 스텁이 있어야 한다.
```JAVA
class Bank {
	Money reduce(Expression source, String to) {
		return null;
	}
}
```
이제 컴파일이 되고, 바로 실패한다. 만세! 진전이다. 간단히 가짜 구현을 할 수 있다.
```JAVA
class Bank {
	Money reduce(Expression source, String to) {
		// return null;
		return Money.dollar(10);
	}
}
```

## 우리는
 - 큰 테스트를 작은 테스트($5 + 10CHF에서 $5 + $5로) 줄여서 발전을 나타낼 수 있도록 했다.
 - 우리에게 필요한 계산(computation)에 대한 가능한 메타포들을 신중히 생각해봤다.
 - 새 메타포에 기반하여 기존의 테스트를 재작성했다.
 - 테스트를 빠르게 컴파일했다.
 - 그리고 테스트를 실행했다.
 - 진짜 구현을 만들기 위해 필요한 리팩토링을 약간의 전율과 함께 기대했다.






























