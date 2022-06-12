# 19장. 테이블 차리기
테스트를 작성하다 보면 공통된 패턴을 발견하게 될 것이다.
 - 준비(arrange) - 객체를 생성한다.
 - 행동(act) - 어떤 자극을 준다.
 - 확인(assert) - 결과를 검사한다.

## TODO List)
 - ~~테스트 메서드 호출하기~~
 - **먼저 setUp 호출하기**
 - 나중에 tearDown 호출하기
 - 테스트 메서드가 실패하더라도 tearDown 호출하기
 - 여러 개의 테스트 실행하기
 - 수집된 결과를 출력하기

두 번쨰와 새 번째 단계인 행동과 확인 단계는 항상 다르지만, 처음 단계인 준비 단계는 여러 테스트에 걸쳐 동일한 경우가 종종 있다.  
만약 이 패턴이 서로 다른 스케일에서 반복된다면 테스트를 위해 새로운 객체를 얼마나 자주 생성해야 하는가 하는 문제에 직면하게 된다. 이 떄 다음 두 가지 제약이 상충한다.
 - 성능: 우린 테스트가 될 수 있는 한 빨리 실행되길 원한다. 여러 테스트에서 같은 객체를 사용한다면, 객체 하나만 생성해서 모든 테스트가 이 객체를 쓰게 할 수 있을 것이다.
 - 격리: 우린 한 테스트에서의 성공이나 실패가 다른 테스트에 영향을 주지 않기를 원한다. 만약 테스트들이 객체를 공유하는 상태에서 하나의 테스트가 공유 객체의 상태를 변경한다면 다음 테스트의 결과에 영향을 미칠 가능성이 있다.

테스트 사이의 커플링은 확실히 지저분한 결과를 야기한다.  
 1. 한개의 테스트가 꺠지면 결합도가 묶인 테스트들이 몇개든 깨진다.
 2. 순서가 중요하다면 뒤의 테스트가 실패하거나, 실패해야 하는 테스트이지만 성공하는 경우가 생긴다.

**테스트 커플링을 만들지 말것!**  
  
이전 예시에서 우리는 테스트를 실행하기 전에 어떤 플래그를 거짓으로 두기를 원했다.
```Python
class TestCaseTest(TestCase):
	def testRunning(self):
		test = WasRun("testMethod")
		assert(not test.wasRun)
		test.run()
		assert(test.wasRun)
	TestCaseTest("testRunning").run()	

	def testSetUp(self): # 추가
		test = WasRun("testMethod")
		test.run()
		assert(test.wasSetUp)
	TestCaseTest("TestSetUp").run()
```
이 테스트를 실행하면 wasSetUp 속성이 없다고 알려준다. 세팅해보자.

```Python
class TestCase: 
	def __init__(self, name):
		self.name = name

	def run(self):
		method = getattr(self, self.name)
		method()

	def setUp(self): # setUp을 호출하는 것은 TestCase가 할것이다. 
		pass

class WasRun(TestCase): 
	def __init__(self, name):
		self.wasRun = None
		TestCase.__init__(self, name)

	def testMethod(self):
		self.wasRun = 1

	def setUp(self): # 추가
		self.wasSetUp = 1 
```
> 테스트가 잘 되긴 한다.

뭔가를 배우고 싶다면, 한 번에 메서드를 하나 이상 수정하지 않으면서 테스트가 통과하게 만들 수 있는 방법을 찾아보자.  

```Python
class WasRun(TestCase): 
	def __init__(self, name):
		self.wasRun = None
		TestCase.__init__(self, name)

	def testMethod(self):
		self.wasRun = 1

	def setUp(self):
		self.wasRun = None # wasRun 플래그를 setUp에서 설정하도록 하면 WasRun을 단순화할 수 있다. 
		self.wasSetUp = 1 
```
테스트를 실행하기 전에 플래그를 검사하지 않도록 testRunning을 단순화해야 한다.  
다른 테스트가 존재하고 잘 돌아갈때에만 테스트를 단순화할 수 있다. 
```Python
class TestCaseTest(TestCase):
	def testRunning(self):
		test = WasRun("testMethod")
		#assert(not test.wasRun)
		test.run()
		assert(test.wasRun)
	TestCaseTest("testRunning").run()	

	def testSetUp(self):
		test = WasRun("testMethod")
		test.run()
		assert(test.wasSetUp)
	TestCaseTest("TestSetUp").run()
```
  


테스트 자체도 단순화할 수 있다. 두 경우 모두 WasRun의 인스턴스를 생성하는데 WasRun을 SetUp에서 생성하고 테스트메서드에서 그걸 사용하게 할 수 있다.  
각 테스트 메서드는 깨끗한 새 TestCaseTest 인스턴스를 사용하므로 두 개의 테스트가 커플링 가능성은 없다(전역변수는 그러니까 쓰지 말자.)

```Python
class TestCaseTest(TestCase):
	def setUp(self):
		self.test = WasRun("testMethod")

	def testRunning(self):
		#test = WasRun("testMethod")
		#test.run()
		self.test.run()

		assert(test.wasRun)

	def testSetUp(self):
		# test = WasRun("testMethod")
		# test.run()
		self.test.run()

		assert(test.wasSetUp)
```

### TODO List)
 - ~~테스트 메서드 호출하기~~
 - **~~먼저 setUp 호출하기~~**
 - 나중에 tearDown 호출하기
 - 테스트 메서드가 실패하더라도 tearDown 호출하기
 - 여러 개의 테스트 실행하기
 - 수집된 결과를 출력하기

## 우리는
 - 일단은 테스트를 작성하는 데 있어 갈결함이 성능 향상보다 더 중요하다고 생각하기로 했다.
 - setUp()을 테스트하고 구현했다.
 - 예제 테스트 케이스를 단순화하기 위해 setUp()을 사용했다.
 - 예제 테스트 케이스에 대한 테스트 케이스를 단순화하기 위해 setUp()을 사용했다.



























