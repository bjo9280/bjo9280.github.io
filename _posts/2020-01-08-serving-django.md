---
title: "Deploying Tensorflow model with Django(1)"
date: 2020-01-08 00:00:00 +0900
categories: Django TensorflowServing
---

> Tensorflow Serving으로 학습 모델을 배포하고 장고로 구현된 웹 어플리케이션을 통해여 inference하는 방법
>
> Serving 관련된 자세한 내용은 이전 포스트를 참고 <https://bjo9280.github.io/tensorflowserving/serving-docker_tensorflow_serving/> 

# Server 실행

Tensorflow serving api와 docker을 이용하여 <https://github.com/tensorflow/serving> 에서 제공되는 half_plus_two 모델을  배포시키는 방법

1. docker로 tensorflow/serving을 pull하고 git으로 tensorflow serving 소스를 clone함

   ```shell
   # Download the TensorFlow Serving Docker image and repo
   docker pull tensorflow/serving
   
   git clone https://github.com/tensorflow/serving
   ```

2. 다음과 같이 server을 실행시킴

   ```shell
   # Location of demo models
   TESTDATA="$(pwd)/serving/tensorflow_serving/servables/tensorflow/testdata"
   
   # Start TensorFlow Serving container and open the REST API port
   docker run -t --rm -p 8501:8501 \
       -v "$TESTDATA/saved_model_half_plus_two_cpu:/models/half_plus_two" \
       -e MODEL_NAME=half_plus_two \
       tensorflow/serving &
   ```

   ![fig1](https://bjo9280.github.io/assets/images/2020-01-08/serving.png)

   

