---
layout: post
title: 도커로 주피터랩 실행
categories: linux
tags: 
  - Docker
  - linux
  - Jupyter
---

본인 설치환경 : 

OS : Ubuntu 18.04.5 LTS

CUDA : 11.1

Docker 20.10.6

nVidia-docker: [https://github.com/NVIDIA/nvidia-docker](https://github.com/NVIDIA/nvidia-docker)

<br/>

# 1. Deepo 이미지 설치

<br/>

딥러닝 / 머신러닝 라이브러리 다수 포함한 이미지 (Jupyter Lab은 미설치)

<br/>

```bash
$ docker pull ufoym/deepo
```

<br/>

설치 확인 

<br/>

```bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
...
ufoym/deepo         all-py36-cu101      5d5a5c342dbf        2 weeks ago         13.3GB
ufoym/deepo         latest              5d5a5c342dbf        2 weeks ago         13.3GB
...
```
<br/>

# 2. Docker Container 실행

<br/>

```bash
$ docker run -p {접속할 포트 번호}:8888 --gpus all -it -v {host-dir}:{container-dir} ufoym/deepo bash
```

<br/>

* run: 도커 실행 명령어
* "-p 8888:8888": 컨테이너의 8888번 포트를 호스트 OS 8888번 포트로 포워딩
* -gpus all": 컨테이너의 gpu를 사용
* "-it": 인터랙티브 터미널을 사용 (컨테이너 내에서 명령어 사용 가능해짐. 반대로 터미널로 접속 없이 외부에서 명령어 내릴땐 exec)
* "-v {host-dir}:{container-dir}": 호스트와 컨테이너가 각각 공유용으로 사용할 폴더
* "bash": 터미널에서 실행할 명령어

<br/>

# 3. Jupyter Lab 설치

<br/>

```bash
$ pip install jupyterlab   ## 터미널 내부에서 설치할 때
$ docker exec -it {container-name} pip install jupyterlab  ## 터미널 외부에서 설치할때
```

<br/>

# 4. Jupyter Lab 실행

<br/>

```bash
$ jupyter lab --ip 0.0.0.0 --port 8888 --allow-root --no-browser &
```

<br/>

# 5. 변경사항이 반영된 이미지를 새로 저장

<br/>

```bash
$ docker ps  ## 실행중인 도커 컨테이너 아이디 확인
$ docker commit {container-id} {new-image-name}
$ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
{new-image-name}    latest              19ea1eeb6d17        2 hours ago         13.3GB 
ufoym/deepo         all-py36-cu101      5d5a5c342dbf        2 weeks ago         13.3GB
ufoym/deepo         latest              5d5a5c342dbf        2 weeks ago         13.3GB
```

<br/>

# 6. 도커 컨테이너 및 주피터 백그라운드 / 포그라운드 조작

<br/>

* 컨테이너 실행 유지하면서 인터랙티브 터미널 빠져나오기(deattach) : ctrl+p 후 ctrl+q
* 컨테이너로 다시 접속(attach) :
```bash
$ docker attach {container-id}
```
* 컨테이너 종료 :
```bash
$ exit
```
* 주피터랩 백그라운드 전환 : ctrl+z (이 경우 프로세스는 일시정지 상태)
* 백그라운드 프로세스 목록 확인 :
```bash
$ jobs
```
* 정지된 프로세스 백그라운드에서 실행 :
```bash
$ bg  ## 모든 정지된 프로세스 전부 백그라운드 전환
$ bg {num}  ## 특정 번호의 프로세스만 백그라운드 전환
```
* 백그라운드 프로세스 포그라운드로 전환 :
```bash
$ fg % {num}
```
* 백그라운드 프로세스 PID 조회 및 강제 종료:
```bash
$ ps -ef|grep {process}  ## PID 조회
$ kill -9 {PID}  ## 강제종료
$ kill -9 `ps -ef|grep {process}`  ## 한번에
```

<br/>

출처 : 
* [https://velog.io/@vanang7/도커를-이용한-딥러닝-환경-구축하기](https://velog.io/@vanang7/도커를-이용한-딥러닝-환경-구축하기)
* [https://brownbears.tistory.com/166](https://brownbears.tistory.com/166)
