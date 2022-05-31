# 07. 사과와 오렌지
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
 - Dollar/Franc 중복
 - 공용 equals
 - 공용 times
 - **Franc과 Dollar 비교하기**

지난장에서 상속을 이용해서 Money를 상속받는 Dollar 클래스와 Franc 클래스를 만들었다.  
그리고 Dollar와 Franc에 중복되는 코드 equals() 를 Money로 옮겼다. 하지만 여기서 문제가 발생한다.  
본질적으로 Dollar나 Franc나 같은 클래스를 상속받았지만 이 둘은 엄연히 서로 다른 클래스이다.  
결국 Money에서 상속받는 equals()에는 단지 amount의 필드 값만 보고 비교를 하기 때문에 오류가 발생한다.  
그래서 이번에는 equals()를 바꿔보도록하자.

## 실패하는 코드 작성
```JAVA
@Test
void testEquality() {
  assertTrue(new Dollar(5).equals(new Dollar(5)));
  assertFalse(new Dollar(5).equals(new Dollar(6)));
  assertTrue(new Franc(5).equals(new Franc(5)));
  assertFalse(new Franc(5).equals(new Franc(6)));   
  assertFalse(new Franc(5).equals(new Dollar(5)));
}

class Money {
  protected int amount;

  public boolean equals(Object object) {
    Money money = (Money) object;
    return amount == money.amount;
  }
}
```
> 당연히 실패하게 된다. 우리의 목적은 금액도 같고, 클래스도 같을 때 true 반환하는 것이기 때문에 리팩토링해보자.

```JAVA
public boolean equals(Object object) {
  Money money = (Money) object;
  return amount == money.amount                   // 금액이 같고
          && getClass().equals(object.getClass());  // 클래스도 같을 때
}
```

모델코드에서 클래스를 이런식으로 사용하는 것은 지저분하지만, 현재로서는 통화(currency)의 개념도 없고  
통화개념을 도입할 충분한 이유가 없으므로 일단 이렇게 넘어가도록 하자.

## 결론
 - 우리를 괴롭히던 결함을 끄집어내서 테스트에 담아냈다.
 - 완벽하진 않지만 그럭저럭 봐줄만한 방법(getClass())으로 테스트를 통과하게 만들었다.
 - 더 많은 동기가 있을때까지는 더 많은 설계를 도입하지 않기로 했다.


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
 - Dollar/Franc 중복
 - ~~공용 equals~~
 - 공용 times
 - **~~Franc과 Dollar 비교하기~~**
 - 통화?
