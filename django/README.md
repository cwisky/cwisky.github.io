# 계층 구조형 BBS(Bulletin Board System)
## 프로젝트의 구조 추천
bbs_project/  
├── post_create/       # 글 작성  
├── post_edit/         # 글 수정  
├── post_delete/       # 글 삭제  
├── post_search/       # 글 검색  
├── post_list/         # 글 목록  
├── common/            # 공통 기능(모델, 템플릿 필터 등)  
└── bbs_project/       # 프로젝트 설정  
* 기능별 앱 분리는 재사용성, 유지보수성, 협업 효율성을 높이는 설계 전략이며, Django의 권장 아키텍처 방향성과도 일치함.
* 다만, 프로젝트의 규모와 팀의 구조에 따라 분리 범위는 유연하게 조절하는 것이 좋음
* 작은 규모의 프로젝트라면 앱을 너무 세분화하면 오히려 복잡도만 증가하고 관리 부담이 생길 수 있음.
* 기능이 서로 강하게 연결된 경우(예: 글 수정/삭제/작성 모두 동일한 모델에 접근)라면 하나의 앱(posts)으로 묶는 것이 더 적절할 수 있음

## 계층 구조형 BBS(게시판) 1단계
* 1단계 작업 : 글쓰기, 글목록, 상세보기
* Threaded Comments 또는 Nested Comments, 트리형 구조 게시판이라고 불림
* Django에서 ForeignKey를 활용하여 쉽게 구현 가능

## 프로젝트, 앱 생성 및 프로젝트 설정
1. 프로젝트 및 앱 생성
  * django-admin startproject mybbs
  * cd mybbs
  * python manage.py startapp board
2. 앱 등록
```python
# mybbs/settings.py
INSTALLED_APPS = [
    ...
    'board.apps.BoardConfig',
]
```
3. Model 클래스 Post 선언
* 한개의 게시글을 데이터베이스에 저장하거나 게시글을 데이터베이스로부터 가져오는 역할
```python
# board/models.py
from django.db import models

class Post(models.Model):
    title = models.CharField(max_length=200)
    author = models.CharField(max_length=100)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    hit = models.PositiveIntegerField(default=0)
    attachment = models.FileField(upload_to='attachments/', null=True, blank=True) # MEDIA_ROOT/attachments/

    # 부모글 번호 (자기 자신을 참조)
    parent = models.ForeignKey('self', null=True, blank=True, on_delete=models.CASCADE, related_name='replies')

    def __str__(self):
        return self.title
```
4. 데이터베이스 테이블 생성(마이그레이션 생성 및 적용)
* python manage.py makemigrations
* python manage.py migrate

5. 모델을 관리자 페이지에서 볼 수 있도록 등록
```python
# board/admin.py
from django.contrib import admin
from .models import Post

admin.site.register(Post)
```

5. 뷰(View) 작성
```python
# board/views.py
from django.shortcuts import render, get_object_or_404, redirect
from .models import Post
from .forms import PostForm

def post_list(request):
    posts = Post.objects.filter(parent=None).order_by('-created_at')
    return render(request, 'board/post_list.html', {'posts': posts})

def post_detail(request, pk):
    post = get_object_or_404(Post, pk=pk)
    post.hit += 1
    post.save()
    return render(request, 'board/post_detail.html', {'post': post})

def post_create(request, parent_id=None):
    if request.method == "POST":
        form = PostForm(request.POST, request.FILES)
        if form.is_valid():
            post = form.save(commit=False)
            if parent_id:
                post.parent = Post.objects.get(id=parent_id)
            post.save()
            return redirect('board:post_list')
    else:
        form = PostForm()
    return render(request, 'board/post_form.html', {'form': form})
```
6. 폼 클래스(django.forms.ModelForm) 선언
   * 이용자의 폼 데이터를 받아서 모델 클래스(Post)에 연결하거나 모델 인스턴스를 이용자 폼에 전달함
```python
# board/forms.py
from django import forms
from .models import Post

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'author', 'content', 'attachment']
```
7. 앱 URL 설정
```python
# board/urls.py
from django.urls import path
from . import views

app_name = 'board'

urlpatterns = [        # 프로젝트 urls.py의 url 접두어를 제거한 나머지 url 등록
    path('', views.post_list, name='post_list'),
    path('post/<int:pk>/', views.post_detail, name='post_detail'),
    path('post/new/', views.post_create, name='post_new'),
    path('post/<int:parent_id>/reply/', views.post_create, name='post_reply'),
]
```
8. 프로젝트 URL 설정
```python
# mybbs/urls.py
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('board/', include('board.urls')),   # 모든 'board/'요청은 앱 urls.py의 path('', ...) 와 연결됨
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```
9. 템플릿 구성(계층구조 출력)
```django
<!-- board/templates/board/post_list.html -->
<h2>게시글 목록</h2>
<a href="{% url 'board:post_new' %}">새 글 쓰기</a>
<ul>
  {% for post in posts %}
    <li>
      <a href="{% url 'board:post_detail' post.pk %}">{{ post.title }}</a> - {{ post.author }} - {{ post.created_at }}
      <a href="{% url 'board:post_reply' post.pk %}">답글</a>
      {% include 'board/post_replies.html' with replies=post.replies.all %}
    </li>
  {% endfor %}
</ul>
```
10. 위의 템플릿에 포함될 조각 파일
```django
<!-- board/templates/board/post_replies.html -->
<ul style="margin-left: 20px;">
  {% for reply in replies %}
    <li>
      <a href="{% url 'board:post_detail' reply.pk %}">{{ reply.title }}</a> - {{ reply.author }} - {{ reply.created_at }}
      <a href="{% url 'board:post_reply' reply.pk %}">답글</a>
      {% include 'board/post_replies.html' with replies=reply.replies.all %}
    </li>
  {% endfor %}
</ul>
```
11. 글 작성 템플릿
```django
<!-- board/templates/board/post_form.html -->
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>글 작성</title>
</head>
<body>
    <h2>{% if form.instance.pk %}글 수정{% else %}글 작성{% endif %}</h2>

    <form method="post" enctype="multipart/form-data">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">저장</button>
    </form>

    <a href="{% url 'board:post_list' %}">목록으로</a>
</body>
</html>
```
12. 글 상세보기 템플릿
```django
<!-- board/templates/board/post_detail.html -->
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ post.title }}</title>
</head>
<body>
    <header>
        <h1>{{ post.title }}</h1>
        <p>By {{ post.author }} on {{ post.created_at }}</p>
        <p>Views: {{ post.hit }}</p>
    </header>
    <main>
        <article>
            <p>{{ post.content }}</p>
        </article>
    </main>
    <footer>
        <a href="{% url 'board:post_list' %}">Back to Post List</a>
    </footer>
</body>
</html>
```
12. 첨부파일 저장을 위한 설정
```python
# mybbs/settings.py 하단에 추가
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'
```
13. 웹브라우저에서 테스트
* python manage.py runserver
* localhost:8000/board/ 접속

## 첨부파일 다운로드
* HTML5에서부터 지원하는 다운로드 기능
```html
<a href="/media/example.pdf" download>PDF 다운로드</a> <--링크에 연결된 파일을 브라우저에 열지 않고 다운로드함-->
<a href="/media/report.csv" download="sales_report.csv">보고서 다운로드</a>  <!--파일명 지명 가능-->
```
1. attachment/file-name 경로에서 file-name 부부만 보여주기위해 아래의 필터 선언(필터 함수로 basename()선언)
```python
# board/templatetags/board_extras.py
import os
from django import template

register = template.Library()

@register.filter
def basename(value):
    return os.path.basename(value)
```
2. board/templatetags/\_\_init\_\_.py 파일도 빈 파일로 생성
3. 필터를 사용하려는 템플릿 상단에 아래의 선언문 추가
   * {% load board_extras %}
```django
{% load board_extras %}
<!-- board/templates/board/post_detail.html -->
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>{{ post.title }}</title>
</head>
<body>
    <h2>{{ post.title }}</h2>
    <p>작성자: {{ post.author }}</p>
    <p>작성일: {{ post.created_at }}</p>
    <p>조회수: {{ post.hit }}</p>
    <p>내용:</p>
    <div>{{ post.content|linebreaks }}</div>  <!--줄바꿈은 <br>, 빈 줄은 단락 <p>로 바꾸는 내장필터-->

    {% if post.attachment %}
        <p>첨부파일:
            <a href="{{ post.attachment.url }}" download> <!--HTML5에서 지원하는 다운로드 기능-->
                {{ post.attachment.name|basename }}  <!--업로드 저장 경로에서 파일명만 이용자에게 표시됨-->
            </a>  <--attachment.url:MEDIA_URL 경로를 통해 파일의 URL을 반환-->
        </p>
    {% endif %}

    <a href="{% url 'board:post_list' %}">← 목록으로</a>
</body>
</html>
```

## 글 수정
1. board/urls.py 에 수정기능 추가
* path('post/<int:pk>/edit/', views.post_edit, name='post_edit'),  # ✅ 수정 URL 추가
2. board/views.py 에 post_edit 함수 선언
```python
def post_edit(request, pk):
    post = get_object_or_404(Post, pk=pk)
    if request.method == "POST":
        form = PostForm(request.POST, request.FILES, instance=post)
        if form.is_valid():
            form.save()
            return redirect('board:post_detail', pk=post.pk)
    else:
        form = PostForm(instance=post)
    return render(request, 'board/post_form.html', {'form': form})
```
3. post_detail.html 하단에 수정 링크 추가
```django
<a href="{% url 'board:post_edit' post.pk %}">✏️ 수정</a>
```
4. 기존 post_form.html 템플릿 재사용
   * post_form.html은 form.instance.pk를 활용해서 글쓰기/수정 여부를 판별
```django
<h2>{% if form.instance.pk %}글 수정{% else %}글 작성{% endif %}</h2>
```

## 글 삭제
1. 삭제 URL 설정(board/urls.py에 아래 항목 추가)
* path('post/<int:pk>/delete/', views.post_delete, name='post_delete'),  # ✅ 삭제 URL 추가
2. 삭제 뷰 추가
```python
# board/views.py

def post_delete(request, pk):
    post = get_object_or_404(Post, pk=pk)
    if request.method == "POST":
        post.delete()
        return redirect('board:post_list')
    return render(request, 'board/post_confirm_delete.html', {'post': post})
```
3. 삭제 재확인을 위한 템플릿 생성
   * board/post_confirm_delete.html
```django
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>글 삭제</title>
</head>
<body>
    <h2>정말 삭제하시겠습니까?</h2>
    <p><strong>{{ post.title }}</strong> - {{ post.author }}</p>

    <form method="post">
        {% csrf_token %}
        <button type="submit">✅ 예, 삭제합니다</button>
        <a href="{% url 'board:post_detail' post.pk %}">❌ 아니요, 돌아가기</a>
    </form>
</body>
</html>
```
4. 상세보기 템플릿(board/post_detail.html)에서 삭제 버튼 추가
```django
<a href="{% url 'board:post_edit' post.pk %}">✏️ 수정</a> |
<a href="{% url 'board:post_delete' post.pk %}">🗑 삭제</a>
```

## 글 검색
*

## Pagination
*

## AI 모델 로드 및 사용
*

## 부모 템플릿 상속 오버라이드
*

## CSS, Layout 적용
*
