---
title: "Tensorflow serving"
date: 2019-10-01 00:00:00 +0900
categories: Serving
---

> docker에 tesnorflow serving을 bazel로 빌드하고 학습한 모델을 serving하는 방법 
>
> <https://github.com/tensorflow/serving> 을 참고하여 작성

## Docker 설치

ubuntu16.04 + python3 설치

공식github에 docker에 설치를 권장

## Bazel 설치

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