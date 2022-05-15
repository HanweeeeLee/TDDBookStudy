# 2장. 타락한 객체
TDD 최종 목적은 "작동하는 깔끔한 코드"를 얻는 것이다.  
작동하는 깔끔한 코드를 얻는 것은 때로는 최고의 프로그래머들 조차 도달하기 힘든 목표이며, 평범한 프로그래머들에게는 거의 불가능한 일이다.  
따라서, "작동하는 깔끔한 코드"를 얻기 위해 문제를 분할하여 접근해보자.  
결론부터 말하면, '작동하는 코드'를 만들고 나서 '깔끔한 코드' 로 변모시키는 것이다. 

## TDD 주기
일반적인 TDD 주기는 다음과 같다.
1. 테스트를 작성한다.
> 오퍼레이션이 코드에 어떤 식으로 나타나길 원하는 지 생각해보고, 인터페이스 대한 이야기를 작성해보는 작업이다.
2. 실행 가능하게 만든다.
> 이 단계에서 가장 중요한 것은 빠르게 초록 막대를 보는 것이다.  
> 깔끔하고 단순한 해법이 명백히 보인다면 그것을 입력한다.  
> 만약 깔끔하고 단순한 해법이지만 구현 시간이 몇 분 걸린다면 일단 적어 놓은 뒤 본래의 목적으로 돌아온다.
3. 올바르게 만든다.
> 이 단계에서 이전에 시스템 동작(초록 막대)을 보기 위해 저질렀던 죄악을 수습한다. 중복을 제거하고 초록 막대로 되돌린다.

### 테스트 목록
> $5 + 10CHF = $10(환율이 2:1 일 경우)  
> ~~$5 * 2 = 10$~~  
> amount를 private으로 만들기  
> **Dollar 부작용(side effect)?**  
> Money 반올림?

테스트를 하나 통과했지만 뭔가 이상하다.  
Dollar에 대해 연산을 수행한 후에 해당 Dollar의 값이 바뀌는 점이다.  
우리는 다음과 같이 사용할 수 있기를 바란다.
```JAVA
@Test
public void testMultiplication(){
  Dollar five = new Dollar(5);
  five.times(2);
  assertEquals(10, five.amount);
  five.times(3);
  assertEquals(15, five.amount);
}
```

하지만 처음 호출한 이후에 five 객체는 더이상 5가 아니라 10이 되어서 두번째 호출시 30이 된다.  
그렇다면 times에서 새로운 객체를 반환하게 만들면 어떠할까?  
그렇게 되면 어떠한 연산을 하더라도 원래 5달러의 값은 변하지 않게 될것이다.
```JAVA
@Test
public void testMultiplication(){
  Dollar five = new Dollar(5);
  Dollar product = five.times(2);
  assertEquals(10, product.amount);
  product = five.times(3);
  assertEquals(15, product.amount);
}
```
Dollar.times()를 아래와 같이 수정하기 전에는 새 테스트는 컴파일 조차 되지 않을 것이다.
```JAVA
public class Dollar{  
  public int amount;
  
  public Dollar(int amount){
    this.amount = amount;
  }
  
  public void times(int multiplier){
    amount = amount * multiplier;
    return null;
  }
}
```

이제 테스트가 컴파일된다. 하지만 실행되지는 않는다.  
테스트를 통과하기 위해서는 올바른 금액을 갖는 새 Dollar를 반환해야 한다.
```JAVA
public class Dollar{  
  public int amount;
  
  public Dollar(int amount){
    this.amount = amount;
  }
  
  public void times(int multiplier){
    return new Dollar(amount * multiplier);
  }
}
```
## 결론
### 테스트 목록
> $5 + 10CHF = $10(환율이 2:1 일 경우)  
> ~~$5 * 2 = 10$~~  
> amount를 private으로 만들기  
> ~~Dollar 부작용(side effect)?~~  
> Money 반올림?

이전 과정과 달리, 위 과정은 가짜 구현으로 시작해서 점차 실제 구현으로 바꿔나간 것이 아닌  
올바른 구현이라고 생각한 내용으로 바로 실제 구현 과정을 적용하였다.  
이와 같이, 최대한 빨리 초록 막대를 보기 위해 취할 수 있는 전략 중 두 가지는 아래와 같다.

 - 가짜로 구현하기 : 상수를 반환하게 만들고 진짜 코드를 얻을 때까지 단계적으로 상수를 변수로 일반화해 나간다.
 - 명백한 구현 사용하기 : 실제 구현을 입력한다.

여기서 켄트벡은 보통 실무에서 두 방법을 모두 사용한다고 설명한다.  
**명백하게 무엇을 구현해야 할 지를 알고 있다면 실제 구현을 입력**하고 테스트를 하고,  
만약 **예상치 못한 빨간 막대를 만나게 된다면 가짜 구현 하기 방법을 사용**하면서 올바른 코드로 리팩토링을 한다는 것이다.  
마지막 한 가지 전략은 다음장에서 보게 될 삼각측량이라는 방법이다.
