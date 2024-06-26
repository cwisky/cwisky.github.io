# cwisky.github.io
## Jekyll Documentation Theme 사용법 정리

[마크다운 문법 참고] (https://gist.github.com/ihoneymon/652be052a0727ad59601)
<pre>
### [사이드바 사용하기]

사이드바 선택 : 각 페이지(*.md)의 전문(frontmatter)에 선언, 아래의 예 참조
사이드바 파일 : _data/sidebars/mydoc_sidebar.yml
---
title: Alerts
tags: [formatting]
keywords: notes, tips, cautions, warnings, admonitions
last_updated: July 3, 2016
summary: "You can insert notes, tips, warnings, and important alerts in your content. These notes are stored as shortcodes made available through the linksrefs.hmtl include."
sidebar: mydoc_sidebar
permalink: mydoc_alerts
---

사이드바에서 메뉴 계층구조 깊이는 2차까지이고 그 이상의 깊이는 사용성에 불리함
사이드바의 Tag archives 메뉴는 3차까지 구성한 예를 보이고 있다
sidebar를 선언하지 않으면 디폴트로 home_sidebar가 사용됨(_config.yml에 설정되어 있음)
sidebar를 사용하지 않고 Top Navigation에서 메뉴를 클릭하면 바로 페이지가 표시되도록 하려면...
hide_siderbar: true
페이지 파일의 디렉토리에 따라 다른 사이드바를 적용하고자 할 때는 _config.yml에 아래처럼 설정한다
예를 들어, pages/mydoc 안에 있는 페이지를 화면에 표시할 때는 mydoc_sidebar가 사용되게 하려면...
-
  scope:
    path: "pages/mydoc"
    type: "pages"
  values:
    layout: "page"
    comments: true
    search: true
    sidebar: mydoc_sidebar
    topnav: topnav

### [Top navigation 사용하기] 

Top navigation도 sidebar와 유사하게 작동한다.
모든 페이지를 대상으로 모두 동일한 _data/topnav.yml을 사용하려면 _config.yml에 아래처럼 할 수 있다
-
  scope:
    path: ""
    type: "pages"
  values:
    layout: "page"
    comments: true
    search: true
    sidebar: home_sidebar
    topnav: topnav

pages/mydoc 안에 포함된 페이지에는 _data/topnav.yml 을 적용하려면 _config.yml에 아래처럼 설정한다
-
  scope:
    path: "pages/mydoc"
    type: "pages"
  values:
    layout: "page"
    comments: true
    search: true
    sidebar: home_sidebar
    topnav: topnav

### [Sidebar 작성 규칙]
  
YAML에 몇가지 규칙을 더함
folders: 메뉴 전체를 포함하는 노드
folder 는 메뉴들의 그룹 개념
folders 아래의 모든 계층구조에는 title, output 노드가 포함되어야 한다
추가로 item 하위 계층구조에는 title, output, url 노드가 포함되어야 한다

### [Comments (유료서비스)]
  
 해제하려면 _config.yml 에서 아래처럼 설정
 comments: false

### [Page frontmatter]   
  
html 페이지나 md 페이지에 반드시 좌측 상단에 위치해야 함
---
title: "Some title"         <-- 제목 안에 콜른이 있다면 따옴표는 필수,  \" 등의 이스케이프 문자 가능
tags: [sample1, sample2]
keywords: keyword1, keyword2, keyword3
last_updated: Month day, year
summary: "optional summary here"
sidebar: sidebarname
permalink: filename.html
---

페이지의 상단에 작은 TOC를 제거하려면 각 페이지 전문(frontmatter)에 아래처럼 설정을 추가한다
  toc: false

permalink : 파일명에 확장자는 .html 을 사용한다

### [새 페이지 추가]
  
pages 디렉토리 안에 html, md 파일을 저장하거나, pages 아래에 하위 디렉토리 생성, 그 안에 하위 디렉토리 혹은 페이지 파일 생성   
html, md 파일은 페이지 빌드시 top navigation, sidebar, page 이들 3가지 요소가 결합되어 하나의 완전한 html 문서가 생성된다   
각 페이지(html, md)는 frontmatter를 선언하여 페이지에 연결될 topnav, sidebar가 결정된다   
확장자가 md 이더라도 최종적으로 html 문서로 완성되기 때문에 링크 등에 사용할 때는 html 확장자를 사용해야 한다
Jupyter notebook 파일(ipynb)을 웹문서로 게시할 때는 juypter notebook에서 md 포맷으로 다운로드하여 상단에 frontmatter를 추가하면 된다
소스파일을 웹문서로 게시할 때는 상단에 frontmatter 를 추가하고 소스 코드 부분은 ``` ```으로 감싸주면 아래와 같은 형태가 된다   
```python
파이썬 프로그램 코드
```

### [Top navigation 설정]
  
_data/topnav.yml 에 정의
각 product으로 이동하거나 외부 리소스로 이동할 수 있다
url : 사이트 내부 링크 url
external_url : 외부 리소스 링크 url
topnav : single links
topnav_dropdowns : dropdown menus 표현
  
### [Markdown]
  
Github.com 의 마크다운과 같지만 리스트를 표현할 때 넘버링 숫자와 아이템 사이에 공백문자 2개가 필요함
1.  First item
2.  Second item
3.  Third item

리스트 아이템 안에 문단이나 다른 것들이 포함될 때는 아이템의 첫 문자와 들여쓰기 정도가 동일해야 한다(앞에 공백문자 4개가 요구됨)
1.  First item

    ```
    alert("hello");
    ```

2.  Second item

    Some pig!

3.  Third item
</pre>

<hr/>

# 마크다운 간단 문법 소개      
- https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax
## 제목 표시하기     
- 큰 제목(h1), 작은 제목(h2> 2개로 구분할 때는 '\=\=\=\=\=', '\-\-\-\-\-' 을  제목 아래에 적용한다
   
- 여기는 큰 제목   
\=============

여기는 큰 제목   
==============

- 부제목(h2)   
\--------

부제목(h2)   
----------   
   
   
## 수평선    
- \-\-\-    
---   
- \* * *   
* * *   
- \***
***   
- \___
___   
- \- - -
- - -   

## 제목(\# 오른쪽에 공백문자가 있어야 제목 효과를 낸다)
- # h1   
- ## h2   
- ### h3   
- #### h4   
- ##### h5   
- ###### h6   

## 순서 있는 목록 표시하기 (번호 사용)
1. Python
2. Java
3. Javascript

## 순서 없는 목록 표시하기 ('-' 사용)
- Python
- Java
- Javascript

## 순서 없는 목록 표시하기 ('*' 사용)
* Python
* Java
* Javascript

## 코드 들여쓰기
  - 문장 왼쪽에 4개의 공백문자가 있을 때 들여쓰기 됨
  - 들여쓰기 시작하는 행 위에는 빈 라인이 있어야 줄바꿈이 정상 작동함.

aaaaaaaaaaaaaaaaaaaaaaaaaaa

    bbbbbbbbbbbbbbbbbbbbbbb
    ccccccccccccccccccccccc   
    

## 코드블럭
코드 전체를 \`\`\` 으로 감싸준다   
```java
public class BootSpringBootApplication {
  public static void main(String[] args) {
    System.out.println("Hello, Honeymon");
  }
}
```

## 기본 링크   
- http:// 혹은 https:// 으로 시작되는 URL을 입력하면 자동으로 링크가 설정됨   
- http://google.com   
- [Google](http://www.google.com)   
- [Google](http://www.google.com "구글 바로가기")   

### 아이디를 사용하여 보여지는 문자열 표시하기
- \[보여질 문자열][아이디]  형식으로 아이디를 지정하고 아래처럼 아이디를 사용하여 보여질 문자열을 표시한다
- \[아이디]: URL "Optional Title"   
- [google 바로가기][google]   
- [google]:http://google.com "google사이트로 이동"   


### 내부참조 링크   
- \# 공백 오른쪽에 참조할 제목을 적되 제목에 포함된 공백문자는 '-'으로 대체한다
- [마크다운 문법 소개 처음으로 가기](#마크다운-간단-문법-소개)   

## 강조
- \*sing asterisks\*   --> *sing asterisks* 
- \_single underscores\_   --> _single underscores_
- \*\*double asterisks\*\*   --> **double asterisks**   
- \_\_double underscores\_\_   --> __double underscores__
- \~\~cancle line\~\~   --> ~~cancle line~~


## 이미지
* github 개인 폼페이지인 https://cwisky.github.io/ 안에 있는 images/markdown.png 파일을 표시하려면
* \<img src="https://cwisky.github.io/images/markdown.png"  alt="마크다운 로고" title="마크다운 로고"\>   
<img src="https://cwisky.github.io/images/markdown.png"  alt="마크다운 로고" title="마크다운 로고">   


## 줄바꿈
문장의 중간이나 끝에 공백 2개 이상이 이어지면 줄바꿈된다   
문장의 중간이나 끝에 '_' 2개 이상이 이어지면 줄바꿈된다   
그러므로 여기에 3개의 문장은 빈 라인을 사용하거나 목록의 원소가 아니어도 줄바꿈된다   

## 인용문(BlockQuote)
- 아래는 인용문입니다. 문장 앞에 각각 \>, \>\> 기호를 사용한 예입니다
> 인용된 문장
>> 인용된 문장 안에서 또 인용된 문장

