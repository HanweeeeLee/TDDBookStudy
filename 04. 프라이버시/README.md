# 04. 프라이버시

### TODO
 - $5 + 10CHF = $10(환율이 2:1일 경우)
 - ~~$5 X 2 = $10~~
 - **amount를 private으로 만들기**
 - ~~Dollar 부작용?~~
 - Money 반올림?
 - ~~equals()~~
 - hasCode()
 - Equal null
 - Equal Object

## 진행
개념적으로 Dollar.times()연산은 호출을 받은 객체의 값에 인자로 받은 곱수만큼 곱한 값을 갖는 Dollar를 반환해야한다. 하지만 테스트가 그렇지 못하다.  
```JAVA
public void testMultiplication() {
	Dollar five = new Dollar(5);
	Dollar product = five.times(2);
	assertEquals(10, product.amount);
	product = five.times(3);
	assertEquals(15, product.amount);
}
```
첫 번째 단언(assertion)을 Dollar와 Dollar를 비교하는 것으로 재작성할 수 있다.
```JAVA
public void testMultiplication() {
	Dollar five = new Dollar(5);
	Dollar product = five.times(2);
	// assertEquals(10, product.amount);
	assertEquals(new Dollar(10, product.amount);
	product = five.times(3);
	assertEquals(15, product.amount);
}
```
이게 더 좋아보이므로 두 번쨰 단언도 고친다.
```JAVA
public void testMultiplication() {
	Dollar five = new Dollar(5);
	Dollar product = five.times(2);
	assertEquals(new Dollar(10), product.amount);
	product = five.times(3);
	// assertEquals(15, product.amount);
	assertEquals(new Dollar(15), product.amount);
}
```
이제 임시 변수인 product는 더 이상 쓸모없어 보인다. 인라인시키자.
```JAVA
public void testMultiplication() {
	Dollar five = new Dollar(5);
	// Dollar product = five.times(2);
	// assertEquals(new Dollar(10), product.amount);
	assertEquals(new Dollar(10), five.times(2));
	// product = five.times(3);
	// assertEquals(new Dollar(15), product.amount);
	assertEquals(new Dollar(15), five.times(3));
}
```
테스트를 고치고 나니 이제 Dollar의 amount 인스턴스 변수를 사용하는 코드는 Dollar 자신밖에 없게 됐다. 따라서 변수를 private으로 변경할 수 있다.
```JAVA
class Dollar {
	// int amount;
	private int amount;
}
```
### TODO
#### amount를 private으로 만들기 완료
 - $5 + 10CHF = $10(환율이 2:1일 경우)
 - ~~$5 X 2 = $10~~
 - ~~amount를 private으로 만들기~~
 - ~~Dollar 부작용?~~
 - Money 반올림?
 - ~~equals()~~
 - hasCode()
 - Equal null
 - Equal Object

할일을 하나 지웠지만 만약 동치성 테스트가 동치성에 대한 코드가 정확히 작동한다는 것을 검증실패하면 곱하기 테스트 역시 검증실패하게 된다.  
TDD를 하면서 관리해야 할 위험요소. 하지만 우리는 완벽함을 위해 노력하지 않는다.  
테스트를 수정하고 초록색으로 만들자!

## 지금까지 배운 것을 검토해보면, 우리는
 - 오직 테스트를 향상시키기 위해서만 개발된 기능을 사용했다.
 - 두 테스트가 동시에 실패하면 망한다는 점을 인식했다.
 - 위험 요소가 있음에도 계속 진행했다.
 - 테스트와 코드 사이의 결합도를 낮추기 위해, 테스트하는 객체의 새 기능을 사용했다.
