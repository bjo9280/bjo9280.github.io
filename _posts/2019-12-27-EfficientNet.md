#  EfficientNet: Rethinking Model Scaling for Convolutional Neural Networks

## 개요

* 구글발표 2019.5 / Rethinking Model Scaling / MnasNet저자
* 현재 CNN은 두가지 방향으로 발전되고 있음
  1. 더 높은 성능을 내도록 연구 
  2. 성능은 괜찮으니 효율성을 높여서 비슷한 성능을 높이는 연구
* 이 논문은 1, 2번을 다 고려함(large모델에서 어떻게 효율을 늘릴지)

![1](D:\문서\EfficientNet\img\실험결과.png)

![1](D:\문서\EfficientNet\img\modelsize.png)



### Model scaling

* 성능을 높이기 위해 네트워크를 키우는 방식
* 가장  일반적인 방법으로는 depth(레이어), width(채널), image resolution을 키움 
* 기존 논문들은 3가지중에  한개만 스케일링하는 식으로 했음
  * 여러개를 고려 하려면 노가다 ->> optimal한 솔루션이라는 보장도없음
* Input 키우면 그만큼 레이어가 더 필요하며 resolution 커지면 특징을 더 잘뽑기위해 채널을 늘려야됨

![1](D:\문서\EfficientNet\img\기법예시.png)



## 실험

* 본 논문에서는 실제로 3가지 scaling 기법에 대해 각 scaling 기법마다 나머지는 고정해두고 1개의 scaling factor만 키워가며 정확도의 변화를 측정
* 비슷한 방식으로 이번엔 depth(d)와 resolution(r)을 고정해두고 width만 조절하며 정확도의 변화를 측정하는 실험을 수행
* 아래 실험들을 통해 3가지 scaling factor를 동시에 고려하는 것이 좋다는 것을 간단하게 입증

![1](D:\문서\EfficientNet\img\single실험.png)



![1](D:\문서\EfficientNet\img\compound실험.png)



## Compound Scaling

* 이 논문에서는 모델(F)를 고정하고 depth(d), width(w), resolution(r) 3가지를 조절하는 방법을 제안
* 이때 고정하는 모델(F)를 좋은 모델로 선정하는 것이 굉장히 중요하며 MnasNet(Platform-Aware Neural Architecture Search for Mobile)을 사용

![1](D:\문서\EfficientNet\img\efficientnet.png)

모델 구조는 MnasNet과 거의 유사하며 위의 표와 같은 구조로 구성

이 모델을 기점으로 3가지 scaling factor를 동시에 고려하는 **Compund Scaling** 을 적용

![1](D:\문서\EfficientNet\img\compound방법.png)

1. depth, width, resolution 은 각각 알파, 베타, 감마 나타냄
2. 베이스라인에 파이를 1로하고 grid search를 사용하여 타겟 데이터셋에서 좋은 성능을 보이는 값을 찾음

   > 이때 각각의 비율은 노란색으로 강조한 조건을 만족시킴(depth는 2배 키우면 FLOPS도 비례하여 2배 증가되지만 width와 resolution은 가로와 세로가 곱해지므로 제곱 배 증가함)
3. 본 논문에서는 알파 1.2 배타 1.1 감마1.15 찾음
4. 알파, 배타, 감마를 고정하고 파이를 조정해서 더 큰 네트워크로 키움 B1->B7까지함

## 결과

* ConvNet들에 비해 비슷한 정확도를 보이면서 parameter수와 FLOPS 수를 굉장히 많이 절약
* ImageNet 데이터셋에서 가장 높은 정확도를 달성했던 GPipe 보다 더 높은 정확도
* 3개의 scaling factor를 각각 고려할 때 보다 동시에 고려하였을 때 더 정교한 CAM을 얻을 수 있음 

![1](D:\문서\EfficientNet\img\결과.png)



![1](D:\문서\EfficientNet\img\classmap.png)

![1](D:\문서\EfficientNet\img\classmap2.png)







