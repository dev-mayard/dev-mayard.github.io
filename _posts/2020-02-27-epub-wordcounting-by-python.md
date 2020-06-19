---
layout: post
title: 파이썬으로 Epub 파일 글자 수 세기
subtitle: 파이썬으로 개발한 Epub 파일 글자수 세는 간단한 툴
tags: [python]
comments: true
---

Epub 파일의 글자 수를 카운팅 하여, 도서 정보를 제공 합니다.


### #개발환경
**Language: python3**
**Library: BeautifulSoup, lxml**


### #Epub 파일 구조

![epub](/assets/img/avatar-icon.png)

Epub 파일은 대부분 위와 같은 구조로 되어 있는데, 위 파일에서는 Text 디렉토리 하위에 있는 Section*.xhtml 파일이 실제 도서의 내용을 담고 있는 파일들입니다.

단, 디렉토리의 구조는 Epub 파일마다 조금씩 차이가 있습니다.

그러므로,  파일의 구조를 명시하고 있는 content.opf 파일을 열어 도서 내용 파일의 정보를 가져와야 합니다.