# 14. 바꾸기
## TODO)
 - $5 + 10CHF = $10(환율이 2:1일 경우)
 - $5 + $5 = $10
 - $5 + $5에서 Money 반환하기
 - ~~Bank.reduce(Money)~~
 - **Money에 대한 통화 변환을 수행하는 Reduce**
 - Reduce(Bank, String)

## 내용
변화에 대해서 생각해보자. 2프랑이 있는데, 이걸 달러로 바꾸고싶다.

```JAVA
public void testReduceMoneyDifferentCurrency() {
	Bank bank = new Bank();
	bank.addRate("CHF", "USD", 2);
	Money result = bank.reduce(Money.franc(2), "USD");
	assertEquals(Money.dollar(1), result);
}
```
> 일단 대충 프랑을 달러로 변환할떄 나누기 2를 하자.

```JAVA
class Money implements Expression {
	...
	Expression plus(Money addend) {
		return new Sum(this, addend);
	}

	public Money reduce(String to) {
		// return this;
		int rate = (currency.equals("CHF") && to.equals("USD")) ? 2 : 1;
		return new Money(amount / rate, to);
	}
}
```
갑자기 Money가 환율을 알게되었다.. 환율에 대한 일은 모두 Bank가 처리해야 한다. Expression.reduce()의 인자로 Bank를 넘겨야 할 것이다.  

```JAVA
class Bank {
	Money reduce(Expression source, String to) {
		return source.reduce(to);
	}
}

interface Expression
Money reduce(String to);

class Sum implements Expression {
	Money augend;
	Money addend;

	Sum(Money augend, Money addend) {
		this.augend = augend;
		this.addend = addend;
	}

	public Money reduce(String to) {
		int amount = augend.amount + addend.amount;
		return new Money(amount, to);
	}
}

class Money implements Expression {
	...
	Expression plus(Money addend) {
		return new Sum(this, addend);
	}

	public Money reduce(String to) {
		int rate = (currency.equals("CHF") && to.equals("USD")) ? 2 : 1;
		return new Money(amount / rate, to);
	}
}
```
> 그냥 모아서 보기... 이제 Bank로 이관해보자.
```JAVA
class Bank {
	Money reduce(Expression source, String to) {
		return source.reduce(to);
	}

	int rate(String from, String to) {
		return (currency.equals("CHF") && to.equals("USD")) ? 2 : 1;
	}
}

class Money implements Expression {
	...
	Expression plus(Money addend) {
		return new Sum(this, addend);
	}

	// public Money reduce(String to) { 
	public Money reduce(Bank bank, String to) { // 그리고 올바른 환율을 Bank에게 물어보자
		// int rate = (currency.equals("CHF") && to.equals("USD")) ? 2 : 1;
		int rate = bank.rate(currency, to);
		return new Money(amount / rate, to);
	}
}
```

환율 매핑이 필요할듯..
```JAVA
public void testArrayEquals() {
	assertEquals(new Object[] {"abc"}, new Object[] {"abc"});
}
```
> 테스트를 실패한다. 배열의 각각 원소의 동치성은 지원하지 않는다.  

키를 위한 객체를 따로 만들자.
```JAVA
private class Pair {
	private String from;
	private String to;

	Pair(String from, String to) {
		this.from = from;
		this.to = to;
	}
}
```
우린 Pair를 키로 쓸 거니까 equals()와 hashCode()를 구현해야 한다.  
하지만 리팩토링하는 중에 코드를 작성하는 것이기 떄문에 테스트를 작성하지는 않을 것이다.  

```JAVA
private class Pair {
	private String from;
	private String to;

	Pair(String from, String to) {
		this.from = from;
		this.to = to;
	}

	public boolean equals(Object object) {
		Pair pair = (Pair) object;
		return from.equals(pair.from) && to.equals(pair.to);
	}

	public int hashCode() {
		return 0; // 일단 대충 박아둔다. TODO) 나중에 진짜로 바꿀것.
	}
}
```

일단 확율을 저장할 뭔가가 필요하다. 환율을 설정할 수도, 환율을 얻어낼 수도 있어야 한다.

```JAVA
class Bank {

	private Hashtable rates = new Hashtable(); // 추가

	Money reduce(Expression source, String to) {
		return source.reduce(to);
	}

	int rate(String from, String to) {
		return (currency.equals("CHF") && to.equals("USD")) ? 2 : 1;
	}

	void addRate(String from, String to, int rate) { // 추가 환율 설정
		rates.put(new Pair(from, to), new Integer(rate));
	}

	int rate(String from, String to) { // 추가 환율 얻어오기
		Integer rate = (Integer) rates.get(new Pair(from, to));
		return rate.intValue();
	}
}
```

빨간 막대다.....  
USD에서 USD로의 환율을 요청하면 그 값이 1이 되어야 한다고 기대하고있다.. 테스트코드를 만들어두자.
```JAVA
public void testIdentityRate() {
	assertEquals(1, new Bank().rate("USD", "USD"));
}
```
빨간 막대를 없애보자.
```Java
class Bank {

	private Hashtable rates = new Hashtable();

	Money reduce(Expression source, String to) {
		return source.reduce(to);
	}

	int rate(String from, String to) {
		return (currency.equals("CHF") && to.equals("USD")) ? 2 : 1;
	}

	void addRate(String from, String to, int rate) { 
		rates.put(new Pair(from, to), new Integer(rate));
	}

	int rate(String from, String to) {
		if (from.equals(to)) return 1; // 같은놈끼리 변환할라고하면 1 리턴
		Integer rate = (Integer) rates.get(new Pair(from, to));
		return rate.intValue();
	}
}
```
> 초록 막대!

## TODO)
 - $5 + 10CHF = $10(환율이 2:1일 경우)
 - **~~$5 + $5 = $10~~**
 - $5 + $5에서 Money 반환하기
 - ~~Bank.reduce(Money)~~
 - **~~Money에 대한 통화 변환을 수행하는 Reduce~~**
 - **~~Reduce(Bank, String)~~**

## 우리는
 - 필요할 거라고 생각한 인자를 빠르게 추가했다.
 - 코드와 테스트 사이에 있는 데이터 중복을 끄집어냈다.
 - 자바의 오퍼레이션에 대한 가정을 검사해보기 위한 테스트(testArrayEquals)를 작성했다.
 - 별도의 테스트 없이 전용(private) 도우미(helper) 클래스를 만들었다.
 - 리팩토링하다가 실수를 했고, 그 문제를 분리하기 위해 또 하나의 테스트를 작성하면서 계속 전진해 가기로 선택했다.





































