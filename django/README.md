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

## 계층 구조형 BBS(게시판)
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
    attachment = models.FileField(upload_to='attachments/', null=True, blank=True)

    # 부모글 번호 (자기 자신을 참조)
    parent = models.ForeignKey('self', null=True, blank=True, on_delete=models.CASCADE, related_name='replies')

    def __str__(self):
        return self.title
```
4. 데이터베이스 테이블 생성(마이그레이션 생성 및 적용)
* python manage.py makemigrations
* python manage.py migrate
