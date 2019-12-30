---
title: "EfficientDet : Scalable and Efficient Object Detection Review"
date: 2019-12-30 00:00:00 +0900
categories: Object Detection
---
* EfficientNet을 사용하여 기존 모델들의 성능 보다 높으며, 특히 연산량, 연산 속도 관점에서는 굉장히 효율적인 모델

![실험결과](https://bjo9280.github.io/assets/images/2019-12-30/실험결과.png)

# Main Contribution

- Weighted bidirectional feature network (BiFPN)을 제안
- Object Detection에도 Compound Scaling을 적용하는 방법을 제안.
- BiFPN과 Compound Scaling을 접목하여 좋은 성능을 보이는 EfficientDet 구조를 제안

## FPN(Feature Pyramid Network)

* Top-down 방식으로 추출된 결과들인 low-resolution 및 high-resolution 들을 묶는 방식
* 상위 레벨의 이미 계산 된 특징을 재사용 하므로 멀티 스케일 특징들을 효율적으로 사용할 수 있음

![fpns](https://bjo9280.github.io/assets/images/2019-12-30/fpn.png)

## BiFPN

### 배경

- One-Stage Detector의 대표격인 모델인 RetinaNet, M2Det, AutoML의 Neural Architecture Search를 FPN 구조에 적용한 NAS-FPN 등 FPN을 적용하고, 성능을 개선하고자 하는 연구들이 많이 진행됨

  ![ob_list](https://bjo9280.github.io/assets/images/2019-12-30/ob_list.jpg)

- 선행 연구들은 모두 서로 다른 input feature들을 합칠 때 구분없이 단순히 더하는 방식을 사용하고 있음

- 서로 다른 input feature들은 해상도가 다르기 때문에 output feature에 기여하는 정도를 다르게 가져가야 함을 주장하며, (단순히 더하면 같은 weight로 기여하게 됨) 

- BiFPN 구조를 사용하면 input feature들의 중요성을 학습을 통해 배울 수 있으며, 이를 통해 성능을 많이 향상시킬 수 있음

### Cross-Scale Connections 

* (c) 구조부터는 scale이 다른 경우에도 connection이 존재

* (f) 그림의 보라색 선처럼 같은 scale에서 edge를 추가하여 더 많은 feature들이 fusion되도록 구성을 한 방식이 BiFPN

* 이를 통해 더 high-level한 feature fusion을 할 수 있음 

![bifpn](https://bjo9280.github.io/assets/images/2019-12-30/bifpn.png)

### 실험

![bifpn성능](https://bjo9280.github.io/assets/images/2019-12-30/bifpn성능.png)

### Weighted Feature Fusion

* 각 input feature에 가중치를 주고, 학습을 통해 가중치를 배울 수 있는 방식을 제안 
* weight는 scalar (per-feature)로 줄 수 있고, vector (per-channel)로 줄 수 있고 multi-dimensional tensor (per-pixel)로 줄 수 있는데, 본 논문에서는 scalar를 사용하는 것이 정확도와 연산량 측면에서 효율적임을 실험을 통해 밝혔고, scalar weight를 사용 

### 실험

![featurefusion](https://bjo9280.github.io/assets/images/2019-12-30/featurefusion.png)

- Unbounded fusion  : unbounded 되어있기 때문에 학습에 불안정성을 유발
- SoftMax-based fusion : GPU하드웨어에서 slowdown을 유발
- Fast normalized fusion 
  - weight들은 ReLU를 거치기 때문에 non-zero임이 보장이 되고 분모가 0이 되는 것을 막기 위해 0.0001 크기의 입실론을 넣어줌 
  - Weight 값이 0~1사이로 normalize가 되는 것은 SoftMax와 유사하며 ablation study를 통해 SoftMax-based fusion 방식보다 좋은 성능을 보임 

![featurefusion결과](https://bjo9280.github.io/assets/images/2019-12-30/featurefusion결과.png)

학습을 거치면서 weight가 빠르게 변하는 것을 보여주고 있고, 이는 각 feature들이 동등하지 않게 output feature에 기여를 하고 있음을 보여주고 있으며, Fast fusion을 사용하여도 SoftMax fusion과 양상이 비슷함을 보여주고 있음

![featurefusion결과](https://bjo9280.github.io/assets/images/2019-12-30/featurefusion결과2.png)

SoftMax fusion과 Fast Fusion을 비교한 결과이며, Fast Fusion을 사용하면 약간의 mAP 하락은 있지만 약 30%의 속도 향상 

## EfficientDet

 BiFPN을 기반으로 **EfficientDet** 이라는 One-Stage Detector 구조를 제안

### EfficientDet Architecture

* EfficientDet의 backbone으로는 ImageNet-pretrained EfficientNet을 사용
* BiFPN을 Feature Network로 사용하였고, level 3-7 feature에 적용
* top-down, bottom-up bidirectional feature fusion을 반복적으로 사용

![effcientdet](https://bjo9280.github.io/assets/images/2019-12-30/effcientdet.png)

### Compound Scaling

* Backbone network에는 EfficientNet-B0 부터 B6까지 사용
* ImageNet-pretrained network를 사용
* input의 resolution과 backbone network의 크기를 늘려주었고, BiFPN과 Box/class network 도 동시에 키워줌

![compound](https://bjo9280.github.io/assets/images/2019-12-30/compound.png)

## 결과

* COCO 데이터셋에서 가장 높은 mAP를 달성하여, 2019년 11월 기준 State-of-the-art(SOTA) 성능을 보임
* 기존 방식들 대비 연산 효율이 압도적으로 좋음 

![effcientdet결과](https://bjo9280.github.io/assets/images/2019-12-30/effcientdet결과.png)

![featurefusion결과2](https://bjo9280.github.io/assets/images/2019-12-30/effcientdet결과2.png)

 