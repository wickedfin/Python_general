# 3장 : 함수

<div style="text-align: right"> 마지막 수정 : 2020-12-30


[TOC]

## <span style="color:green">[개요]</span>

- 함수는 이제 어느 정도 다루지만, 그래도 근본을 다진다는 의미에서 공부해보자.





## <span style="color:green">[본문]</span>

### 1. 함수에서는 웬만해선 None을 반환하지 말자

- 아래 함수의 문제점은 값이 0이거나 None이거나, 둘 다 검사식에서 False로 처리된다는 것이다.

  ~~~python
  def divide(a, b):
      try:
          return a / b
      except ZeroDivisionError:
          return None
      
  if not None: print("hello")
  >>  hello
  ~~~
  
- 따라서 아래처럼 예외를 일으켜주는 것이 좋다.

  ~~~python
  def divide_fixed(a, b):
      try:
          return a / b
      except ZeroDivisionError as e:
          raise ValueError("Invalid inputs") from e
  ~~~

- 저렇게 예외를 일으키면 함수를 사용하는 입장에서 편해진다.

  ~~~python
  x, y = 5, 0
  try:
      result = divide_fixed(x, y)
  except ValueError:
      print("Invalid inputs")
  else:
      print("Result is %.1f" % result)
  # 즉 함수에서 None이 아니라 exceptions을 띄우도록 설계하라는 뜻
  ~~~



### 2. 클로저 함수의 변수 스코프 상호작용에 대하여

- 아래 함수는 `group`에 속한 얘들을 우선적으로 보여주면서 sorting하는 함수이다.

  ~~~python
  numbers = [8, 3, 1, 2, 5, 4, 7, 6]
  group = {2, 3, 5, 7}
  def sort_priority(values, group):
      def helper(x):
          if x in group:    # 함수 helper에는 group이 정의 되지 않았지만 참조할 수 있음, 클로저 함수의 특징임
              return(0, x)
          return (1, x)
      values.sort(key=helper)
  sort_priority(numbers, group)
  print(numbers)
  ~~~

- 볼 수 있듯이, **함수 helper**가 group을 상위 함수로부터 참조했다. 하지만 클로저 함수 내에서 **할당**을 하게되면 이름이 같아도 새로운 변수로써 취급된다.

- 이럴 때에는 nonlocal이나 global문 같은 걸 사용해서 참조되는 변수를 맞출 수 있다. global문은 전역 변수이기 때문에 더 안전한 nonlocal을 쓰자.

  ~~~python
  def sort_priority_found(numbers, group):
      found = False
      def helper(x):
          nonlocal found
          if x in group:
              found = True    # "할당"을 했기 때문에 위에 nonlocal 있어야만 상위 함수에서 정의한 found가 된다.
              return (0, x)
          return (1, x)
      numbers.sort(key=helper)
      return found
  
  print(sort_priority_found(numbers, group))
  print(numbers)
  ~~~

- 위의 클로저 함수를 쓰는 것이 복잡해지기 시작했다면 클래스로 표현하는 게 나을 수도 있다.

  ~~~python
  class Sorter(object):
      def __init__(self, group):
          self.group = group
          self.found = False
  
      def __call__(self, x):
          if x in self.group:
              self.found = True
              return (0, x)
          return (1, x)
      
  sorter = Sorter(group)
  numbers.sort(key=sorter)
  assert sorter.found is True  # 보장 받기 위해 사용하는 구문, 개발 과정에서 보증을 위해 많이 사용한다.
  ~~~



### 3. 제너레이터 내용 추가

- 내장 함수 list를 통해 손쉽게 리스트로 변환 가능하다

  ~~~python
  result = list(index_words_iter(address))
  ~~~
  
- 제너레이터의 정의에 있어서 주의해야할 것은 이터레이터는 **현 상태**라는 것이 있고 재사용이 불가능하다는 것이다.

  - 즉 인수를 순회할 때는 방어적으로 해야 한다.

- 재사용이 불가능하다는 점을 커버하기 위해서 입력 데이터를 여러 번 읽어 각각 새 interator로 만드는 방법을 사용할 수 있다.

  - 이터레이터 프로토콜을 따르는 어떤 컨테이너 타입을 만들면 되는 것

  ~~~python
  class ReadVisits(object):
      def __init__(self, data_path):
          self.data_path = data_path
          
      def __iter__(self):
          with open(self.data_path) as f:
              for line in f:
                  yield int(line)
                  
  def normalize_defensive(numbers):
      if iter(numbers) is iter(numbers):
          raise TypeError("Must supply a container")  # 단순 이터러블 밴
      total = sum(numbers)
      result = []
      for value in numbers:
          percent = 100 * value / total
          result.append(percent)
      return result
  
  visits = [15, 35, 80]
  normalize_defensive(visits)
  visits = ReadVisits(path)
  normalize_defensive(visits)
  
  # 즉 visits가 이터레이터 프로토콜을 따르는 어떤 컨테이너 타입이면 normailze_defensive가 잘 작동하게 됨.
  # 하지만 단순 이터러블이면 sum에서 소진될 때 for 문에서 사용할 수 없으므로 단순 이터러블 넣으면 예상대로 작동 안하게 됨.
  ~~~

- `if iter(numbers) is iter(numbers):` 이 줄이 좀 tricky한데 컨테이너 타입의 경우 iter method가 불러질 때마다 매번 다른 이터러블이 호출되므로 is 로 테스트해보면 다른 것으로 판명이 됨



### 4. 가변 위치 인수를 활용하자

- 선택적인 위치 인수 또는 **star args**라고 불리는 `*args`를 활용해서 함수 호출을 더 명확하게 할 수 있다. 

  ~~~python
  def log(message, *values):
      if not values:
          print(message)
      else:
          values_str = ', '.join(str(x) for x in values)
          print('%s: %s' % (message, values_str))
  
  
  log('My numbers are', 1, 2)
  log('Hi there')
  
  favorites = [7, 33, 99]
  log('Favoriate colors', *favorites)  # 이런 식으로 하면 favoriate이라는 list를 *values의 인수로 받을 수 있다.
                                       # 만약 이런 식으로 이터레이터를 넘기면 튜플로 바꾸면서 소진됨. 메모리 문제도 있을 수 있음.
  ~~~

- 여기서 주의해야할 점은 가변 인수는 함수 전달 전 <b>항상 튜플로 변환된다는 것</b>이다.

- 또 한 가지 주의해야 할 점은 인수의 위치에 주의해야 한다는 것



### 5. 키워드 인수를 활용하자

- 키워드 인수를 활용하면 함수 호출 시 더 명확하게 의도를 알 수 있다. (또는 드러낼 수 있다.)

  ~~~python
  def remainder(number, divisor):
      return number % divisor
  
  assert remainder(20, 7) == 6
  
  # 아래는 모두 동일한 호출
  remainder(20, 7)
  remainder(20, divisor=7)
  remainder(number=20, divisor=7)
  remainder(divisor=7, number=20)
  
  # 아래는 불가능한 호출
  remainder(number=20, 7)  # 위치 인수를 키워드 인수의 앞에 지정해야한다. 
  remainder(20, number=7)  # number 인수에 이미 20이 들어간 상태에서 7이 다시 들어갔다.
  ~~~

- 동적 기본 인수를 지정하려면 None과 docstring을 사용하자

  ~~~python
  def log(message, when=datetime.now()):
      print("%s: %s" % (when, message))
  # 이런 식으로 하면 when은 함수 정의 시의 datetime.now()로 고정되어 버림
      
  
  def log(message, when=None):
      """Log a message with a timestamp.
      
      Args:
      	message: Message to print
      	when: datetime of when the message occurred.
      		Defaults to the present time.
      """
      when = datetime.now() if when is None else when
      print("%s: %s" % (when, message))
  ~~~



### 6. 키워드 전용 인수로 명료성을 강요하자

- 함수 정의 시에 * 별표를 위치인수와 키워드 전용인수 사이에 놓아줌으로써 키워드 전용 인수를 정의할 수 있다.

  ~~~python
  def safe_division(number, divisor, *, ignore_overflow=False, ignore_zero_division=False):
      try:
          return number / divisor
      except OverflowError:
          if ignore_overflow:
              return 0
          else:
              raise
      except ZeroDivisionError:
          if ignore_zero_division:
              return float('inf')
          else:
              raise
  ~~~
  
- 위처럼 구현해두면 이제 키워드 인수를 함수 호출에서 사용 시, 반드시 키워드를 명시해야 함.



### 7. 함수가 여러 값을 반환할 때, 4개 이상의 값을 언패킹하지 말자

- 이는 실수하기 쉽기 때문이고, 만약 해야한다면 *경량 클래스*나 *namedtuple*을 사용하자



### 8. functools.wrap을 사용해 함수 데코레이터를 정의하라

- 데코레이터는 자신이 감싸고 있는 함수가 호출되기 전, 또는 후에 코드를 추가로 실행해준다.

  - 가령 어떤 함수에 들어가기 전후에 **타입 변환등**을 정의하는 함수를 실행할 수 있다.
  - 또한 오류를 미연에 방지해주기 위해 `functools.wrap`를 사용해주는 것이 좋다.

- 아래는 html tag를 붙이는 데코레이터 예시. Hi 구문 앞뒤로 html tag를 붙여주는 역할을 한다.

  ~~~python
  from functools import wraps
  
  def html(func):
  	@wraps(func)
      def wrapper(*args, **kwargs):
          print("<html>")
          func(*args, **kwargs)
          print("</html>")
      return wrapper
  
  @html
  def test(text):
      print(text)
      
  test("Hi")
  ~~~

  



## <span style="color:green">[비고]</span>

- 3장 모두 공부하였음.
- 데코레이터 함수 개념이 재밌네. 자주 써 봐야겠다.
- 이제 4장으로 가봅시다.