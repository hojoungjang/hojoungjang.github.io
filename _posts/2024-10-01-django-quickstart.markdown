---
layout: post
title:  "Django 살펴보기 (근데 이제 빠르게랑 간단하게를 곁들인...)"
date:   2024-10-01 15:20:00 +0900
categories: [backend, web framework]
tags: [python, django]
---
토스 Next Developer 공채에 Python developer 포지션이 흥미있어보여서 지원하게 됬다.

채용 프로세스 1차는 과제 전형으로 주어진 시간 안에 사전에 공유된 기술스택을 배이스로한 과제를 해결해 제출하는 방식이다. 기술스택은 Django 가 메인인데 써본적이 없다. 시간이 많이 없었기 때문에 이 친구의 핵심만 빠르게 익힐수 있도록 목표하였다. 

추가로 몇몇 기업에서 장고를 인터널 툴 개발할때 사용하는 것 같은데 이 유즈케이스가 개인적으로 궁금해서 추후 더 알아보고 싶다.

# Django
Django (장고) 는 파이썬 웹 프레임워크이다. 장고를 사용하여 웹 어플리케이션의 간단한 UI 와 백엔드를 구성 할 수 있다. 구성요소가 꽤나 많고 그만큼 러닝커브는 가파르지만 프레임워크에서 기본적으로 제공하고 지원하는 부분이 많아서 규모가 있는 어플리케이션 개발시 이점이 될 것 같다.

## 시작하는 법
프로젝트 디렉토리를 하나 만들고 그 안에서 파이썬 virtual environment 를 생성한다.

```
mkdir my-django-app
cd my-django-app
```

```
python -m venv .venv
source .venv/bin/activate
```

### 장고 설치
```
pip install Django
```

### 장고 프로젝트 만들기
```
django-admin startproject <project name>
```

장고 프로젝트를 만들면 필수 파일들이 생성된다. 여기서 `myproject` 는 위에서 입력한 \<project name\> 이다.
```
myproject/
    manage.py
    myproject/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
```

주요 파일 정리:
* `manage.py` - 장고에서 제공하는 여러 cli command 를 실행할때 사용하는 스크립트. 장고의 cli 프로그램이라고 봐도 거의 무방하다.
* `settings.py` - 장고 애플리케이션의 각종 설정값을 지정 할 수 있다. 예를 들어 사용하는 장고 앱, 템플릿, DB 등.
* `urls.py` - 애플리케이션의 모든 라우트를 지정한다. 장고 앱 전체의 최상위 라우트 정의이고 각각의 장고 앱은 `urls.py` 따로 가지고 있고 독립적으로 URL 경로와 뷰의 관계를 정의해 준다.

ASGI 와 WSGI 는 파이썬 웹 프레임워크 앞단에 연결되는 웹서버의 종류인데 `asgi.py` 와 `wsgi.py` 는 웹 서버와 장고 코드를 연결해 서빙하는 역할을 한다.

### 장고 앱 만들기
장고 앱의 개념은 전체 웹 애플리케이션의 구성하는 한가지의 서비스/역할을 하는 요소 이다.

```
python manage.py startapp <app name>
```

앱을 만들면 몇가지 boilerplate 코드가 생성이된다.

```
myapp/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
```

주요 파일 정리:
* `admin.py` - `models.py` 에 정의한 데이터 모델을 admin 페이지에서 관리가 가능하게 정의하는 용도
* `apps.py` - 앱을 대표하는 클래스/객체 정의. 여기서 정의된 클래스를 `settings.py` 의 `INSTALLED_APPS` 에 추가해주어야 한다.
* `models.py` - 앱의 데이터 모델을 정의하는 용도
* `tests.py` - 테스트 코드를 담는 파일
* `views.py` - view 로직을 담는 파일. 더 설명되겠지만 간단히 생각하면 view 는 애플리케이션 URL 경로와 연결시키고 해당 URL 이 방문이 되었을 때 실행되어 페이지 데이터를 반환하는 코드이다.

## View 만드는 법

### View 로직 정의하기
View 는 클래스나 함수 형태로 `views.py` 에 정의한다.

간단한 예시로는 아래와 같이 함수로 응답 객체를 반환한다.
```py
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world")

# URL path parameter 가 있는 view
def detail(request, item_id):
    return HttpResponse(f"You're looking at item detail {item_id}.")
```

### URLconf 정의하기
URLconf 다음과 같은 형식으로 `urls.py` 에 정의한다.

```py
from django.urls import path
from . import views

urlpatterns = [
    path("", views.index, name="index"),
    path("<int:item_id>/detail/", views.detail, name="detail")
]
```

### Global URLconf 설정하기
프로젝트 root 앱에도 `urls.py` 가 존재한다. 여기에 각 앱의 URLconf 를 포함시켜준다.

다음과 같이 작성 가능하다.
```py
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("myapp/", include("myapp.urls")),
    path("admin/", admin.site.urls),
]
```

`include()` 룰 사용하여 앱의 `urls.py` 모듈을 지정해 준다.

## Model
모델은 애플리케이션의 데이터 모델 정의를 담고 있다.

### 새로운 모델 정의하기
`models.py` 파일에 클래스 정의를 한다. 이때 해당 클래스는 `django.db.models.Model` 를 상속한다.

예시
```py
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")
```

### 모델 활성화 하기
정의된 모델을 기반으로 장고는 데이터베이스 스키마를 만들 수 있고 ORM 을 제공 할 수 있다.

migration 파일을 생성한다.
```
python manage.py makemigrations myapp
```

migration 이 실행할 SQL statements 을 미리 볼 수 있다
```
python manage.py sqlmigrate myapp 0001
```

dry run
```
python manage.py check
```

데이터베이스 스키마 업뎃
```
python manage.py migrate
```

## Admin 기능

### Admin 유저 생성
```
python manage.py createsuperuser
```
username, email 하고 password 를 입력하면 생성이 된다.

### Admin 페이지에서 앱 모델 관리하기
해당 앱의 `admin.py` 파일에 코드 추가
```py
from django.contrib import admin
from .models import Question

admin.site.register(Question)
```

### Admin 페이지 커스터마이징
```py
class QuestionAdmin(admin.ModelAdmin):
    fields = ["pub_date", "question_text"]


admin.site.register(Question, QuestionAdmin)
```

보통 이 패턴을 가져간다. 어드민 페이지에 보여지는 모델에 대하여 `admin.ModelAdmin` 을 상속하는 클래스를 정의하고 특정 클래스 attribute 값을 설정해서 페이지에 보여지는 형태를 변경 할 수 있다.

몇가지 attribute example
* fieldsets - model 추가/변경 폼 레이아웃
    ```py
    fieldsets = [
        (None, {"fields": ["question_text"]}),
        ("Date information", {
            "fields": ["pub_date"],
            "classes": ["collapse"],
        }),
    ]
    ```

* inlines - you can edit another model in the current model. The two models are in a relationship like one-to-many.
    ```py
    inlines = [ChoiceInline]
    ```

* list_display - model 리스트 테이블에서 column 에 표시될 데이터 필드
    ```py
    list_display = ["question_text", "pub_date", "was_published_recently"]
    ```

* list_filter - model 리스트 테이블 필터 기능
    ```py
    list_filter = ["pub_date"]
    ```

* search_fields - model 리스트 테이블 검색 바 기능
    ```py
    search_fields = ["question_text"]
    ```

### 어드민 페이지 탬플릿 자체를 변경도 가능
`admin` 디렉토리에 `templates` 추가하고 오리지널 탬플릿 코드 복붙해서 수정하기.

## Template 사용하기
1. 앱 폴더 안에서 `templates` 폴더 만들기
2. `settings.py` 에서 `TEMPLATES` > `APP_DIRS` 옵션 `True` 로 세팅

### Template namespacing
`templates` 폴더안에 추가적으로 앱 이름의 폴더 만들고 안에 템플릿 HTML 파일 생성.

예를 들어 `myapp/templates/myapp/index.html`

### 템플릿 작성
템플릿은 특별한 syntax 를 사용해 작성.
괄호를 열어 Python 코드와 템플릿 파일에 전달된 객체 사용 가능.

예를 들어
```
{% raw %}
{% if items_list %}
    <ul>
    {% for item in items_list %}
        <li><a href="/polls/{{ item.id }}/">{{ item.description }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No items are available.</p>
{% endif %}
{% endraw %}
```

[템플릿 가이드](https://docs.djangoproject.com/en/5.1/topics/templates/)

### View 에서 템플릿 사용
`render()` 를 사용한 방법
```py
from django.shortcuts import render
from .models import Item

def index(request):
    items_list = Question.objects.order_by("-create_date")[:5]
    context = {"items_list": items_list}
    return render(request, "myapp/index.html", context)
```

### 404 에러
`Http404()` 를 이용한 404 에러 전달
```py
from django.http import Http404
from django.shortcuts import render

from .models import Item

def detail(request, item_id):
    try:
        item = Item.objects.get(pk=item_id)
    except Item.DoesNotExist:
        raise Http404("Item does not exist")
    return render(request, "myapp/detail.html", {"item": item})
```

`get_object_or_404()` 를 이용한 404 에러 전달
```py
from django.shortcuts import get_object_or_404, render
from .models import Item

def detail(request, item_id):
    item = get_object_or_404(Item, pk=item_id)
    return render(request, "myapp/detail.html", {"item": item})
```

There’s also a get_list_or_404() function, which works just as get_object_or_404() – except using filter() instead of get(). It raises Http404 if the list is empty.

### URLconf 에 정의한 이름 가지고 URL 경로 참조하기
```html
{% raw %}
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
{% endraw %}
```

`url` 키워드 뒤에 URL 이름 붙이고 추가로 path param 붙이기

### URL 이름 네임스페이싱 하기
urls.py 에 `app_name` variable 추가. 값이 namepsace value 가 됨

예를 들어
```py
app_name = "polls"
```

namespace 를 이용해서 템플릿 이름 참조
```html
{% raw %}
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
{% endraw %}
```

<네임스페이스:이름> 형태로 참조 가능
