---
layout: post
title: (MacOS) Vim을 기본 텍스트 에디터로 사용하기
categories: mac_os
tags: 
  - m1_mac
  - vim
  - automator
---

맥에서 Automator를 활용해 기본 텍스트 에디터를 Vim으로 설정하는 법을 정리해본다.

---

<br/>

### Automator & AppleScript 설정

<br/>

아래 사진에 따라 설정을 진행한다

<br/>

<p align="center">
	<img src="/images/2022/03/22/1.png" width="800px">
	<figcaption style="text-align:center; font-size:12px; color:#808080">
		런치패드 또는 Spotlight 검색으로 Automator 실행 -> '새로운 문서' 선택
 	</figcaption>
</p>

<br/>

<p align="center">
	<img src="/images/2022/03/22/2.png" width="800px">
	<figcaption style="text-align:center; font-size:12px; color:#808080">
		'응용 프로그램' 선택
 	</figcaption>
</p>

<br/>

<p align="center">
	<img src="/images/2022/03/22/3.png" width="800px">
	<figcaption style="text-align:center; font-size:12px; color:#808080">
		검색창에 'AppleScript' 검색 후 'AppleScript 실행' 더블클릭, 스크립트 입력창에 아래 코드 작성
 	</figcaption>
</p>

<br/>

```
on run {input, parameters}
	if (count of input) > 0 then
		tell application "System Events"
			set runs to false
			try
				set p to application process "iTerm"
				set runs to true
			end try
		end tell
		tell application "iTerm"
			activate
			set numItems to the count of items of input
			set launchPaths to ""
			repeat with x from 1 to numItems
				set filePath to quoted form of POSIX path of item x of input
				set launchPaths to launchPaths & " " & filePath
			end repeat
			tell current window
				delay 0.01
				create tab with default profile command "vim " & launchPaths
			end tell
		end tell
	end if
end run
```
<br/>

코드의 출처는 글 아래에 명시해 놨는데, 해당 글에서 소개한 코드에서 `create tab ...` 바로 위 라인에 `delay 0.01` 코드가 추가된 것을 알 수 있다.

맥 버전이나 기종마다 다른지는 확실히 알 수 없지만, 내 경우 이렇게 딜레이 코드를 넣지 않으면 iterm이 실행중이지 않을 때 텍스트 파일을 열면 iterm만 실행되고 Vim은 실행되지 않는 문제가 생겼다.

<br/>

<p align="center">
	<img src="/images/2022/03/22/4.png" width="800px">
	<figcaption style="text-align:center; font-size:12px; color:#808080">
		앱 이름 및 위치 지정
 	</figcaption>
</p>

<br/>

<p align="center">
	<img src="/images/2022/03/22/5.png" width="800px">
	<figcaption style="text-align:center; font-size:12px; color:#808080">
		'응용 프로그램'과 같은 적당한 위치에 작성한 앱 저장
 	</figcaption>
</p>

<br/>

<p align="center">
	<img src="/images/2022/03/22/6.png" width="800px">
	<figcaption style="text-align:center; font-size:12px; color:#808080">
		열고싶은 텍스트 파일 우클릭 -> 다음으로 열기 -> 기타
 	</figcaption>
</p>

<br/>

<p align="center">
	<img src="/images/2022/03/22/7.png" width="800px">
	<figcaption style="text-align:center; font-size:12px; color:#808080">
		저장한 앱 선택 -> '항상 선택한 응용 프로그램으로 열기' 체크
 	</figcaption>
</p>

<br/>

---

<br/>

### 실행 결과

<p align="center">
	<img src="/images/2022/03/22/8.png" width="800px">
	<figcaption style="text-align:center; font-size:12px; color:#808080">
		이제 같은 확장자를 가진 텍스트 파일들은 바로 Vim으로 다이렉트로 열 수 있다
 	</figcaption>
</p>

<br/>

 ---

 <br/>

### 출처

<br/>

* [https://gist.github.com/Huluk/5117702](https://gist.github.com/Huluk/5117702)
