# post목록

## saved model

<https://github.com/tensorflow/tensorflow/tree/master/tensorflow/python/saved_model> 

일반적인 tensorflow model export 이후 serving이 가능한 형태로 변환해야됨

Frozen model을 불러와 입력과 출력에 serving이 가능한 형태의 input과 output을 연결하여 Protocol buffer 파일 생성   

출처: <https://hoororyn.tistory.com/21?category=792594> [후로링의 프로그래밍 이야기] 

## batching

<https://github.com/tensorflow/serving/tree/master/tensorflow_serving/batching> 

```bash
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



