---
layout: post
title: 파이썬 커스텀 패키지 import중 ModuleNotFoundError
categories: python
tags: 
  - python
  - linux
---

Argument Parser 구현 후 콘솔창에서 파이썬 파일을 실행하려 했으나 커스텀 파일의 모듈들이 import가 불가능한 문제가 발생했다.

```bash
>>> from [file_path.file_name.py] import [module_name]
Traceback (most recent call last):
  File "", line 1, in 
ModuleNotFoundError: No module named '[module_name]'
```

구글링 시 환경변수 문제임을 알 수 있었으나, 해결방법이 복잡하고 같은 문제가 생길때마다 커스텀 폴더의 패키지들을 환경변수에 등록하는 것이 너무 번거롭다고 생각되어 다른 방법을 찾아봤는데 해결방법은 의외로 간단했다. 실행파일을 최상위 폴더에 두고, 하위 폴더에 패키지가 포함되도록 아래와 같이 폴더 구조를 리팩토링 하면 된다.

```markdown
[최상위 폴더]
│
└ [하위 폴더]
│ │
│ └ [모듈 포함 패키지]
│
└ [실행파일]
```
