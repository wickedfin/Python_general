# 4장 : 컴프리헨션과 제너레이터

<div style="text-align: right"> 마지막 수정 : 2020-12-29


[TOC]

## <span style="color:green">[개요]</span>

- 개정판에서는 이쪽 내용이 좀 많이 추가되었음.





## <span style="color:green">[본문]</span>

- **내포에 관하여**

  - 파이썬 기초쪽에서 이미 언급하였지만 한 번 더 요약해서 적는다.

    ~~~python
    a = [1, 2, 3, 4, 5]
    squares = [x**2 for x in a]       # good
    squares = map(lambda x: x**2, a)  # bad
    
    even_squares = [x**2 for x in a if x % 2 == 0]              # good
    list(map(lambda x: x**2, filter(lambda x: x % 2 == 0, a)))  # bad
    
    # 사전과 세트에서도 비슷한 것을 할 수 있다
    chile_ranks = {'ghost': 1, 'habanero': 2, 'cayeene': 3}
    rank_dict = {rank: name for name, rank in chile_ranks.items()}
    child_len_set = {len(name) for name in rank_dict.values()}
    ~~~

  - 리스트 컴프리헨션에서 웬만하면 표현식을 두 개 넘게 쓰지 말자 (조건문, 루프문 포함 최대 2개)

  - 표현식이 2개가 넘어가게 되면 일반적인 조건문과 루프문을 (if, for)을 사용하자

  

- **generator에 대하여**

  - 파일이 너무 클 때에는 하나씩 불러오는 generator을 사용하자

    ~~~python
    value = [len(x) for x in open("/tmp/my_file.txt")]  # 파일이 너무 크면 문제 생길 수 있음
    it = (len(x) for x in open("/tmp/my_file.txt"))     # 이런 식으로 하나씩 참조 가능
    print(next(it))
    print(nexx(it))
    # generator의 연계
    roots = ((x, x**0.5) for x in it)
    ~~~

  - enumerate과 zip을 적극적으로 활용하는 것이 좋다.  이 중은 zip은 generator이다. (python2에서는 튜플임)

  - zip은 여러 이터레이터에 대해 나란히 루프를 수행할 때 유용하다.

  - zip은 포함되는 리스트들끼리의 길이가 같을 때만 사용하는 것이 좋다. 다르다면 zip_logest의 사용을 고려해야한다.
  





### <span style="color:lightgreen">[마치며]</span>

- 아직 더 봐야함