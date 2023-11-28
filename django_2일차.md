<h1>Django Rest framework</h1>

<h2>✔️[231102] 2일차</h2>

`Django rest framework` API 만들기 (tutorial 3까지) 
    
📍 **DRF(Django Rest Framework)** <br>
  👉🏻 Django안에서 REstfulAPI 서버를 쉽게 만들게 도와주는 오픈소스 라이브러리
***

📌 **사용하는 이유**

- 웹 브라우저 API는 범용성이 큼 -> 개발을 쉽게 만들어준다.<br>
- 시리얼라이즈 기능을 제공(DB data -> JSON)

📌 **Serializer**

- Serializer란 말 그대로 직렬화 하는 클래스
- 사용자의 DB안에 사용자 프로필사진, 이메일, 이름, 성별이 있다고 가정하면,<br>
 사용자 모델 인스턴스를 JSON 또는 Dictionary 형태로 직렬화

🛠️**ORM 와 동일하게 (anaconda prompt) 가상환경에서 실행**🛠️ <br>
  👉🏻 `django_test` 환경 만들고, 파일명도 동일하게 <br>
  `cd django_test`  `cd tutorial`파일 안에서 실행 (접속할 때 참고하기)

<h5> 1) 가상환경 만든 후, 패키지 요구사항 설치 </h5>
  
```python
pip install django
pip install djangorestframework
pip install pygments
```

<h5> 2) 새프로젝트 만들기 </h5>

```python
django-admin startproject tutorial
cd tutorial
# 완료 되면 간단한 웹 API 만들기
python manage.py startapp snippets
```
<h5>✂️탐색기 -> tutorial -> settinngs.py -> INSTALLED_APPS 파일 편집✂️</h5>

`rest_framework, snippets`  추가<br>

```python
INSTALLED_APPS = [
    'rest_framework',
    'snippets',
    ...
]
```

<h5> 3) 모델 만들기</h5>

`snippets - models.py`  파일을 편집

```python
from django.db import models
from pygments.lexers import get_all_lexers
from pygments.styles import get_all_styles

LEXERS = [item for item in get_all_lexers() if item[1]]
LANGUAGE_CHOICES = sorted([(item[1][0], item[0]) for item in LEXERS])
STYLE_CHOICES = sorted([(item, item) for item in get_all_styles()])


class Snippet(models.Model):
    created = models.DateTimeField(auto_now_add=True)
    title = models.CharField(max_length=100, blank=True, default='')
    code = models.TextField()
    linenos = models.BooleanField(default=False)
    language = models.CharField(choices=LANGUAGE_CHOICES, default='python', max_length=100)
    style = models.CharField(choices=STYLE_CHOICES, default='friendly', max_length=100)

    class Meta:
        ordering = ['created']
```

<h5> 4) 초기 마이그레이션 생성과 처음으로 DB 동기화</h5>

```python
python manage.py makemigrations snippets
python manage.py migrate snippets
```

<h5> 5) Serializer 클래스 만들기</h5> 

`snippets -> serializer.py` 파일 만들기

```python
from rest_framework import serializers
from snippets.models import Snippet, LANGUAGE_CHOICES, STYLE_CHOICES


class SnippetSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(required=False, allow_blank=True, max_length=100)
    code = serializers.CharField(style={'base_template': 'textarea.html'})
    linenos = serializers.BooleanField(required=False)
    language = serializers.ChoiceField(choices=LANGUAGE_CHOICES, default='python')
    style = serializers.ChoiceField(choices=STYLE_CHOICES, default='friendly')

    def create(self, validated_data):
        """
        Create and return a new `Snippet` instance, given the validated data.
        """
        return Snippet.objects.create(**validated_data)

    def update(self, instance, validated_data):
        """
        Update and return an existing `Snippet` instance, given the validated data.
        """
        instance.title = validated_data.get('title', instance.title)
        instance.code = validated_data.get('code', instance.code)
        instance.linenos = validated_data.get('linenos', instance.linenos)
        instance.language = validated_data.get('language', instance.language)
        instance.style = validated_data.get('style', instance.style)
        instance.save()
        return instance
```

<h6>
  1. serializer 클래스의 첫번째 부분은 직렬화/역직렬화 되는 필드로 정의 <br>
  2. create(), update() 호출 시 완전한 기능을 갖춘 인스턴스가 생성 또는 수정 serializer.save() <br>
  3. Serializer 클래스는 Django 클래스와 매우 유사,<br>
  같은 Form 다양한 필드에 유사한 유효성 검사 플래그 포함(required, max_length, default) <br>
</h6>

<h5> 6) 직렬 변환기 작업</h5>

```python
python manage.py shell

# 아래 코드 입력
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser

snippet = Snippet(code='foo = "bar"\n')
snippet.save()

snippet = Snippet(code='print("hello, world")\n')
snippet.save()
```

<h5> 7) 인스턴스 중 하나를 직렬화 -> 데이터 유형 변환</h5>

```python
# 직렬화
serializer = SnippetSerializer(snippet)
serializer.data

# data 유형 변환
content = JSONRenderer().render(serializer.data)
content

# 역직렬화
import io

stream = io.BytesIO(content)
data = JSONParser().parse(stream)
```

<h6>
   SnippetSerializer 사용하여, snippet 객체를 직렬화 -> serializer.data로 얻어내기 <br>
  1. 직렬화 클래스는 모델클래스의 데이터를 원하는 형식으로 변환 -> JSON형식으로 변환 <br>
  2. 'serializer.data'는 직렬화 된 데이터 ''snippet'객체의 필드를 JSON형식으로 변환한 결과 저장 <br>
  Python dictionary 형식으로 표현, 각 필드의 이름과 값을 가지고 있음 <br>
  3. 'snippet' 객체의 'linenos' 필드가 'False' 값을 가지고 있음 다른 필드들도 유사한 방식으로 직렬화 <br>
  📌 SnippetSerializer를 사용하여 snippet 객체를 JSON 형식으로 직렬화한 후, <br>
  직렬화된 데이터를 Python 사전으로 나타내어 serializer.data에 저장
</h6>

<h5> 8) 기본 데이터 유형을 완전히 채워진 객체 인스턴스로 복원</h5>

```python
serializer = SnippetSerializer(data=data)
serializer.is_valid()
# True 결과값 출력
serializer.validated_data
# OrderedDict([('title', ''), ('code', 'print("hello, world")\n'), ...
serializer.save()
# <Snippet: Snippet object>
```

<h5> 9) 모델 인스턴스 대신 쿼리 세트 직렬화</h5>

```python
serializer = SnippetSerializer(Snippet.objects.all(), many=True)
serializer.data
```
👉🏻 `many=True`  직렬 변환기 인수에 플래그를 추가하면 된다.


<h5> 10) ModelSerializer 사용</h5>

👉🏻    `Form` 은 Django가 클래스와 클래슬르 모두 제공하는 것

✔️ 클래스를 사용하여 직렬 변환기를 리팩토링 하는 방법

👉🏻    `Snippets/serializers.py` 열고, `SnippetSerializer` **변경해주기**

```python
class SnippetSerializer(serializers.ModelSerializer):
    class Meta:
        model = Snippet
        fields = ['id', 'title', 'code', 'linenos', 'language', 'style']
```
👉🏻   `ModelSerializer` 클래스는 단순히 직렬 변환기 클래스를 생성

  - 자동으로 결정된 필드 집합<br>
  - `create()` 및 메소르에 대한 간단한 기본 구현 `update()`

👉🏻 Django shell 열기 `python manage.py shell` 하고 난 후 아래 코드 실행

 ```python
from snippets.serializers import SnippetSerializer
serializer = SnippetSerializer()
print(repr(serializer))
```
<h5> 11) Serializer를 사용하여 일반 Django 뷰 작성</h5>
 <h6>REST 프레임워크의 다른 기능을 사용하지 않고, 일반 Django 뷰로 뷰를 작성</h6>

  파일을 편집  `snippets/view.py`  다음 추가

```python
from django.http import HttpResponse, JsonResponse
from django.views.decorators.csrf import csrf_exempt
from rest_framework.parsers import JSONParser
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer

@csrf_exempt
def snippet_list(request):
    """
    List all code snippets, or create a new snippet.
    """
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return JsonResponse(serializer.data, safe=False)

    elif request.method == 'POST':
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data, status=201)
        return JsonResponse(serializer.errors, status=400)

@csrf_exempt
def snippet_detail(request, pk):
    """
    Retrieve, update or delete a code snippet.
    """
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return HttpResponse(status=404)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return JsonResponse(serializer.data)

    elif request.method == 'PUT':
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(snippet, data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data)
        return JsonResponse(serializer.errors, status=400)

    elif request.method == 'DELETE':
        snippet.delete()
        return HttpResponse(status=204)
```
 
<h5> 12) 뷰 연결하기 위해 파일 생성</h5>

`snippets/urls.py`

```python
from django.urls import path
from snippets import views

urlpatterns = [
    path('snippets/', views.snippet_list),
    path('snippets/<int:pk>/', views.snippet_detail),
]
```

`tutorial/urls.py` 코드 조각앱의 URL을 포함하려면 파일에 루트urlconf를 연결

```python
from django.urls import path, include

urlpatterns = [
    path('', include('snippets.urls')),
]
```
✔️ 잘못된 형식을 보내 `JSON` 거나 뷰가 처리하지 않는 메서드로 요청이 이루어진 경우 

500"서버오류" 응답으로 표시

