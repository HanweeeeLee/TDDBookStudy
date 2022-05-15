# 1장. 다중 통화를 지원하는 Money 객체
주식 거래소를 운영하고 있다고 가정한다. 현재는 USD만 지원하지만 Franc도 지원하게 수정하고 싶다. 앞으로의 과정들은 이것을 설명한다.  
목표는
> 통화가 다른 두 금액을 더해서 주어진 환율에 맞게 변한 금액을 결과로 얻을 수 있어야 한다.
> 어떤 금액을 어떤 수에 곱한 금액을 결과로 얻을 수 있어야 한다.
이에 대한 예제를 보자면
> $5 + 10CHF = $10(환율이 2:1 인 경우) -> 통화가 다른 두 금액을 더해서 주어진 환율에 맞게 변한 금액을 결과로 얻을 수 있어야 한다.
> $5 * 2 = $10 -> 어떤 금액(주가)을 어떤 수(주식의 수)에 곱한 금액을 결과로 얻을 수 있어야 한다.

이 중 곱셈에 관한 두번째 예를 테스트 해보자.

## 간단한 곱셈의 예
```JAVA
public void testMultiplication(){
  Dollar five = new Dollar(5);
  five.times(2);
  assertEquals(10, five.amount);
}
```
작성한 위 테스트는 아직 컴파일조차 되지 않는다. 현재 상태에서는 네개의 컴파일 에러가 있는데, 살펴보자면
 - Dollar 클래스가 없음
 - 생성자가 없음
 - times(int) 메서드가 없음
 - amount 필드가 없음

하나씩 에러를 제거해 보자.  
일단 Dollar 클래스를 정의하면 에러 하나는 없앨 수 있다.
```JAVA
public class Dollar{
}
```

하나의 에러가 사라졌다. 이제 생성자를 만들어보자. 그냥 컴파일만 넘어갈수있게 생성자 안에서는 아무것도 하지 않아도 된다.
```JAVA
public class Dollar{  
  public Dollar(int amount){
  }
}
```

이제 times()의 구현이 필요하다. 마찬가지로 컴파일만 될수있게 최소한의 구현만 한다.
```JAVA
public class Dollar{  
  public Dollar(int amount){
  }
  
  public void times(int multiplier){
  }
}
```

마지막으로 amount필드를 추가해주자
```JAVA
public class Dollar{  
  public int amount;
  
  public Dollar(int amount){
  }
  
  public void times(int multiplier){
  }
}
```
