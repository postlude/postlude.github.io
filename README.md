[GitHub Pages Blog](https://postlude.github.io)
=================

## Branch
master : 실제 배포되는 코드가 올라가는 브랜치<br>
github_pages_hexo : 포스트를 작성하는 브랜치<br>
dev : 여러 설정들을 테스트하거나 변경하기 전에 적용해보기 위한 브랜치

## Clone
### repository clone
<pre><code> git clone https://github.com/postlude/postlude.github.io.git </code></pre>

### theme clone
<pre><code> git clone -b themes/hueman https://github.com/postlude/postlude.github.io.git themes/hueman </code></pre>

## 작성
<pre><code> npx hexo new [post/draft] 'post_name'</code></pre>

## 배포
**github_pages_hexo branch에서 실행**
<pre><code> npx hexo g -d</code></pre>