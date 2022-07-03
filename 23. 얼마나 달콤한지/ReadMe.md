# 23. 얼마나 달콤한지

## TODO List)
 - ~~테스트 메서드 호출하기~~
 - ~~먼저 setUp 호출하기~~
 - ~~나중에 tearDown 호출하기~~
 - 테스트 메서드가 실패하더라도 tearDown 호출하기
 - **여러 개의 테스트 실행하기**
 - ~~수집된 결과를 출력하기~~
 - ~~WasRun에 로그 문자열 남기기~~
 - ~~**실패한 테스트 보고하기**~~
 - setUp 에러를 잡아서 보고하기
 
## 내용
파일의 끝부분에는 모든 테스트들을 호출하는 코드가 있는데, 좀 비참해 보인다.
```Python
print TestCaseTest("testTemplateMethod").run().summary()
print TestCaseTest("testResult").run().summary()
print TestCaseTest("testFailedResultFormatting").run().summary()
print TestCaseTest("testFailedResult").run().summary()
```

놓친 디자인 요소를 찾기위해 일부러 만드는 중복이 아니라면, 중복은 언제나 나쁘다.  
여기서 우리는 테스트들을 모아서 한번에 실행할 수 있는 기능을 원한다.  
TestSuite를 만들고 거기에 테스트를 몇개 넣은 다음 이것을 모두 실행하고 결과를 얻어내고 싶다.
```Python
//TestCaseTest
def testSuite(self):
  suite = TestSuite()
  suite.add(WasRun("testMethod"))
  suite.add(WasRun("testBrokenMethod"))
  result = suite.run()
  assert("2 run, 1 failed" == result.summary())
```

add() 메소드를 구현하는 것은 그냥 테스트들을 리스트에 추가하는 작업으로 끝난다.
```Python
//TestSuite
class TestSuite:
  def __init__(self):
    self.tests = []    //[]는 빈 컬렉션을 생성한다.    
  def add(self, test)
  	self.tests.append(test)
```

run() 메소드는 약간 어렵다. 우린 하나의 TestResult 가 모든 테스트에 대해 쓰이길 바라기 때문에 다음과 같이 작성해야 한다.
```Python
//TestSuite
def run(self):
  result = TestResult()
  for test in self.tests:
    test.run(result)
  return result 
```

컴포지트의 주요 제약 중 하나는 컬렉션이 하나의 개별 아이템인 것처럼 반응해야 한다는 것이다.  
TestCase.run()에 매개변수를 추가하게 되면 TestSuite.run()에도 같은 매개변수를 추가해야 한다.  
이를 해결하기 위한 세가지 대안이 있다.
 - 파이썬의 기본 매개 변수 기능을 사용한다. 불행히 기본값은 컴파일타임에 평가되므로 하나의 TestResult를 재사용할 수 없게 된다.
 - 메서드를 두 부분으로 나눈다. 하나는 TestResult를 할당하는 부분, 할당된 TestResult를 가지고 테스트를 수행하는 부분. 그런데 이 두 부분에 대한 좋은 이름이 떠오르질 않는다. 이는 이렇게 나누는것이 그리 좋은 전략이 아니라는것을 뜻한다.
 - 호출하는 곳에서 TestResults를 할당한다.
 
호출하는 곳에서 TestResults 를 할당하는 전략을 이용하자.  
이 패턴은 매개 변수 수집(collecting parameter)이라 부른다. 
```Python
//TestCaseTest
def testSuite(self):
  suite = TestSuite()
  suite.add(WasRun("testMethod"))
  suite.add(WasRun("testBrokenMethod"))
  result = TestResult()
  suite.run(result)
  assert("2 run, 1 failed" == result.summary())
```

이 해법은 run() 이 명시적으로 반환하지 않아도 된다는 추가적인 장점이 있다.
 ```Python
//testSuite
def run(self, result):
  for test in self.tests:
    test.run(result)
    
//testCase
def run(self, result):
  result.testStarted()
  self.setUp()
  try:
    method = getattr(self, self.name)
    method()
  except:
    result.testFailed()
  self.tearDown()
```

이제 파일 뒷부분에 있는 테스트 호출 코드를 정리할수 있다.
```Python
suite= TestSuite()
suite.add(TestCaseTest("testTemplateMethod"))
suite.add(TestCaseTest("testResult"))
suite.add(TestCaseTest("testFailedResultFormatting"))
suite.add(TestCaseTest("testFailedResult"))
suite.add(TestCaseTest("testSuite"))
result = TestResult()
suite.run(result)
print result.summary()
```

### TODO 추가
 - ~~테스트 메서드 호출하기~~
 - ~~먼저 setUp 호출하기~~
 - ~~나중에 tearDown 호출하기~~
 - 테스트 메서드가 실패하더라도 tearDown 호출하기
 - **여러 개의 테스트 실행하기**
 - ~~수집된 결과를 출력하기~~
 - ~~WasRun에 로그 문자열 남기기~~
 - ~~실패한 테스트 보고하기~~
 - setUp 에러를 잡아서 보고하기
 - **TestCase 클래스에서 TestSuite 생성하기(추가)**

중복이 상당히 많다. 주어진 테스트 클래스에 대한 테스트 슈트를 자동 생성할 방법이 있다면 그 중복을 제거할 수 있을 것이다.  
우선 실패하는 테스트 세개를 고쳐야 한다.
```Python
//TestCaseTest
def testTemplateMethod(self):
    test= WasRun("testMethod")
    result= TestResult()
    test.run(result)
    assert("setUp testMethod tearDown " == test.log)
def testResult(self):
    test= WasRun("testMethod")
    result= TestResult()
    test.run(result)
    assert("1 run, 0 failed" == result.summary())
def testFailedResult(self):
    test= WasRun("testBrokenMethod")
    result= TestResult()
    test.run(result)
    assert("1 run, 1 failed" == result.summary())
def testFailedResultFormatting(self): 
    result= TestResult() 
    result.testStarted() 
    result.testFailed()
    assert("1 run, 1 failed" == result.summary())
```

각 테스트들이 TestResult를 할당하고 있음을 주목하자.  
이것은 정확히 setUp()으로 해결할 수 있는 문제다. TestResult를 setUp()에서 생성하게 만들면 테스트를 단순화할 수 있다.
```Python
//TestCaseTest
def setUp(self):
    self.result= TestResult()
def testTemplateMethod(self):
    test= WasRun("testMethod")
    test.run(self.result)
    assert("setUp testMethod tearDown " == test.log)
def testResult(self):
    test= WasRun("testMethod")
    test.run(self.result)
    assert("1 run, 0 failed" == self.result.summary())
def testFailedResult(self):
    test= WasRun("testBrokenMethod") 
    test.run(self.result)
    assert("1 run, 1 failed" == self.result.summary())
def testFailedResultFormatting(self): 
    self.result.testStarted()
    self.result.testFailed()
    assert("1 run, 1 failed" == self.result.summary())
def testSuite(self):
    suite= TestSuite() 
    suite.add(WasRun("testMethod")) 
    suite.add(WasRun("testBrokenMethod")) 
    suite.run(self.result)
    assert("2 run, 1 failed" == self.result.summary())
```
파이썬이 객체지원이 추가된 스크립트 언어이다보니 전역참조가 암묵적으로 이루어지고 self에 대한 참조를 명시적으로 적어야 한다.

### TODO
 - ~~테스트 메서드 호출하기~~
 - ~~먼저 setUp 호출하기~~
 - ~~나중에 tearDown 호출하기~~
 - 테스트 메서드가 실패하더라도 tearDown 호출하기
 - ~~**여러 개의 테스트 실행하기**~~
 - ~~수집된 결과를 출력하기~~
 - ~~WasRun에 로그 문자열 남기기~~
 - ~~실패한 테스트 보고하기~~
 - setUp 에러를 잡아서 보고하기
 - TestCase 클래스에서 TestSuite 생성하기

이 목록의 나머지 항목은 새로 배운 TDD기술을 이용해 직접해볼수 있도록 남겨두자.

## 우리는
 - TestSuite 를 위한 테스트를 작성했다.
 - 테스트를 통과시키지 못한 채 일부분만 구현 하였다. 테스트를 통과 시키고 리팩토링할 가짜구현이 떠오르지 않는다.
 - 아이템과 아이템의 모음(컴포지트)이 동일하게 작동할 수 있도록 run메서드의 인터페이스를 변경하였고 테스트를 통과 시켰다.
 - 공통된 셋업 코드를 분리했다.
