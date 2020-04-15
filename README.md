[GitHub Pages Blog](https://postlude.github.io)
=================

## Branch
- master : 빌드 후 배포되는 코드가 올라가는 브랜치<br>
- develop : 포스트 작성 및 설정 브랜치

## 최초 Clone
### repository clone
submodule까지 한 번에 clone
<pre><code> git clone --recursive https://github.com/postlude/postlude.github.io.git </code></pre>

### theme checkout
clone만 받게 되면 해당 submodule은 *detached HEAD* 상태이기 때문에 해당 원격 브랜치로 checkout
<pre><code>cd themes/hueman
git checkout -b themes/hueman
</code></pre>

## 작성
<pre><code> npx hexo new [post/draft] 'post_name'</code></pre>

## 배포
develop 브랜치에서 실행
<pre><code> npx hexo g -d</code></pre>