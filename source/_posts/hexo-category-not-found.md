---
categories:
  - Hexo
tags:
  - hexo
date: 2019-01-21 00:21:44
title: Hexo 블로그에서 카테고리(태그)가 제대로 보이지 않을 때
updated:
---

결론부터 말하자면 예전 {% post_link hexo-image-upload Hexo 이미지 업로드 %} 와 굉장히 유사한 상황이 발생했다.

블로그의 Category 항목에서 **다른 카테고리로는 이동(검색)이 되는데 유독 Hexo 카테고리만 404 not found**가 나오는 현상이 발생한 것이다.

검색을 해보니 다음과 같은 글을 볼 수 있었다.

{% blockquote https://github.com/iissnan/hexo-theme-next/issues/1903 %}
  Always remember: Linux (instead of Windows, for example) always split all files/directories with case-sensitive. It mean in same directory can be JavaScript, Javascript and javascript — and all of they will be with exclusive, different pathes.

  So, actual trouble with Hexo — we want to use case-sensitive in categories/tags with pretty-fine vision (JavaScript), but it will generate case-sensitive links on categories/tags too (categories/JavaScript/) and always create any problems with rendering and SEO.
{% endblockquote %}

글을 읽고 아차 싶어 배포한 github repo에 들어가보니 다음과 같이 되어 있었다.

{% asset_img hexo-category-not-found-1.jpg %}

처음에 카테고리가 Hexo인 글을 쓸 때 'h'exo로 만들고 이후에 대문자로 수정했는데 github에서는 동일한 것으로 판단하여 수정되지 않은 것 같았다.

아래 링크에서는 2가지 해결방법을 제시하고 있었다.

{% blockquote https://github.com/iissnan/hexo-theme-next/issues/1904 %}
  1. In main Hexo _config.yml (auto):
  {% codeblock %}
    # Writing
    filename_case: 1
  {% endcodeblock %}

  2. You can use maps on Categories and Tags (manual):
  {% codeblock %}
    default_category: uncategorized

    category_map:
    NexT: next
    Hexo: hexo

    tag_map:
    JavaScript: javascript
  {% endcodeblock %}

  And in *.md:
  {% codeblock %}
    categories:
    - NexT
    - Hexo
    tags:
    - JavaScript
    #OR
    categories: [NexT, Hexo]
    categories: [JavaScript]
  {% endcodeblock %}
{% endblockquote %}

2번 방법은 카테고리가 추가될 때마다 수정을 해야된는 것으로 보여 번거로울 것 같았다.

`filename_case` 옵션의 의미는 다음과 같다고 한다.

{% blockquote %}
  It will generate case-sensitive in blog vision (JavaScript),
  but do all links of categories/tags to lowercase (categories/javascript/).
{% endblockquote %}

즉, 링크 주소는 무조건 소문자로 만들어 준다는 것.

아무래도 주소는 대문자가 있는 것보단 소문자만 있는 것이 보기 편하므로 옵션을 적용하기로 했다.

그리고 기존에 있던 대문자로된 디렉토리들을 전부 바꿔야하기 때문에

1. `_config.yml`에 위의 filename_case 옵션 적용
2. `public/categories/` 하위의 영문으로 된 디렉토리를 전부 삭제하고
3. `hexo d` 명령어로 배포만 실시(이렇게 하면 카테고리 디렉토리가 github repo에서 삭제된다.)
4. `hexo g -d` 명령어로 generate 및 deploy(위의 옵션을 바꿨으므로 포스트에서 대문자로 썼더라도 github에는 모두 소문자로 배포된다.)

이렇게 진행했다.

{% asset_img hexo-category-not-found-2.JPG %}

깔끔하게 해결.