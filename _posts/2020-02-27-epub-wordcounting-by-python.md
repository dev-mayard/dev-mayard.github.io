---
layout: post
title: Epub 파일 글자 수 세기
subtitle: 파이썬으로 Epub 파일 글자 수 세는 간단한 툴 개발하기
tags: [python]
comments: true
---

전자책 표준 스펙인 Epub 파일의 글자 수 카운팅 기능을 파이썬으로 개발해봤습니다.

***
### #개발환경
**Language: python3**

**Library: BeautifulSoup, lxml**


  
***
### #Epub 파일 구조

![epub](/assets/img/20200227/epub_hierarchy.png)

Epub 파일은 대부분 위와 같은 구조로 되어 있는데, 

위 파일에서는 Text 디렉토리 하위에 있는 Section*.xhtml 파일이 실제 도서의 내용을 담고 있는 파일들입니다.

단, 디렉토리의 구조는 Epub 파일마다 조금씩 차이가 있습니다.

그러므로,  파일의 구조를 명시하고 있는 **content.opf** 파일을 열어 도서 내용 파일의 정보를 가져와야 합니다.



![content.opf](/assets/img/20200227/content_opf.png)


**content.opf** 파일은 위와 같은 **xml** 포멧으로 되어 있습니다.

여기서 **spine** 태그 하위의 **itemref** 태그에서 **idref** Attribute 값을 가져 온 뒤, 

이를 **manifest** 태그 하위에 있는 **item** 태그의 id Attribute 값과 비교하여 동일 한 값을 가지고 있는 item 들의 href Attribute 값을 얻을 수 가 있는데,

이 값들이 Epub 파일 내 실제 도서 내용이 담긴 파일을 의미합니다.



이는 Python3.x 코드로 아래와 같이 처리 할 수 있습니다.

```python
def __read_items__(self):
    try:
        with ZipFile(self.epub, 'r') as zf:
            zipInfo = zf.infolist()
 
            wc = 0;
            for content in zipInfo:
                if '.opf' in content.filename:
                    lxml = BS(zf.read(content.filename), 'lxml')
 
                    itemrefs = lxml.find_all('itemref')
                    items = lxml.find_all('item')
                    content_files = []
 
                    for item in items:
                        for itemref in itemrefs:
                            if itemref['idref'] == item['id']:
                                content_files.append(item['href'])
                                break
 
                    ### 코드 중략 (아래 코드 참조)
 
            zf.close()
 
            return wc
    except BadZipFile:
        return 0
```


위에서 얻은 파일들 **(content_files)** 은 **html**포멧의 파일이므로,  lxml 을 사용하여 파일을 연 뒤, 

**BeautifulSoup**의 **find_all()** 함수를 통해 **html** 파일 내의 **body** 태그의 내용을 모두 가져 와 ***.text** 로 태그를 모두 제거 할 수 있습니다.

그 후, 공백과 개행 그리고 **html**에서 **&amp;nbsp;** 에 해당하는 **u00A0** 를 모두 없앤 나머지 글자 수를 계산할 수 있습니다.



이는 아래 코드와 같습니다.


```python
### 위 코드에서 이어짐
                        if len(content_files) > 0:
                            zip_file_info = zf.infolist()
                            for content_file in content_files:
                                for file_info in zip_file_info:
                                    if content_file in file_info.filename:
                                        print(file_info.filename)
                                        lxml = BS(zf.read(file_info.filename), 'lxml')
                                        body_book = lxml.find_all('body')
 
                                        for body in body_book:
                                            body_content = body.text
                                            wc += len(body_content.replace(" ", "").replace("\n", "").replace(u"\u00A0", ""))

```




***
### #검증하기

특정 **Epub** 파일의 경우 R*** 사이트에서 제공하는 글자수와 차이가 나는 경우들이 있었습니다.


R***사이트: **약 106,000자**

![r_wc](/assets/img/20200227/r_wc.png)


개발한 코드: **약 103,000자**

![f_wc](/assets/img/20200227/f_wc.png)




확인을 위해, 해당 **Epub** 파일을 직접 열어서, <a href="https://atom.io/" target="_blank">에디터</a>를 사용해 수동으로 글자 수를 세봤습니다.



위 코드와 동일하게, **html** 파일에서 **body** 내용을 가져 온 뒤, 

1. 태그 제거 (regexp: (<([^>]+)>) )
2. 공백 제거
3. 개행 제거
4. &amp;nbsp; 제거
5. HTML Escape(NonBreakingSpace) 제거

를 한 결과,


![epub_detail](/assets/img/20200227/epub_detail.png)


102,801 자, 약 **103,000** 자가 나왔습니다.



글자 수 차이가 나는 원인은 아래와 같은 것으로 보입니다.



특정 문자 제거 과정에서 **HTML Escape(NonBreakingSpace)** 제거 를 하지 않을 경우, 

R***사이트에서 제공하는 글자수와 동일한 글자 수가 나오게 됩니다(아래 그림 참고).



![epub_detail2](/assets/img/20200227/epub_detail2.png)



**NonBreakingSpace**는 에디터로 확인시 &amp;#160; 으로 보이게 됩니다.

참고: <http://www.htmlhelp.com/reference/charset/iso160-191.html>



***
![empty_space](/assets/img/20200227/empty_space.png)


위 그림에서 드래그한 영역과 같이, 

이 문자 역시 빈 공백이므로 글자수에서 제외하고 카운팅을 하는 게 맞다고 생각됩니다.



