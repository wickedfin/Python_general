## <span style="color:lightyellow">8장 : 강건성과 성능</span> 

<div style="text-align: right"> 마지막 수정 : 2020-12-30




### <span style="color:lightgreen">[개요]</span>

- 일부 내용이 8장으로 옮겨가서 내용 옮겨둠






### <span style="color:lightgreen">[본문]</span>

- **예외처리 과정에 대하여**

  - **try/except/else/finally**의 모든 구문을 활용해 이상적인 예외처리를 구현하자.

    ~~~python
    handle = open('/tmp/random_data.txt')
    try:
        data = handle.read()  # 여기서 예외 오류가 있어도
    finally:
        handle.close()        # 여기서는 정상적으로 파일 종료시킴
    
        
    def load_json_key(data, key):
        try:
            result_dict = json.loads(data)  # 예외 오류가 일어날 수 있는 부분
        except ValueError as e:
            raise KeyError from e           # 예외 발생의 종류를 바꿔주는 부분 (user-friendly)
        else:
            return result_dict[key]         # else를 활용하면 try구문의 내용을 최소화 할 수 있음
    ~~~

  - 위처럼 **finally**는 어떤 상황에서도 실행된다.

  - **else**는 **try**문이 예외를 일으키지 않으면 실행된다.






### <span style="color:lightgreen">[마치며]</span>

- 3장 이제 읽기 시작함. 2장 내용은 형식을 위해 남겨둠. 읽으면서 지울 것.