---
title: "Deploying Tensorflow model with Django(2)"
date: 2020-01-21 00:00:00 +0900
categories: Django TensorflowServing
---

> 이번 포스트에서는 간단하게 Tensorflow Serving api에서 제공되는 half_plus_two모델을 serving해보고 Django로 구현된 웹 어플리케이션을 통해여 결과값을 request하는 방법을 작성
>
> Tensorflow Serving 관련된 자세한 내용은 이전 포스트를 참고 <https://bjo9280.github.io/tensorflowserving/serving-docker_tensorflow_serving/> 

# Server 실행

1. 

   ```shell
   
   ```
   ```shell
   
   ```

   

   ```shell
   
   ```


#  Web Application 만들기



1. ##### 

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
                           <!--<button type="submit" class="btn btn-primary btn" style="float: right;">submit</button>-->
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

   ##### 

   ```html
   {% raw %}
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

   

   ##### 

   ```python
   def image_cls(request):
   
       if request.method == 'POST':
   
           myfile = request.FILES['file']
           fs = FileSystemStorage()
           filename = fs.save(myfile.name, myfile)
           uploaded_file_url = fs.url(filename)
           result = img_cls(CLS_SERVING_IP, BASE_DIR + uploaded_file_url)
           print(BASE_DIR + uploaded_file_url)
           return render(request, 'page/image_cls.html', {'uploaded_file_url':uploaded_file_url,'results': result })
   
       return render(request, 'page/image_cls.html')
   ```
   ##### 

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

2. 최종 페이지

   ![fig3](https://bjo9280.github.io/assets/images/2020-01-21/result.png)

   





