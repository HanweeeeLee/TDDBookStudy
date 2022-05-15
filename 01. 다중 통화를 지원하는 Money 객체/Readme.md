# 1장. 다중 통화를 지원하는 Money 객체
주식 거래소를 운영하고 있다고 가정한다. 현재는 USD만 지원하지만 Franc도 지원하게 수정하고 싶다. 앞으로의 과정들은 이것을 설명한다.  
목표는
> 통화가 다른 두 금액을 더해서 주어진 환율에 맞게 변한 금액을 결과로 얻을 수 있어야 한다.
> 어떤 금액을 어떤 수에 곱한 금액을 결과로 얻을 수 있어야 한다.
이를 목록으로 정리해보자면
### 테스트 목록
> $5 + 10CHF = $10(환율이 2:1 인 경우) -> 통화가 다른 두 금액을 더해서 주어진 환율에 맞게 변한 금액을 결과로 얻을 수 있어야 한다.
> $5 * 2 = $10 -> 어떤 금액(주가)을 어떤 수(주식의 수)에 곱한 금액을 결과로 얻을 수 있어야 한다.

이 중 곱셈에 관한 두번째 예를 테스트 해보자.

## 간단한 곱셈의 예
```JAVA
@Test
public void testMultiplication(){
  Dollar five = new Dollar(5);
  five.times(2);
  assertEquals(10, five.amount);
}
```
위 테스트는 예기치 못한 부작용을 고려하지 않았고, 금액 계산도 정수형으로 하였다. 
하지만, 우선 이러한 문제 사항들은 목록에 추가하고, 작은 단계로 시작해보는 것이 중요하다.  
### 테스트 목록
> $5 + 10CHF = $10(환율이 2:1 일 경우)
> $5 * 2 = 10$ 
> amount를 private으로 만들기
> Dollar 부작용(side effect)?
> Money 반올림?

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

현재 우리는 주기의 1번부터 4번까지의 항목을 수행하였다.  
이제 중복을 제거할 차례이다. 하지만 어디에 중복이 존재한단 말인가?  
이번 경우에는 코드가 아닌 데이터의 중복이 있는 것을 확인할 수 있다.  
```JAVA
public class Dollar{  
  public int amount = 5 * 2;    //중복된 데이터
  
  public Dollar(int amount){
  }
  
  public void times(int multiplier){
  }
}
```
10이었던 amount 필드를 5 * 2 로 나누어 생각해보면, 테스트에서의 5, 2와 데이터가 중복되는 것을 확인할 수 있다.
하지만 5와 2 를 한번에 제거할 수 있는 방법은 없으니, amount 초기화 단계를 times() 메서드 안으로 옮겨 보자. 
```JAVA
public class Dollar{  
  public int amount;
  
  public Dollar(int amount){
  }
  
  public void times(int multiplier){
    amount = 5 * 2;
  }
}
```
테스트는 여전히 통과된다.  
하지만, 아직도 중복 데이터는 여전히 존재한다. 다음으로 5라는 값을 제거할 수 있을까??  
이건 생성자를 통해서 넘어오는 값이기 때문에 amount 초기화 단계를 변경해볼 수 있을 것이다.
```JAVA
public class Dollar{  
  public int amount;
  
  public Dollar(int amount){
    this.amount = amount;
  }
  
  public void times(int multiplier){
    amount = amount * 2;
  }
}
```
테스트에서 times()의 인자 multiplier 의 값이 2이므로 이것도 중복이며, 상수를 multiplier 로 일반화할 수 있을 것이다.
```JAVA
public class Dollar{  
  public int amount;
  
  public Dollar(int amount){
    this.amount = amount;
  }
  
  public void times(int multiplier){
    amount = amount * multiplier;
  }
}
```
times 메서드 안에서의 amount 가 변수도 중복이므로 자바 *= 문법을 통해 간결하게 수정할 수 있다.
```JAVA
public class Dollar{  
  public int amount;
  
  public Dollar(int amount){
    this.amount = amount;
  }
  
  public void times(int multiplier){
    amount *= multiplier;
  }
}
```

## 결론
