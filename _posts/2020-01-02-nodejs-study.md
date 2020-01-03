---
title: "Node.js CRUD 스터디"
date: 2020-01-02 00:00:00 +0900
categories: Node.js
---

> tensorflow.js 공부하기 전 Node.js 스터디
>
> <https://opentutorials.org/course/3332> 에서 강의 내용을 참고하여 작성
>
> 전체 소스는 <https://github.com/web-n/Nodejs> 

## Synchronous vs Asynchronous

* 동기식 처리 모델(Synchronous processing model)은 직렬적으로 테스크(task)를 수행

  ```javascript
  var fs = require('fs');
  
  console.log('A');
  var result = fs.readFileSync('syntax/sample.txt', 'utf8');
  console.log(result);
  console.log('C');
  ```

  ```
  A
  B
  C
  ```

* 비동기식 처리 모델(Asynchronous processing model 또는 Non-Blocking processing model)은 병렬적으로 태스크를 수행

  ```javascript
  var fs = require('fs');
  
  console.log('A');
  fs.readFile('syntax/sample.txt', 'utf8', function(err, result){
      console.log(result);
  });
  console.log('C');
  ```

  ```
  A 
  C 
  B
  ```

* Collback

  ```javascript
  var a = function(){
    console.log('A');
  }
    
  function slowfunc(callback){
    callback();
  }
   
  slowfunc(a);
  ```

## PM2(Process Manager)

* Node.js의 프로세스 매니저로 프로세스가 죽었을 때에 대한 처리 
* 코드를 수정했을때 자동으로 프로세스 처리 

### 설치 및 실행

```
npm install pm2 -g
pm2 start main.js
```

### 기능

```
pm2 list
pm2 stop main
pm2 start main.js --watch
pm2 log
```

## HTML - Form

* 웹을 통해서 컨텐츠를 생성/삭제할때 html form 생성 방법

  ```javascript
  <form action="http://localhost:3000/process_create" method="post">
    <p><input type="text" name="title"></p>
    <p>
      <textarea name="description"></textarea>
    </p>
    <p>
      <input type="submit">
    </p>
  </form>
  ```

  

## 글쓰기 UI 만들기

* 글쓰기 template 생성 

  ```javascript
  //글쓰기 화면 만들기
  function templateHTML(title, list, body){
    return `
    <!doctype html>
    <html>
    <head>
      <title>WEB1 - ${title}</title>
      <meta charset="utf-8">
    </head>
    <body>
      <h1><a href="/">WEB</a></h1>
      ${list}
      <a href="/create">create</a>
      ${body}
    </body>
    </html>
    `;
  }
  ```

* 글쓰기 create페이지

  ```javascript
  else if(pathname === '/create'){
        fs.readdir('./data', function(error, filelist){
          var title = 'WEB - create';
          var list = templateList(filelist);
          //위에서 작성햇던 form  
          var template = templateHTML(title, list, `
            <form action="http://localhost:3000/process_create" method="post">
              <p><input type="text" name="title" placeholder="title"></p>
              <p>
                <textarea name="description" placeholder="description"></textarea>
              </p>
              <p>
                <input type="submit">
              </p>
            </form>
          `);
          response.writeHead(200);
          response.end(template);
        });
  ```

## POST방식으로 전송된 데이터 받기

* create페이지 -> create_process페이지

* post데이터 가져오기

  ```javascript
  var qs = require('querystring');
  
  //서버에 전달된 callback함수의 request를 사용
  var app = http.createServer(function(request,response){
  ...생략
  else if(pathname === '/create_process'){
        var body = '';
      
        //post에 데이터가 많을 경우를 대비해서 callback함수를 호출
        request.on('data', function(data){
            body = body + data;
        });
        //더 이상 데이터가 없으면 end에 해당되는 callback함수가 호출
        request.on('end', function(){
            //body의 데이터를 post로 선언하여 객체화해줌
            var post = qs.parse(body);
            var title = post.title;
            var description = post.description
        });
  ```

## 파일생성과 리다이렉션

* 사용자가 입력한 정보를 받아 데이터 디렉토리에 파일을 생성

* 작업을 완료 후에 특정 페이지로 보내고자 할 때 리다이렉션을 사용

  ```javascript
  else if(pathname === '/create_process'){
        var body = '';
        request.on('data', function(data){
            body = body + data;
        });
        request.on('end', function(){
            var post = qs.parse(body);
            var title = post.title;
            var description = post.description;
            //file, data, callback
            fs.writeFile(`data/${title}`, description, 'utf8',                   function(err){
              //302 리다이렉션, location 페이지위치
              response.writeHead(302, {Location: `/?id=${title}`});
              response.end();
            })
        });
      }
  ```

## 글수정-수정 링크 생성

* templateHTML에 control변수를 주고 template에 update 링크를 생성

  ```javascript
  else {
          fs.readdir('./data', function(error, filelist){
            fs.readFile(`data/${queryData.id}`, 'utf8', function(err, description){
              var title = queryData.id;
              var list = templateList(filelist);
              var template = templateHTML(title, list,
                `<h2>${title}</h2>${description}`,
                //title id를 가지는 update링크를 생성
                `<a href="/create">create</a> <a href="/update?id=${title}">update</a>`
              );
              response.writeHead(200);
              response.end(template);
            });
          });
        }
  ```



## 글수정-수정할 정보 전송

* 기존정보를 가져와서 보여줌

* submit 버튼을 눌르고 update_process로 갈때 기존 title 정보가 필요하며 hidden으로 보여지지 않도록함

  ```javascript
  else if(pathname === '/update'){
        fs.readdir('./data', function(error, filelist){
          fs.readFile(`data/${queryData.id}`, 'utf8', function(err, description){
            var title = queryData.id;
            var list = templateList(filelist);
            var template = templateHTML(title, list,
              `
              <form action="/update_process" method="post">
                <input type="hidden" name="id" value="${title}">
                <p><input type="text" name="title" placeholder="title" value="${title}"></p>
                <p>
                  <textarea name="description" placeholder="description">${description}                       </textarea>
                </p>
                <p>
                  <input type="submit">
                </p>
              </form>
              `,
                                       
              `<a href="/create">create</a> <a href="/update?id=${title}">update</a>`
            );
            response.writeHead(200);
            response.end(template);
          });
        });
  ```

  

## 글수정 - 수정된 내용 저장

* update_process에서 fs.rename(oldPath, newPath, callback)을 사용하여 기존 타이틀 이름을 변경

* fs.writeFile으로 newPath의 정보를 변경

  ```javascript
  else if(pathname === '/update_process'){
        var body = '';
        request.on('data', function(data){
            body = body + data;
        });
        request.on('end', function(){
            var post = qs.parse(body);
            var id = post.id;
            var title = post.title;
            var description = post.description;
            fs.rename(`data/${id}`, `data/${title}`, function(error){
              fs.writeFile(`data/${title}`, description, 'utf8', function(err){
                response.writeHead(302, {Location: `/?id=${title}`});
                response.end();
              })
            });
        });
      }
  ```



## 글삭제 - 삭제버튼 구현

* 삭제할때 링크로 구현하면 안되며 form으로 hidden속성을 갖도록함

*  delete_process로 post방식으로 보냄

* onsubmit 옵션으로 삭제확인버튼 구현가능

  ```javascript
  else {
          fs.readdir('./data', function(error, filelist){
            fs.readFile(`data/${queryData.id}`, 'utf8', function(err, description){
              var title = queryData.id;
              var list = templateList(filelist);
              var template = templateHTML(title, list,
                `<h2>${title}</h2>${description}`,
                ` <a href="/create">create</a>
                  <a href="/update?id=${title}">update</a>
                  <form action="delete_process" method="post">
                    <input type="hidden" name="id" value="${title}">
                    <input type="submit" value="delete">
                  </form>`
              );
              response.writeHead(200);
              response.end(template);
            });
          });
        }
      }
  ```



## 글삭제 - 기능 완성

* delete_process페이지로 title정보를 가져와 fs.unlink(path, callback)을 사용하여 file을 삭제

* 리다이렉션으로 홈으로 이동시킴

  ```javascript
  else if(pathname === '/delete_process'){
        var body = '';
        request.on('data', function(data){
            body = body + data;
        });
        request.on('end', function(){
            var post = qs.parse(body);
            var id = post.id;
            fs.unlink(`data/${id}`, function(error){
              response.writeHead(302, {Location: `/`});
              response.end();
            })
        });
      }
  ```

  

