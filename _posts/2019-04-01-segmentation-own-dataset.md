---
title: "Semantic segmentation 데이터생성 방법"
date: 2021-04-01 00:00:00 +0900
categories: Segmantation
---

> Semantic segmentation에서 데이터셋을 생성하는 방법 정리

# 데이터셋 구성방법
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

![fig](https://bjo9280.github.io/assets/images/2019-04-01/annotation.png)

## Convert to Dataset

labels.txt 파일생성(background, ignore포함한 label명)

![fig](https://bjo9280.github.io/assets/images/2019-04-01/label.png)

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

 ![fig](https://bjo9280.github.io/assets/images/2019-04-01/labelme.PNG)



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

![fig](https://bjo9280.github.io/assets/images/2019-04-01/labelme2.png)