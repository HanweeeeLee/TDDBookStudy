# 14. 바꾸기
## TODO)
 - **$5 + 10CHF = $10(환율이 2:1일 경우)**
 - ~~$5 + $5 = $10~~
 - $5 + $5에서 Money 반환하기
 - ~~Bank.reduce(Money)~~
 - ~~Money에 대한 통화 변환을 수행하는 Reduce~~
 - ~~Reduce(Bank, String)~~

## 내용
드디어 모든 작업의 시초인 $5 + 10CHF에 대한 테스트를 추가할 준비가 되었다.

```JAVA
public void testMixedAddition() {
  Expression fiveBucks = Money.dollar(5);
  Expression tenFrancs = Money.franc(10);
  
  Bank bank = new Bank();
  bank.addRate("CHF", "USD",2);
  
  Money result = bank.reduce(fiveBucks.plus(tenFrancs),"USD");
  assertEquals(Money.dollar(10),result);
}
```

Money에서 Expression으로 일반화 할때 어설프게 남겨둔 부분들이 많기에 무수히 많은 컴파일 에러가 생기고,  
이런 컴파일 에러들을 해결해야 한다.  
```JAVA
public void testMixedAddition() {
  Money fiveBucks = Money.dollar(5);    //Expression -> Money로 변경
  Money tenFrancs = Money.franc(10);    //Expression -> Money로 변경
  
  Bank bank = new Bank();
  bank.addRate("CHF", "USD",2);
  
  Money result = bank.reduce(fiveBucks.plus(tenFrancs),"USD");
  assertEquals(Money.dollar(10),result);
}
```

Expression을 Money로 변경 후 다시 테스트해보면 10USD 대신 15USD가 나오게 된다.  
Sum.reduce()가 인자를 축약하지 않는것 같아 보인다.
```JAVA
//Sum 클래스
public class Sum implements Expression {
	Money augend;
	Money addend;
  
  Sum(Money augend, Money addend) {
		this.addend = addend;
		this.augend = augend;
	}
  
  public Money reduce(Bank bank, String to) {
    int amount = augend.amount + addend.amount;
    return new Money(amount,to);
  }
  
}
```

다음과 같이 두 인자를 모두 축약하면 테스트가 통과하게 될것이다.
```JAVA
//Sum 클래스
public class Sum implements Expression {
	Money augend;
	Money addend;
  
  Sum(Money augend, Money addend) {
		this.addend = addend;
		this.augend = augend;
	}
  
  public Money reduce(Bank bank, String to) {
    int amount = reduce(bank, to).amount + reduce(bank, to).amount;
    return new Money(amount,to);
  }
  
}
```

테스트를 통과하게 되고, 이제 우리는 Expression이어야 하는 Money들을 조금씩 없엘수 있다.  
파급 효과를 피하기 위해 가장자리에서 작업해 나가기 시작해서 그 테스트 코드까지 거슬러 올라오도록 하자.  
예를들어 피가산수와 가수는 이제 Expression으로 취급 할 수 있다.  
생성자의 인자 또한 Expression일 수 있다.
```JAVA
//Sum 클래스
public class Sum implements Expression {
	Expression augend;
	Expression addend;
  
  Sum(Expression augend, Expression addend) {
		this.addend = addend;
		this.augend = augend;
	}
  
  public Money reduce(Bank bank, String to) {
    int amount = reduce(bank, to).amount + reduce(bank, to).amount;
    return new Money(amount,to);
  }
  
}
```

Sum을 변경해 주었으니, plus()의 인자가 Expression으로 취급될 수 있다.
```JAVA
//Money 클래스
Expression times(int multiplier) {
  return new Money(amount*multiplier, currency);
};
```

times() 의 반환 값도 Expression 일수 있다.
```JAVA
//Money 클래스
Expression plus(Expression addend) {
  return new Sum(this, addend);
}
```

이제 테스트 케이스에 나오는 plus()의 인자도 바꿀 수 있다.
```JAVA
public void testMixedAddition() {
  Money fiveBucks = Money.dollar(5);
  Expression tenFrancs = Money.franc(10);    //Money ->Expression로 다시 변경
  
  Bank bank = new Bank();
  bank.addRate("CHF", "USD",2);
  
  Money result = bank.reduce(fiveBucks.plus(tenFrancs),"USD");
  assertEquals(Money.dollar(10),result);
}
```

fiveBucks 를 Expression 으로 바꾸고 나면 몇 군데를 같이 수정해야 한다.  
컴파일러가 Expression에 plus()가 정의 되지 않았다고 알려준다. 정의해주자.
```JAVA
//Expression 클래스
Expression plus(Expression addend);
```

이제 Money와 Sum에도 추가해야 한다.  
Money는 공용으로 변경해 준다.
```JAVA
//Money 클래스
public Expression plus(Expression addend) {
  return new Sum(this, addend);
}
```

Sum의 구현을 스텁구현으로 바꾸고 할일 목록에 적어두자.

```JAVA
//Sum 클래스
public Expression plus(Expression addend) {
  return null;
}
```
## TODO)
 - ~~**$5 + 10CHF = $10(환율이 2:1일 경우)**~~
 - ~~$5 + $5 = $10~~
 - $5 + $5에서 Money 반환하기
 - ~~Bank.reduce(Money)~~
 - ~~Money에 대한 통화 변환을 수행하는 Reduce~~
 - ~~Reduce(Bank, String)~~
 - Sum.plus
 - Expression.times

# 우리는
 - 원하는 테스트를 작성하고, 한 단계에 달성할 수 있도록 뒤로 물렀다.
 - 좀더 추상적인 선언을 통해 가지에서 뿌리(애초의 테스트 케이스)로 일반화했다.
 - 변경 후, 그 영향을 받은 다른 부분들을 변경하기 위해 컴파일러의 지시를 따랐다.
