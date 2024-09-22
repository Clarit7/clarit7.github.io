---
layout: post
title: AWS Bedrock Tutorial - 음성파일로부터 자막과 이미지 생성하기(1)
categories: cloud
tags: 
  - aws
  - bedrock
---

<br/>

---

<br/>

### AWS Bedrock 활용한 토이 프로젝트 제작

<br/>

음성 파일을 업로드해 자막을 생성하고, 그 자막과 연관된 이미지를 생성하는 AWS Bedrock 토이 프로젝트의 튜토리얼이다.

사용된 AWS 서비스는 다음과 같다.

* IAM : 역할과 정책 관리
* API Gateway : 각종 API 생성 및 관리
* S3 : 파일 저장소
* Lambda : 서버리스 컴퓨팅
* Transcribe : 음성 파일 텍스트 변환 모델
* Bedrock : 생성형 AI 모델 사용

1편에선 음성파일을 S3버킷에 업로드하는 부분까지 설명한다.

<br/>

---

### 0. 주의점

<br/>

* 모든 서비스의 리전은 전부 같은 리전이어야 한다. 이 글에선 미국 동부(us-east-1)으로 진행한다.
* S3 버킷에 파일 생성을 트리거로 입력 파일을 받아 출력 파일을 생성하고, 그 출력 파일을 같은 버킷에 넣으면 안된다!!! **입력파일생성 - 트리거 - 출력파일생성(=입력파일생성) - 트리거 - 출력파일생성(=입력파일생성) - ...** 의 무한루프에 빠져 무한히 파일이 생성되며 **요금 폭탄이 나올 수 있다.** 따라서 반드시 입력파일이 생성되는 버킷과 출력파일이 생성되는 버킷은 구분해주어야 한다.

<br/>

### 1. IAM 역할 생성

<br/>

모든 프로세스에 대해 IAM에서 총 3개의 역할이 필요하다.

* S3에 음성 파일을 업로드 하는 역할
* 업로드된 음성 파일로부터 자막을 생성하는 역할
* 생성된 자막 파일로부터 이미지파일을 생성하는 역할

그리고 각 역할에는 다음과 같은 권한들이 필요하다.

* 음성파일 업로드
    1. API Gateway 기본 권한 (CloudWatch에 로그 생성)
    2. 음성파일이 업로드될 S3 버킷의 쓰기 권한
* 자막 생성
    1. Lambda 기본 권한 (CloudWatch에 로그 생성)
    2. 음성파일이 업로드된 S3 버킷의 읽기 권한
    3. 자막파일이 업로드될 S3 버킷의 쓰기 권한
    4. Transcribe 사용 권한
* 이미지 생성
    1. Lambda 기본 권한 (CloudWatch에 로그 생성)
    2. 자막파일이 업로드된 S3 버킷의 읽기 권한
    3. 이미지파일이 업로드될 S3 버킷의 쓰기 권한
    4. Bedrock 사용 권한

각 서비스들을 하나씩 생성할때마다 기본적으로 역할을 생성할 수 있기 때문에, 기본 생성된 역할에 나열된 권한을 추가해주면 된다.
단, API Gateway의 메서드는 미리 역할을 만들어놔야 하기 때문에, 역할 하나를 미리 만들고 진행한다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/0.png" width="800px">
</p>

<br/>

역할 탭에서 '역할 생성' 버튼을 클릭한다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/1.png" width="800px">
</p>

<br/>

신뢰할 수 있는 엔티티 유형에서 AWS 서비스를 선택 후, 아래 서비스 또는 사용 사례 목록에서 API Gateway를 선택하고 '다음'을 누른다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/2.png" width="800px">
</p>

<br/>

권한 추가 탭에선 따로 선택하지 말고 '다음'으로 넘어간다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/3.png" width="800px">
</p>

<br/>

<p align="center">
    <img src="/images/2024/09/22/4.png" width="800px">
</p>

<br/>

적당히 구분하기 좋은 이름을 입력 후, 밑으로 쭉 내려 '역할 생성'을 누른다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/5.png" width="800px">
</p>

<br/>

방금 생성한 오디오 업로드 역할에 대해, 추가 정책(권한)을 부여해야 한다. 정책 탭으로 가 '정책 생성'을 누른다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/6.png" width="800px">
</p>

<br/>

서비스에서 'S3'를 선택한다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/7.png" width="800px">
</p>

<br/>

권한 지정에서 작업 검색창에 PutObject를 검색 후, 밑에 나오는 리스트 중 'PutObject'에 체크한다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/8.png" width="800px">
</p>

<br/>

밑으로 가 리소스 항목에서 '특정'을 선택하고, 'ARN 추가'를 클릭한다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/9.png" width="800px">
</p>

<br/>

Resource bucket name에는 앞으로 만들 오디오 파일 업로드용 버킷 이름을 입력한다.

Resource object name에는 앞으로 업로드할 오디오 파일 이름을 입력한다.

ARN은 Amazon Resource Name의 약자로, 사진과 같이 입력하게 된다면 지금 설정하는 PutObjet 정책은 위의 ARN에 대해서만 유효하다.

즉, 아무 버킷에 아무 파일명으로 PutObject 할 수 있는게 아닌, my-toy-bucket-upload-audio-jay 버킷에만 my_audio로 시작하는 파일명으로 제한적인 업로드가 가능한 것이다.

다 입력했으면 'ARN 추가'를 누른다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/10.png" width="800px">
</p>

<br/>

ARN이 정상적으로 추가되었다면 '다음'을 누른다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/11.png" width="800px">
</p>

<br/>

<p align="center">
    <img src="/images/2024/09/22/12.png" width="800px">
</p>

<br/>

적당히 구분하기 쉬운 정책 이름을 입력하고, 밑으로 가 '정책 생성'을 누른다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/13.png" width="800px">
</p>

<br/>

정책 생성이 완료되었다면, 다시 역할 탭으로 가 앞에서 생성했던 S3 업로드용 역할을 클릭한다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/14.png" width="800px">
</p>

<br/>

'권한 추가' - '정책 연결'을 클릭한다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/15.png" width="800px">
</p>

<br/>

검색창에 방금 생성한 S3 putObject 정책을 검색 후, 체크박스를 선택하고 '권한 추가'를 누른다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/16.png" width="800px">
</p>

<br/>

최종적으로 S3 업로드 역할엔, 다음과 같이 CloudWatch 로그 권한과 putObject 권한이 생기게 된다.


### 2. API Gateway 생성

<br/>

<p align="center">
    <img src="/images/2024/09/22/17.png" width="800px">
</p>

<br/>

API 서비스에 접속 후, API 탭으로 이동해 'API 생성'버튼을 누른다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/18.png" width="800px">
</p>

<br/>

REST API에서 '구축'을 누른다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/19.png" width="800px">
</p>

<br/>

적당한 API이름을 입력하고 '생성'을 누른다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/20.png" width="800px">
</p>

<br/>

방금 생성한 API 하위 항목의 리소스 탭에서 '/'가 선택된 상태로 리소스 생성 버튼을 누른다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/21.png" width="800px">
</p>

<br/>

리소스 경로에 '/'가 선택되어있는지 확인하고, 리소스 이름에 {folder}를 입력 후 리소스 생성 버튼을 누른다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/22.png" width="800px">
</p>

<br/>

이번엔 '/{folder}'가 선택된 상태로 리소스 생성 버튼을 누른다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/23.png" width="800px">
</p>

<br/>

리소스 경로에 '/{fodler}/'가 선택되어있는지 확인하고, 리소스 이름에 {object}를 입력 후 리소스 생성 버튼을 누른다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/24.png" width="800px">
</p>

<br/>

리소스 경로에 '/{object}/'가 선택되어있는지 확인하고, 메서드 생성을 클릭한다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/25.png" width="800px">
</p>

<br/>

메서드 유형: PUT, 통합 유형: AWS 서비스, AWS 리전: us-east-1, AWS 서비스: S3, HTTP 메서드: PUT을 각각 선택한다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/26.png" width="800px">
</p>

<br/>

<p align="center">
    <img src="/images/2024/09/22/27.png" width="800px">
</p>

<br/>

경로 재정의에 {bucket}/{key}를 입력하고, 실행 역할에 아까 생성한 역할의 ARN을 복사해 붙여넣는다. 콘텐츠 처리는 패스스루를 선택하고 '다음'을 누른다.
여기서 아까 생성한 역할의 ARN은 IAM 서비스로 돌아가 위와 같은 위치에서 확인 및 복사를 할 수 있다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/28.png" width="800px">
</p>

<br/>

다시 API Gateway 리소스로 돌아와, PUT 메서드가 선택된 상태에서 통합 요청 탭의 '편집'을 누른다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/29.png" width="800px">
</p>

<br/>

하단에서 '경로 파라미터 추가'를 누른다. 총 두번 눌러 두개를 추가한다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/30.png" width="800px">
</p>

<br/>

위 입력칸의 이름에 bucket, 다음에서 매핑됨에 method.request.path.folder를 입력한다.

아래 입력칸의 이름에 key, 다음에서 매핑됨에 method.request.path.object를 입력하고 '저장'을 누른다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/30-1.png" width="800px">
</p>

<br/>

API 설정 탭으로 가 이진 미디어 유형 항목의 '미디어 유형 관리'를 클릭한다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/30-2.png" width="800px">
</p>

<br/>

이진 미디어 유형 추가 클릭 후, \*/\*를 입력 후 '변경 사항 저장'을 누른다. (모든 이진 미디어 유형을 허용하겠다는 의미)

<br/>

<p align="center">
    <img src="/images/2024/09/22/31.png" width="800px">
</p>

<br/>

다시 API의 리소스 창으로 가 'API 배포'를 누른다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/32.png" width="800px">
</p>

<br/>

\*새 스테이지\*를 선택하고, 스테이지 이름에 'v1'을 입력한다. '배포'를 누른다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/33.png" width="800px">
</p>

<br/>

API 배포가 완료되었다.

<br/>

### 3. S3 버킷 생성

<br/>

<p align="center">
    <img src="/images/2024/09/22/34.png" width="800px">
</p>

<br/>

S3 서비스로 이동해 버킷 만들기를 클릭한다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/35.png" width="800px">
</p>

<br/>

<p align="center">
    <img src="/images/2024/09/22/36.png" width="800px">
</p>

<br/>

앞에서 S3 업로드 정책에서 사용했던 ARN과 동일하게 S3 버킷을 만들어야 한다. 버킷 이름은 전 세계 모든 버킷에 대해 이름이 중복되면 안되므로, 만약 입력한 버킷명이 중복되었다면 변경 후 IAM 정책도 수정하도록 하자. 다른 설정들은 기본으로 두고 밑으로 쭉 내려 '버킷 만들기'를 완료한다.

<br/>

### 4. 테스트

Postman 앱과 같은 API 테스트 툴을 다운로드 받는다. 다른 툴이 익숙하다면 그대로 사용해도 된다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/36-1.png" width="800px">
</p>

<br/>

또한 [링크](https://www.espressoenglish.net/500audio/)에서 테스트용 샘플 mp3파일 모음을 다운받아 압축을 해제한다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/37.png" width="800px">
</p>

<br/>

API Gateway 스테이지 탭으로 가 앞서 생성한 API의 URL을 복사한다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/38.png" width="800px">
</p>

<br/>

Postman 앱에 URL을 복사한다.

메서드 종류는 PUT, URL은 복사한 텍스트 뒤에 '/업로드할 버킷명/업로드할 파일명'을 추가한다. 파일명은 앞서 정책에서 ARN 지정했던대로 my_audio로 시작해야 한다. 여기선 my_audio_1.mp3로 입력하자.

Body 탭에선 입력 음성파일을 업로드할 수 있는데, 파일 타입은 'binary'(이진 타입)를 선택하고, 앞서 다운로드 받았던 음성 파일 샘플 중 하나를 선택한다.

마지막으로 Send를 눌러 아래에 정상 응답(특별한 에러메세지가 없으며, 200 OK)이 도착했는지 확인한다.

<br/>

<p align="center">
    <img src="/images/2024/09/22/39.png" width="800px">
</p>

<br/>

생성한 파일이 정상적으로 버킷에 존재하는지 확인해보자.

<br/>

---

<br/>

### 출처

* https://www.youtube.com/watch?v=gXMZqaQC-T8
* https://www.espressoenglish.net/500audio/

<br/>
