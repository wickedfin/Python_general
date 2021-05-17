# 5장 : 클래스와 인터페이스

<div style="text-align: right"> 마지막 수정 : 2021-05-17




[TOC]

## <span style="color:green">[개요]</span>

- 객체 지향 언어인 파이썬을 잘 다루기 위해서는 결국 클래스를 잘 다뤄야한다.
- 클래스를 잘 알아두면...
  - 유지보수가 용이한 코드를 작성할 수 있다.
  - 프로그램의 기능 개선, 확장, 요구 사항이 바뀌는 환경에 유용하다.
- "유지, 보수의 악몽 속으로..."라는 표현이 참 마음에 든다.





## <span style="color:green">[본문]</span>

### 1. 내장 타입을 여러 단계로 내포시키기보다는 클래스를 합성하라

- 학생 성적을 관리하는 프로그램이 있다 가정할 때, (학생별로, 과목별로, 시험 분류별 가중치로) 점수를 매기다보면 하나의 클래스가 매우 복잡해진다.

- 이 경우 학생별로 사전을 만들고, 과목별로 만드는 등, 여러 단계의 사전 내포가 필요한데, 이럴 경우 코드가 길어지더라도 클래스를 나눠 만든 뒤, 합성하는 것이 보기 좋다.

- 유연성이 중요치 않고, 불변 데이터 컨테이너가 필요하다면 **namedtuple**을 사용하는 것이 유용하다.

  ~~~python
  from collections import namedtuple, defaultdict
  
  Grade = namedtuple('Grade', ('score', 'weight'))
  
  
  class Subject:
      def __init__(self):
          self._grades = []
  
      def report_grade(self, score, weight):
          self._grades.append(Grade(score, weight))
  
      def average_grade(self):
          total, total_weight = 0, 0
          for grade in self._grades:
              total += grade.score * grade.weight
              total_weight += grade.weight
          return total / total_weight
  
  
  test = Subject()
  test.report_grade(80, 0.2)
  test.report_grade(80, 0.4)
  test.report_grade(80, 0.6)
  print(test.average_grade())
  
  
  class Student:
      def __init__(self):
          self._subjects = defaultdict(Subject)
  
      def get_subject(self, name):
          return self._subjects[name]
  
      def average_grade(self):
          total, count = 0, 0
          for subject in self._subjects.values():
              total += subject.average_grade()
              count += 1
          return total / count
  
  
  class Gradebook:
      def __init__(self):
          self._students = defaultdict(Student)
  
      def get_student(self, name):
          return self._students[name]
  
  
  book = Gradebook()
  albert = book.get_student('알버트 아인슈타인')
  math = albert.get_subject('수학')
  math.report_grade(75, 0.05)
  math.report_grade(score=75, weight=0.05)
  gym = albert.get_subject('체육')
  gym.report_grade(100, 0.4)
  gym.report_grade(85, 0.6)
  print(albert.average_grade())
  ~~~

- 위 코드를 보면 단계별로 3개의 클래스를 구성해서 연결시켜둔 것이다.

- **Gradebook**에서 학생 이름을 넣으면, **Student 클래스**를  부르고, Student 클래스에서 과목을 입력하면 **Subject 클래스**를 부르는 식이다.

- **_instance**를 사용해서 내부 작동짜는 게 깔끔해보이는 코드다.



### 2. 인터페이스가 간단하면 클래스 대신 함수를 받자

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
  
- 아래는 defaultdict에서 새로운 것이 추가될 때마다 횟수를 세는 것과 관련된 코드이다.

  - 근데 위 제목이랑 무슨 상관이 있는 것인지 잘 모르겠다.

  ~~~python
  from collections import defaultdict
  
  current = {'초록': 12, '파랑': 3}
  increments = [('빨강', 5), ('파랑', 17), ('주황', 9)]
  
  class BetterCountMissing:
      def __init__(self):
          self.added = 0
  
      def __call__(self):
          self.added += 1
          return 0
  
  counter = BetterCountMissing()
  result = defaultdict(counter, current)
  for key, amount in increments:
      result[key] += amount
  
  assert counter.added == 2
  ~~~
  
- 설명을 좀 하자면, **defaultdict**의 초기값으로 함수를 넘길 수가 있음. 함수가 **return 0**하기 때문에 0을 초기값으로 갖게 한다.

  - 하지만 여기에 **self.added**를 통해 특정 상태를 저장하는 원리이다. 
  - **call**을 통해서 함수가 불러질 때만 작동하도록 할 수 있다.



### 3. @property

- 여긴 책에 나오는 내용은 아니고, 그냥 찾아봄

- 클래스 인스턴스의 내부 데이터를 보호하기 위해 접근용 메서드를 작성하는 것은 객체 지향 프로그래밍에서의 흔한 패턴

  - 즉 **getter와 setter**를 사용하는 건데, 이걸 파이썬에서는 **property 데코레이터**로 예쁘게 구현할 수 있다.

- 그냥 인스턴스로 구현하는 것보다 외부로부터 안전하게 만들 수 있다. (가령 나이는 음수가 될 수 없다는 식으로)

  ~~~python
  class Person:
      def __init__(self, first_name, last_name, age):
          self.first_name = first_name
          self.last_name = last_name
          self.age = age
          # 여기가 _age가 아니라 그냥 age여야 setter의 조건이 적용되던데 이유는 모르겠음.
  
      @property
      def age(self):
          return self._age
  
      @age.setter
      def age(self, age):
          if age < 0:
              raise ValueError("Invalied age")
          # 나이에 음수를 설정할 수 없도록 인스턴스 보호
          self._age = age
  
      @age.deleter  # 사용할 일이 많지는 않을 거 같다.
      def age(self):
          del self._age
  
      @property
      def full_name(self):
          return self.first_name + " " + self.last_name
  
  james = Person("James", "Lee", 28)
  print(james.age)
  james.age += 1
  print(james.age)
  james.age = 20
  print(james.age)
  # del(james.age)
  # print(james.age)
  print(james.full_name)
  
  mason = Person("Mason", "Lee", -1)  # 오류를 일으킴
  ~~~



### 4. @classmethod를 활용해 다형성을 확보하라

- 이 부분 책의 내용이 복잡하여, @staticmethod과 @classmethod가 무엇인지에 대한 내용으로 대체해둔다.

- 우선 파이썬 클래스 안에서 정의되는 메소드는 크게 3가지로 아래와 같다.

  > 인스턴스 메소드, 스태틱 메소드, 클래스 메소드

- 위 3개 중 후자의 2개를 **정적 메소드**라고 한다. 

  - 본래 익숙한 방식은 `1)클래스 정의 2)객체 생성 3)메소드 호출`의 방식일 것이다.
  - 정적 메소드는 **객체의 생성 없이, 호출이 가능한**(접근 가능한) 메소드를 의미한다.

- 인스턴스 메소드는, 첫번째 인자로 인스턴스 자신(self)을 전달하지만 ,클래스 메소드는 첫번째 인자로 클래스 자신(cls)을 전달한다. 

- 스태틱 메소드는 인스턴스나 클래스를 인자로 받지 않는다.

- 클래스 메서드는 클래스 속성에 접근하거나 클래스 메서드를 호출할 수 있으나, 인스턴스 속성이나 인스턴스 메서드에 접근하면 안된다.

  - 클래스메서드는 팩토리 메서드 작성 시에 유용하게 사용된다.

  ~~~python
  class User:
      def __init__(self, email, password):
          self.email = email
          self.password = password
  
      @classmethod
      def fromTuple(cls, tup):
          return cls(tup[0], tup[1])
  
      @classmethod
      def fromDictionary(cls, dic):
          return cls(dic["email"], dic["password"])
  
  
  # 기본 생성자로 객체 생성
  user = User("user@test.com", "1245")
  print(user.email, user.password)
  
  # 클래스메서드-tuple로 객체 생성
  user = User.fromTuple(("user@test.com", "1245"))
  print(user.email, user.password)
  
  # 클래스메서드-dictionary로 객체 생성
  user = User.fromDictionary({"email": "user@test.com", "password": "1245"})
  print(user.email, user.password)
  ~~~

  - 클래스 메서드는 상속 받는 하위 클래스에서의 다형성을 위해서도 많이 활용된다.

- 정적메소드의 경우, 사실 사용할 일이 그렇게 많지는 않다. 클래스 method와 다르게, 절대경로로 class를 적어주어야한다.

  - 비슷한 기능의 유틸리티 메서드들을 하나의 class 아래에 정리할 때 주로 사용한다. 

  ~~~python
  class StringUtils:
      @staticmethod
      def toCamelcase(text):
          words = iter(text.split("_"))
          return next(words) + "".join(i.title() for i in words)
  
      @staticmethod
      def toSnakecase(text):
          letters = ["_" + i.lower() if i.isupper() else i for i in text]
          return "".join(letters).lstrip("_")
  ~~~



### 5. super로 부모 클래스를 초기화하라

- 자식 클래스에서 부모 클래스를 초기화하는 오래된 방법은 자식 인스턴스에서 부모 클래스의 `__init__` 메서드를 직접 호출하는 것

  - 하지만 이 경우 다중 상속을 할 때에는 문제가 생길 수 있다. (다중 상속도 사실 하면 안 됨)

  ~~~python
  class MyBaseClass:
      def __init__(self, value):
          self.value = value
          
  class TimesTwo:
      def __init__(self):
          self.value *= 2
          
  class PlusFive:
      def __init__(self):
          self.valeu += 5
          
  class OneWay(MyBaseClass, TimesTwo, PlusFive):  # 여기의 나열 순서가 아닌
      def __init__(self, value):  # 여기에서의 생성자 호출 순서를 따르게 됨.
          MyBaseClass.__init__(self, value)
          TimesTwo.__init__(self)
          PlusFive.__init__(self)
          
  foo = OneWay(5)
  ~~~

  - 위의 예시에서 `foo`라는 변수에는 15라는 값이 들어가게 된다. `OneWay`부모 클래스를 나열한 순서가 아닌, 생성자 호출 순서를 따라가게 된다.
    - 즉, 헷갈리고 실수를 했을 시 잡아내기 힘들어짐
  - 또한 다이아몬드 상속에서도 문제를 일으킬 수 있음. 후에 호출되는 생성자가 다이아몬드 꼭지점 클래스의 생성자를 호출하면 앞 선 호출자의 생성자에서 수행된 코드는 무시될 수도 있기 때문

- 따라서, `super`를 사용해서 MRO(표준 메서드 결정 순서)를 따라가게 해주면 명료해진다.

  ~~~python
  class TimesSevenCorrect(MyBaseClass):
      def __init__(self, value):
          super().__init__(value)
          self.value *= 7
          
  class PlusNineCorrect(MyBaseClass):
      def __init__(self, value):
          super().__init__(value)
          self.valeu += 9
          
  class GoodWay(TimesSevenCorrect, PlusNineCorrect):
      def __init__(self, value):
          super().__init__(value)  # 일일이 부모 클래스의 이름을 적어가면 초기화할 필요가 없다.
                           
  foo = GoodWay(5)
  ~~~

  - `mro_str = '\n'.join(repr(cls) for cls in GoodWay.mro ())` 를 실행해보면 MRO 순서를 알 수 있다.
  - 다이아몬드 상속의 꼭지점을 만나면 그 역순으로 호출을 실행하기에 9가 먼저 더해지고 후에 7이 곱해진 것
  - `super`에 파라미터를 제공해야하는 경우는, 자식 클래스에서 상위 클래스의 특정 기능에 접근해야할 때 뿐이라고 하는데, 이 경우가 뭔지 모르겠다.



### 6. 기능을 합성할 때는 믹스인 클래스를 사용하라

- 이거 몰랐던 기능인데 꽤 유용할 거 같음. 자세히 보자

- 믹스인은 자식 클래스가 사용할 메서드 몇 개만 정의하는 클래스이다.

  - 다중 상속이 제공하는 편의와 캡슐화가 필요하지만, 다중 상속의 골치 아픈 상황을 피하고 싶을 때 좋다.
  - 반복적인 코드를 최소화하고 재사용성을 극대화할 수 있다.

- 근데 설명 읽어보니까, 결국 다중 상속으로 구현해야하는 거 같은데, 다중 상속 사용하는 거랑 무슨 차이인 지 모르겠다.

  - 책의 설명을 봤을 때는 이해가 안돼서, 구글링을 좀 해봤음.
  - 믹스인을 구현할 때에만 다중 상속을 사용하는 것이 좋다고하네.

- 일단 믹스인 클래스는 **자체 인스턴스 속성을 가지지 않음**, 또 **__init__** 메서드를 구현하지 않음. 이게 핵심 포인트인 거 같음.

  - 그러니까 정말 기능 만을 정의해둔 클래스라는 느낌인 거 같네. (참고로 클래스 속성은, 클래스의 인스턴스 모두가 공유함)
  - 필요에 따라 믹스인 안에 인스턴스 메서드는 물론 클래스 메서드도 포함될 수 있다.

  ~~~python
  class HelloMixIn:
      def greeting(self):
          print("안녕하세요.")
       
  class Person():
      def __init__(self, name):
          self.name = name
          
  class Student(HelloMixIn, Person):  # 기능을 합성 후, 나머지를 구현하는 방식인 거 같네. Person은 인스턴스 속성 있으니 믹스인 아님.
      def study(self):
          print("공부하기")
          
  class Teacher(HelloMixIn, Person):
      def teach(self):
          print("가르치기")
  ~~~

  - 보니까 분류별로 기능을 간단히 만들어두고, 합성을 통해 필요한 것들을 만들어가는 거 같은데 지금 내 수준에서 크게 쓸 일이 생길까?
  - 그냥 Util 같은 클래스 하나 만들어두고, 전부 staticmethod로 정의한 후에 쓰는 게 더 편리할 것도 같은데 흠



### 7. 비공개 애트리뷰트보다는 공개 애트리뷰트를 사용하자

- 파이썬에서는 비공개라고해서 접근 못하는 것 아니다. 귀찮을 뿐.
- 따라서 어차피 접근성을 막지 못하는데, 비공개 애트리뷰트를 사용하면 확장이나 하위 클래스에서의 오버라이드를 귀찮게 할 뿐이다.
- 따라서 밑줄 한 줄을 통해 보호 필드로 만들고, 크래스 외부에서 이 필드를 사용하는 경우, 조심해야한다는 경고 정도만 남겨주는 것이 좋다.
  - 이 편이 다른 프로그래머나 미래의 자신에게 클래스를 안전하게 확장하는 가이드 라인을 제공하기에 좋다.
- 비공개 애트리뷰트를 사용하는 거의 유일한 경우는, 하위 클래스의 필드와 이름이 충돌할 수 있는 경우 뿐이다.
  - 주로 공개 API에 속한 클래스에서 신경 써야 한다. 특히 `value`처럼 흔한 애트리뷰트 명일 경우에는 비공개로 처리해서 충돌을 막아주는 것이 좋다.



## <span style="color:green">[비고]</span>

- 4장에 앞서서 보고 있음.
- 5장 이제 한 챕터 남음.
- 일부 내용을 아래의 링크에서 가져왔음
  - https://www.daleseo.com/python-class-methods-vs-static-methods/
  - 위 링크 유용한 거 같아서, 후에 한 번 다 봐둘 예정