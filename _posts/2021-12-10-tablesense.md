---
layout: post
title: Tablesense 논문 리뷰 (TableSense - Spreadsheet Table Detection with Convolutional Neural Networks)
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

<br/>

Spreadsheet table detection은 엑셀 파일 등에서 테이블이 존재하는 영역, 정확히는 top, left, bottom, right 네 방향의 boundary를 감지하는 종류의 과제를 말한다.
스프레드 시트라는 2차원 좌표계 내에서 bounding box를 추출하는 과제이므로 얼핏 보면 이미지 object detection과 비슷한 느낌이 있다.
실제로도 저자는 이 문제를 해결하기 위한 base algorithm으로 딥러닝 이미지 처리 분야에서 real-time object detection의 포문을 열었던 모델인 Faster R-CNN을 사용했다.
그러나 이 과제는 object detection과 결정적인 차이점이 존재한다.

<br/>

먼저, object detection은 평가지표가 Intersection-over-Union(IoU)라는 것이다.
Bounding Box라는 것이 어떤 절대적인 기준에 의해 라벨링 된 것이 아닌 인간의 시각을 기준으로 임의로 부여된 것이다 보니, 아래 그림과 같이 예측 bounding box가 ground truth와 약간의 차이가 있더라도 IoU는 충분히 높게 측정되고, 인간의 눈으로 보기에도 정답이라고 인정할 수준이 된다.

<br/>

-그림-

<br/>

하지만 엑셀 파일에서 테이블을 추출하는 작업은 가능한 한 셀의 오차도 없이 정확해야 한다. 이미지 처리에 비교하자면 1픽셀의 오차도 허용하면 안되는 상황인 것이다.
만약 테이블의 가장 우측 또는 최하단에 중요한 정보가 포함되어 있고, bounding box에서 이런 row나 column만 제외된다면 추출된 테이블 활용에 문제가 생길 것이다.

<br/>

그리고 입력 데이터의 성격도 다르다. 이미지는 각 픽셀당 3채널에 R, G, B 색상 정보를 가지고 있는데, 엑셀에서 하나의 셀이 품고 있는 정보는 배경색, 선 스타일, 입력값, 수식 등등... 훨씬 많다.
게다가 가로 또는 세로가 편향적으로 길쭉한 이미지나 객체는 거의 없지만, 엑셀 파일에선 가로보다 세로 길이가 100배 이상 길쭉한 비율을 가진 테이블을 흔히 찾을 수 있다.

<br/>

저자는 위 문제를 해결하기 위해 새로운 모델 구조와 평가방법을 제시했고, 관련 데이터셋을 구축하는 성과를 올렸다.
비록 이 분야 자체가 사람들의 관심도가 높은 편은 아니지만, 논문은 엑셀의 본고장 마이크로소프트의 연구진들에 의해 작성됐고 연구진들의 후속 논문에서도 이 Tablesense 논문이 꾸준하게 사용되고 있기 때문에 한번쯤 볼만한 논문이다.

<br/>

## IoU vs EoB

<br/>

여기서부턴 bounding box를 편의상 bbox로 줄여 부르겠다. 
Object detection의 가장 보편적인 평가지표는 Intersection-over-Union이다. 이는 예측 bbox $(B)$와 실제 bbox $(B')$의 일치도를 나타내는데, 두 bbox간의 교집합 넓이를 합집합 넓이로 나눈 것이다.

<br/>

$$
\mathrm{IoU} = \frac{\mathrm{area}(B \cap B')}{\mathrm{area}(B \cup B')}
$$

<br/>

이는 bbox의 절대적인 크기와 상관 없이 각 bbox간 오차의 상대적 비율만 고려한다. 따라서 큰 bbox간 IoU를 구할 수록 작은 절대 오차는 무시된다. 같은 크기지만 해상도가 다른 20pixel x 20pixel 이미지와 1000pixel x 1000pixel 이미지가 있을 때, 20 x 20 해상도 이미지에선 한쪽 boundary가 5픽셀 차이가 나면 인간의 시각 기준으로 매우 큰 차이로 느껴지겠지만, 1000 x 1000 해상도 이미지에선 한쪽 boundary가 5픽셀 차이나도 인간의 시각은 문제 없이 bbox를 정답으로 인식한다. 따라서 전체 영역이 커질수록 작은 차이가 무시되는 IoU는 엑셀 테이블 추출의 평가지표로 사용하기에 매우 부적합하기에 저자들은 새로운 평가지표를 제시한다.

<br/>

Error-of-Boundary는 예측과 정답 boundary의 최대 절대 오차가 기준이다.

<br/>

$$
\begin{align*}
\mathrm{EoB} = \mathrm{max}(
&\vert\mathrm{row_{top}^\mathit{B}} - \mathrm{row_{top}^\mathit{B'}}\vert, \vert\mathrm{row_{bottom}^\mathit{B}} - \mathrm{row_{bottom}^\mathit{B'}}\vert, \\ 
&\vert\mathrm{row_{left}^\mathit{B}} - \mathrm{row_{left}^\mathit{B'}}\vert, \vert\mathrm{row_{right}^\mathit{B}} - \mathrm{row_{right}^\mathit{B'}}\vert)
\end{align*}
$$

<br/>

예를 들어, 상/하/좌/우 boundary의 예측값과 정답이 각각 2/0/1/1 셀 만큼씩 차이가 난다면, top-boundary의 오차가 2로 가장 크고, 따라서 EoB는 이 경우 2가 된다.
덕분에 테이블이 크기와 상관 없이 영역이 아닌 boundary를 기준으로 평가할 수 있다.

<br/>

## Datasets & Framework

<br/>

스프레드 시트 파일을 웹에서 크롤링해 그 중 10220개의 시트에 라벨링을 해 훈련 셋으로 사용하고 이와 겹치지 않는 400개의 시트를 테스트 셋으로 사용했다.

<br/>

Tablesense는 다음 다섯 단계에 걸쳐 테이블을 추출한다.

<br/>

1. Cell Featurization
2. CNN Backbone
3. Region Proposal Network
4. Bounding Box Regresssion
5. Precise Bounding Box Regression

<br/>

Cell Featurization은 시트를 텐서로 변환하는 단계이다. 이미지 파일은 한 픽셀에 색상 3채널의 정보를 담고 있지만, 엑셀 파일은 한 셀당 20채널로 나타낸다. 각 채널마다 영문자 비율, 숫자 비율, 입력값 길이, 선 스타일 적용 유무, 배경색, 글자색, 수식 적용 여부 등등의 정보를 담게 된다.

<br/>

이후 CNN backbone, Region Proposal Network, Bounding Box Regression은 Faster R-CNN의 알고리즘을 그대로 사용한다. 약간의 차이점이 있다면 resnet backbone에서 pooling layer를 제거한것과, RPN의 anchor세트가 base size 8 ~ 4096, ratio 1/256 ~ 256 까지 존재하는 등 다양한 크기와 극단적으로 편향된 비율을 가진 케이스까지 포함한다는 점이다.

<br/>

주목해야 할 부분은 이 모델의 핵심 구조인 PBR(Precise Bounding Box Regression) 모듈이다. BBR모듈의 Regrion of Interest를 기반으로 세부적인 boundary를 보정하는데, 어떤 원리인지 자세히 알아보자.

<br/>

BBR 모듈로 출력된 RoI는 부정확한 boundary를 가지고 있기 때문에, 이 boundary를 기준으로 receptive field를 새로 설정해 예측값과 정답값의 차이를 구하게 된다.
좌, 우 boundary에 대해서는 수평방향으로 $2k$, 상/하 boundary에 대해서는 수직방향으로 $2k$의 사이즈를 가지는 좁은 receptive 필드는 예측 boundary와 실제 boundary의 오차를 최소화하는데 적합하다. 논문에선 적절한 $k$를 7로 설정했다. 네 방향의 receptive field로부터 추출된 feature map은 RoIAlign을 통해 $2k \times 2k$ 크기의 텐서로 고정되고, regression 이후 세부적인 보정값을 출력하게 된다. Receptive field의 폭 또는 높이와 RoIAlign 후의 폭 또는 높이가 같기 때문에 정보의 손실이 거의 없게 된다. PBR모듈의 출력값을 통해 BBR모듈이 출력한 RoI를 보정하게 되면 경계 오차가 매우 적은 bbox를 얻을 수 있다.

<br/>

## Target & Loss Function

<br/>

기존 Faster R-CNN의 손실함수는 다음과 같이 smooth L1을 사용한다.

<br/>

$$
L_\mathrm{reg}(t, t^{*}) = \sum_{i \in \{ x, y, w, h\}} \mathrm{smooth_\mathit{L_1}} (t_i - t_i^{*})
$$

<br/>

타겟은 다음과 같이 설정된다.

<br/>

$$
\begin{align*}
&t_x = (x - x_a) / w_a, t_w = \log(w/w_a) \\
&t_x^* = (x^* - x_a) / w_a, t_w^a = \log(w^*/w_a)
\end{align*}
$$

<br/>

$x, w$는 각각 box의 중심 $x$좌표와 폭을 나타내고, $y, h$에 대해서도 동일한 식을 사용한다. 첨자가 없는 문자는 예측값, 아래첨자 $a$가 붙은 문자는 anchor box의 값, 윗첨자 $\*$가 붙은 문자는 ground-truth의 값을 나타낸다. 이 식에서 $x$에 대한 손실함수의 기울기는 $w_a$에 대해 반비례하고, $w$에 대한 손실함수의 기울기는 $w$에 대해 반비례한다. 즉 anchor box가 클 수록 중심 좌표에 대한 가중치 업데이트 값이 작아지고, 마찬가지로 예측 bounding box가 클 수록 폭과 높이에 대한 가중치 업데이트 값이 작아진다. 이는 bbox의 크기와 상관 없이 안정적인 훈련을 보장하지만 boundary를 정확하게 예측하는 데에는 적합하지 않다.

<br/>

따라서 PBR 모듈은 새로운 손실함수와 타겟을 사용한다.

<br/>

$$
L_\mathrm{reg}(t, t^{*}) = \sum_{i \in \mathrm\{ top, bottom, left, right\}}} R (t_i - t_i^{*})
$$

$$
R(x) = \begin{cases}
      0.5x^2, & if \vert x \vert < k\\
      0.5k^2, & otherwise
    \end{cases}
$$

$$
\begin{align*}
&t_{left} = x - x_a - w/2, t_{right} = x - x_a + w/2 \\
&t_{left}^* = x^* - x_a - w^*/2, t^*_{right} = x^* - x_a + w^*/2
\end{align*}
$$

<br/>


## Evaluation Results

<br/>

<br/>

## Implementation

<br/>

오픈 소스로 여러 구현이 존재하는 Faster R-CNN 코드를 기반으로 이 Tablesense를 구현하면서 몇 가지 어려움에 부딛혔고, 약간의 수정으로 해결했다.

