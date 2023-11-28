<h1>Django Rest framework</h1>

<h2>✔️[231103] 3일차</h2>

`Django rest framework` API 만들기 (tutorial 3까지)

📍 **DRF(Django Rest Framework)** <br>
  👉🏻 Django안에서 REstfulAPI 서버를 쉽게 만들게 도와주는 오픈소스 라이브러리
***

<h6>📌[2일차] 내용 이어서 진행</h6>

<h5> 1) 웹 API에 대한 첫 번째 시도 테스트</h5>

`exit() or quit()`  서버 나가기 

```python
python manage.py runserver
```

<h6> **다른 터미널 창 열기** 컬 또는 httpie 사용하여 API 테스트 할 수 있다.<br>
Httpie는 Python을 작성된 http 클라이언트 </h6>

```python
# pip 사용
pip install httpie
http http://127.0.0.1:8000/snippets/

# 해당 ID를 입력하면 특정 조각을 얻을 수 있다.(ID = 2)
http http://127.0.0.1:8000/snippets/2/
```
👉🏻 스니펫 목록들을 볼 수 있다.

📍아래와 같이 `terminal` 두개 열고, 왼쪽은 서버, 오른쪽은 클라이언트 -> 오른쪽에서 수정을 하면 왼쪽이 받아진다.📍
![image](https://github.com/jinnyyyyyy/study_django/assets/125633789/9d7b2205-e154-494d-af81-cc5ddedb72fc)


<h4> Tutorial 2 요청과 응답 </h4>

<h5> 이해 후 정리</h5>

<h4> Tutorial 3 클래스 기반 뷰</h4>

```
# 클래스 기반 뷰를 사용하여 API 재작성
# 믹스인 사용
# 일반 클래스 기반 뷰 사용
총 3가지를 확인했다. 점점 코드가 짧아지는 것을 확인 할 수 있었다.
```

<h5> 일반 클래스 기반 뷰 사용 </h5>

```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework import generics


class SnippetList(generics.ListCreateAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer


class SnippetDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
```

👉🏻 `view.py` REST 프레임워크는 모듈을 더욱 축소하는데 사용 <br>
사용 할 수 있는 이미 혼합된 일반 뷰를 제공
