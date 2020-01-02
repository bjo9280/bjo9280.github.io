---
title: "Node.js 스터디(진행중)"
date: 2020-01-02 00:00:00 +0900
categories: Node.js
---

> tensorflow.js 공부하기전에 Node.js에 대해서 공부
>
> <https://opentutorials.org/course/3332> 참고

## Synchronous vs Asynchronous

* 동기식 처리 모델(Synchronous processing model)은 직렬적으로 태스크(task)를 수행

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

