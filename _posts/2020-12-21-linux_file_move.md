---
layout: post
title: 리눅스 폴더 덮어쓰기 (rsync)
categories: linux
tags: linux
---

리눅스 파일 이동시 다음과 같은 방법으로 덮어쓰기를 시도했는데 실패했다. 아무래도 타겟 폴더 안에 파일이 있으면 안되는 모양이다.

<br/>

```bash
mv -f [origin_directory/] [target_directory/]
```

<br/>

구글링으로 찾은 해결 방법은 다음과 같다.

<br/>

```bash
rsync -a [origin_directory/] [target_directory/]  # 폴더 동기화
rm -rf [origin_directory/]                        # 원본 폴더 삭제
```

<br/>

출처 : [https://moonlighting.tistory.com/158](https://moonlighting.tistory.com/158)
