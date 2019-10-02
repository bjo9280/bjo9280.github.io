---
title: "Serving a Tensorflow Model in Docker -Tensorflow Serving"
date: 2019-10-01 00:00:00 +0900
categories: Tensorflow Serving
---

> docker에 tesnorflow serving을 bazel로 빌드하고 학습한 모델을 serving하는 방법 
>
> <https://github.com/tensorflow/serving> 을 참고하여 작성

## Tensorflow Serving이란?

* Tensorflow serving api or bazel로 빌드하는 방식이 있음

## Serve a Tensorflow model in 60 seconds

* tensorflow serving api를 이용하여 serving하는 방법

  ```
  # Download the TensorFlow Serving Docker image and repo
  docker pull tensorflow/serving
  
  git clone https://github.com/tensorflow/serving
  # Location of demo models
  TESTDATA="$(pwd)/serving/tensorflow_serving/servables/tensorflow/testdata"
  
  # Start TensorFlow Serving container and open the REST API port
  docker run -t --rm -p 8501:8501 \
      -v "$TESTDATA/saved_model_half_plus_two_cpu:/models/half_plus_two" \
      -e MODEL_NAME=half_plus_two \
      tensorflow/serving &
  
  # Query the model using the predict API
  curl -d '{"instances": [1.0, 2.0, 5.0]}' \
      -X POST http://localhost:8501/v1/models/half_plus_two:predict
  
  # Returns => { "predictions": [2.5, 3.0, 4.5] }
  ```

## Docker 설치

* 공식 Gitbub에서 Tensorflow serving을 Docker에 설치하는 걸 권장
* ubuntu16.04 + python3 설치

## Bazel 설치

bazel은 다양한 플랫폼에 설치할때나 여러가지 언어를 컴파일할때 사용한다.

> 설치방법은 <https://docs.bazel.build/versions/master/install-ubuntu.html>을 참고 
>

* 필요한 packages 설치

  ```
  apt-get install pkg-config zip g++ zlib1g-dev unzip python3
  ```

* <version>/bazel-<version> -installer-linux-x86_64.sh 형식으로 원하는 버전 다운로드

  ```
  wget https://github.com/bazelbuild/bazel/releases/download/0.24.1/bazel-0.24.1-installer-linux-x86_64.sh
  ```

* bazel 설치

  ```
  chmod +x bazel-0.24.1-installer-linux-x86_64.sh
  ./bazel-0.24.1-installer-linux-x86_64.sh
  ```

* 환경변수 설정

  ```
  export PATH="$PATH:$HOME/bin"
  ```

  

## gRPC 설치

* gRPC설치

  ```
  pip install grpcio
  ```

## Server / Client 실행

* tensorflow serving clone

  ```
  git clone https://github.com/tensorflow/serving
  ```

* bazel build

   ```
     cd serving
     bazel build -c opt --local_resources 5000,1.0,1.0 tensorflow_serving/…
   ```

* mnist 학습 파일 생성 (Docker환경에 tensorflow가 설치되어 있지 않으므로 설치하거나 or 호스트환경에서 학습 후 학습파일을 옮김)

  ```
  python tensorflow_serving/example/mnist_saved_model.py /tmp/mnist_model
  ```

* ifconfig로 docker환경의 ip확인(client 요청할때를 위해 확인)

* tensorflow serving 서버실행

  ```
  bazel-bin/tensorflow_serving/model_servers/tensorflow_model_server \
  --port=9000 \
  --model_name=mnist \ 
  --model_base_path=/tmp/mnist_model/
  ```

* cntl + p+ q 로 호스트환경으로 돌아오기

* Client 요청(mnist test이미지 1000장으로 inference한후에 성능측정하는 스크립트)

  ```
  python mnist_client.py --num_tests=1000 --server=<Docker IP>:9000
  ```
  ![fig1](https://bjo9280.github.io/assets/images/2019-10-01/fig1.png)

## Inception model 테스트

AWS에서 제공하는 Tensorflow serving 예제로 테스트 해보기 

> <https://docs.aws.amazon.com/ko_kr/dlami/latest/devguide/tutorial-tfserving.html#tutorial-tfserving-mnist> 

* Inception모델, TEST이미지 다운로드

  ```
  curl -O https://s3-us-west-2.amazonaws.com/aws-tf-serving-ei-example/inception.zip
  curl -O https://upload.wikimedia.org/wikipedia/commons/b/b5/Siberian_Husky_bi-eyed_Flickr.jpg
  ```

* 다운로드받은 모델을 원하는 위치에 압축해제

* Server 실행

  ```
  bazel-bin/tensorflow_serving/model_servers/tensorflow_model_server \
  --port=9000 \
  --model_name=inception \
  --model_base_path=/tmp/SERVING_INCEPTION
  ```

* 편집기를 사용하여 inception_client.py 스크립트 생성

  ```
  from __future__ import print_function
  
  import grpc
  import tensorflow as tf
  
  from tensorflow_serving.apis import predict_pb2
  from tensorflow_serving.apis import prediction_service_pb2_grpc
  
  
  tf.app.flags.DEFINE_string('server', 'localhost:9000',
                             'PredictionService host:port')
  tf.app.flags.DEFINE_string('image', '', 'path to image in JPEG format')
  FLAGS = tf.app.flags.FLAGS
  
  
  def main(_):
    channel = grpc.insecure_channel(FLAGS.server)
    stub = prediction_service_pb2_grpc.PredictionServiceStub(channel)
    # Send request
    with open(FLAGS.image, 'rb') as f:
      # See prediction_service.proto for gRPC request/response details.
      data = f.read()
      request = predict_pb2.PredictRequest()
      request.model_spec.name = 'inception'
      request.model_spec.signature_name = 'predict_images'
      request.inputs['images'].CopyFrom(
          tf.contrib.util.make_tensor_proto(data, shape=[1]))
      result = stub.Predict(request, 10.0)  # 10 secs timeout
      print(result)
    print("Inception Client Passed")
  
  if __name__ == '__main__':
    tf.app.run()
  ```

  

* client 요청으로 이미지 inference

  ```
  python inception_client.py --server=<Docker IP>:9000 --image Siberian_Husky_bi-eyed_Flickr.jpg
  ```

  ![fig2](https://bjo9280.github.io/assets/images/2019-10-01/fig2.png)

  

## 여러 모델을 동시에 serving

* config file을 생성하고 model_config_file 옵션으로 server실행

  ```
  # /models/models.config
  
  model_config_list: {
    config: {
      name: "mnist",
      base_path: "/tmp/mnist_model",
      model_platform: "tensorflow"
    },
    config: {
      name: "inception",
      base_path: "/tmp/SERVING_INCEPTION",
      model_platform: "tensorflow"
    },
  }
  ```

  ```
  bazel-bin/tensorflow_serving/model_servers/tensorflow_model_server --port=9000 --model_config_file=/models/models.config
  ```

  

  