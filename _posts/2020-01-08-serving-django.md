---
title: "Deploying Tensorflow model with Django(1)"
date: 2020-01-08 00:00:00 +0900
categories: Django TensorflowServing
---

> 이번 포스트에서는 간단하게 Tensorflow Serving api에서 제공되는 half_plus_two모델을 serving해보고 Django로 구현된 웹 어플리케이션을 통해여 결과값을 request하는 방법을 작성
>
> Serving 관련된 자세한 내용은 이전 포스트를 참고 <https://bjo9280.github.io/tensorflowserving/serving-docker_tensorflow_serving/> 

# Server 실행

Tensorflow serving api와 docker을 이용하여 half_plus_two 모델을  배포시키는 방법

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

3. Query로 결과 값을 request하는 방법

   ```shell
   # Query the model using the predict API
   curl -d '{"instances": [1.0, 2.0, 5.0]}' \
       -X POST http://localhost:8501/v1/models/half_plus_two:predict
   
   # Returns => { "predictions": [2.5, 3.0, 4.5] }
   ```

#  Web Application 만들기

* Tensorflow serving에 request를 보내는 역할을 django로 구현된 웹 페이지에서 수행하도록 만들것임
* django로 구현된 base어플리케이션은 [이곳](https://nachwon.github.io/django-1-setting)에서 blog 만드는 방법을 따라했으며
* 위의 코드에 serving하는 코드를 추가하였음 전체 소스는  [여기](https://github.com/bjo9280/django_tensorflow_serving)를참고

1. 버튼 / url 생성

   * url path name이 serving_half_plus_two으로 링크되도록 버튼생성 

   ##### base.html

   ```html
   {% raw %}
   <a class="serving" href="{% url 'serving_half_plus_two' %}">
       <button type="button" class="btn btn">half_plus_two</button>
   </a>
   {% endraw %}
   ```

   ##### urls.py

   ```python
   from blog.views import serving_half_plus_two
   
   urlpatterns = [
       ...
       path('serving/', serving_half_plus_two, name='serving_half_plus_two'),
       ...
   ]
   ```

   

2. input값을 보낼 form과 prediction 결과를 받아올 form 생성 

   * input name이 x_pred1,2,3인 form에서 value값을 가져와서 view.py에 POST방식으로 전송해줌
   * views.py에서 전송해준 데이터의 결과값을 예측하여 {{ result }}로 보냄

   ##### serving_half_plus_two.html

   ```html
   {% raw %}
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
   {% endraw %}
   ```

3. 데이터를 처리할 view 코드

   * html페이지에서 POST로 전송된  x_pred1,2,3값을 가져옴
   * float로 타입 변환 후에 http://localhost:8501/v1/models/half_plus_two:predict로 requests함
   * return한 결과를 result에 저장 후 리다이렉션

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
   ​```
   ```

4. 최종 페이지

   ![fig3](https://bjo9280.github.io/assets/images/2020-01-08/web2.png)

   





