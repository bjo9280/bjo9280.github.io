---
title: "Deploying Tensorflow model with Django(2)"
date: 2020-01-21 00:00:00 +0900
categories: Django TensorflowServing
---

> 이번 포스트에서는 Tensorflow Serving API를 이용하여 배포한 이미지 분류 모델을 inference하는 웹 어플리케이션을 django로 구현하는 방법을 설명
>
> Tensorflow Serving 관련된 자세한 내용은 이전 포스트를 참고 <https://bjo9280.github.io/tensorflowserving/serving-docker_tensorflow_serving/> 

# Inference Server 실행

1. AWS에서 제공하는 예제에서 imagenet 데이터셋으로 학습시킨 Inception모델을 다운로드 (직접학습시킨 모델을 사용하고 싶으면 [여기](https://bjo9280.github.io/tensorflowserving/serving-saved_model/) 를 참고)

   ```shell
   curl -O https://s3-us-west-2.amazonaws.com/aws-tf-serving-ei-example/inception.zip
   ```

2. model.config에 serving할 모델을 설정한 후에  tensorflow api를 이용하여 Server를 실행

   ```shell
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

   ```shell
   bazel-bin/tensorflow_serving/model_servers/tensorflow_model_server --port=9000 --model_config_file=./models.config
   ```

   

#  Web Application(client) 만들기

* 업로드시킨 이미지를 inferecne 하고 결과를 리다이렉션하여 테이블로 결과값을 리턴하는 웹페이지 구현

## 이미지 Upload

1. HTML에서 이미지를 upload하는 form을 구현

   ##### image_cls.html

   ```html
   {% raw %}
   <div class="row">
           <div class="ml-1 col-sm-6">
               <div id="msg"></div>
               <form method="post" id="image-form" enctype="multipart/form-data">
                   {% csrf_token %}
                   <input type="file" name="file" class="file" accept="image/*" onchange="form.submit()">
                   <div class="input-group my-3">
                       <input type="text" class="form-control" disabled placeholder="Upload File"
                              id="file">
                       <div class="input-group-append">
                           <button type="button" class="browse btn btn-primary">Browse...</button>
                           &nbsp;&nbsp;
                       </div>
                   </div>
   
               </form>
   
           </div>
           <div class="col-lg-6">
               <!-- Circle Buttons -->
               <div class="card shadow mb-4">
                   <div class="card-header py-3">
                       <h6 class="m-0 font-weight-bold text-primary">Preview</h6>
                   </div>
                   <div class="card-body">
                       <div class="ml-2 col-sm-12">
                           {% if uploaded_file_url %}
                           <img src="{{ uploaded_file_url }}" class="img-thumbnail">
                           {% else %}
                           <img src="https://placehold.it/250x250" id="preview" class="img-thumbnail">
                           {% endif %}
                       </div>
                   </div>
               </div>
           </div>
       {% endraw %}
   ```

2. upload 기능 구현(해당되는 js파일을 찾아서 소스추가)

   ##### sd-admin-2.js

   ```javascript
   $('input[type="file"]').change(function(e) {
     var fileName = e.target.files[0].name;
     $("#file").val(fileName);
   
     var reader = new FileReader();
     reader.onload = function(e) {
       // get loaded data and render thumbnail.
       document.getElementById("preview").src = e.target.result;
     };
     // read the image file as a data URL.
     reader.readAsDataURL(this.files[0]);
   });
   ```

3. MEDIA폴더에 업로드(설정된 폴더로 이미지를 저장시킴)

   ##### settings.py

   ```python
   ...
   MEDIA_URL = '/media/'
   
   STATICFILES_DIRS = [
       os.path.join(BASE_DIR,'static')
   ]
   MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
   ```

4. 업로드시킬 파일의 title, file의 변수를 선언

   ##### forms.py

   ```python
   class UploadFileForm(forms.Form):
       title = forms.CharField(required=False)
       file = forms.FileField()
   ```

5. POST로 전송되는 request.FILES['file']을 저장하고 저장한 url을 리다이렉션(preview 되도록)

   ##### view.py

   ```python
   from django.core.files.storage import FileSystemStorage
   
   def image_cls(request):
   
       if request.method == 'POST':
           myfile = request.FILES['file']
           fs = FileSystemStorage()
           filename = fs.save(myfile.name, myfile)
           uploaded_file_url = fs.url(filename)
   
           return render(request, 'page/image_cls.html', {'uploaded_file_url':uploaded_file_url})
   
       return render(request, 'page/image_cls.html')
   ```

## Client 요청

1. client요청하는 img_cls 함수에서 결과값을 가져와 리다이렉션

   ##### view.py

   ```python
   from django.conf import settings
   from .serving import img_cls
   
   BASE_DIR = getattr(settings, "BASE_DIR", None)
   CLS_SERVING_IP = getattr(settings, "CLS_SERVING_IP", None)
   
   def image_cls(request):
   
       if request.method == 'POST':
   
           myfile = request.FILES['file']
           fs = FileSystemStorage()
           filename = fs.save(myfile.name, myfile)
           uploaded_file_url = fs.url(filename)
           result = img_cls(CLS_SERVING_IP, BASE_DIR + uploaded_file_url)
      
           return render(request, 'page/image_cls.html', {'uploaded_file_url':uploaded_file_url,'results': result })
   
       return render(request, 'page/image_cls.html')
   ```



2. 아래 이미지처럼 표현되는 result값에서 classes, scores값만 list로 저장하여 return (top3의 결과값만 받아옴)

   ![fig](https://bjo9280.github.io/assets/images/2019-10-01/fig2.png)

   ##### serving.py

   ```python
   def img_cls(server, image):
   
       channel = grpc.insecure_channel(server)
       stub = prediction_service_pb2_grpc.PredictionServiceStub(channel)
       # Send request
       with open(image, 'rb') as f:
           # See prediction_service.proto for gRPC request/response details.
           data = f.read()
           request = predict_pb2.PredictRequest()
           request.model_spec.name = 'inception'
           request.model_spec.signature_name = 'predict_images'
           request.inputs['images'].CopyFrom(
               tf.contrib.util.make_tensor_proto(data, shape=[1]))
           result_ = stub.Predict(request, 10.0)  # 10 secs timeout
   
       result = []
       
       for i in range(3):
           result.append({'label': result_.outputs['classes'].string_val[i].decode('utf-8'), 'score' : result_.outputs['scores'].float_val[i]})
   
       return result
   ```

   

## 결과 페이지

1. view.py에서 results를 받아와 table로 결과를 보여줌

   ##### image_cls.html 

   ```html
   {% raw %}
   ...
   <div class="col-lg-3">
               <div class="card shadow mb-4">
                   <div class="card-header py-3">
                       <h6 class="m-0 font-weight-bold text-primary">Prediction</h6>
                   </div>
                   <div class="card-body">
                       <table class="table">
                           <thead>
                           <tr>
                               <th scope="col">#</th>
                               <th scope="col">label</th>
                               <th scope="col">score</th>
                           </tr>
                           </thead>
                           <tbody>
                           {% for result in results %}
                           <tr>
                               <th scope="row">{{ forloop.counter }}</th>
                               <td>{{ result.label }}</td>
                               <td>{{ result.score }}</td>
                           </tr>
                           {% endfor %}
                           </tbody>
                       </table>
   
                   </div>
               </div>
           </div>
       </div>
       {% endraw %}
   ```

2. 최종페이지

   ![fig3](https://bjo9280.github.io/assets/images/2020-01-21/result.png)







