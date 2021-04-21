# 5장 : 클래스와 인터페이스

<div style="text-align: right"> 마지막 수정 : 2020-01-04



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





## <span style="color:green">[비고]</span>

- 4장에 앞서서 보고 있음.