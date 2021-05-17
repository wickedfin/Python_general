# 1장 : 파이썬다운 생각

<div style="text-align: right"> 마지막 수정 : 2020-12-29

[TOC]

## <span style="color:green">[개요]</span>

- **파이썬다운 생각**이란 표현이 참 마음에 든다.
- 이 책은 파이썬 코딩을 함에 있어서 유용한 방법이나 기술 등을 기재해두었다.
- 개정 2판으로 공부하였다.





## <span style="color:green">[본문]</span>

### 1. bytes & str

- python3에서 bytes와 str은 심지어 빈 문자열이라 하더라도 같지 않다. 따라서 반드시 하나의 타입으로 맞출 필요가 있다.

  - 타입이 같다면 + 연산자를 통해 붙일 수 있다. (<, > 같은 이항연산자를 통해 비교도 할 수 있다.)

- bytes는 바이너리 값을, str은 유니코드 문자를 가지고 있다.

  ~~~python
  def to_str(bytes_or_str):
      if isinstance(bytes_or_str, bytes):
          value = bytes_or_str.decode('utf-8')
      else:
          value = bytes_or_str
    return value
  
  
  def to_bytes(bytes_or_str):
      if isinstance(bytes_or_str, str):
          value = bytes_or_str.encode('utf-8')
      else:
          value = bytes_or_str
      return value
  ~~~

- python3의 내장 함수 **open**은 기본적으로 str 타입. 즉, **UTF-8** 인코딩을 사용한다.

  - 따라서 바이너리 데이터를 읽으려면 쓰기 모드가 'wb'로 되어야한다. 읽기도 마찬가지

  ~~~python
  # 시스템 인코딩을 검사하는 방법
  python3 -c 'import locale; print(locale.getpreferredencoding())'
  # 디폴트 인코딩이 의심스러운 경우에는 명시적으로 encoding 파라미터를 전달해주자
  with open('data.bin', 'r', encoding='cp1252') as f: # 보통은 utf-8
      data = f.read()
  ~~~



### 2. 문자열 형식화에 관해서

- 파이썬에서 주로 사용하는 문자열 형식화에는 크게 3가지 방식이 있다.

  > C 스타일 형식 문자열, str.format, f-문자열

- 이 중 f-문자열이 가장 간결하고 표현력도 좋으니 f-문자열을 사용하는 습관을 들이도록하자

  ~~~python
  # 같은 것을 표현할 때
  f_string = f'{key:<10} = {value:.2f}'  # 가장 간결
  c_tuple  = '%-10s = %.2f' % (key, value)  # c 스타일 형식 
  str_args = '{:<10} = {:.2f}'.format(key, value)  # format 스타일 형식
  str_kw   = '{key:<10} = {value:.2f}'.format(key=key, value=value)
  c_dict   = '%(key)-10s = %(value).2f' % {'key': key, 'value': value}
  
  # 예시1
  for i, (item, count) in enumerate(pantry):
      print(
          f'#{i+1}: '
          f'{item.title():<10s} = '
          f'{round(count)}'
      	 )
  
  # 예시2
  places = 3
  number = 1.23456
  print(f'내가 고른 숫자는 {number:.{places}f}')
  ~~~
  
- f-문자열 안에서 객체를 사용하면 해당 객체의 `__str__()`메서드가 호출된 결과가 삽입된다.

  - 만약 `__repr__()`메서드가 호출된 결과를 삽입하고 싶다면 `!r`을 뒤에 붙여주면 된다.

  ~~~python
  from datetime import date
  print(f"오늘은 {data.today()} 입니다.")
  >> '오늘은 2021-02-27 입니다.'
  print(f"오늘은 {data.today()!r} 입니다.")
  >> '오늘은 datetime.date(2021, 2, 27) 입니다.'
  ~~~

  



### 3. 인덱스를 사용하는 대신 대입을 사용한 언패킹을 하자

- 아래처럼 인덱스 사용을 피하면 더 깔끔해보인다.

  ~~~python
  snacks = [('베이컨', 350), ('도넛', 240), ('머핀', 190)]
  # 인덱스를 이용하는 경우 -> 피해야하는 경우
  for i in range(len(snacks)):
      item = snacks[i]
      name = item[0]
      calories = item[1]
      print(f'#{i+1}: {name}은 {calories} 칼로리입니다.')
      
  # enumerate을 활용해 대입하는 경우 -> 좋은 경우
  for rank, (name, calories) in enumerate(snakcs):
      print(f'#{rank+1}: {name}은 {calories} 칼로리입니다.')
  ~~~

- 추가로 언패킹을 잘 활용하면 임시 변수를 사용하지 않고도 원소 바꾸기를 할 수 있다.

  ~~~python
  def bubble_sort(a):
      for _ in range(len(a)):
          for i in range(1, len(a)):
              if a[i] < a[i-1]:
                  a[i-1], a[i] = a[i], a[i-1]  # 맞바꾸기
  ~~~




### 4. 기타

- 파이썬에선 for이나 while등의 반복문 뒤에 else 구문을 사용할 수 있으나, 가독성이 떨어지므로 사용하지 말도록 하자
- **왈러스 연산자**라고 하는 새로운 대입 구문이 파이썬 3.8부터 도입되었다.
  - 하지만 3.8은 너무 최신 버전이라 나는 일단은 사용을 보류할 예정





## <span style="color:green">[비고]</span>

- 1 장 모두 공부하였음.
- 랩사람들마다 평가가 좀 갈리기는 하는데, 개인적으론 재밌고 유용한 책인 거 같다.