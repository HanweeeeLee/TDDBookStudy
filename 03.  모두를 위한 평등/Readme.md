# 3장. 모두를 위한 평등
이전장에서 Dollar의 부수작용을 값 객체를 사용하여 해결하였다.  
값 객체에 대한 제약사항중 하나는 객체의 인스턴스 변수가 생성자를 통해서 생성된 후에는 결코 변하지 않는다는 것이다.(불변성)  
값 객체가 암시하는 것 중 하나는 2장에서와 같이 모든 연산은 새 객체를 반환해야 한다는 것이며,  
이를 통해 알수있는 또다른 것은 값 객체는 equals() 를 구현해야한다는 것을 암시하는데, 예를들어 $5 라는 것은 항상 다른 $5 와 똑같은 것이기 때문이다. (동일성)  
따라서 equals() 오퍼레이션에 대한 테스트를 진행해보자.

### 테스트 목록
> $5 + 10CHF = $10(환율이 2:1 일 경우)  
> ~~$5 * 2 = 10$~~  
> amount를 private으로 만들기  
> ~~Dollar 부작용(side effect)?~~  
> Money 반올림?  
> **equals()**  
> hashCode()

만약 Dollar를 해시 테이블의 키로 쓸 생각이라면 equals()를 구현할 때에 hashCode()를 같이 구현해야 하므로,  
테스트 목록에 적어 놓고 이것때문에 문제가 생기면 그때 다시 다루도록 하자.

```JAVA
@Test
void testEquality() {
  assertTrue(new Dollar(5).equals(new Dollar(5)));    
}
```

컴파일은 성공하지만, 테스트에 실패하게 된다. 빠르게 초록 막대를 보기 위해 가짜 구현을 해보자.
```JAVA
public class Dollar{  
  int amount;
  
  public Dollar(int amount){
    this.amount = amount;
  }
  
  public void times(int multiplier){
    amount = amount * multiplier;
    return null;
  }
  
  public boolean equals(Object object) {
    return true;    //단순히 true를 반환하게 구현
  }
}
```

사실 true 값은 다음과 같은 단계를 밟아 일반화할 수 있다.
> 5 == 5  
> amount == 5  
> amount == dollar.amount

이러한 명확한 케이스의 경우에는 위와 같이 쉽게 일반화할 수 있으나, 그렇지 않은 경우에는 켄트벡은 **삼각 측량**전략을 제시해준다.

## 삼각 측량
두 개 이상의 예제를 통해서 코드를 일반화해 나가는 것이며, 새로 추가된 두번째 예제가 일반적인 해를 필요로 할때만 비로소 일반화한다.  
위의 테스트에서 삼각측량을 하기 위해 두번째 예제가 필요하기에 $5 != $6 을 해보는게 어떨까?
```JAVA
@Test
void testEquality() {
  assertTrue(new Dollar(5).equals(new Dollar(5)));
  assertFalse(new Dollar(5).equals(new Dollar(6)));
}
```

테스트는 당연히 실패할 것이고, 이제 동치성(equality)을 일반화해야한다.
```JAVA
public class Dollar{  
  int amount;
  
  public Dollar(int amount){
    this.amount = amount;
  }
  
  public void times(int multiplier){
    amount = amount * multiplier;
    return null;
  }
  
  public boolean equals(Object object) {
    Dollar dollar = (Dollar) object;
    return amount == dollar.amount;
  }
}
```
또한, 켄트벡은 어떻게 리팩토링 해야 할지 전혀 감이 오질 않을 때만 삼각측량 전략을 사용한다고 말한다.  
굳이 삼각측량 방법만이 아닌, 코드와 테스트의 중복을 제거하고 일반적인 해법이 존재할 경우에는 그 방법을 구현하면 된다는 것이다.

## 결론
### 테스트 목록
> $5 + 10CHF = $10(환율이 2:1 일 경우)  
> ~~$5 * 2 = 10$~~  
> amount를 private으로 만들기  
> ~~Dollar 부작용(side effect)?~~  
> Money 반올림?
> ~~equals()~~
> hashCode()
> Equal null
> Equal object

지금까지 동일성 문제는 일시적으로 해결됐다.  
하지만, 현재 프로그램이 어떤 변화 가능성을 지원해야 하는가? 몇몇 부분을 변경시켜보면 답이 명확해질 것이다.  
만약 널값이나 다른 객체들과 비교한다면 어떻게 될까?  
이런 상황은 일반적이긴 하지만, 지금 당장은 필요하지 않으므로 테스트 목록에 추가만 해두자.
