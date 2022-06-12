# 16. 드디어 추상화
## TODO)
 - ~~$5 + 10CHF = $10(환율이 2:1일 경우)~~
 - ~~$5 + $5 = $10~~
 - $5 + $5에서 Money 반환하기
 - ~~Bank.reduce(Money)~~
 - ~~Money에 대한 통화 변환을 수행하는 Reduce~~
 - ~~Reduce(Bank, String)~~
 - Sum.plus
 - Expression.times

## 내용
Expression.plus를 끝마치려면 Sum.plus()를 구현해야 한다.  
그리고 나서 Expression.times()를 구현하면 전체 예제가 끝난다.  
다음은 Sum.plus()에 대한 테스트다.
```JAVA
public void testSumPlusMoney() {
  Expression fiveBucks = Money.dollar(5);
  Expression tenFrancs = Money.franc(10);
        
  Bank bank = new Bank();
  bank.addRate("CHF","USD",2);
        
  Expression sum = new Sum(fiveBucks,tenFrancs).plus(fiveBucks);
        
  Money result = bank.reduce(sum,"USD");
  assertEquals(Money.dollar(15),result);        
}
```
두 Expression fiveBucks와 tenFrancs을 더해서 Sum을 만드는 대신, 명시적으로 Sum을 생성하였는데,  
이게 더 직접적으로 우리 의도를 드러내기에, 후대 우리의 코드를 읽게 될 독자를 생각하여 이런식으로 작성하였다.  
테스트코드가 구현 코드보다 더 길다. 심지어 구현할 코드는 Money의 코드와 똑같다.
```JAVA
//Sum 클래스
public Expression plus(Expression addend) {
  return new Sum(this, addend);
}
```
TDD로 구현할 땐 테스트 코드의 줄 수와 모델 코드의 줄 수가 거의 비슷한 상태로 끝난다.  
TDD가 경제적이기 위해서는 매일 만들어 내는 코드의 줄 수가 두배가 되거나 동일한 기능을 구현하되 절반의 줄 수로 해내야 한다.  
  
이제 Sum.times()가 작동하게 만들 수만 있다면, 마지막인 Expression.times()를 선언하는 일이야 어렵지 않을 것이다.  
테스트는 다음과 같다.
```JAVA
public void testSumTimes() {
  Expression fiveBucks = Money.dollar(5);
  Expression tenFrancs = Money.franc(10);
        
  Bank bank = new Bank();
  bank.addRate("CHF","USD",2);
        
  Expression sum = new Sum(fiveBucks,tenFrancs).times(2);
  Money result = bank.reduce(sum,"USD");
  assertEquals(Money.dollar(20),result);
}
```
이번에도 테스트가 코드보다 길다.
```JAVA
//Sum 클래스의 기존 times()
Expression times(int multiplier) {
  return new Sum(augend.times(multiplier), addend.times(multiplier));
}
```
지난 장에서 피가산수와 가수를 Expression으로 추상화 했기 때문에 코드가 컴파일 되게 만들려면  
Expression에 times()를 선언해야 한다.
```JAVA
//Expression
Expression times(int multiplier);
```
이제 Money.times()와 Sum.times()의 가시성을 높여주자.
```JAVA
//Sum 클래스
public Expression times(int multiplier) {
  return new Sum(augend.times(multiplier), addend.times(multiplier));
}

//Money 클래스
public Expression times(int multiplier) {
  return new Money(amount * multiplier, currency);
}
```
이제 테스트가 통과한다.  
뭔가 깔끔하게 정리해야 할 것이 있다면, $5 + $5 가 Money를 반환하는지를 실험해본다.  
테스트는 다음과 같을 것이다.
```JAVA
public void testPlusSameCurrencyReturnsMoney() {
  Expression sum = Money.dollar(1).plus(Money.dollar(1));
  assertTrue(sum instanceof Money);
}
```
이 테스트코드는 구현 중심이기 때문에 좀 지저분하지만, 단지 실험일 뿐더러 우리가 원하는 변화를 가할 수 있게 해준다.  
인자가 Money일 경우에 필요충분조건으로 인자의 통화를 확인하는 분명하고 깔끔한 방법이 없으니, 실험은 실패하였다.  
테스트를 삭제하고 떠나자.

## TODO)
 - ~~$5 + 10CHF = $10(환율이 2:1일 경우)~~
 - ~~$5 + $5 = $10~~
 - ~~$5 + $5에서 Money 반환하기~~
 - ~~Bank.reduce(Money)~~
 - ~~Money에 대한 통화 변환을 수행하는 Reduce~~
 - ~~Reduce(Bank, String)~~
 - ~~Sum.plus~~
 - ~~Expression.times~~

## 우리는
 - 미래에 코드를 읽을 다른 사람들을 염두에 둔 테스트를 작성했다.
 - TDD와 여러분의 현재 개발 스타일을 비교해 볼수 있는 실험방법을 제시했다.
 - 또 한번 선언부에 대한 수정이 시스템 나머지 부분으로 번져갔고, 문제를 고치기 위해 역시 컴파일러의 조언을 따랐다.
 - 잠시 실험을 시도했는데, 제대로 되지 않아서 버렸다.
