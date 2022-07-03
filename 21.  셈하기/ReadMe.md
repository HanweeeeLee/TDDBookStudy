# 21. 셈하기

## TODO List)
 - ~~테스트 메서드 호출하기~~
 - ~~먼저 setUp 호출하기~~
 - ~~나중에 tearDown 호출하기~~
 - 테스트 메서드가 실패하더라도 tearDown 호출하기
 - 여러 개의 테스트 실행하기
 - **수집된 결과를 출력하기**
 - ~~WasRun에 로그 문자열 남기기~~

## 내용
테스트 메소드에서 예외가 발생하건 말건 tearDown() 이 호출되도록 보장해 주는 기능을 구현하려고 했다.  
하지만 테스트가 작동하도록 하려면 예외를 잡아야 한다.  
일반적으로 테스트를 구현하는 순서는 중요하다.  
켄트 백은 다음에 구현할 테스트를 선택할 때, 뭔가 가르침을 줄 수 있고 내가 만들 수 있다는 확신이 드는 것을 선택한다고 한다.  
만약 테스트를 하나 성공시켰는데 그 다음 테스트를 만들며 문제가 생기면 두 단계 뒤로 물러서는 것을 고려한다.  
우리는 TestCase.run() 이 테스트 하나의 실행 결과를 기록하는 TestResult 객체를 반환하게 만들 것이다.

```Python
//TestCaseTest
def testResult(self):
  test = wasRun("testMethod")
  result = test.run()
  assert("1 run, 0 failed" == result.summary())
```

일단 가짜구현으로 시작하자.
```Python
//TestResult
class TestResult:
  def summary(self):
    return "1 run, 0 failed"
```

그리고 TestCase.run() 이 TestResult 를 결과로 반환하도록 하자.
```Python
//TestCase
def run(self):
  self.setUp();
  method = getattr(self, self.name)
  method()
  self.tearDown()
  return TestResult()
```

이제 테스트가 실행된다. 이제 summary() 의 구현을 조금씩 실체화를 해보자.  
우선 실행된 테스트의 수를 상수로 만들 수 있다.
```Python
//TestResult
def __init__(self):
  self.runCount = 1  
def summary(self):
  return "%d run, 0 failed" % self.runCount
```

하지만 runCount 가 상수일 수는 없다. 이 수치는 실제로 실행된 테스트 수를 세서 계산되어야 한다.  
runCount 를 0으로 초기화 하고 테스트가 실행될 때마다 1씩 증가하게 만들자.
```Python
//TestResult
def __init__(self):
  self.runCount = 0  
def summary(self):
  return "%d run, 0 failed" % self.runCount
def testStarted(self):
  self.runCount = self.runCount + 1
```

이제 이 메소드를 실제로 호출하게 만들어야 한다.
```Python
//TestCase
def run(self):
  result = TestResult()
  result.testStarted()
  self.setUp();
  method = getattr(self, self.name)
  method()
  self.tearDown()
  return result
```

실패하는 테스트의 수를 나타내는 문자열 상수 0을 runCount 처럼 변수로 만들어줘야 하지만  
아직 실패하는 테스트가 존재하지 않는다.  
그러니 또 다른 테스트를 하나 작성해야 한다.
```Python
//TestCaseTest
def testFailedResult(self):
  test = WasRun("testBrokendMethod")
  result = test.run()
  assert("1 run, 1 failed" == run.summary())
```

testBrokendMethod() 는 다음과 같다.
```Python
//WasRun
def testBrokenMethod(self):
  raise Exception 
```

### TODO 추가
 - ~~테스트 메서드 호출하기~~
 - ~~먼저 setUp 호출하기~~
 - ~~나중에 tearDown 호출하기~~
 - 테스트 메서드가 실패하더라도 tearDown 호출하기
 - 여러 개의 테스트 실행하기
 - ~~**수집된 결과를 출력하기**~~
 - ~~WasRun에 로그 문자열 남기기~~
 - **실패한 테스트 보고하기(추가)**
 
 우리가 관심 가져야 할 사항은 WasRun.testBrokenMethod 에서 던진 예외를 처리하지 않았다는 점이다.  
 예외를 잡고 테스트가 실패했음을 테스트 결과에 적어 넣고 싶겠지만 잠시 할일 목록에 남겨두자.
 
 ## 우리는
  - 가짜 구현을 한 뒤에 단계적으로 상수를 변수로 바꾸어 실제 구현으로 만들었다.
  - 또 다른 테스트를 작성했다.
  - 테스트가 실패했을 때 좀더 작은 스케일을 또 다른 테스트를 만들어서 실패한 테스트가 성공하게 만드는 것을 보조할 수 있었다.
