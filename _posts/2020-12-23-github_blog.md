---
layout: post
title: github.io 블로그 제작 (1)
categories: github
tags:
  - github
  - github.io
  - jekyll
---

미루고 미루던 [깃허브 블로그](https://clarit7.github.io/)를 드디어 만들었다. 디자인과 css파일은 손봐야할 곳이 많지만 우선 원하는 기능들을 구현하는 데에 집중했고 긴 시간동안 몇번이나 비교적 편한 벨로그로 만들까 하는 유혹을 뿌리쳤다. 힘들었던 만큼 제작 과정을 자세히 정리해보고자 한다.

<br/>

- github.io 레포지터리 생성

  ![/images/2020/12/23/blog1.png](/images/2020/12/23/blog1.png)

  <br/>
  
  깃허브 레포지터리(1)를 선택 후 새로운 저장소를 만들어준다.(2)
  
  <br/>
  
  ![/images/2020/12/23/blog2.png](/images/2020/12/23/blog2.png)

  <br/>
  
  Repository name은 `[자신의 깃허브 아이디].github.io` 로 지정해준다.(3) 이미 완성 한 후이기 때문에 이미 있다는 오류 메세지가 뜨지만 처음 만드는 것이라면 정상적으로 진행이 가능하다. 이후 Create repository버튼을 눌러(4) 마무리한다.
  
  <br/>
  
- jekyll 테마 적용

  다음 단계는 [jekyll](https://jekyllrb.com/) 테마를 찾는 것이다. 깃허브 블로그는 무궁무진한 커스터마이징이 가능한 만큼 0부터 100까지 모든 과정을 전부 만드는 것은 나같은 프론트엔드 초보자에게는 굉장히 힘든 일이다. 따라서 이미 만들어져 있는 jekyll 테마중 마음에 드는 것을 다운받는것이 효율적이다. 다운로드 링크는 아래와 같다.
  
  <br/>
  
  [https://github.com/topics/jekyll-theme](https://github.com/topics/jekyll-theme)

  [http://jekyllthemes.org/](http://jekyllthemes.org/)

  [https://jekyllthemes.io/](https://jekyllthemes.io/)
  
  <br/>
  
  ![/images/2020/12/23/blog3.png](/images/2020/12/23/blog3.png)

  <br/>
  
  내가 선택한 테마는 'thunder'라는 테마이다. 다른 사람들이 잘 쓰지 않으면서 최대한 심플한 느낌의 테마를 골랐다. 컬러가 들어간 가로줄이 딱 취향에 맞았다.
  
  <br/>
  
  ![/images/2020/12/23/blog4.png](/images/2020/12/23/blog4.png)

  <br/>
  
  이 테마를 자신의 깃허브 블로그에 적용시키기 위해선, 우선 테마의 깃허브 페이지에 내려가 파일들을 모두 내려 받는다. `git clone` 으로도 받을 수 있고 압축 파일로도 받을수 있다. 본인의 local git 폴더에 테마 파일을 전부 옮겨넣고 원격 저장소(본인의 [github.io](http://github.io) 레포지터리)와 연결시킨 후 `git push` 를 통해 업로드한다. 깃 사용법은 검색시 다른 좋은 글이 많으므로 이 글에서 자세히는 다루지는 않겠다.
  
  <br/>
  
  참고로 이 jekyll 테마를 자신의 레포지터리로 fork하고 저장소명을 github.io로 바꿔서 사용하는 방식으로도 만들 수 있지만, 그러면 **fork한 프로젝트로 인식돼서 잔디가 심어지지 않는다!** 따라서 포트폴리오 관리도 염두에 두고 있으면 가능한 fork저장소로 만들기보다는 오리지널 저장소를 따로 만들어 파일을 옮기는 이 방식을 추천한다.
  
  <br/>
  
  ![/images/2020/12/23/blog5.png](/images/2020/12/23/blog5.png)
  
  <br/>
  
  이렇게 jekyll 테마 파일들을 본인의 [github.io](http://github.io) 레포지터리에 모두 업로드하게 되면 빌드와 호스팅 모두 깃허브에서 알아서 해주므로 편하게 사용만 하면 된다. 인터넷 주소창에 `[자신의 깃허브 아이디].github.io` 를 입력하면 접속이 가능하다.
  
  <br/>
  
  ![/images/2020/12/23/blog6.png](/images/2020/12/23/blog6.png)
  
  <br/>
  
- 글 쓰기

  이제 원하는대로 블로그가 만들어졌으면, `_post` 폴더 안에 마크다운 파일(.md)들을 업로드하는 것으로 블로그에 글을 쓸 수 있다. 따로 카테고리를 나눠놓지 않으면 가장 최신 글이 위에 표시되고 오래된 글이 아래에 표시된다. 
  
  <br/>
  
  ![/images/2020/12/23/blog7.png](/images/2020/12/23/blog7.png)

  <br/>
  
  가장 먼저 Add file(1) - Create new file(2)을 통해 새 파일을 생성한다.
  
  <br/>
  
  ![/images/2020/12/23/blog8.png](/images/2020/12/23/blog8.png)

  <br/>
  
  파일명에 `_posts/YYYY-MM-DD-[파일명].md` 를 입력한다. 그러면 파일을 생성하는 동시에 _post 폴더가 자동으로 생성된다.

  <br/>
  
  ![/images/2020/12/23/blog9.png](/images/2020/12/23/blog9.png)

  <br/>
  
  파일의 내용은 다음과 같이 layout, title, categories, tags 등을 입력해줄 수 있다. 물론 카테고리 기능과 태그 기능은 해당 테마에서 지원을 해주거나 직접 기능을 추가해줘야 동작한다. 하단의 'commit new file' 버튼을 클릭해 커밋을 완료한다. 이후 빌드에 필요한 시간이 조금 지나면 블로그에 글이 올라와 있는 것을 확인할 수 있다.

  <br/>
  
  ![/images/2020/12/23/blog10.png](/images/2020/12/23/blog10.png)

  <br/>
  
  지금은 원하는 기능을 전부 구현한 상태지만, 막 저장소에 업로드를 한 시점에서는 원하는 기능들도 구현이 잘 돼있지 않았고 필요하지 않은 기능들이 남아있기도 했다. 따라서 내 입맛에 맞게 커스터마이징이 된 블로그를 만들기 위해선 추가적인 과정이 더 필요하다.
  
  <br/>
  
  내가 원했지만 이 테마에서 구현되지 않은 기능들은 다음과 같았다.
  
  <br/>
  
  1. 카테고리별 모아 보기
  2. 게시글 내 같은 카테고리의 다른 글 목록 보기
  3. 태그별 모아 보기
  4. 게시글 내 태그 목록 표시
  5. 게시글 검색 기능
  
  <br/>
  
  해당 기능들에 대한 구현은 다음 글에 이어서 쓰도록 하겠다.
