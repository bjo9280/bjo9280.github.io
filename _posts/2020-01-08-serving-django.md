---
title: "Deploying Tensorflow model with Django(1)"
date: 2020-01-08 00:00:00 +0900
categories: django TensorflowServing
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

3.  Query로 결과값을 request하는 방법

```shell
# Query the model using the predict API
curl -d '{"instances": [1.0, 2.0, 5.0]}' \
    -X POST http://localhost:8501/v1/models/half_plus_two:predict

# Returns => { "predictions": [2.5, 3.0, 4.5] }
```



# Web Application 만들기

tensorflow serving에 request를 보내는 역할을 django로 구현된 웹페이지에서 수행하도록 만들것임

1. 버튼 생성

##### base.html

```html
<a class="serving" href="{% url 'serving_half_plus_two' %}">
    <button type="button" class="btn btn">half_plus_two</button>
</a>
```

![fig3](https://bjo9280.github.io/assets/images/2020-01-08/web1.png)

2. instances value와 prediction result를 받아올 폼생성 

##### serving_half_plus_two.html

```html
<form action="" method="POST" class="form-horizontal">
    {% csrf_token %}
    <h2 class="post-add">half_plus_two </h2>
        <div class="row">
        <div class="col-sm-9">
            <div class="form-group">
                <label class="col-md-2 control-label">input1</label>
                <div class="col-md-3">
                    <input name="x_pred1" type="number" class="form-control" value="{{ x_pred1 }}" placeholder="">
                </div>
            </div>

            <div class="form-group">
                <label class="col-md-2 control-label">input2</label>
                <div class="col-md-3">
                    <input name="x_pred2" type="number" class="form-control" value="{{ x_pred2 }}" placeholder="">
                </div>
            </div>

            <div class="form-group">
                <label class="col-md-2 control-label">input3</label>
                <div class="col-md-3">
                    <input name="x_pred3"  type="number" class="form-control" value="{{ x_pred3 }}" placeholder="">
                </div>
            </div>
            <div class="form-group">
                <label class="col-md-2 control-label">Result</label>
                <div class="col-md-3">
                    <h4>{{ result }}</h4>
                    <button type="submit" class="btn btn-primary btn-lg">submit</button>
                </div>
            </div>
        </div>
    </div>

</form>
```

3. 페이지와 데이터를 처리할 view 코드

##### views.py

```python
def serving_half_plus_two(request):
    if request.method == 'POST':
        x_pred1 = request.POST['x_pred1']
        x_pred2 = request.POST['x_pred2']
        x_pred3 = request.POST['x_pred3']

        if x_pred1 == '' or x_pred2 == '' or x_pred3 == '' :
            return render(request, 'blog/serving_half_plus_two.html')

        load = {"instances": [float(x_pred1), float(x_pred2), float(x_pred3)]} #[1.0, 2.0, 5.0]
        r = requests.post(' http://localhost:8501/v1/models/half_plus_two:predict', json=load)
        y_pred = json.loads(r.content.decode('utf-8'))
        y_pred = y_pred['predictions']

        context = {
            'result': y_pred,
        }

        return render(request, 'blog/serving_half_plus_two.html', context)

    elif request.method == 'GET':
        return render(request, 'blog/serving_half_plus_two.html')
```

4. url링크

##### urls.py

```python
from django.contrib import admin
from django.conf.urls import url
from django.urls import path
from blog.views import post_list, post_detail, post_add, post_delete, serving_half_plus_two

urlpatterns = [
    path('admin/', admin.site.urls),
    path('serving/', serving_half_plus_two, name='serving_half_plus_two'),
]
```



![fig3](https://bjo9280.github.io/assets/images/2020-01-08/web2.png)