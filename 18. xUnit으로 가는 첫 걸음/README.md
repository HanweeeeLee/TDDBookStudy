# 18. xUnit으로 가는 첫 걸음

## 테스트 프레임워크에 대한 할일 목록 TODO)
 - 테스트 메서드 호출하기
 - 먼저 setUp 호출하기
 - 나중에 tearDown 호출하기
 - 테스트 메서드가 실패하더라도 tearDown 호출하기
 - 여러 개의 테스트 실행하기
 - 수집된 결과를 출력하기

## 내용
우리의 첫 번째 원시테스트에는 테스트 메서드가 호출되면 true, 그렇지 않으면 false를 반환할 프로그램이 필요하다.  
플래그를 가지고 있는 테스트 케이스를 만들것이다. 테스트 메서드가 실행되기 전에는 플래그가 false이고, 테스트 메서드가 실행된 후에는 플래그가 true로 바뀐다.  
메서드가 실행되었는지를 알려주는 테스트 케이스이므로 클래스 이름을 WasRun으로 하자. 플래그 이름은 wasRun으로..!

```Python
test = WasRun("testMothod")
print test.wasRun
test.testMethod()
print test.wasRun
```
> 이 코드는 메서드가 실행되기 전에 'Nonce'를 출력하고, 그 후에 '1'을 출력할것이다. 하지만 이 코드는 실행되지 않는다 -> WasRun클래스를 정의하지 않았기 때문에(테스트 먼저!)

```Python 
class WasRun: 
	pass #pass라는 키워드는 클래스나 메서드 구현이 없을 떄 사용한다. 
```
이제 우리는 wasRun이란 속성이 필요하다는 말을 듣는다. 

```Python
class WasRun: 
	#pass
	def __init__(self, name):
		self.wasRun = None
```
실행하면 'None'을 출력한 후, testMethod를 정의해야 한다고 알려준다.  

```Python
class WasRun: 
	def __init__(self, name):
		self.wasRun = None

	def testMethod(self): # 추가
		pass
```
실행하면 'None', 'None'이 출력된다. 우리가 원하는건 'None', '1'이다.

```Python
class WasRun: 
	def __init__(self, name):
		self.wasRun = None

	def testMethod(self):
		#pass
		self.wasRun = 1
```
이제 초록 막대다! 이제 리팩토링을 해보자.  
  
다음으로 필요한 것은 테스트 메서드를 직접 호출하는 대신 진짜 인터페이스인 run() 메서드를 사용하는 것이다. 테스트는 다음과 같이 변한다.
```Python
test = WasRun("testMothod")
print test.wasRun

#test.testMethod()
test.run()

print test.wasRun
```
  
구현 코드는 다음과 같다.
```Python
class WasRun: 
	def __init__(self, name):
		self.wasRun = None

	def testMethod(self):
		self.wasRun = 1

	def run(self): # 추가
		self.testMethod()
```
이제 테스트는 다시 올바른 값을 출력한다.  
  
다음 단계는 testMethod()를 동적으로 호출하는 것이다.  
파이썬의 가장 멋진 특징 중 하나는 클래스의 이름이나 메서드의 이름을 함수처럼 다룰 수 있다는 점이다.  
테스트 케이스의 이름과 같은 문자열을 갖는 필드가 주어지면, 함수로 호출될 때 해당 메서드를 호출하게끔 하는 객체를 얻어낼 수 있다.  

```Python
class WasRun: 
	def __init__(self, name):
		self.wasRun = None
		self.name = name # 추가

	def testMethod(self):
		self.wasRun = 1

	def run(self):
		#self.testMethod()
		method = getattr(self, self.name)
		method()
```
이제 WasRun 클래스는 독립된 두 가지 일을 수행한다
1. 메서드가 호출되었느지 그렇지 않은지를 기억하는 일
2. 메서드를 동적으로 호출하는 일
  
비어있는 TestCase 상위 클래스를 만들고 WasRun이 이를 상속받게 만들자

```Python
class TestCase: 
	pass

#class WasRun: 
class WasRun(TestCase): 
	def __init__(self, name):
		self.wasRun = None
		self.name = name

	def testMethod(self):
		self.wasRun = 1

	def run(self):
		method = getattr(self, self.name)
		method()
```
이제 name 속성을 상위 클래스로 끌어올리자.
```Python
class TestCase: 
	# pass
	def __init__(self, name):
		self.name = name

class WasRun(TestCase): 
	def __init__(self, name):
		self.wasRun = None
		#self.name = name
		TestCase.__init__(self, name)

	def testMethod(self):
		self.wasRun = 1

	def run(self):
		method = getattr(self, self.name)
		method()
```
마지막으로, run() 메서드는 상위 클래스에 있는 속성만을 사용하므로 이것도 올리자.

```Python
class TestCase: 
	def __init__(self, name):
		self.name = name

	def run(self): # 상위클래스로 올려짐
		method = getattr(self, self.name)
		method()

class WasRun(TestCase): 
	def __init__(self, name):
		self.wasRun = None
		TestCase.__init__(self, name)

	def testMethod(self):
		self.wasRun = 1

	# def run(self):
	# 	method = getattr(self, self.name)
	# 	method()
```

우리가 방금 만든 코드를 사용하여 이제 다음과 같이 바꿔쓸 수 있다.
```Python
class TestCaseTest(TestCase):
	def testRunning(self):
		test = WasRun("testMethod")
		assert(not test.wasRun)
		test.run()
		assert(test.wasRun)
TestCaseTest("testRunning").run()
```
> 테스트 코드의 내용은 단순히 프린트 문이 assertion으로 바뀐것임..

### TODO List)
 - **~~테스트 메서드 호출하기~~**
 - 먼저 setUp 호출하기
 - 나중에 tearDown 호출하기
 - 테스트 메서드가 실패하더라도 tearDown 호출하기
 - 여러 개의 테스트 실행하기
 - 수집된 결과를 출력하기

## 우리는
 - 자기 과신에 차서 몇 번의 잘못된 출발을 한 후, 아주 자그마한 단계로 시작하는 법을 알아냈다.
 - 일단 하드코딩을 한 다음 상수를 변수로 대체하여 일반성을 이끌어내는 방식으로 기능을 구현했다.
 - 플러거블 셀렉터(Pluggable Selector)를 사용했다. 플러거블 셀렉터는 정적 코드 분석을 어렵게 만들기 때문에 앞으로 최소 4개월 안에는 사용하지 않기로 약속하자.
 - 테스트 프레임워크를 작은 단계로만 부트스트랩했다.
























