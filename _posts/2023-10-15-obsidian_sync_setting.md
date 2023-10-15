---
layout: post
title: Obsidian github 동기화 세팅하기(iOS 자동화, 단축어 활용)
categories: tech
tags: 
  - obsidian
  - github
  - automation
  - shortcut
---

이 글은 Joshua Kim 님의 [옵시디언 사용해 보실래요? - 동기화, 백업 환경 구축](https://velog.io/@joshuara7235/옵시디언-사용해-보실래요)의 내용을 참고해 작성되었습니다.


<br/>

---

<br/>

### 옵시디언 입문

<br/>

메모 습관과 시스템을 바꿔보고자 obsidian에 입문했다.

그렇지만 공식 동기화 플러그인 obsidian sync는 너무 비싸다는 단점이 있었다. 메모앱 하나에 일년에 10만원을 태워?

현재 내가 사용하는 기기는 windows pc, mac, iphone, ipad 총 네대로, 서로 다른 플랫폼에서의 동기화 방법을 고민하다가 Joshua Kim 님의 글을 참고해 github를 사용한 동기화 세팅을 진행했다.

여기에 약간의 추가 세팅과 아이폰 자동화, 단축어 등을 사용해 조금 더 편하게 사용할 수 있게 만든 방법을 공유하고자 한다.

<br/>

---

<br/>

### 1. 깃헙에 원격 repository 만들기

<br/>


<p align="center">
    <img src="/images/2023/10/15/0.png" width="800px">
</p>

<br/>

깃헙의 Repositories 탭에서 New 버튼으로 새 원격 저장소를 생성한다.

<br/>

<p align="center">
    <img src="/images/2023/10/15/1.png" width="800px">
</p>

<br/>

저장소 이름 입력, 공개/비공개 설정, Create repository 클릭

<br/>

### 2. PC에 git client 활용해 원격 저장소 내려받기

<br/>

원격 저장소에서 로컬 기기로 원격 저장소를 내려받고 연결해야한다.

이 글에선 Mac을 기준으로 설명하지만, 리눅스와 윈도우즈에서도 크게 차이나는 점은 없다.

혹시라도 git client를 어떻게 사용하는지 모른다면 [이 글](https://www.lainyzine.com/ko/article/git-clone-command/)을 참고하자.

<br/>

먼저, 아래 명령어를 사용해 로컬 기기로 원격 저장소를 내려받는다.

```bash
git clone https://github.com/{본인 계정 이름}/{저장소 이름}

# 예시
git clone https://github.com/Clarit7/obsidian-sync.git
```

<br/>

만약 private로 설정한 저장소의 경우엔 아래 명령어를 사용한다.

```bash
git clone https://{본인 계정 이름}@github.com/{본인 계정 이름}/{저장소 이름}

# 예시
git clone https://Clarit7@github.com/Clarit7/obsidian-sync.git
```

<br/>

해당 폴더 내에 .gitignore 파일을 생성한다.

```vim .gitignore``` 명령어로 생성해도 좋고, 다른 텍스트 편집기가 있다면 써도 좋다.

.gitignore파일의 내용은 다음과 같이 작성한다.

```
.DS_Store
.obsidian/
```

<br/>

.DS_Store는 맥에서 파인더를 사용할 때 생성되는 metadata이며, .obsidian폴더는 obsidiana의 설정값 등이 저장되는 폴더이다.

만약 .obsidian폴더를 ignore처리하지 않을 경우 여러 기기에서 pull, push를 진행하며 설정값 때문에 conflict가 날 확률이 크기 때문에 꼭 위와 같이 작성하는 것을 추천한다.

<br/>

.gitignore 작성이 완료되었으면, 최초 1회는 upstream branch 설정을 위해 수동으로 push하는 것을 추천한다.

```bash
git commit -m "커밋메세지"
git push --set-upstream origin main
```

만약 옛날에 만들어 놓은 저장소를 사용한다면 branch명이 master일 수도 있는데, 깃헙은 노예제를 연상시킨다는 이유로 더이상 master라는 브랜치명을 사용하지 않는다.

<br/>

혹시라도 이후 과정에서 옵시디언에서 원격 저장소에 연결되지 못하는 에러가 계속 발생한다면, 원격 저장소 연결을 삭제 후 다시 연결해보자.

```bash
git remote remove origin
git remote add origin https://github.com/{본인 계정 이름}/{저장소 이름} # public
git remote add origin https://{본인 계정 이름}@github.com/{본인 계정 이름}/{저장소 이름} # private
```

<br/>

### 3. 맥, 윈도우, 리눅스에서 Obsidian Git 플러그인을 활용한 동기화 세팅

<br/>

Obsidian을 실행한다. 만약 이미 다른 vault가 열려있다면 왼쪽 아래의 open another vault 버튼으로 vault 탐색기를 연다.

<p align="center">
    <img src="/images/2023/10/15/2.png" width="800px">
</p>

<br/>

Open folder as vault 로 아까 로컬 저장소에 클론한 폴더를 연다.

<p align="center">
    <img src="/images/2023/10/15/3.png" width="800px"
</p>

<br/>

설정 - Community plugins - Turn on community plugins

<p align="center">
    <img src="/images/2023/10/15/4.png" width="800px">
</p>

<br/>

Community plugins가 활성화되면 Browse 버튼을 클릭한다.

<p align="center">
    <img src="/images/2023/10/15/5.png" width="800px">
</p>

<br/>

Obsidian Git을 검색하고, 해당 플러그인을 선택해 설치한다.

<p align="center">
    <img src="/images/2023/10/15/6.png" width="800px">
</p>

<br/>

Obsidian Git을 활성화시킨다.

<p align="center">
    <img src="/images/2023/10/15/7.png" width="800px">
</p>

<br/>

메인화면으로 돌아와 command 창을 연다. 이후 git을 검색해 Switch branch를 클릭한다.

<p align="center">
    <img src="/images/2023/10/15/8.png" width="800px">
</p>

<br/>

main 브랜치를 클릭한다.

<p align="center">
    <img src="/images/2023/10/15/9.png" width="800px">
</p>

<br/>

이후 command 창을 열어 수동으로 commit과 push를 진행해도 좋고, 주기적으로 commit & push와 pull을 진행하도록 설정해도 좋다.

아래는 push, pull을 주기적으로 진행하도록 설정하는 방법이다.

원하는 push와 pull 간격을 분 단위로 입력한다.

<p align="center">
    <img src="/images/2023/10/15/10.png" width="800px">
</p>

<br/>

Pull updates on startup 옵션도 키는 것을 추천한다.

<p align="center">
    <img src="/images/2023/10/15/11.png" width="800px">
</p>

<br/>

테스트 파일을 만들어 실제 자동 동기화가 잘 이루어지는지 확인해보았다.

<p align="center">
    <img src="/images/2023/10/15/12.png" width="800px">
</p>

<br/>

### 4. iOS, iPadOS 에서 Working Copy 활용한 동기화 세팅

<br/>

모바일에선 Obsidian Git이 설치만 되고 작동하지 않으니, 다른 git client 앱을 활용해야 한다.

앱스토어에서 Working Copy 앱을 내려받고, 앱 내부에서 모든 기능 언락을 결제한다. (구독형이 아닌 일회성 결제 30,000원이라 충분히 살만하다.)

아래에선 아이폰을 기준으로 설명한다. 아이패드에선 앱의 인터페이스가 조금 다를 수 있는 점은 참고.

<br/>

<p align="center">
    <img src="/images/2023/10/15/13.png" width="800px">
</p>



<p align="center">
    <img src="/images/2023/10/15/14.png" width="800px">
</p>

앱 실행 후 Clone repository를 선택한다.

<p align="center">
    <img src="/images/2023/10/15/15.png" width="800px">
</p>

<br/>

GitHub 탭에서 저장소를 선택해준다.

<p align="center">
    <img src="/images/2023/10/15/16.png" width="800px">
</p>

<br/>

선택하면 URL 탭으로 넘어가는데, 이때 프로토콜은 https를 선택해주고, User에 본인 깃헙 닉네임을 입력해준다. (ssh프로토콜에선 문제가 생길 수 있다는 reddit 글을 본 것 같은데 확실하진 않음) 이후 Clone을 눌러 원격 저장소를 내려받는다.

<p align="center">
    <img src="/images/2023/10/15/17.png" width="800px">
</p>

<br/>

옵시디언을 실행해 왼쪽 위 탭을 누른다.

<p align="center">
    <img src="/images/2023/10/15/18.png" width="800px">
</p>

<br/>

Vault 이름 옆 화살표를 누르고 Manage vaults... 를 선택.

<p align="center">
    <img src="/images/2023/10/15/19.png" width="800px">
</p>

<br/>

Create new vault를 선택한다.

<p align="center">
    <img src="/images/2023/10/15/20.png" width="800px">
</p>

<br/>

원격 저장소와 같은 이름으로 Vault name을 입력하고, Store in iCloud는 체크 해제상태로 Create를 실행한다.

<p align="center">
    <img src="/images/2023/10/15/21.png" width="800px">
</p>

<br/>

Working Copy 앱으로 돌아가, 내려받은 저장소를 선택 후 Repository를 설정 창으로 넘어간다.

<p align="center">
    <img src="/images/2023/10/15/22.png" width="800px">
</p>

<br/>

저장소명 오른쪽의 화살표를 선택 후, Link Repository to를 선택한다.

<p align="center">
    <img src="/images/2023/10/15/23.png" width="800px">
</p>

<br/>

Directory를 누른다.

<p align="center">
    <img src="/images/2023/10/15/24.png" width="800px">
</p>

<br/>

파일탐색기가 열리면 나의 iPhone 최상단 폴더 - Obsidian 폴더로 들어가면 아까 옵시디언에서 생성한 Vault가 있다.

<p align="center">
    <img src="/images/2023/10/15/25.png" width="800px">
</p>

<br/>

해당 Vault를 선택 후 열기를 누른다.

<p align="center">
    <img src="/images/2023/10/15/26.png" width="800px">
</p>

<br/>

이러면 Obsidian mobile에서 생성한 Vault와 Working Copy를 통해서 Clone한 저장소가 연결된다. 아까 만들었던 Test.md 파일도 정상적으로 표시되고 있다.

<p align="center">
    <img src="/images/2023/10/15/27.png" width="800px">
</p>

<br/>

이후 Working Copy를 통해 commit, pull, push를 진행해도 되지만...

옵시디언에서 메모 후 Working Copy를 통해 수동으로 백업을 해야 한다는 점이 귀찮으니 자동화와 단축어를 이용해 앱이 실행되고 종료될때마다 자동으로 백업과 메모 최신화가 될 수 있도록 세팅해보자.

<br/>

### 5. pull, push 단축어 생성 및 자동화 설정

<br/>

아래 단축어를 다운로드 받는다.

<br/>


[Pull 단축어 다운로드](https://www.icloud.com/shortcuts/8e800ff3399949cd857131c1336766d5)

<p align="center">
    <img src="/images/2023/10/15/28.png" width="800px">
</p>

<br/>

[Push 단축어 다운로드](https://www.icloud.com/shortcuts/83864d76d5574a49b2749b9ed1c801bd)

<p align="center">
    <img src="/images/2023/10/15/29.png" width="800px">
</p>

<br/>

Pull, Push 둘 다 모든 빈칸에 전부 obsidian 저장소를 선택해 채워넣는다.

<p align="center">
    <img src="/images/2023/10/15/30.png" width="800px">
</p>

<br/>

자동화 탭으로 가서 추가를 선택한다.

<p align="center">
    <img src="/images/2023/10/15/31.png" width="800px">
</p>

<br/>

앱이 열리거나 닫힐때 조건을 선택한다.

<p align="center">
    <img src="/images/2023/10/15/32.png" width="800px">
</p>

<br/>

앱 - Obsidian, 조건 - 열릴 때, 즉시실행 으로 세팅 후 다음을 누른다.

<p align="center">
    <img src="/images/2023/10/15/33.png" width="800px">
</p>

<br/>

나의 단축어로 들어간다.

<p align="center">
    <img src="/images/2023/10/15/34.png" width="800px">
</p>

<br/>

Pull shortcut을 선택한다.

<p align="center">
    <img src="/images/2023/10/15/35.png" width="800px">
</p>

<br/>

Push shorcut도 동일하게 세팅하면 된다. 단, 조건은 앱이 닫힐 때로 바꿔준다.

이제 제대로 업데이트가 이루어지는지 확인해보자.

옵시디언에서 새로운 문서를 작성한 후 앱을 종료하면

이렇게 귀찮은 commit과 push가 앱을 닫을때 자동으로 스무스하게 실행된다!

<p align="center">
    <img src="/images/2023/10/15/36.png" width="800px">
</p>

<br/>

마찬가지로 앱을 실행할때 자동으로 pull이 된다.

<br/>

혹시라도 이 세팅으로 잘 사용하다가 pull, push가 어느 순간부터 에러가 난다면 Working Copy에서 발생하는 에러일 수 있다.

그럴땐 4번 과정에서 Link Repository to를 다시 해주면 해결된다.

<br/>

---

<br/>

### 세팅을 마치며

<br/>

이렇게 안드로이드를 제외한 모든 플랫폼에서 자동으로 백업과 동기화가 이루어지도록 옵시디언 세팅을 완료할 수 있었다.

알아본 바로는 Remotely Save라는 커뮤니티 플러그인과 드롭박스, 구글 드라이브의 조합, 또는 icloud 등을 이용해도 충분히 동기화 세팅을 진행할 수 있다.

그러나 이렇게 github을 이용하는 방법이 메모의 히스토리까지 저장 가능하다는 점, conflict가 생기는 상황에서 익숙한 방법으로 해결이 가능하다는 점이 좋아 이 방법을 선택했다.

사실 어떻게 세팅하든, 정작 메모를 적극적으로 활용하지 못한다면 의미가 없을 것이다.

앞으로 [PARA 노트 정리법](https://youtu.be/lkRQuMIbFYc?si=a94NSEd40HPS8RUW)에 따라 메모를 작성하며 생산성을 높여봐야겠다.
