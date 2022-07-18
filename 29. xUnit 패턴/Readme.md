# 29. xUnit 패턴

## 단언(assertion)
### 테스트를 완전히 자동화하려면 결과를 평가하는 데 개입되는 인간의 판단을 모조리 끄집어내야 한다. 이것은 다음 사항을 의미한다.
 - 판단 결과가 불리언 값이어야 한다. 일반적으로 참 값은 모든 테스트가 통과했음을 의미하고, 거짓 값은 뭔가 예상치 못했던 일이 발생했음을 의미한다. 
 - 이 불리언 값은 컴퓨터에 의해 검증되어야 한다. 보통 다양한 형태의 assert() 메서드를 호출하여 이 값을 얻어낸다.
  
### 단언은 구체적이어야 한다. ㅁ만약 면적이 50이어야 하면 다음과 같이 써주자.
```JAVA
assertTrue(rectangle.area() == 50)
```
많은 xUnit 구현들이 동등성 검사를 위한 특별한 형태의 단언을 제공한다.  
### 객체를 블랙박스처럼 취급하기란 쉽지 않다. 
청약됨(Offered) 혹은 시행중(Running)이라는 상태를 가질 수 있는 계약에 대한 테스트를 작성하려한다.  
```Java
Contract contract = new Contract(); // 디폴트로 Offered status
contract.begin(); // Running으로 status 변경
assertEquals(Running.class, contract.status.class);
```
이 테스트는 status에 대한 현재 구현에 너무 의존적이다. status가 불리언 값으로 표현되도록 구현이 바뀌더라도 테스트가 통과할 수 있어야 한다.  
```JAVA
assertEquals(..., contract.startData()); // status가 Offered라면 예외
```
> 아마도 status 가 Running으로 바뀐다면 시행일이 언제인지 알아낼 수 있을 것이다.  
  
## 픽스처
여러 테스트에서 공통으로 사용하는 객체들을 생성할 떄 어떻게 하면 좋을까? 각 테스트 코드에 있는 지역 변수를 인스턴스 변수로 바꾸고 setUp()메서드를 재정의하여 이 메서드에서 인스턴스 변수들을 초기화하도록 한다.  
우리는 모델코드에서 중복을 제거하길 원한다.  
테스트코드에서도 마찬가지인데, 다음과 같은 이유이다.
 - 복사해서 붙이기를 한다고 하더라도 이런 코드를 반복 작성하는 것엔 시간이 소요되는데, 우리는 테스트를 빨리 작성하길 원한다.
 - 인터페이스를 수동으로 변경할 필요가 있을 경우, 여러 테스트를 고쳐주어야 한다(이는 중복에 대해 정확히 예상되는 결과이다.)  
  
중복되는 코드가 좋은점도 있는데, 그냥 읽기 쉬움..

```Java
class EmptyRectangleTest {
    public void testEmpty() {
        Rectangle empty = new Rectangle(0,0,0,0);
        assertTrue(empty.isEmpty());
    }

    public void testWidth() {
        Rectangle empty = new Rectangle(0,0,0,0);
        assertEquals(0.0, empty.getWidth(), 0.0);
    }
}

```
다음과 같이 중복을 제거할 수 있다.
```Java
class EmptyRectangleTest {
    private Rectangle empty;

    public void setUp() {
        empty = new Rectangle(0,0,0,0);
    }

    public void testEmpty() {
        assertTrue(empty.isEmpty());
    }

    public void testWidth() {
        assertEquals(0.0, empty.getWidth(), 0.0);
    }
}

```

## 외부 픽스처
픽스처 중 외부 자원이 있을 경우 이를 어떻게 해제할 것인가? tearDown()메서드를 재정의해서 해제하자.  
```Python
testMethod(self):
    file = File("foobar").open()
    try:
        # ...run the test...
    finally:
        file.close()
```
그런데 만약 파일이 여러 테스트에서 사용되었다면?
```Python
setUp(self)
    self.file = File("foobar").open()
testMethod(self):
    try:
        # ...run the test...
    finally:
        file.close()
```
file.close() 가 거슬린다.
```Python
setUp(self)
    self.file = File("foobar").open()
testMethod(self):
    try:
        # ...run the test...
tearDown(self):
    self.file.close()
```

## 테스트 메서드
테스트 케이스 하나를 어떻게 표현할 것인가? 'test'로 시작하는 이름의 메서드로 나타내자.  
객체지향 프로그래밍 언어는 세 가지 범주의 구조 계층을 제공한다.  
 - 모듈(자바의 패키지)
 - 클래스
 - 메서드
  
만약 테스트를 일반적인 소스코드처럼 작성하길 원한다면, 테스트를 위의 구조에 끼워 맞출 방법을 찾아내야 한다. (알아서 만들면됨)  
관습에 의해 메서드 이름은 'test'로 시작된다. 툴은 이 패턴을 자동으로 인식하고 주어진 클래스에 대한 테스트 슈트를 생성한다.  
메서드 이름의 나머지 부분은 코드를 읽었을 때 이 테스트가 왜 작성되었는지 알 수 있게 적어야한다.  

## 예외 테스트
예외가 발생하는 것이 정상인 경우에 대한 테스트는 어떻게 작성해야할까? 예상되는 예외를 잡아서 무시하고, 예외가 발생하지 않은 경우에 한해서 테스트가 실패하게 만들면 된다.  
```JAVA
public void testRate() {
    exchange.addRate("USD", "GBP", 2);
    int rate = exchange.findRate("USD", "GBP");
    assertEquals(2, rate);
}
```
예외가 제대로 발생하는지에 대한 테스트를 하려면?
```JAVA
public void testMissingRate() {
    try {
        exchange.findRate("USD", "GBP");
        fail();
    } catch (IllegalArgumentException expected) {

    }
}
```
> findRate()가 예외를 던지지 않는다면 fail()이 호출될 것이다. fail()은 테스트가 실패했음을 알려주기 위한 xUnit메서드이다.

## 전체 테스트
모든 테스트를 한번에 실행하려면 어떻게 해야 할까? 모든 테스트 슈트에 대한 모음을 작성하면 된다.(XCode는 지원 되는듯 ㅎㅎ)  
```Java
public class AllTests {
    public static void main(String[] args) {
        junit.swingui.TestRunner.run(AllTests.class);
    }

    public static Test suite() {
        TestSuite result = new TestSuite("TFD tests");
        result.addTestSuite(MoneyTest.class);
        result.addTestSuite(ExchangeTest.class);
        result.addTestSuite(IdentityRateTest.class);
        return result;
    }
}
```

