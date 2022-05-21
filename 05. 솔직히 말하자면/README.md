# 05. 솔직히 말하자면
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
 - 5CHF X 2 = 10CHF

## 진행
우선은 Dollar 객체와 비슷하지만 달러 대신 프랑(Fanc)을 표현할 수 있는 객체가 필요할 것 같다.  
만약 Dollar 객체와 비슷하게 작동하는 Franc이라는 객체를 만든다면 단위가 섞인 덧셈 테스트를 작성하고 돌려보는게 더 가까워질 것이다.

```JAVA
public void testFeancMultiplication() { // 달러 테스트 개조버전
	Franc five = new Franc(5);
	assertEquals(new Franc(10), five.times(2))
	assertEquals(new Franc(15), five.times(3))
}
```
> 복붙을 이용해서 재사용?! 

-> 우리의 주기를 다시 한번 생각해보자.
1. 테스트 작성
2. 컴파일되게 하기
3. 실패하는지 확인하기 위해 실행
4. 실행하게 만듦
5. 중복 제거

각 단계에는 다 다른 목적이 있으며 처음 네단계는 빠르게 진행해야한다.  
빠르게 진행하기 위해 어떤 죄든 저지를 수 있다. 그동안 만큼은 속도가 설계보다 더 높은 가치이기 떄문이다.  

```JAVA
class Franc {
	private int amount;

	Franc(int amount) {
		this.amount = amount;
	}

	Franc times(int multiplier) {
		return enw Franc(amount * multiplier);
	}

	public boolean equal(Object object) {
		Franc franc = (Franc) object;
		return amount == franc.amount;
	}
}
```

### 결론
코드를 실행시키기까지의 단계가 짧았기 때문에 '컴파일되게 하기' 단계도 넘어갈 수 있었다.  
중복이 많고, 이것들을 제거해야한다.  
우리는  
 - 큰 테스트를 공략할 수 없다. 그래서 진전을 나타낼 수 있는 자그마한 테스트를 만들었다. 
 - 뻔뻔스럽게도 중복을 만들고 조금 고쳐서 테스트를 작성했다.
 - 설상가상으로 모델 코드까지 도매금으로 복사하고 수정해서 테스트를 통과했다.
 - 중복이 사라지기 전에는 집에 가지 않겠다고 약속했다.

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





























