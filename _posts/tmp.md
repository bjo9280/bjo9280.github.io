# post목록

## saved model

<https://github.com/tensorflow/tensorflow/tree/master/tensorflow/python/saved_model> 

일반적인 tensorflow model export 이후 serving이 가능한 형태로 변환해야됨

Frozen model을 불러와 입력과 출력에 serving이 가능한 형태의 input과 output을 연결하여 Protocol buffer 파일 생성   

출처: <https://hoororyn.tistory.com/21?category=792594> [후로링의 프로그래밍 이야기] 

## batching

<https://github.com/tensorflow/serving/tree/master/tensorflow_serving/batching> 