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
오류는 모두 잡혔고 테스트를 실행해보면 에러가 발생한다.  
![KakaoTalk_20220515_162701412](https://user-images.githubusercontent.com/50142323/168462077-697fef1e-07e9-4b71-bb2b-66628d670659.jpg)
우리는 10을 바랐지만 값은 0으로 나왔다. 우선 테스트를 통과하기 위해서 아래와 같이 수정한 후 다시 실행해보자.
```JAVA
public class Dollar{  
  public int amount = 10; //수정
  
  public Dollar(int amount){
  }
  
  public void times(int multiplier){
  }
}
```
![KakaoTalk_20220515_162701412_01](https://user-images.githubusercontent.com/50142323/168462157-58ecd7df-c21f-466f-91e3-a750690a2127.jpg)
테스트를 통과한다!  
진행하기 전에 테스트 주기에서의 현재 자신의 단계를 생각해보자.

### 단위 테스트 주기 
1. 작은 테스트를 하나 추가한다.
2. 모든 테스트를 실행해서 테스트가 실패하는것을 확인한다.
3. 조금 수정한다.
4. 모든 테스트를 실행해서 테스트가 성공하는 것을 확인한다.
5. 중복을 제거하기 위해 리팩토링을 한다.

### 의존성과 중복
> 테스트를 작성하게 되면, 자연스럽게 테스트와 코드 사이에 의존성이 발생하게 된다. 
> 즉, 코드나 테스트 중 한쪽을 수정하면 반드시 다른 한쪽도 수정해야만 한다는 것이다.
> 의존성이 문제 그 자체라면 중복은 문제의 징후이다. 
> 중복의 가장 흔한 예는 로직의 중복이며, 중복된 로직을 제거하는 가장 좋은 방법은 객체를 이용하는 것이다.
> 프로그램에서는 중복만 제거해주면 의존성도 제거된다. 
> 이게 바로 TDD의 두 번째 규칙이 존재하는 이유이며, 다음 테스트로 진행하기 전에 중복을 제거하여, 
> 오직 한 가지의 코드 수정을 통해 다음 테스트도 통과되게 만들 가능성을 최대화하는 것이다.
