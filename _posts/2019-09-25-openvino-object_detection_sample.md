---
title: "Windows에서 OpenVINO을 활용한 Object Detection모델 최적화 및 Inference방법"
date: 2019-09-25 00:00:00 +0900
categories: OpenVINO
---

> 윈도우에서 Neural Compute Stick2(NCS2)와 OpenVINO을 활용하여 Object Detecion모델을 최적화하여 Inference하는 방법
>
> 공식 가이드 문서 [https://docs.openvinotoolkit.org](https://docs.openvinotoolkit.org/) 을 참고하여 작성

## Model Optimizer Developer이란?

학습한 모델을 배포하기위해 변환

- `.xml` - 모델의 네트워크정보
- `.bin` - weights 및 biases를 포함

![fig1](https://bjo9280.github.io/assets/images/2019-09-25/fig1.png)

## Converting a Tensorflow model

* openVINO 설치  <https://software.intel.com/en-us/articles/get-started-with-neural-compute-stick> 

* pretrained model을 [Object Detection Model Zoo](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md) 에서 다운로드

* C:\Program Files (x86)\IntelSWTools\openvino\deployment_tools\model_optimizer\requirements_tf.txt 패키지설치 

  ```
  pip install -r requirements_tf.txt
  ```

* mo_tf.py를 사용하여 .bin, .xml파일로 변환(cmd 관리자 권한으로 실행)

  ```
  python mo_tf.py --input_model=tmp/ssd_inception_v2_coco_2018_01_28/frozen_inference_graph.pb \
  --tensorflow_use_custom_operations_config extensions/front/tf/ssd_v2_support.json \
  --tensorflow_object_detection_api_pipeline_config tmp/ssd_inception_v2_coco_2018_01_28/pipeline.config \
  --reverse_input_channels \
  --data_type FP16 
  ```

  --data_type을 설정하지 않으면 default 값인 FP32로 설정됨 하지만 MYRIAD로 inference할때

   **[VPU] Unsupported precision FP32**에러가 나므로 --data_type FP16으로 해줌(CPU사용때는 상관없음)

  ![fig2](https://bjo9280.github.io/assets/images/2019-09-25/fig2.png)

  

## Inference Sample 빌드

* C:\Program Files (x86)\IntelSWTools\openvino\inference_engine\samples에서 빌드

  > Microsoft Windows* 10
  >
  > Microsoft Visual Studio* 2015, 2017, or 2019
  >

  ```
  build_samples_msvc.bat VS2015
  ```

* setupvars실행(PYTHONPATH 설정)

  ```
  C:\Program Files (x86)\IntelSWTools\openvino\bin\setupvars.bat
  ```

* 사용자 환경변수 Path에 추가

  * C:\Program Files (x86)\IntelSWTools\openvino\deployment_tools\inference_engine\bin\intel64\Release
  * C:\Program Files (x86)\IntelSWTools\openvino\deployment_tools\inference_engine\bin\intel64\Debug
  * C:\Program Files (x86)\IntelSWTools\openvino\opencv\bin

  ![fig3](https://bjo9280.github.io/assets/images/2019-09-25/fig3.png)

## Inference방법 

* C:\사용자계정\Documents\Intel\OpenVINO\inference_engine_samples_build\intel64\Release 경로에 빌드된 object_detection_sample_ssd.exe으로 inference (RFCN, SSD, Faster RCNNs지원)

  ```
  object_detection_sample_ssd -i image1.jpg -m frozen_inference_graph.xml -d MYRIAD
  ```

  ![fig4](https://bjo9280.github.io/assets/images/2019-09-25/fig4.png)

* 출력이미지

  ![fig5](https://bjo9280.github.io/assets/images/2019-09-25/fig5.png)

* 이슈 : faster rcnn로 학습한 모델을 Myrid device로 inference할때  Failed to allocate graph: NC_ERROR 에러발생



