---
title: "TensorFlow.js MNIST 예제(1)"
date: 2020-01-06 00:00:00 +0900
categories: TensorFlow.js
---

> tensorflow.js MNIST예제와 <https://github.com/tensorflow/tfjs-examples/tree/master/mnist-node>  
>
> javascript에 익숙치 않기 때문에 예제 소스의 생소한 부분도 스터디



## async & await

1. Callback은 비동기적인 작업이 길어질수록 콜백이 깊어지고 콜백 내에서 if문 분기와 에러 핸들링을 어렵게함
2. 이를 해결하기위해 Pomise패턴이 등작하여 비동기 작업을 콜백이 아닌 then으로 연결하고 catch로 에러 핸들링을 편하게 할 수 있다.
3. 하지만 잘못하용하면 then이 깊어 질 수 있고 if문 분기와 특정 에러 핸들링은 여전히 어려움 
4. 이를 해결하기위해 ES7에 async await가 등장함
5. async 함수 내부에서 await을 사용해 동기적으로 코드를 작성할수있음.
6. 에러 해들링과 if문 분기 또한 동기적으로 작성가능 

출처 : <https://velog.io/@ashnamuh/자바스크립트-콜백부터-async-await까지-비동기-처리>

## var let const

ES6는 var의 단점을 보완하기 위해 let과 const를 도입

* var는 *Hoisting되는 문제를 가짐
* let은 변수에 재 할당이 가능
* const는 변수 재 선언, 재 할당 모두 불가능

*Hoisting : 함수 안에 있는 선언들을 모두 끌어올려서 해당 함수 유효 범위의 최상단에 선언하는 것

참고 : <https://velog.io/@marcus/Javascript-Hoisting>

## NPM

Node Package Manager로 Node.js에서 사용할 수 있는 모듈들을 패키지화하여 모아둔 저장소 역할과 패키지 설치 및 관리를 위한 CLI(Command line interface)를 제공 

## 실행방법

npm install로 package.json에 명시된 패키지를 한번에 설치

```
npm install
```

```json
{
  "name": "tfjs-examples-mnist-node",
  "version": "0.1.0",
  "description": "",
  "main": "index.js",
  "license": "Apache-2.0",
  "private": true,
  "engines": {
    "node": ">=8.11.0"
  },
  "dependencies": {
    "@tensorflow/tfjs-node": "^1.5.1",
    "@tensorflow/tfjs-node-gpu": "^1.5.1",
    "argparse": "^1.0.10"
  },
  "scripts": {
    "train": "node main.js"
  },
  "devDependencies": {
    "clang-format": "~1.2.2"
  }
}
```

scripts에 명시된 방법으로 실행 (or npm으로 설치하지않고 node main.js로 실행가능) 

```
npm run train
```

## main

require를 사용하여  tfjs-node, data.js, model.js에서 module.exports됨 함수를 가져옴

* model.js : conv 모델 정보를 가짐
* data.js : mnist dataset 처리에 필요한 함수

```javascript
const tf = require('@tensorflow/tfjs-node');
const argparse = require('argparse');

const data = require('./data');
const model = require('./model');
```

data.getTrainData()에서 가져온 train, test이미지와 레이블링 정보를 가져와 model.fit()으로 학습 

```javascript
async function run(epochs, batchSize, modelSavePath) {
  await data.loadData();

  const {images: trainImages, labels: trainLabels} = data.getTrainData();
  model.summary();

  let epochBeginTime;
  let millisPerStep;
  const validationSplit = 0.15;
  const numTrainExamplesPerEpoch =
      trainImages.shape[0] * (1 - validationSplit);
  const numTrainBatchesPerEpoch =
      Math.ceil(numTrainExamplesPerEpoch / batchSize);
    
  await model.fit(trainImages, trainLabels, {
    epochs,
    batchSize,
    validationSplit
  });
```

model.evaluate()로 testset 성능 평가 후 acc, loss값 출력 후 학습된 모델을 model.save()로 저장

```javascript
  const {images: testImages, labels: testLabels} = data.getTestData();
  const evalOutput = model.evaluate(testImages, testLabels);

  console.log(
      `\nEvaluation result:\n` +
      `  Loss = ${evalOutput[0].dataSync()[0].toFixed(3)}; `+
      `Accuracy = ${evalOutput[1].dataSync()[0].toFixed(3)}`);

  if (modelSavePath != null) {
    await model.save(`file://${modelSavePath}`);
    console.log(`Saved model to path: ${modelSavePath}`);
  }
}
```

parser 인자값을 받아와 run함수를 실행

```javascript
const parser = new argparse.ArgumentParser({
  description: 'TensorFlow.js-Node MNIST Example.',
  addHelp: true
});
parser.addArgument('--epochs', {
  type: 'int',
  defaultValue: 2,
  help: 'Number of epochs to train the model for.'
});
parser.addArgument('--batch_size', {
  type: 'int',
  defaultValue: 128,
  help: 'Batch size to be used during model training.'
})
parser.addArgument('--model_save_path', {
  type: 'string',
  help: 'Path to which the model will be saved after training.'
});
const args = parser.parseArgs();

run(args.epochs, args.batch_size, args.model_save_path);
```





