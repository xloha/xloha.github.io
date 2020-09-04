---
layout: post
title:  "SimpleJekyllSearch Failed to get JSON 해결하기"
date:   2020-09-04 23:50:00 +0900
categories: gitblog jekyll SimpleJekyllSearch json
tags: 
- gitblog 
- jekyll 
- SimpleJekyllSearch 
- json
---

<br>

내가 공부해왔던것을 정리하고, 후에 정리했던 것을 보면서 되새김질하고 필요할 때 쉽게 찾을 수 있도록 예전에 git blog를 만들었다.

깃 블로그는 사실상 만든지는 꽤 되었지만, 그 동안 정리해놓은 read me파일들을 막 올리자니 맘에 안들어서 차근차근 올리겠다 마음을 먹고, 우선 블로그부터 꾸미기 시작했다.

> 원래 개발도 예쁘게 해야지 할 맛이 남  

<br>

테마는 `plain-white` 를 쓰고 있는데, 댓글과 검색 기능도 제공하고 엄청 깔끔하고 예쁘다는 장점과 인용구의 표시가 잘 나지 않는다는 단점이 있다.

어쨋든 검색기능을 제공해주기 때문에 **나는 이 검색기능을 빠르게 쓸 수 있을 줄 알았다. 그것은 나의 오산이였다,,,,🤯** 하라는대로 다 했지만, 어째서인지 자꾸 아래와 같은 에러가 났다.   

`Uncaught Error: SimpleJekyllSearch --- failed to get JSON`

![image-20200904105001](/assets/image/image-20200904105001.png)


<br>



# 해결방법

1. [Simple-Jekyll-Search-Options](https://github.com/christian-fei/Simple-Jekyll-Search/wiki#options) 에도 나와있듯이 컨텐츠의 문자열들을 아래의 **옵션으로 가공**해준다. (부득이하게 중괄호 두개를 띄어썼다!! { { -> 원래 붙여써야함!!)	

   ```json
   "content" : "{ { post.content | strip_html | strip_newlines | remove_chars | escape | truncate:200 } .}"
   ```

   하지만 계속해서 위와 같이 `failed to get JSON` 과 같은 에러가 난다면 그것은,,,, ***당신 포스트에 문제가 있는 것이다!!***  

   > 나는 이것을 무려,,,,1년만에 깨달았다,,,,,,,,그 동안 블로그에 무관심한 적도 있지만,,,검색 기능을 활성화하려고 한번 붙잡으면 정말 3시간까지 붙잡았는데,,,,내 포스트에 문제가 있을 것이라고는 상상도 못했다.  
  
    

2. Invalid JSON 고치기

    ![image-20200904232029538](/assets/image/image-20200904232029538.png)

    * chrome의 개발자 도구를 열고 `Network`탭을 열면 다음과 같이 search.json의 Response값을 긁어서 JSON의 Validation을 확인해준다. [JSON Validator](https://jsonlint.com/)

   * 그럼 분명히 Invalid라고 뜰 것이다,,,,

   * 글을 포스팅할 때 코드를 넣게 되는데 그 코드에 포함되어 있는 **탭 문자가 JSON을 Valid하게 만들지 않는다!**
     * 다들 알겠지만 탭은 `\t` 와 같은 형태로 이스케이프 문자로 작성해줘야한다. 하지만 왜인지,,,탭만은 escape가 적용되지 않아서 문제를 일으키게 되는 것이다.  
  

3. 꼼수

   문제가 되는 포스트는 대부분 글의 상단에 코드가 있는데, 코드의 탭을 없애게 되면 가독성이 구리다. 그럼 블로그를 예쁘게 꾸민 이유도 없어지게 되는 것이다. 그래서 나는 다음과 같은 방법을 사용하였다.

   * 글의 상단에 주저리주저리 글을 써 넣어서 200자를 채운다,,,,,(찌질,,,🙃)

   * 컨텐츠을 가공할 때 `truncate:200` 를 `truncate:100`로 바꾸어 코드가 JSON에 포함되지 않도록 한다.  


<br>


킄킄,,,,,,,, SimplejekyllSearch에 고통을 받고 있던 사람들이 있다면,,,,부디 나의 포스팅을 보고 빠르게 해결했으면 좋겠다,,,,,,,난,,,1년이 걸렸지만,,,,,,아니면 나만 몰랐던 걸까,,,,,? 주르륽,,,,,,,,,,





