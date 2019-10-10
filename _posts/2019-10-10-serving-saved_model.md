---
title: "직접 학습시킨 모델로 Serving하기"
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

```
saved_model_cli show --dir /tmp/mnist_model/1 --all
```



![fig1](https://bjo9280.github.io/assets/images/2019-10-10/fig1.png)

작성중