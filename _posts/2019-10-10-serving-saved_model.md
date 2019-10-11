---
title: "직접 학습시킨 모델(ckpt)로 Serving하기"
date: 2019-10-10 00:00:00 +0900
categories: TensorflowServing
---

> 이전 POST에서는 Saved_model포맷으로 되어 있는 학습파일을 다운받아서 serving해봤다면 
>
> 이번에는 직접 학습시킨 모델로 Serving하는 과정을 작성

# 개요

* 일반적인 tensorflow model은 checkpoint(.ckpt)또는 protobuf(.pb)파일로 변환하여 저장됨

* Tensorflow Serving은 예제에서 보았듯이 saved_model을 사용

* 즉, 직접 학습시킨 모델을 serving하려면 변환해야됨

* 방법은 checkpoint(.ckpt)또는 protobuf(.pb)을 불러와 입력과 출력에 serving이 가능한 형태의input과output을 연결하여 protobuf 파일 생성할 것임

  > ckpt: weight 체크포인트파일
  >
  > pb: graph + weight
  >
  > saved_model: input/output(signature_def) + pb

* Serving에서 사용되는 그래프에는 입력 및 출력이 있으며 signature라고 함

* SignatureDefs는 용도에 따라서 Classification SignatureDef와 Predict SignatureDef 두가지로 나뉨

  * Cassification SignatureDef : 분류 모델에 최적화되어 클래스 종류나 클래스별 정확도등을 옵션으로 가짐
  * Predict SignatureDef : 분류 모델뿐 아니라 모든 모델에 범용적으로사용될 수 있는 형태

* <https://github.com/tensorflow/tensorflow/tree/master/tensorflow/python/saved_model> 를 보면 SavedModel을 생성하는 방법에 대해 나와있음

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

# Signature_defs

saved_model_cli 커맨드를 이용하여 SavedModel을 검사할 수 있으며 예제에서 사용된 mnist와 incetpin모델을 확인해보면 predict_image와 serving_default로 정의되어 있음

```bash
saved_model_cli show --dir /tmp/mnist_model/1 --all
```

![fig0](https://bjo9280.github.io/assets/images/2019-10-10/fig0.png)

```bash
saved_model_cli show --dir ~/SERVING_INCEPTION/SERVING_INCEPTION/1 --all
```

![fig1](https://bjo9280.github.io/assets/images/2019-10-10/fig1.png)

# My saved_model생성

직접 학습한 ckpt로 저장되있는 모델을 불러와 SignatureDef(input, scores)를 정의하여 Serving를 위한 saved_model을 생성할 것임

* 직접 학습한 모델은 input_shape=(-1, -1, 3) output_shape=(1, 2) 이며 signatureDef로 정의해 줄 것임

  ![fig2](https://bjo9280.github.io/assets/images/2019-10-10/fig2.png)

* 이전 포스트에서 inception예제는 위에 saved_model_cli로 봤듯이 input_shape=(-1), output_shape=(-1, 5)로 정의되어 잇는데  이미지의 path로만 inference하는듯??이 부분에서 혼란 (Signature_defs build방법을 좀더 공부해야봐겟음..)

* tf.saved_model.builder.SavedModelBuilder로 serving를 위한 saved_model을 생성

  >  <https://github.com/bjo9280/classification-inference/blob/master/cls_ckpt_to_saved_model.ipynb> 

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

* saved_model_cli로 확인해보면 signature_def 이름은 serving_default, input은 (-1, -1, 3), output은 (1, 2)로 생성되었음.

  ```bash
  saved_model_cli show --dir ./saved/1 --all
  ```

  ![fig5](https://bjo9280.github.io/assets/images/2019-10-10/fig5.png)

  

  

# Server/Client 실행

* 위에서 생성한 saved_model를 tensorflow serving을 이용하여 server실행

  ```bash
  bazel-bin/tensorflow_serving/model_servers/tensorflow_model_server \
  --port=9000 \
  --model_name=my_model \ 
  --model_base_path=/tmp/my_model/ 
  ```

  ![fig3](https://bjo9280.github.io/assets/images/2019-10-10/fig3.png)

* client python코드작성할때 Server model_name과 saved_model의 sig_name, input명을 확인

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

* client요청

  ```shell
  python my_client.py --server=<docker ip>:9000 --image test.jpg
  ```

* 2개의 class의 softmax값이 float_val에 출력됨

  ![fig4](https://bjo9280.github.io/assets/images/2019-10-10/fig4.png)





































