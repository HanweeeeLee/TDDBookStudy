# 11. 모든 악의 근원

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
 - **Dollar/Franc 중복**
 - ~~공용 equals~~
 - ~~공용 times~~
 - ~~Franc과 Dollar 비교하기~~
 - ~~통화~~
 - testFuncMultiplication 제거

## 진행
두 하위 클래스 Dollar와 Franc에는 달랑 생성자밖에 없다. 하위클래스를 제거하자.  

```JAVA
class Money {
	static Money dollar(int amount) {
		return new Money(amount, "USD");
	}
}
```
> 이제 Dollar에 대한 참조는 하나도 남아 있지 않으므로 Dollar를 지울 수 있다.

```JAVA
public void testDifferentClassEquality() {
	assertTrue(new Money(10, "CHF").equals(new Franc(10, "CHF")));
}
```
> 반면 Franc은 우리가 작성했던 테스트 코드에서 아직 참조한다. 

```JAVA
public void testEquality() {
	assertTrue(Money.dollar(5).equals(Money.dollar(5)));
	assertFalse(Money.dollar(5).equals(Money.dollar(6)));
	// assertTrue(Money.franc(5).equals(Money.franc(5))); // 3, 4번째 assertion(단언)은 1, 2번쨰 단언과 중복되니 지우자. 왜? 이제는 같은 클래스(Money)다.
	// assertFalse(Money.franc(5).equals(Money.franc(6)));
	assertFalse(Money.franc(5).equals(Money.dollar(5)));
}
```
> 충분한 테스트인 것 같으니 위의 testDifferentClassEquality를 지우자.

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
 - ~~**Dollar/Franc 중복**~~
 - ~~공용 equals~~
 - ~~공용 times~~
 - ~~Franc과 Dollar 비교하기~~
 - ~~통화~~
 - ~~**testFuncMultiplication 제거**~~

## 우리는
 - 하위 클래스의 속을 들어내는 걸 완료하고, 하위 클래스를 삭제했다.
 - 기존의 소스 구조에서는 필요했지만 새로운 구조에서는 필요 없게 된 테스트를 제거했다.





























