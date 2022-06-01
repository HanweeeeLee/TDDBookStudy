# 13. 진짜로 만들기

## TODO)
 - $5 + 10CHF = $10(환율이 2:1일 경우)
 - **$5 + $5 = $10**

## 내용
모든 중복을 제거하기 전까지는 $5 + $5 테스트에 완료 표시를 할 수 없다.

```JAVA
class Bank {
	Money reduce(Expression source, String to) {
		return Money.dollar(10); // 사실 테스트 코드에 있는 $5 + $5와 같다.
	}
}
```

```JAVA
public void testSimpleAddition() {
	Money five = Money.dollar(5);
	Expression sum = five.plus(five); // 이부분..?!
	Bank bank = new Bank();
	Money reduced = bank.reduce(sum, "USD");
	assertEquals(Money.dollar(10), reduced);
}
```
이전에는 가짜 구현이 있을 때 진짜 구현으로 바꾸는 일은..  
-> 단순히 상수를 변수로 치환하는 일이었다.
-> 하지만 지금은 명확하지 않은듯  
  
우선 Money.plus()는 그냥 Money가 아닌 Expression(Sum)을 반환해야 한다.

```JAVA
public void testPlusReturnsSum() { // 새로 추가한 테스트인듯?
	Money five = Money.dollar(5);
	Expression result = five.plus(five);
	Sum sum = (Sum) result;
	assertEquals(five, sum.augend); // augend?? 덧샘의 첫 인자
	assertEquals(five, sum.addend);
}
```
> 이 테스트는 오래가지 못한다. 수행하고자 하는 연산의 외부 행위가 아닌 내부 구현에 대해 너무 깊게 관여하고 있다.  
  
일단 이 테스트를 통과하면 목표에 한 걸음 더 다가가게 될 것이다. 이 코드를 컴파일하기 위해선 augend와 addend 필드를 가지는 Sum클래스가 필요하다.

```JAVA
class Sum {
	Money augend;
	Money addend;
}
```
Money클래스 역시 plus메서드의 리턴타입이 Expression 이기때문에 Sum으로 바꿔주자.
```JAVA
class Money implements Expression {
	...
	Expression plus(Money addend) {
		// return Money(amount + addend.amount, currency);
		return new Sum(this, addend); // 개인적인 궁금증. 앞 단의 Expression은 아무것도 명시되어있지 않은 interface, 그러면 Sum도 Expression을 채택해야 이거 리턴 가능한거 아닌가 ㅡㅡㅋ
	}
}
```

```JAVA
// class Sum {
class Sum implements Expression { // 이걸 뒤에서 하네..
	Money augend;
	Money addend;

	Sum(Money augend, Money addend) { // 생성자 추가

	}
}
```

이제 시스템이 다시 컴파일되는 상태로 돌아왔다. 하지만 테스트는 여전히 실패한다.  
이유는 Sum생성자에서 필드를 설정하지 않기 때문

```JAVA
class Sum implements Expression {
	Money augend;
	Money addend;

	Sum(Money augend, Money addend) {
		// 추가.. 자바는 이걸 뺴먹어도 컴파일이 되는게 놀랍네..
		this.augend = augend;
		this.addend = addend;
	}
}
```

이제 Bank.reduce()는 Sum을 전달받는다.  
만약 Sum이 가지고 있는 Money의 통화가 모두 동일하고, reduce를 통해 얻어내고자 하는 Money의 통화 역시 같다면, 결과는 Sum내에 있는 Money들의 amount를 합친 값을 갖는 Money객체여야 한다.  
```JAVA
public void testReduceSum() {
	Expression sum = new Sum(Money.dollar(3), Money.dollar(4));
	Bank bank = new Bank();
	Money result = bank.reduce(sum, "USD");
	assertEquals(Money.dollar(7), result);
}
```
우리가 Sum을 계산하면 결과는 Money가 되어야 하며, 그 Money의 양은 두 Money양의 합이고, 통화는 우리가 축약하는 통화여야 한다.  

```JAVA
class Bank {
	Money reduce(Expression source, String to) {
		// return Money.dollar(10); // 사실 테스트 코드에 있는 $5 + $5와 같다.
		Sum sum = (Sum) source;
		int amount = sum.augend.amount + sum.addend.amount;
		return new Money(amount, to);
	}
}
```
이 코드는 두 가지 이유로 지저분하다.
 - 캐스팅(형변환). 이 코드는 모든 Expression에 대해 작동해야 한다.
 - 공용(public) 필드와 그 필드들에 대한 두 단계에 걸친 레퍼런스.
  
리팩토링 해보자.
```JAVA
class Bank {
	Money reduce(Expression source, String to) {
		Sum sum = (Sum) source;
		// int amount = sum.augend.amount + sum.addend.amount;
		// return new Money(amount, to);
		return sum.reduce(to)
	}
}

class Sum implements Expression {
	Money augend;
	Money addend;

	Sum(Money augend, Money addend) {
		this.augend = augend;
		this.addend = addend;
	}

	public Money reduce(String to) { // 메서드 이동
		int amount = augend.amount + addend.amount;
		return new Money(amount, to);
	}
}
```
> 외부에서 접근 가능한 필드 몇 개를 들어내기 위해서 reduce메서드의 Sum Property접근 부분을 Sum의 reduce라는 메서드를 생성해서 이동시켰다. 언젠가는 Bank가 매개변수가 될 거라 생각하지만 일단 이렇게.. 

### TODO)
 - $5 + 10CHF = $10(환율이 2:1일 경우)
 - **$5 + $5 = $10**
 - $5 + $5에서 Money 반환하기
 - Bank.reduce(Money) // 추가

뱅크 관련 테스트를 하나 추가하자.  
막대가 초록색이니 일단 테스트를 바로 추가한다.  
```JAVA
public void testReduceMoney() {
	Bank bank = new Bank();
	Money result = bank.reduce(Money.dollar(1), "USD");
	assertEquals(Money.dollar(1), result);
}
```
```JAVA
class Bank {
	Money reduce(Expression source, String to) {
		if (source instanceof Money) return (Money) source; // 추가
		Sum sum = (Sum) source;
		return sum.reduce(to);
	}
}
```
지저분하다.. 그래도 막대가 초록색이니 리팩토링을 해보자.  
클래스를 명시적으로 검사하는 코드가 있을 때에는 항상 다형성을 사용하는게 좋다.
```JAVA
class Bank {
	Money reduce(Expression source, String to) {
		// if (source instanceof Money) return (Money) source;
		if (source instanceof Money)
			return (Money) source.reduce(to);

		Sum sum = (Sum) source;
		return sum.reduce(to);
	}
}

class Money implements Expression {
	...
	Expression plus(Money addend) {
		return new Sum(this, addend);
	}

	public Money reduce(String to) { // 추가
		return this;
	}
}
```

이 상태에서 Expression 인터페이스에 reduce(String)를 추가하자.
```JAVA
interface Expression
Money reduce(String to);
```

이렇게 하면 지저분한 캐스팅과 클래스 검사 코드를 제거할 수 있다.
```JAVA
class Bank {
	Money reduce(Expression source, String to) {
		// if (source instanceof Money)
		// 	return (Money) source.reduce(to);

		// Sum sum = (Sum) source;
		// return sum.reduce(to);
		return source.reduce(to);
	}
}
```
Expression과 Bank에 이름은 동일하지만 매개 변수 형이 다른 메서드가 있다는 것이 만족스럽지는 않다.  

### TODO)
 - $5 + 10CHF = $10(환율이 2:1일 경우)
 - **$5 + $5 = $10**
 - $5 + $5에서 Money 반환하기
 - **~~Bank.reduce(Money)~~**
 - Money에 대한 통화 변환을 수행하는 Reduce
 - Reduce(Bank, String)

## 우리는
 - 모든 중복이 제거되기 전까지는 테스트를 통과한 것으로 치지 않았다.
 - 구현하기 위해 역방향이 아닌 순방향으로 작업했다.
 - 앞으로 필요할 것으로 예상되는 객체(Sum)의 생성을 강요하기 위한 테스트를 작성했다.
 - 빠른 속도로 구현하기 시작했다(Sum의 생성자).
 - 일단 한 곳에 캐스팅을 이용해서 코드를 구현했다가, 테스트가 돌아가자 그 코드를 적당한 자리로 옮겼다.
 - 명시적인 클래스 검사를 제거하기 위해 다형성을 사용했다.












