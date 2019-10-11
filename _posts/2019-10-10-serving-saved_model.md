---
title: "직접 학습시킨 모델로 Serving하기(작성중)"
date: 2019-10-10 00:00:00 +0900
categories: TensorflowServing
---

일반적인 tensorflow model은 checkpoint(.ckpt)또는 -->> protobuf(.pb)파일로 변환하여 저장됨

serving에서는 예제에서 보았듯이 saved_model을 사용

직접학습시킨 모델을 serving하려면 변환해야됨

방법은 Frozen model을 불러와 입력과 출력에 serving이 가능한 형태의input과output을 연결하여 protobuf 파일 생성

ckpt: weight 체크포인트파일

pb: graph + weight

pb(saved_model): input/output(signature_def) + pb

Inference에서 사용되는 그래프에는 입력 및 출력이 있으며 signature라고 함

<https://github.com/tensorflow/tensorflow/tree/master/tensorflow/python/saved_model> 

<https://www.tensorflow.org/guide/saved_model> 



```python
export_dir = ...
...
builder = tf.saved_model.builder.SavedModelBuilder(export_dir)
with tf.Session(graph=tf.Graph()) as sess:
  ...
  builder.add_meta_graph_and_variables(sess,
                                       [tf.saved_model.tag_constants.TRAINING],
                                       signature_def_map=foo_signatures,
                                       assets_collection=foo_assets)
...
with tf.Session(graph=tf.Graph()) as sess:
  ...
  builder.add_meta_graph(["bar-tag", "baz-tag"])
...
builder.save()
```

```python
export_dir = ...
...
with tf.Session(graph=tf.Graph()) as sess:
  tf.saved_model.loader.load(sess, [tag_constants.TRAINING], export_dir)
  ...
```

<https://www.tensorflow.org/tfx/serving/signature_defs> 

SignatureDefs는 용도에 따라서 Classification SignatureDef와 Predict SignatureDef 두가지로 나뉘어 진다. Cassification SignatureDef는 분류 모델에 최적화되어 정의된 시그네쳐로 출력값들이 클래스 종류나 클래스별 정확도등을 옵션으로 가질 수 있고, Predict SignatureDef는 분류 모델뿐 아니라 모든 모델에 범용적으로사용될 수 있는 형태로 입력과 출력값을 정의할 수 있다. 

```
saved_model_cli show --dir /tmp/mnist_model/1 --all
```



![fig1](https://bjo9280.github.io/assets/images/2019-10-10/fig1.png)

## saved_model생성

학습했던 ckpt로 저장되있는 모델을 불러와 SignatureDef(input, scores)를 정의하여 inference를 위한 saved_model을 생성

* tf.saved_model.utils.build_tensor_info()를 이용하여 확인하면 input_shape=(-1, -1, 3), output_shape=(1, 2)

  ![fig2](https://bjo9280.github.io/assets/images/2019-10-10/fig2.png)

* tf.saved_model.builder.SavedModelBuilder로 serving를 위한 saved_model을 생성

  ```python
  from tensorflow.python.saved_model import signature_constants
  from tensorflow.python.saved_model import tag_constants
  
  saved_model_path = './saved/1'
  
  saver.restore(sess, input_checkpoint)
  builder = tf.saved_model.builder.SavedModelBuilder(saved_model_path)
  
  tensor_info_inputs = {
            'inputs': tf.saved_model.utils.build_tensor_info(image_input)}
  tensor_info_outputs = {
      'scores' : tf.saved_model.utils.build_tensor_info(tf.nn.softmax(logits))
  }
  
  detection_signature = (
          tf.saved_model.signature_def_utils.build_signature_def(
                inputs=tensor_info_inputs,
                outputs=tensor_info_outputs,
          method_name=signature_constants.PREDICT_METHOD_NAME))
  
  builder.add_meta_graph_and_variables(
            sess, [tf.saved_model.tag_constants.SERVING],
            signature_def_map={
                signature_constants.DEFAULT_SERVING_SIGNATURE_DEF_KEY:
                    detection_signature,
            },
        )
  
  builder.save()
  ```

## serving

위에서 생성한 saved_model를 tensorflow serving을 이용하여 server실행

```
bazel-bin/tensorflow_serving/model_servers/tensorflow_model_server \
--port=9000 \
--model_name=my_model \ 
--model_base_path=/tmp/my_model/ 
```

![fig3](https://bjo9280.github.io/assets/images/2019-10-10/fig3.png)

## client

client코드작성

```python
from __future__ import print_function
  
import grpc
import tensorflow as tf
import cv2
import numpy as np

from tensorflow_serving.apis import predict_pb2
from tensorflow_serving.apis import prediction_service_pb2_grpc
 
  
tf.app.flags.DEFINE_string('server', 'localhost:9000',
                           'PredictionService host:port')
tf.app.flags.DEFINE_string('image', '', 'path to image in JPEG format')
FLAGS = tf.app.flags.FLAGS
  
def read_image(im_path,flags=cv2.IMREAD_UNCHANGED):
    if im_path.endswith('.npy'):
        im = np.load(im_path)
    else:
        with open(im_path,'rb') as f:
            im_encoded = f.read()
        im = cv2.imdecode(np.frombuffer(im_encoded,dtype=np.uint8),flags=flags)
        if len(im.shape) > 2:
            im = im[:,:,::-1]
    return im
    
def main(_):
  channel = grpc.insecure_channel(FLAGS.server)
  stub = prediction_service_pb2_grpc.PredictionServiceStub(channel)
  # Send request
  with open(FLAGS.image, 'rb') as f:
    im = read_image(FLAGS.image, flags=cv2.IMREAD_COLOR)
    data = cv2.resize(im, (224, 224), interpolation=cv2.INTER_LINEAR)
    # See prediction_service.proto for gRPC request/response details.
    #data = f.read()
    
    
    request = predict_pb2.PredictRequest()
    request.model_spec.name = 'my_model'
    request.model_spec.signature_name = 'serving_default'
    request.inputs['inputs'].CopyFrom(
        tf.contrib.util.make_tensor_proto(data, shape=[224, 224, 3]))

    result = stub.Predict(request, 10.0)  # 10 secs timeout
    print(result)
  print("Client Passed")
  
if __name__ == '__main__':
  tf.app.run()
```

python실행

```
python my_client.py --server=<docker ip>:9000 --image test.jpg
```

## 결과

2개의 클래스이므로 float_val에 softmax결과가 출력

![fig4](https://bjo9280.github.io/assets/images/2019-10-10/fig4.png)































