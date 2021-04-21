# 2장 : 리스트와 딕셔너리

<div style="text-align: right"> 마지막 수정 : 2020-12-29



[TOC]

## <span style="color:green">[개요]</span>

- 개정 2판이 되면서 순서가 좀 달라졌다. 이 파트도 그에 맞춰서 내용을 좀 바꿔둔다.





## <span style="color:green">[본문]</span>

### 1. 리스트에 관해서

- 리스트 등의 인덱싱에서 범위로 인덱싱할 때는 리스트의 경계를 벗어나도 허용한다.

  - 즉 somelist[:30] 이런 거는 리스트 길이가 30까지 안되어도 가능하다.
  - 하지만 somelist[30] 같은 직접 인덱싱은 오류가 난다. (길이가 30이 안되면)

  ~~~python
  a = [1, 3, 4, 5, 2, 4, 5, 2]
  print(a)
  a[2:6] = [8, 8]  # 이런 식으로 슬라이싱에 들어갈 길이가 달라도 부여할 수 있다
  print(a)  # 2~5에 해당하는 길이가 길이2로 줄어들었다.
  ~~~

- 아래처럼 하면 a를 복사한 b를 만들 수 있다. 하지만 둘은 서로 다른 리스트이다. 

  ~~~python
  b = a[:]
  print(a == b)  # True
  print(a is b)  # False  => is는 참조를 확인하는 것. 같은 참조가 아니기에 False가 뜸.
  -----------------------------------------------------------------------------------------------
  -----------------------------------------------------------------------------------------------
  b = a
  print(a)
  a[:] = [101, 102, 103]
  print(a)
  print(b)  # 이러면 둘이 같은 값이 나옴
  a = [4, 5, 6]  # => 다른 참조를 가지는 리스트를 할당했음.
  print(a)
  print(b)  # 이러면 둘이 다른 값이 나옴
  ~~~

- 슬라이싱에서 **stride**는 웬만하면 혼자서 쓰자



### 2. 슬라이싱보다는 언패킹을 이용해보자

- 별표식을 활용해 아래처럼 언패킹 할 수 있다.

  ~~~python
  car_ages = [0, 9, 4, 8, 7, 20, 19, 1, 6, 15]
  car_ages_descending = sorted(car_ages, reverse=True)
  
  # 슬라이싱을 이용하는 경우
  oldest = car_ages_descending[0]
  second_oldest = car_ages_descending[1]
  others = car_ages_descending[2:]
  print(oldest, second_oldest, others)
  
  # 별표식을 이용한 언패킹을 하는 경우
  oldest, second_oldest, *others = car_ages_descending
  print(oldest, second_oldest, others)
  ~~~

  - 별표식은 항상 list 인스턴스가 된다. 남는 원소가 없을 경우에는 빈 리스트가 된다.

- iterator에도 적용할 수 있지만 별표식은 항상 리스트를 만들기 때문에 메모리 부족에 주의해야한다.

  ~~~python
  it = generate_csv()
  header, *rows = it  # 첫번째는 header에 들어가고 나머지는 모두 rows에 들어감
  print('CSV 헤더:', header)
  print('행 수:', len(rows))
  ~~~



### 3. 복잡한 기준을 사용해 정렬할 때는 key 파라미터를 이용하자


- **sort()** 함수에는 key라는 파라미터가 있는데 key는 함수 형태여야한다.

- key 함수에는 정렬 중인 리스트의 원소가 인수로 전달된다. **결과값이 자연스러운 순서가 정해진 데이터 타입이어야한다.**

  ~~~python
  class Tool:
      def __init__(self, name, weight):
          self.name = name
          self.weight = weight
          
      def __repr__(self):
          return f'Tool({self.name!r}, {self.weight})'
      
  tools = [
      Tool('수준계', 3.5),
      Tool('해머', 1.25),
      Tool('스크류드라이버', 0.5),
      Tool('끌', 0.25),
  ]
  
  # sort 구현
  tools.sort(key=lambda x: x.name)
  tools.sort(key=lambda x: x.weight)
  tools.sort(key=lambda x: (x.weight, x.name))  # 튜플은 기본적으로 비교 가능하며 자연스러운 순서가 정해져있다.
  										      # weight으로 정렬 후, name으로 정렬한다.
      
  # 역순 구현
  tools.sort(key=lambda x: (x.weight, x.name), reverse=True)  # 모든 정렬 기준을 역순으로 바꾼다.
  tools.sort(key=lambda x: (-x.weight, x.name))  # 부호 반전 기호를 통해 하나만 역순으로 바꾼다.
  # name은 부호 반전이 안되기 때문에 아래처럼 나눠서 해야한다
  tools.sort(key=lambda x: x.name, reverse=True)
  tools.sort(key=lambda x: x.weight)
  ~~~



### 4. 딕셔너리 삽입 순서에 의존할 때는 조심하자

- 파이썬 3.6부터는 딕셔너리에 삽입된 순서대로 이터레이션을 해준다. (3.5 이하에서는 임의의 순서로 해 줌)

- 만약 이터레이션 자체를 삽입 순서대로 하고 싶지 않으면 커스터마이징된 사전을 정의해주면 된다.

  ~~~python
  from collections.abc import MutableMapping
  
  class SortedDict(MutableMapping):
      def __init__(self):
          self.data = {}
          
      def __getitem__(self, key):
          return self.data[key]
      
      def __setitem__(self, key, value):
          self.data[key] = value
          
      def __delitem__(self, key):
          del self.data[key]
          
      def __iter__(self):
          keys = list(self.data.keys())
          keys.sort()
          for key in keys:
              yield key
              
      def __len__(self):
          return len(self.data)
      
   
  # 정의
  votes = {
      'otter': 1281,
      'polar bear': 587,
      'fox': 863,
  }
  
  def populate_ranks(votes, ranks):
      names = list(votes.keys())
      names.sort(key=votes.get, reverse=True)
      for i, name in enumerate(names, 1):
          ranks[name] = i
  
  def get_winner(ranks):
      return next(iter(ranks))
  
  
  # 실행
  ranks = {}
  populate_ranks(votes, ranks)
  print(ranks)
  winner = get_winner(ranks)
  print(winner)
  
  sorted_ranks = SortedDict()
  populate_ranks(votes, sorted_ranks)  # 흠 여기를 sorted_ranks.data 로 인수를 넣어야하는 줄 알았는데 그냥해도 되네 신기
  print(sorted_ranks)
  winner = get_winner(sorted_ranks)
  print(winner)
  
  # 결과
  {'otter': 1, 'fox': 2, 'polar bear': 3}
  otter
  {'otter': 1, 'fox': 2, 'polar bear': 3}
  fox
  ~~~



### 5. in 구문이나, try 구문보다는 get 함수를 이용하자

- 조금 복잡한데, 정리하면 간단한 경우에는 get을 사용하고, 복잡해질 기미가 보이면 defaultdict을 사용하자

  ~~~python
  votes = {
      '바게트': ['철수', '순이'],
      '치아바타': ['하니', '유리'],
  }
  
  key = '브리오슈'
  who = '단이'
  
  # in 구문을 사용하는 경우
  if key in votes:
      names = votes[key]
  else:
      votes[key] = names = []
  names.append(who)
  print(votes)
  
  # try & except 구문을 사용하는 경우
  try:
      names = votes[key]
  except KeyError:
      votes[key] = names = []
  names.append(who)
  print(votes)
  
  # get 함수를 이용하는 경우
  names = votes.get(key)
  if names is None:
      votes[key] = names = []
  names.append(who)
  print(votes)
  ~~~




### 6. 내부 상태에서 원소가 없는 경우를 처리할 때는 defaultdict을 사용하자

- setdefault도 물론 사용할 수 있지만 defaultdict이 더 간단하게 작용할 수 있다.

  ~~~python
  # setdefault를 이용할 경우
  class Visits:
      def __init__(self):
          self.data = {}
          
      def add(self, country, city):
          city_set = self.data.setdefault(country, set())
          city_set.add(city)
          
  visits = Visits()
  visits.add('러시아', '예카테린부르크')
  visits.add('탄자니아', '잔지바르')
  print(visits.data)
  >> {'러시아': {'예카테린부르크'}, '탄자니아': {'잔지바르'}}
  
  # defaultdict을 이용할 경우 -> 잘 이해 안되는데 무튼 이게 더 좋은 방법이래
  from cllections import defaultdict
  
  class Visits:
      def __init__(self):
          self.data = defaultdict(set)
          
      def add(self, country, city):
          self.data[country].add(city)
          
  visits = Visits()
  visits.add('영국', '바스')
  visits.add('영국', '런던')
  print(visits.data)
  >> {'영국': {'바스', '런던'}}
  ~~~

- 커스터마이징이 필요하다면 dict타입의 하위클래스를 만들고 `__missing__`을 사용해 key값이 없을 때 반환해 줄 값을 설정해 줄 수 있다. 





## <span style="color:green">[비고]</span>

- 2 장 모두 공부하였음.
- 개정판에서는 기존보다 책이 좀 매워진 거 같다.