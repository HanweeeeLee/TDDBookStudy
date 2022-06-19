# 20. 뒷정리하기

## TODO List)
 - ~~테스트 메서드 호출하기~~
 - ~~먼저 setUp 호출하기~~
 - **나중에 tearDown 호출하기**
 - 테스트 메서드가 실패하더라도 tearDown 호출하기
 - 여러 개의 테스트 실행하기
 - 수집된 결과를 출력하기

## 내용
테스트가 계속 서로 독립적이길 바란다면 외부 자원을 할당받은 테스트들은 작업을 마치기 전에 tearDown()메서드 같은 곳에서 자원을 다시 반환할 필요가 있다.  
setUp()은 테스트 메서드가 실행되기 전에 호출되어야하고, tearDown()은 테스트 메서드가 실행된 후에 호출되어야 한다.  
이를 테스트하기 위해 호출된 메서드의 로그를 간단히 남기는 식으로 테스트 전략을 바꿔보자.  
항상 로그의 끝부분에만 기록을 추가하면 메서드 호출 순서를 알 수 있게 될 것이다.

### TODO 추가
 - ~~테스트 메서드 호출하기~~
 - ~~먼저 setUp 호출하기~~
 - 나중에 tearDown 호출하기
 - 테스트 메서드가 실패하더라도 tearDown 호출하기
 - 여러 개의 테스트 실행하기
 - 수집된 결과를 출력하기
 - **WasRun에 로그 문자열 남기기**

```Python
class WasRun(TestCase): 
	def __init__(self, name):
		self.wasRun = None
		TestCase.__init__(self, name)

	def testMethod(self):
		self.wasRun = 1

	def setUp(self):
		self.wasRun = None
		self.wasSetUp = 1 
		self.log = "set Up " # 추가
```
> 이제 testSetUp()이 플래그 대신에 로그를 검사하도록 변경할 수 있다. 

```Python
class TestCaseTest(TestCase):
	def setUp(self):
		self.test = WasRun("testMethod")

	def testRunning(self):
		self.test.run()
		assert(test.wasRun)

	def testSetUp(self):
		self.test.run()
		# assert(test.wasSetUp)
		assert("setUp " == self.test.log)
```

다음엔 wasSetup 플래그를 지울 수 있다. 그리고 테스트 메서드의 실행을 기록할 수 있다. 

```Python
class WasRun(TestCase): 
	def __init__(self, name):
		self.wasRun = None
		TestCase.__init__(self, name)

	def testMethod(self):
		self.wasRun = 1
		self.log = self.log + "testMethod" # 추가

	def setUp(self):
		self.wasRun = None
		self.wasSetUp = 1 
		self.log = "set Up "
```
> 이 작업은 testSetUp을 실패하게 만든다. 왜냐하면 실제 로그는 'setUptestMethod'이기 떄문이다. 

```Python
class TestCaseTest(TestCase):
	def setUp(self):
		self.test = WasRun("testMethod")

	def testRunning(self):
		self.test.run()
		assert(test.wasRun)

	def testSetUp(self):
		self.test.run()
		# assert("setUp " == self.test.log)
		assert("setUp testMethod " == self.test.log)
```

이제 이 테스트는 두 개의 테스트가 할 일을 모두 수행한다. 따라서 test-Running을 지우고 testSetUp의 이름을 바꿔주자.

```Python
class TestCaseTest(TestCase):
	def setUp(self):
		self.test = WasRun("testMethod")

	# def testRunning(self):
	# 	self.test.run()
	# 	assert(test.wasRun)

	# def testSetUp(self):
	def testTemplateMethod(self):
		self.test.run()
		assert("setUp testMethod " == self.test.log)
```
  
불행이도 WasRun의 인스턴스를 한 곳에서만 사용하므로 꾀를 써서 setUp 부분을 분리했던 것을 되돌려 놓아야 한다.
```Python
class TestCaseTest(TestCase):
	def setUp(self):
		self.test = WasRun("testMethod")

	def testTemplateMethod(self):
		test = WasRun("testMethod") // 돌려놓자.
		self.test.run()
		assert("setUp testMethod " == self.test.log)
```
> 리펙토링을 했다가 다시 돌려놓는 일은 매우 흔한 일이다. 

### TODO 추가
 - ~~테스트 메서드 호출하기~~
 - ~~먼저 setUp 호출하기~~
 - **나중에 tearDown 호출하기**
 - 테스트 메서드가 실패하더라도 tearDown 호출하기
 - 여러 개의 테스트 실행하기
 - 수집된 결과를 출력하기
 - **~~WasRun에 로그 문자열 남기기~~**

이제 tearDown()을 테스트할 준비가 됐다.
```Python
class TestCaseTest(TestCase):
	def setUp(self):
		self.test = WasRun("testMethod")

	def testTemplateMethod(self):
		test = WasRun("testMethod") // 돌려놓자.
		self.test.run()
		# assert("setUp testMethod " == self.test.log)
		assert("setUp testMethod tearDown " == self.test.log)
```
> 실패한다. 성공하게 만들어보자.

```Python
class TestCase:
	def __init__(self, name):
		self.name = name

	def run(self):
		self.setUp() # setUp호출
		method = getattr(self, self.name) # 메서드 set
		method() # 메서드 호출
		self.tearDown() # tearDown 호출

	def setUp(self): # setUp을 호출하는 것은 TestCase가 할것이다. 
		pass
```

```Python
class WasRun(TestCase): 
	def __init__(self, name):
		self.wasRun = None
		TestCase.__init__(self, name)

	def testMethod(self):
		# self.wasRun = 1
		self.log = self.log + "testMethod"

	def setUp(self):
		# self.wasRun = None
		# self.wasSetUp = 1 
		self.log = "set Up "

	def tearDown(self): # 추가
		self.log = self.log + "tearDown "
```

이제 TestCaseTest에서 에러가 난다. 아직 TestCase에 아무 일도 하지 않는 tearDown()을 구현하지 않았다.

```Python
class TestCase:
	def __init__(self, name):
		self.name = name

	def run(self):
		self.setUp()
		method = getattr(self, self.name)
		method()
		self.tearDown() 

	def setUp(self): 
		pass

	def tearDown(self): # 추가
		pass
```

### TODO 추가
 - ~~테스트 메서드 호출하기~~
 - ~~먼저 setUp 호출하기~~
 - **~~나중에 tearDown 호출하기~~**
 - 테스트 메서드가 실패하더라도 tearDown 호출하기
 - 여러 개의 테스트 실행하기
 - 수집된 결과를 출력하기
 - ~~WasRun에 로그 문자열 남기기~~


## 우리는
 - 플래그에서 로그로 테스트 전략을 구조 조정했다.
 - 새로운 로그 기능을 이용하여 tearDown()을 테스트하고 구현했다.
 - 문제를 반결했는데 뒤로 되돌아가는 대신 과감히 수정했다(이게 잘 한 일일까?)





















