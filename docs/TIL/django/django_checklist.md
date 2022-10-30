---
layout: default
title: Django 프로젝트 만들 때 체크리스트
parent: Django
grand_parent: TIL
---

# Django 프로젝트 만들 때 체크리스트
{: .no_toc }

## TABLE OF CONTENTS
{: .no_toc .text-delta }

1. TOC
{:toc}

---
## 프로젝트 시작
1. `django-admin startproject <project name> .`
	* 맨 마지막에 `.` 붙이는 것 잊지 말기
	* base dir에 manage.py가 있는 것을 꼭 확인해야 한다
2.  `python manage.py startapp <app name>` 
	* 프로젝트에 앱을 추가한 뒤, `settings.py` - `INSTALLED_APPS` 에 앱을 추가해야한다.
3. project의  `urls.py`  수정
```python
from django.urls import include

urlpatterns = [
    path('appname/', include('appname.urls'),
]
``` 
4. app의 `urls.py`  수정
	* `app_name`  잊지말기
```
from django.urls import path
from . import views

app_name = 'appname'
urlpatterns = []
```
5. base dir에 `templates`  폴더 만들고 `base.html`  작성 후 `settings.py` 에 추가하기
	* 다른 템플렛들의 기초가 될 템플렛을 만들어둔다.

## Accounts
1. `models.py`
```python
from django.db import models
from django.contrib.auth.models import AbstractUser


class User(AbstractUser):
    pass
```
2. `admin.py`에 user 모델 등록하기
```python
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from .models import User


admin.site.register(User, UserAdmin)
```
3. `forms.py`
	* 기존의 `UserCreationForm`을 바꿔야 한다.
```python
from django.contrib.auth import get_user_model
from django.contrib.auth.forms import UserCreationForm, UserChangeForm


class CustomUserCreationForm(UserCreationForm):

    class Meta(UserCreationForm.Meta):
        model = get_user_model()
        fields = UserCreationForm.Meta.fields + ('email',)


class CustomUserChangeForm(UserChangeForm):

    class Meta(UserChangeForm.Meta):
        model = get_user_model()
        fields = ('email', 'first_name', 'last_name',)
```
4. templates
	* method에 따라 csrf 토큰을 적절히 활용해야 한다.
	* form을 사용하는 경우 `{{ form.as_p }}`를 통해 페이지에 form이 문단 형식으로 보이게 한다.

5. `views.py`
	* django에서 제공하는 form이 아니라 custom으로 작성한 form을 사용할 것이기 때문에 로그인, 로그아웃의 이름을 바꿔야한다.
	* 회원가입, 비밀번호 변경은 기존의 form을 사용해도 된다.
	* import list
```python
from django.shortcuts import render, redirect
from django.contrib.auth import login as auth_login
from django.contrib.auth import logout as auth_logout
from django.contrib.auth import update_session_auth_hash
from django.contrib.auth.forms import AuthenticationForm, PasswordChangeForm
from django.contrib.auth.decorators import login_required
from django.views.decorators.http import require_POST, require_http_methods
from .forms import CustomUserCreationForm, CustomUserChangeForm
```
	* 적절한 데코레이터를 사용해야 한다.
		* `@require_http_methods(['GET', 'POST'])`
		* `@login_required`
6. `settings.py` 에 추가하기
`AUTH_USER_MODEL = 'accounts.User'`

## App
1. `models.py`
	* `models.py` 를 수정 한 후에는 makemigrations, migrate 잊지말기
	* 혹시 db 관련 오류가 생겼다면 migration 폴더 내의 migration과 `db.sqlite3`을 삭제한 뒤 다시 makemigrations 하면 된다
	* user를 foreignkey로 가져오는 경우
		* `from django.conf import settings`
		* `user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)`
	* django의 헷갈리는 model fields
		* `models.DateTimeField(auto_now_add=True)`
		* `models.ForeignKey(modelname, on_delete=models.CASCADE, related_name='relatedname')`
		* `models.ManyToManyField('self', symmetrical=False, related_name='other_user')`
2. `admin.py`에 모델 등록하기
```python
from django.contrib import admin
from .models import AppModel


admin.site.register(AppModel)
```
3. templates
	* accounts와 비슷
	* `<p>{{ comments|length }}개의 댓글이 있습니다.</p>`
4. `views.py`
	* context에 필요한 데이터를 dict 형태로 보낼 수 있다
	* foreinkey를 사용하는 model을 form을 통해 작성/수정하는 경우 
```
article = form.save(commit=False)
article.user = request.user
article.save()
```

	* 삭제는 `comment.delete()`
