---
<<<<<<< HEAD
title: "Semantic segmentation 데이터생성 방법"
=======
title: "Segmentation 데이터생성 방법"
>>>>>>> 561ed806a7beba8312c62a1d9926b32af95c8727
date: 2021-04-01 00:00:00 +0900
categories: Segmantation
---

<<<<<<< HEAD
> Semantic segmentation에서 데이터셋을 생성하는 방법 정리

# 데이터셋 구성방법
=======
> Segmentation에서 데이터셋을 생성하는 방법 정리

# Segmentation 데이터셋 구성방법
>>>>>>> 561ed806a7beba8312c62a1d9926b32af95c8727

## Annotation tool 설치

### Installation

labelme : https://github.com/wkentaro/labelme

```
# python2
conda create --name=labelme python=2.7
source activate labelme
# conda install -c conda-forge pyside2
conda install pyqt
pip install labelme
# if you'd like to use the latest version. run below:
# pip install git+https://github.com/wkentaro/labelme.git

# python3
conda create --name=labelme python=3.6
source activate labelme
# conda install -c conda-forge pyside2
# conda install pyqt
pip install pyqt5  # pyqt5 can be installed via pip on python3
pip install labelme
```

# Sementic Segmentation

## Annotation 작업

가상환경에서 labelme실행 후 create polygons로 Ground Truth/레이블링 후 저장

이미지파일명.json 파일로 생성됨(label명, point위치, image경로)

![fig](https://bjo9280.github.io/assets/images/2021-04-01/annotation.png)

## Convert to Dataset

labels.txt 파일생성(background, ignore포함한 label명)

![fig](https://bjo9280.github.io/assets/images/2021-04-01/label.png)

```
생성방법
python labelme/examples/semantic_segmentation/labelme2voc.py \
labels.txt 생성된json경로 생성할폴더명

# It generates:
#   - data_dataset_voc/JPEGImages : 원본이미지
#   - data_dataset_voc/SegmentationClass : npy형식(semantic)
#   - data_dataset_voc/SegmentationClassPNG : ground truth이미지(semantic)
#   - data_dataset_voc/SegmentationClassVisualization :시각화 이미지(semantic)
```

 ![fig](https://bjo9280.github.io/assets/images/2021-04-01/labelme.PNG)



## Grayscale image생성

필요시 SegmentationClassPNG폴더에 있는 rgb이미지를 grayscale로 변환작업

```
folder =  'C:/labelme/examples/semantic_segmentation/data_dataset_voc'

for root, dirs, files in os.walk('{}/SegmentationClassPNG'.format(folder)):
    for fname in files:
        im = Image.open("{}/SegmentationClassPNG/{}".format(folder, fname))
        arr = np.array(im, dtype=np.float32)
        cv2.imwrite("{}/SegmentationClassAug/{}".format(folder, fname), arr)
```

![fig](https://bjo9280.github.io/assets/images/2021-04-01/labelme2.png)
<<<<<<< HEAD
=======

# Instance Segmentation

annotation방법은 동일하지만 sementic segmentation과 다르게 label명을 person-1, persion-2로 구분해줘야됨(labels.txt파일 생성시 sementic segmentation과 label명은 동일함)

## Convert to Dataset

```
생성방법
python labelme/examples/instance_segmentation/labelme2voc.py \
labels.txt 생성된json경로 생성할폴더명

# It generates:
#   - data_dataset_voc/JPEGImages : 원본이미지
#   - data_dataset_voc/SegmentationClass : npy형식(semantic)
#   - data_dataset_voc/SegmentationClassPNG : ground truth이미지(semantic)
#   - data_dataset_voc/SegmentationClassVisualization :시각화 이미지(semantic)
#   - data_dataset_voc/SegmentationObject : npy형식(instance)
#   - data_dataset_voc/SegmentationObjectPNG : ground truth이미지(instance)
#   - data_dataset_voc/SegmentationObjectVisualization : 시각화 이미지(instance)
```

 ![fig](https://bjo9280.github.io/assets/images/2021-04-01/instance.png)

왼쪽이미지: sementic segmentation / 오른쪽이미지: instance segmentation

# COCO Dataset의 구성

다운로드 경로 : http://cocodataset.org/#download

```
$ ls
000000000139.jpg  000000098018.jpg  000000190307.jpg  000000289702.jpg
000000384666.jpg  000000481413.jpg  000000000285.jpg  000000098261.jpg
000000190637.jpg  000000289741.jpg  000000384670.jpg  000000481480.jpg
...
$ ls annotations
captions_train2017.json  instances_train2017.json  person_keypoints_train2017.json
captions_val2017.json    instances_val2017.json    person_keypoints_val2017.json
```

val2017에는 5000장의 jpg 파일이 있습니다. (다른 데이터셋에는 더 많은 jpg 파일이 있습니다.) 그리고, annotations 디렉토리에는 6개의 json 파일이 있는데요. Training, validation 별로 captions, instances, person_keypoints 파일이 있습니다. 이것들은 각각

- captions - 텍스트로 된 그림에 대한 설명
- instances - 그림에 있는 사람/사물에 대한 category와 영역 mask
- person_keypoint - 사람의 자세 데이터 를 가지고 있습니다.

## Instances json file

Instances json file의 첫 부분은 아래와 같이 information과 license의 종류에 대한 내용입니다.

```
{
  "info": {
    "description": "COCO 2017 Dataset",
    "url": "http://cocodataset.org",
    "version": "1.0",
    "year": 2017,
    "contributor": "COCO Consortium",
    "date_created": "2017/09/01"
  },
  "licenses": [
    {
      "url": "http://creativecommons.org/licenses/by-nc-sa/2.0/",
      "id": 1,
      "name": "Attribution-NonCommercial-ShareAlike License"
    },
    ...
  ],
```

그 다음엔, 그림 파일에 대한 내용이 나옵니다. 

```
  "images": [
    ...
    {
      "license": 1,
      "file_name": "000000324158.jpg",
      "coco_url": "http://images.cocodataset.org/val2017/000000324158.jpg",
      "height": 334,
      "width": 500,
      "date_captured": "2013-11-19 23:54:06",
      "flickr_url": "http://farm1.staticflickr.com/169/417836491_5bf8762150_z.jpg",
      "id": 324158
    },
    ...
  ],
```

그 다음엔, 각 그림에 대한 annotation 정보가 나옵니다. Annotation이란, 그림에 있는 사물/사람의 segmentation mask와 box 영역, 카테고리 등의 정보를 말합니다. 아래 예는 [COCO API Demo](https://github.com/cocodataset/cocoapi/blob/master/PythonAPI/pycocoDemo.ipynb)에서 사용된 image인 324159 그림의 annotation 중 일부 입니다. 

![fig](https://bjo9280.github.io/assets/images/2021-04-01/coco.png) 

```
"annotations": [
    ...
    {
      "segmentation": [
        [
          216.7,
          211.89,
          216.16,
          217.81,
          215.89,
          220.77,

          ...
          212.16
        ]
      ],
      "area": 759.3375500000002,
      "iscrowd": 0,
      "image_id": 324158,
      "bbox": [
        196.51,
        183.36,
        23.95,
        53.02
      ],
      "category_id": 18,
      "id": 10673
    },
    ...
    {
      "segmentation": [
        [
          44.2,
          167.74,
          48.39,
          162.71,
          ...
          167.58
        ]
      ],
      "area": 331.9790999999998,
      "iscrowd": 0,
      "image_id": 324158,
      "bbox": [
        44.2,
        161.19,
        36.78,
        13.78
      ],
      "category_id": 3,
      "id": 345846
    },
    ...
  ],
```

마지막으로, category 리스트가 나옵니다.

```
  "categories": [
    {
      "supercategory": "person",
      "id": 1,
      "name": "person"
    },
    ...
  ]
}
```

출처

https://github.com/wkentaro/labelme

https://ukayzm.github.io/cocodataset
>>>>>>> 561ed806a7beba8312c62a1d9926b32af95c8727
