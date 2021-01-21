---
layout: post
title: 텐서보드 원격 실행 (bind_all 옵션)
categories: linux
tags: 
  - linux
  - tensorboard 
---

리눅스에선 ngrok로 localhost의 포트를 http로 연결해 원격으로 접속할 수 있다.
그러나 localhost에서 실행중인 tensorboard를 ngrok를 통해 원격 호스팅을 시도하면 연결이 되지 않는다.
다음과 같이 tensorboard 실행시 원격 접속을 허용하는 ```--bind_all```옵션을 붙여야만 ngrok를 통해 원격으로 접속할 수 있다.

<br/>

```bash
tensorboard --logdir runs --bind_all
```
