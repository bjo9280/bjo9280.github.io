---
title: "Serving a Tensorflow model - Tensorflow Serving"
date: 2019-10-01 00:00:00 +0900
categories: Serving
---

> docker에 tesnorflow serving을 bazel로 빌드하고 학습한 모델을 serving하는 방법 
>
> <https://github.com/tensorflow/serving> 을 참고하여 작성

## Serve a Tensorflow model in 60 seconds

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

ubuntu16.04 + python3 설치

공식github에서 docker에 설치를 권장

## Bazel 설치

Terving Server는 C++기반에 gRPC 인터페이스 기반  

```
wget https://github.com/bazelbuild/bazel/releases/download/0.24.1/bazel-0.24.1-installer-linux-x86_64.sh
```

```
chmod +x bazel-0.24.1-installer-linux-x86_64.sh
./bazel-0.24.1-installer-linux-x86_64.sh
```

```
export PATH="$PATH:$HOME/bin"
```

## gRPC

```
pip install grpcio
```



## TF-serving

```
git clone https://github.com/tensorflow/serving
```

```
cd serving
bazel build -c opt --local_resources 5000,1.0,1.0 tensorflow_serving/…
```

```
python tensorflow_serving/example/mnist_saved_model.py /tmp/mnist_model
```

```
bazel-bin/tensorflow_serving/model_servers/tensorflow_model_server \
--port=9000 \
--model_name=mnist \ 
--model_base_path=/tmp/mnist_model/
```

```
bazel-bin/tensorflow_serving/example/mnist_client --num_tests=1000 --server=localhost:9000
```

docker설치

bazel설치

serving clone

bazel build

tensorflow.bzl

pip install numpy ,

```
pip install keras_applications==1.0.4 --no-deps
pip install keras_preprocessing==1.0.2 --no-deps
pip install h5py==2.8.0
```

apt-get install autoconf automake libtool   

작성중