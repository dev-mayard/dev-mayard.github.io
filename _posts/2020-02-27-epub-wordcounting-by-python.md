---
layout: post
title: 파이썬으로 Epub 파일 글자 수 세기
subtitle: 파이썬으로 Epub 파일 글자 수 세는 간단한 툴 개발하기
tags: [python]
comments: true
---

Epub 파일의 글자 수를 카운팅 하여, 도서 정보를 제공 합니다.


### #개발환경
**Language: python3**

**Library: BeautifulSoup, lxml**



### #Epub 파일 구조

![epub](/assets/img/epub_hierarchy.png)

Epub 파일은 대부분 위와 같은 구조로 되어 있는데, 

위 파일에서는 Text 디렉토리 하위에 있는 Section*.xhtml 파일이 실제 도서의 내용을 담고 있는 파일들입니다.

단, 디렉토리의 구조는 Epub 파일마다 조금씩 차이가 있습니다.

그러므로,  파일의 구조를 명시하고 있는 **content.opf** 파일을 열어 도서 내용 파일의 정보를 가져와야 합니다.



![epub](/assets/img/20200227/content_opf.png)


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

BeautifulSoup의 find_all() 함수를 통해 html 파일 내의 <body> 태그의 내용을 모두 가져 와 *.text 로 태그를 모두 제거 할 수 있습니다.

그 후, 공백과 개행 그리고 html에서 &nbsp; 에 해당하는 u00A0 를 모두 없앤 나머지 글자 수를 계산할 수 있습니다.



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

