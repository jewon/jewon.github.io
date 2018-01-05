---
layout: post
title:  "Jekyll 테마를 활용한 Github page만들기"
date:   2018-01-05 20:00:00 +0900
categories: ordinary
---


첫 포스트로 Jekyll을 이용한 github page를 만드는 방법에 대해 알아보자.

**1. github page?**

github page는 쉽게 말해서 버전 관리 서비스인 github의 repository에 웹 페이지를 업로드 하면, 이를 자동으로 호스팅해주고 name.github.io라는 도메인과 연결시켜주는 서비스라고 할 수 있다. 가장 간단하게는 아무것도 없는 git repository에 index.html이라는 이름을 가진 파일 하나만 딱 커밋해도 이를 name.github.io에서 볼 수 있는 것이다! 따로 호스팅 과정이 필요 없어 매우 편리하다. git repo안에서 컨텐츠를 관리할 수 있으므로, 버전관리까지 가능한 것은 덤이다.

**2. Jekyll**

Jekyll은 Ruby gem의 하나로, 블로그와 같이 간단한 페이지를 만드는 데 적합한 다양한 기능들을 포함하고 있다. markdown, html등의 컨텐츠 + Jekyll테마만 있으면 하나의 웹 사이트를 완성할 수 있다. 주소 관리, 포스트 카테고리 등의 기능이 모두 내장되 있기 때문에 따로 신경을 쓸 필요가 없어 편리하다.

Jekyll의 디렉토리 구조나, 설치 방버 등은 [Jekyllrb-ko](http://jekyllrb-ko.github.io/docs) 에서 자세히 알아볼 수 있다. 특히, Jekyll테마의 [디렉터리 구조](http://jekyllrb-ko.github.io/docs/structure/)를 보면, `_include` 에 블로그에서 고정적으로 필요한 부분 (상단바, 하단부, 카테고리목록, 각종아이콘 등등...)을 재료로 넣고, `_layout`을 이런 재료를 활용하여 포스트가 어디에 들어가고, 재료들의 배치를 어떻게 할 것인지 등을 설정하면, `_post`에서는 레이어를 선택해 컨텐츠만 만들어주면 되는 형태인 것을 알 수 있다.

`_post`디렉터리 내의 컨텐츠가 작성되는 파일은 markdown형태로 되어야 하며, 가장 앞에 YAML형식으로 레이아웃 등을 명시해야 한다.
예를 들면,  
`---  
layout: post  
title:  "Hello World!"  
date:   2017-12-30 13:05:00 +0900  
categories: ordinary  
---`   
이런 식으로 명시를 하고, 아래쪽에 markdown문법에 맞게 컨텐츠를 작성하면 된다. 사용할 markdown문법은 루트디렉터리의 `_config.yml`에서 정의할 수 있다. 이 파일에서는 페이지 제목이나, 작성자 정보 등 블로그 전체에 사용할 재료들에 있어 일종의 변수가 되는 기본적인 것들을 정의하게 된다.

**3. github page + Jekyll**

github page를 만들기 위해 새로운 repository를 만들었다면, settings에서 githup page에 대한 설정을 할 수 있다.

> githu page sting 설정화면]

기본적으로 Jekyll 테마를 골라서 적용할 수 있는데, 이렇게 하여 설정하면, repository의 Root Directory에 테마에 대한 정보가 생성되며 자동으로 새로운 commit이 이루어진다. 이는 repository에 명시된 테마를 온라인에서 소스로 불러서 렌더링 하는 것이며, 세부적인 설정이나 컨텐츠 페이지가 다양한 관계를 갖고 있을 때 모두에 적용하기는 어려운 점이 있다.

따라서, 테마 파일 자체를 다운로드하여 따로 git repository안에 두어 테마에 대한 참조를 내부적으로 해결하고, customize할 필요가 있다. Jekyll theme파일은 검색만 해도 매우 다양하게 나오며, 기본적으로 Jekyll을 설치했다면 minima라는 테마를 사용하게 된다. 현재 이 블로그의 테마다.
