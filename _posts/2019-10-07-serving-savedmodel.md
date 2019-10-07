---
title: "직접 학습시킨 모델로 Serving하기"
date: 2019-10-07 00:00:00 +0900
categories: Tensorflow Serving
---
일반적인 tensorflow model은 checkpoint(.ckpt) protobuf(.pb)파일로 저장됨
serving에서는 예제에서 보았듯이 saved_model을 사용하며 직접학습시킨 모델을 serving이 가능한 형태로 변환해야됨
방법은 Frozen model을 불러와 입력과 출력에 serving이 가능한 형태의input과output을 연결하여 Protocol buffer 파일 생성
작성중