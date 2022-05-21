# 06. 돌아온 '모두를 위한 평등' 
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


## 진행
이제는 중복코드를 지우는 작업을 할것임.  
가능한 방법 한 가지는 우리가 만든 클래스 중 하나가 다른 클래스를 상속받게 하는 것이다.
![KakaoTalk_Photo_2022-05-21-21-20-43](https://user-images.githubusercontent.com/60125719/169651321-c734a9f2-212d-431a-8faf-ffc07388df6e.jpeg)

Money클래스가 공통의 equals를 갖게 하는게 나으려나..?

### Money클래스 생성, 공통된 코드를 상위 클래스로 옮기기

```JAVA
class Money {

}

class Dollar extends Money {
	private int amount;
}
```
> 여전히 테스트는 잘 돌아간다. 이제 amount 인스턴스 변수를 Money로 옮길 수 있다.

```JAVA
class Money {
	protected int amount;
}

class Dollar extends Money {
	// private int amount;
}
```
> 이제 equals()코드를 위로 올리는 일을 할 수 있게됐다

```JAVA
class Dollar extends Money {
	public boolean equals(Object object) {
		Money dollar = (Dollar) object;
		return amount == dollar.amount;
	}
}
```
> 모든 테스트가 여전히 잘 돈다. 이제 캐스트(cast) 부분을 변경하자.

```JAVA
class Dollar extends Money {
	public boolean equals(Object object) {
		// Money dollar = (Dollar) object;
		Money dollar = (Money) object;
		return amount == dollar.amount;
	}
}
```

좀 더 원활한 의사소통을 위해 임시 변수의 이름을 변경하자.

```JAVA
class Dollar extends Money {
	public boolean equals(Object object) {
		// Money dollar = (Money) object;
		Money money = (Money) object;
		// return amount == dollar.amount;
		return amount == money.amount;
	}
}
```
> 이제 이 메서드를 Dollar에서 Money로 옮길 수 있다.

```JAVA
class Money {
	protected int amount;

	public boolean equals(Object object) {
		Money money = (Money) object;
		return amount == money.amount;
	}
}

class Dollar extends Money {
	// public boolean equals(Object object) {
	// 	Money money = (Money) object;
	// 	return amount == money.amount;
	// }
}
```

### Franc.equals() 제거하기
동치성 테스트가 Franc끼리의 비교에 대해서는 다루지 않는다는 점에 주목하자. 코드를 복사했던 과거의 죄가 우리 발목을 붙잡고 있다.  
우리는 코드를 변경하기 전에 애초에 그곳에 있어야 했던 테스트를 작성할 것이다.  
  

충분한 테스트가 없다면 리팩토링을 하면서 실수를 했어도 테스트가 성공하는 일이 발생한다.  
있으면 좋을 것 같은 테스트를 작성하라.  
테스트를 안만듬 -> 리팩토링이 안좋은줄 알게됨 -> 코드 질 떨어짐 -> 해고 -> 굶음 -> 이 빠짐 ->-> 이 건강을 위해 테스트를 작성하자.
  
다행이도 이번 테스트는 작성하기 쉽다. 그냥 Dollar테스트를 복사하자
```JAVA
public void testEquality() {
	assertTrue(new Dollar(5).equals(new Dollar(5)));
	assertFalse(new Dollar(5).equals(new Dollar(6)));
	assertTrue(new Franc(5).equals(new Franc(5)));
	assertFalse(new Franc(5).equals(new Franc(6)));
}
```
> 또 중복이다!

```JAVA
class Franc extends Money {
	// private int amount;
}
```
> Money 클래스에 있는 필드를 이용하면 Franc의 amount 필드를 제거할 수 있다.
  
Franc.equals()는 Money.equals()와 거의 비슷해 보인다.  
이 부분을 완전히 똑같이 만들 수 있다면 프로그램의 의미를 변화시키지 않고도 Franc의 equals()를 지워버릴 수 있게 된다.

```JAVA
class Franc extends Money {
	public boolean equals(Object object) {
		// Franc franc = (Franc) object;
		Money money = (Money) object;
		// return amount == franc.amount;
		return amount == money.amount;
	}
}
```
이제 Franc.equals()와 Money.equals() 사이에 다른 점이 없으므로 Franc의 불필요한 코드를 제거하자. 그리고 테스트를 돌려보자. 잘 돌아간다.  

## 결론
우리는
 - 공통된 코드를 첫 번째 클래스(Dollar)에서 상위 클래스(Money)로 단계적으로 옮겼다.
 - 두 번째 클래스(Franc)도 Money의 하위 클래스로 만들었다.
 - 불필요한 구현을 제거하기 전에 두 equals()구현을 일치시켰다.


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
 - **~~공용 equals~~**
 - 공용 times
 - **Franc과 Dollar 비교하기**



















