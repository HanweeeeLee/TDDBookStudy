# 22. 실패 처리하기

## TODO List)
 - ~~테스트 메서드 호출하기~~
 - ~~먼저 setUp 호출하기~~
 - ~~나중에 tearDown 호출하기~~
 - 테스트 메서드가 실패하더라도 tearDown 호출하기
 - 여러 개의 테스트 실행하기
 - ~~수집된 결과를 출력하기~~
 - ~~WasRun에 로그 문자열 남기기~~
 - **실패한 테스트 보고하기**
 
## 내용
우리는 실패한 테스트를 발견하면 좀더 세밀한 단위의 테스트를 작성해서 올바른 결과를 출력하는 걸 확인할것이다.
```Python
//TestCaseTest
def testFailedResultFormatting(self):
  result = TestResult()
  result.testStarted()
  result.testFailed()
  assert("1 run, 1 failed" == result.summary())
```

testStarted() 와 testFailed() 는 각각 테스트가 시작할 때와 테스트가 실패할 때 보낼 메시지다.  
만약 이 메시지들이 위와 같은 순서로 호출되었을 때 summary 문자열이 제대로 출력된다면 우리의 프로그래밍 문제는 어떻게 이 메시지들을 보낼 것인가로 좁혀진다.  
일단 메시지를 보내기만 하면 전체가 제대로 동작 할 거라는 걸 예상할 수 있다. 구현은 실패 횟수를 세는 것이다.
```Python
//TestResult
def __init__(self):
  self.runCount = 0
  self.failureCount = 0  
def testFailed(self):
  self.failureCount = self.failureCount + 1
```

횟수가 맞다면 우리는 제대로 출력할 수 있다.
```Python
//TestResult
def summary(self):
  return "%d run, %d failed" % (self.runCount, self.failureCount)
```

이제 testFailed() 만 제대로 호출하면 예상되는 답을 얻을 수 있다.  
이걸 호출할 타이밍은 테스트 메소드에서 예외를 잡았을 때다.
```Python
//TestCase
def run(self):
  result = TestResult()
  result.testStarted()
  self.setUp()
  try:
    method = getattr(self, self.name)
    method()
  except:
    result.testFailed()
  self.tearDown()
  return result
```
이 메소드에는 교모하게 숨겨진 부분이 있다.  
위에서 작성한 대로라면 setUp() 에서 발생하는 문제에 대해선 예외가 잡히지 않는다.  
이건 바라는 바가 아니다. 우리가 원하는 건 독립적으로 테스트가 실행되는 걸 원한다.  
하지만 코드를 수정하기 위해서는 또 다른 테스트가 필요하다.  
우리는 항상 테스트를 먼저 작성해야 한다는 사실을 의식적으로 상기해야 한다.  

### TODO 추가
 - ~~테스트 메서드 호출하기~~
 - ~~먼저 setUp 호출하기~~
 - ~~나중에 tearDown 호출하기~~
 - 테스트 메서드가 실패하더라도 tearDown 호출하기
 - 여러 개의 테스트 실행하기
 - ~~수집된 결과를 출력하기~~
 - ~~WasRun에 로그 문자열 남기기~~
 - ~~**실패한 테스트 보고하기**~~
 - **setUp 에러를 잡아서 보고하기(추가)**
 
다음에는 여러 테스트가 같이 실행될 수 있도록 만드는 일을 할 것이다.
 
## 우리는
  - 작은 스케일의 테스트가 통과하게 만들었다.
  - 큰 스케일의 테스트를 다시 도입했다.
  - 작은 스케일의 테스트에서 보았던 매커니즘을 이용해 큰 스케일의 테스트를 빠르게 통과시켰다.
  - 중요한 문제를 발견했는데 이를 바로 처리하기 보단 할 일 목록에 적었다.
