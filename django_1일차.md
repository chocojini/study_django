<h1>Django</h1>



<h2>✔️[231101] 1일차</h2>

 - DB작업 하기 위해
    `Django와 python을 이용한 postgreSQL` ORM 사용해보기
***


<h3> 🛠️anaconda prompt 열고, 실행🛠️</h3>

***

<h5> 1) 가상환경 만들기 -> python3.9 </h5>

```python
conda create -n bolt_db python==3.9
```
<h5> 2) 생성확인 </h5>

```python
conda env list
```

<h5> 3) 가상환경 진입</h5>

```python
conda activate bolt_db (내가 만든 가상환경에 접속)
```

<h5> 4) (진입 상태) 장고 설치, psycopg2설치 </h5>

```python
pip install django==4.1
conda install psycopg2
```

<h5> 5) 폴더 만들고, 해당폴더로 이동, 장고 프로젝트 생성 후 폴더로 이동 </h5>

```python
mkdir bolt_db
cd bolt_db
django-admin startproject backend
cd backend
```

<h5> 6) 서버 구동 확인, 종료 </h5>

```python
# 서버 구동 확인
python manage.py runserver

#종료
ctrl + c
```
<h5> 7) vscode 실행 </h5>

```python
(backend 폴더에서) code .
# vscode 에서 ctrl + ` 누르면 터미널 창이 열리고 cmd로 열기

# 가상환경 활성화
conda ativate bolt_db

# 서버 구동 확인
python manage.py runserver

#종료
ctrl + c
```

<h5> 8) backend project(folder 아님) .env 파일 만들기</h5>

```python
DB_NAME
DB_USER
DB_PASSWORD
DB_HOST
DB_PORT
입력해준다.
```

<h5> 9) accounts 앱 생성</h5>

```python
python manage.py startapp accounts

👉🏻 backend setting.py에서 34번째 줄 'accounts', 추가 해주기
👉🏻 setting.py에서 DATABASES부분 8)내용으로 수정 하기 
```
<h5> 10) Django의 Inspectdb를 이용해 외부 DB에 대해 ORM 쓰는 방법</h5>

```python
python manage.py inspectdb > models.py
```
<h5> 11) shell 실행 </h5>

```python
# models.py를 accounts폴더 models.py에 덮어쓰기
# models.py에서 xtrl + d 이용하여, 각 클래스(DB) id부분 전부 주석 처리
# app.py에서 "default_auto_field = "django.db.models.BigAutoField" 주석처리

# django shell을 이용하여 db에 저장된 값을 CRUD 
python manage.py shell
from accounts.models import AccountsAddress
```

<h5> 12) ORM </h5>

```python
# AccountsAddress에 저장된 데이터 모두 가져오기
AccountsAddress.objects.all()

# 변수 사용
address = AccountsAddress.objects.all()
address[0]
address[0].address_main
```

<h5> 13) filter 사용 </h5>

```python
AccountsAddress.objects.filter(is_default=True)
👉🏻 is_default=True 인 경우만 데이터 가져오기

from django.db.models import Q
👉🏻 여러개 조건 사용
```

<h5> 14) ORM으로 where절 조건문 추가 </h5>

```python
# id=10, 35 인 것
AccountsAddress.objects.filter(Q(id=10) | Q(id=35))
(select절사용)
SELECT * FROM AccountsAddress WHERE id=10 or id=35
```

<h5> 15) 변수에 넣고, for문 이용 </h5>

``` python
q = AccountsAddress.objects.filter(Q(id=10) | Q(id=35))
for i in q:
    print(i.user_id) (enter 2번)
```

<h5> 16) 변수에 넣지 않고 출력 </h5>

```python
for i AccountsAddress.objects.create(address_main__icontains = "서울"):
    print(i.address_main)
👉🏻 '__icontains' : __ 언더바를 2개 사용 -> AccountsAddress 모델에서 address_main 필드를 사용하고, 
__icontains 연산자를 사용하여 "서울"을 포함하는 주소를 필터링하고 출력
```

<h5> 17) 실제 DB에 CREATE 진행하는 방법, 확인</h5>

```python
AccountsAddress.objects.create(address_main = "서울 마포구",
address_detail = "10층", postcode = "05321", is_default=False, user_id=90)

# 위의 명령어를 입력하면 id 번호가 나온다. ex) id=901
AccountsAddress.objects.get(id=901)

# 변수에 저장
instance = AccountsAddress.objects.get(id=901)

# update
instance.address_detail="20층"

#update 후, 꼭 save 해주기
instance.save()
```

<h5> 18) dict 이용하여 update, save 한번에 하는 법</h5>

```
data = {"address_detail" : "25층", "address_main" : "여의도"}
instance = AccountsAddress.objects.get(id=901)
AccountsAddress.objects,filter(id=901).update(**data)
```
