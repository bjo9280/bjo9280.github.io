---
title: "TensorFlow.js MNIST 예제(작성중)"
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



## main.js

```javascript
const tf = require('@tensorflow/tfjs-node');
const argparse = require('argparse');

const data = require('./data');
const model = require('./model');

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



## model.js

```javascript
const tf = require('@tensorflow/tfjs');

const model = tf.sequential();
model.add(tf.layers.conv2d({
  inputShape: [28, 28, 1],
  filters: 32,
  kernelSize: 3,
  activation: 'relu',
}));
model.add(tf.layers.conv2d({
  filters: 32,
  kernelSize: 3,
  activation: 'relu',
}));
model.add(tf.layers.maxPooling2d({poolSize: [2, 2]}));
model.add(tf.layers.conv2d({
  filters: 64,
  kernelSize: 3,
  activation: 'relu',
}));
model.add(tf.layers.conv2d({
  filters: 64,
  kernelSize: 3,
  activation: 'relu',
}));
model.add(tf.layers.maxPooling2d({poolSize: [2, 2]}));
model.add(tf.layers.flatten());
model.add(tf.layers.dropout({rate: 0.25}));
model.add(tf.layers.dense({units: 512, activation: 'relu'}));
model.add(tf.layers.dropout({rate: 0.5}));
model.add(tf.layers.dense({units: 10, activation: 'softmax'}));

const optimizer = 'rmsprop';
model.compile({
  optimizer: optimizer,
  loss: 'categoricalCrossentropy',
  metrics: ['accuracy'],
});

module.exports = model;

```

## data.js

```javascript
const tf = require('@tensorflow/tfjs');
const assert = require('assert');
const fs = require('fs');
const https = require('https');
const util = require('util');
const zlib = require('zlib');

const readFile = util.promisify(fs.readFile);

// MNIST data constants:
const BASE_URL = 'https://storage.googleapis.com/cvdf-datasets/mnist/';
const TRAIN_IMAGES_FILE = 'train-images-idx3-ubyte';
const TRAIN_LABELS_FILE = 'train-labels-idx1-ubyte';
const TEST_IMAGES_FILE = 't10k-images-idx3-ubyte';
const TEST_LABELS_FILE = 't10k-labels-idx1-ubyte';
const IMAGE_HEADER_MAGIC_NUM = 2051;
const IMAGE_HEADER_BYTES = 16;
const IMAGE_HEIGHT = 28;
const IMAGE_WIDTH = 28;
const IMAGE_FLAT_SIZE = IMAGE_HEIGHT * IMAGE_WIDTH;
const LABEL_HEADER_MAGIC_NUM = 2049;
const LABEL_HEADER_BYTES = 8;
const LABEL_RECORD_BYTE = 1;
const LABEL_FLAT_SIZE = 10;

// Downloads a test file only once and returns the buffer for the file.
async function fetchOnceAndSaveToDiskWithBuffer(filename) {
  return new Promise(resolve => {
    const url = `${BASE_URL}${filename}.gz`;
    if (fs.existsSync(filename)) {
      resolve(readFile(filename));
      return;
    }
    const file = fs.createWriteStream(filename);
    console.log(`  * Downloading from: ${url}`);
    https.get(url, (response) => {
      const unzip = zlib.createGunzip();
      response.pipe(unzip).pipe(file);
      unzip.on('end', () => {
        resolve(readFile(filename));
      });
    });
  });
}

function loadHeaderValues(buffer, headerLength) {
  const headerValues = [];
  for (let i = 0; i < headerLength / 4; i++) {
    // Header data is stored in-order (aka big-endian)
    headerValues[i] = buffer.readUInt32BE(i * 4);
  }
  return headerValues;
}

async function loadImages(filename) {
  const buffer = await fetchOnceAndSaveToDiskWithBuffer(filename);

  const headerBytes = IMAGE_HEADER_BYTES;
  const recordBytes = IMAGE_HEIGHT * IMAGE_WIDTH;

  const headerValues = loadHeaderValues(buffer, headerBytes);
  assert.equal(headerValues[0], IMAGE_HEADER_MAGIC_NUM);
  assert.equal(headerValues[2], IMAGE_HEIGHT);
  assert.equal(headerValues[3], IMAGE_WIDTH);

  const images = [];
  let index = headerBytes;
  while (index < buffer.byteLength) {
    const array = new Float32Array(recordBytes);
    for (let i = 0; i < recordBytes; i++) {
      // Normalize the pixel values into the 0-1 interval, from
      // the original 0-255 interval.
      array[i] = buffer.readUInt8(index++) / 255;
    }
    images.push(array);
  }

  assert.equal(images.length, headerValues[1]);
  return images;
}

async function loadLabels(filename) {
  const buffer = await fetchOnceAndSaveToDiskWithBuffer(filename);

  const headerBytes = LABEL_HEADER_BYTES;
  const recordBytes = LABEL_RECORD_BYTE;

  const headerValues = loadHeaderValues(buffer, headerBytes);
  assert.equal(headerValues[0], LABEL_HEADER_MAGIC_NUM);

  const labels = [];
  let index = headerBytes;
  while (index < buffer.byteLength) {
    const array = new Int32Array(recordBytes);
    for (let i = 0; i < recordBytes; i++) {
      array[i] = buffer.readUInt8(index++);
    }
    labels.push(array);
  }

  assert.equal(labels.length, headerValues[1]);
  return labels;
}

/** Helper class to handle loading training and test data. */
class MnistDataset {
  constructor() {
    this.dataset = null;
    this.trainSize = 0;
    this.testSize = 0;
    this.trainBatchIndex = 0;
    this.testBatchIndex = 0;
  }

  /** Loads training and test data. */
  async loadData() {
    this.dataset = await Promise.all([
      loadImages(TRAIN_IMAGES_FILE), loadLabels(TRAIN_LABELS_FILE),
      loadImages(TEST_IMAGES_FILE), loadLabels(TEST_LABELS_FILE)
    ]);
    this.trainSize = this.dataset[0].length;
    this.testSize = this.dataset[2].length;
  }

  getTrainData() {
    return this.getData_(true);
  }

  getTestData() {
    return this.getData_(false);
  }

  getData_(isTrainingData) {
    let imagesIndex;
    let labelsIndex;
    if (isTrainingData) {
      imagesIndex = 0;
      labelsIndex = 1;
    } else {
      imagesIndex = 2;
      labelsIndex = 3;
    }
    const size = this.dataset[imagesIndex].length;
    tf.util.assert(
        this.dataset[labelsIndex].length === size,
        `Mismatch in the number of images (${size}) and ` +
            `the number of labels (${this.dataset[labelsIndex].length})`);

    // Only create one big array to hold batch of images.
    const imagesShape = [size, IMAGE_HEIGHT, IMAGE_WIDTH, 1];
    const images = new Float32Array(tf.util.sizeFromShape(imagesShape));
    const labels = new Int32Array(tf.util.sizeFromShape([size, 1]));

    let imageOffset = 0;
    let labelOffset = 0;
    for (let i = 0; i < size; ++i) {
      images.set(this.dataset[imagesIndex][i], imageOffset);
      labels.set(this.dataset[labelsIndex][i], labelOffset);
      imageOffset += IMAGE_FLAT_SIZE;
      labelOffset += 1;
    }

    return {
      images: tf.tensor4d(images, imagesShape),
      labels: tf.oneHot(tf.tensor1d(labels, 'int32'), LABEL_FLAT_SIZE).toFloat()
    };
  }
}

module.exports = new MnistDataset();
```



