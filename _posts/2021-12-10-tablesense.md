---
layout: post
title: Tablesense 논문 리뷰 (TableSense: Spreadsheet Table Detection with Convolutional Neural Networks)
categories: computer_vision
tags: 
  - computer_vision
  - Faster R-CNN
  - object_detection
use_math: true
---

**TableSense: Spreadsheet Table Detection with Convolutional Neural Networks** [논문](https://www.microsoft.com/en-us/research/publication/tablesense-spreadsheet-table-detection-with-convolutional-neural-networks/) 

<br/>

## Abstract & Problem Statement

Spreadsheet table detection은 엑셀 파일 등에서 테이블이 존재하는 영역, 정확히는 top, left, bottom, right 네 방향의 boundary를 감지하는 종류의 과제를 말한다.
스프레드 시트라는 2차원 좌표계 내에서 bounding box를 추출하는 과제이므로 얼핏 보면 이미지 object detection과 비슷한 느낌이 있다.
실제로도 저자는 이 문제를 해결하기 위한 base algorithm으로 딥러닝 이미지 처리 분야에서 real-time object detection의 포문을 열었던 모델인 Faster R-CNN을 사용했다.
그러나 이 과제는 object detection과 근본적인 차이점이 존재한다.

<br/>

먼저, object detection은 평가지표가 Intersection-over-Union(IoU)라는 것이다.
Bounding Box라는 것이 어떤 절대적인 기준에 의해 라벨링 된 것이 아닌 인간의 시각을 기준으로 임의로 부여된 것이다 보니, 아래 그림과 같이 예측 bounding box가 ground truth와 약간의 차이가 있더라도 IoU는 충분히 높게 측정되고, 인간의 눈으로 보기에도 정답이라고 인정할 수준이 된다.

<br/>

-그림-

<br/>

하지만 엑셀 파일에서 테이블을 추출하는 작업은 한셀의 오차도 없이 정확해야 한다. 이미지 처리에 비교하자면 bounding box에 1픽셀의 오차도 허용하면 안되는 상황인 것이다.
만약 테이블의 가장 우측 또는 최하단에 중요한 정보가 포함되어 있고, bounding box에서 이런 row나 column만 제외된다면 추출된 테이블 활용에 문제가 생길 것이다.

<br/>

그리고 입력 데이터의 성격도 다르다. 이미지는 각 픽셀당 3채널에 R, G, B 색상 정보를 가지고 있는데, 엑셀에서 하나의 셀이 품고 있는 정보는 배경색, 선 스타일, 입력값, 수식 등등... 훨씬 많다.
게다가 가로 또는 세로가 편향적으로 길쭉한 이미지나 객체는 거의 없지만, 엑셀 파일에선 가로보다 세로 길이가 100배 이상 길쭉한 비율을 가진 테이블을 흔히 찾을 수 있다.

<br/>

저자는 위 문제를 해결하기 위해 새로운 모델 구조와 평가방법을 제시했고, 관련 데이터셋을 구축하는 성과를 올렸다.
비록 이 분야 자체가 사람들의 관심도가 높은 편은 아니지만, 논문은 엑셀의 본고장 마이크로소프트의 연구진들에 의해 작성됐고 연구진들의 후속 논문에서도 이 Tablesense 논문이 꾸준하게 사용되고 있기 때문에 한번쯤 볼만한 논문이다.

<br/>

## IoU vs EoB

여기서부턴 bounding box를 편의상 bbox로 줄여 부르겠다. 
Object detection의 가장 보편적인 평가지표는 Intersection-over-Union이다. 이는 예측 bbox와 실제 bbox의 일치도를 나타내는데, 두 bbox간의 교집합 넓이를 합집합 넓이로 나눈 것이다.

<br/>

$$ IoU = \frac{area(B B)} $$

<br/>

이는 bbox의 절대적인 크기와 상관 없이 각 bbox간 오차의 상대적 비율만 고려한다. 따라서 큰 bbox간 IoU를 구할 수록 작은 절대 오차는 무시된다. 같은 크기지만 해상도가 다른 20pixel x 20pixel 이미지와 1000pixel x 1000pixel 이미지가 있을 때, 20 x 20 해상도 이미지에선 한쪽 boundary가 5픽셀 차이가 나면 인간의 시각 기준으로 매우 큰 차이로 느껴지겠지만, 1000 x 1000 해상도 이미지에선 한쪽 boundary가 5픽셀 차이나도 인간의 시각은 문제 없이 bbox를 정답으로 인식한다. 따라서 전체 영역이 커질수록 작은 차이가 무시되는 IoU는 엑셀 테이블 추출의 평가지표로 사용하기에 매우 부적합하기에 저자들은 새로운 평가지표를 제시한다.

<br/>

Error-of-Boundary는 예측과 정답 boundary의 최대 절대 오차가 기준이다.

<br/>

$$ EoB = max() $$

<br/>

예를 들어, 상/하/좌/우 boundary의 예측값과 정답이 각각 2/0/1/1 셀 만큼씩 차이가 난다면, top-boundary의 오차가 2로 가장 크고, 따라서 EoB는 이 경우 2가 된다.
덕분에 테이블이 작든 크든 bbox의 boundary 자체가 얼마나 정확하게 예측되는지 알 수 있다.

