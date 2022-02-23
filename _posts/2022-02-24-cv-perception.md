---
layout: post
title: "Data custom"
date: 2022-02-24
excerpt: "Custom Aihub's data to COCO format for training YOLOX"
project: true
tags: [cv_perception]
comments: false
---


## 0. Goal
   Custom Aihub's data for training YOLOX.
  
## 1. Analyse Data Format
 For changing the Aihub's data format to COCO data format, you need to know the difference between the two formats.
 
 Aihub's Data Format
 --
Download [데이터 설명서](https://aihub.or.kr/aidata/27678) and check p.3~6.

A json file has information about __just one__ image.

Aihub's data format can be decomposed into two: __image, annotation__.

__image__ have filename, imsize

__annotation__ can be decomposed into four classes: __traffic_light, light, traffic_sign, accident_info__.

```
annotation
├── traffic_light
│ ├── light_count : int   //신로등 구 수
│ ├── direction : str     //신호등 몸체의 설치 방향
│ ├── type : str          //신호등 종류
│ ├── color : str         //신호등 몸체 색상
│ └── index : int         //몸체와 구를 묶는 용도
│
├──── light 
│ ├── type : str          //신호등 구의 형태
│ ├── color : str         //신호등 구의 색상
│ ├── arrow : str         //화살표 방향등의 방향
│ └── index : int         //몸체와 구를 묶는 용도
│
├── traffic_sign
│ ├── type : str          //도로교통표지판의 분류
│ ├── shape : str         //도로교통표지판의 모양
│ ├── color : str         //도로교통표지판의 색상
│ ├── command : str       //도로교통표지판의 의미
│ ├── number : int        //도로교통표지판의 고유 Code 번호
│ ├── kinds : str         //일반/LED 구분
│ └── text : int          //별도 표기가 필요한 표지판의 숫자값
│
└── accident_info
  └── type  : str         //유고정보 종류
```

 COCO Data Format
 --

One json file has all information.

COCO data format can be decomposed into five: __info, image, license, annotation, categories__.

you can check specific example and explain [here](https://towardsdatascience.com/how-to-work-with-object-detection-datasets-in-coco-format-9bf4fb5848a4) __(read Json Format part
)__. 

__image__ have a list of images in your dataset.

__annotation__ have a list of annotations which has bounding box information(x, y, w, h).

__categories__ have a list of classes which we have to classify(ex. stop sign, red light).

advice(중요하진 않음): ```Class COCOdataset``` is subclass of ```torch.utils.data.Dataset```

## 2. Check ```__getitem__``` python code.
* According to [YOLO_X official github](https://github.com/Megvii-BaseDetection/YOLOX/blob/main/yolox/data/datasets/coco.py), we should write the corresponding Dataset Class which can load images and labels through ```__getitem__``` method. Since we will use COCO data format, we should check [this code](https://github.com/Megvii-BaseDetection/YOLOX/blob/main/yolox/data/datasets/coco.py).

```
        One image / label pair for the given index is picked up and pre-processed.
        Args:
            index (int): data index
        Returns:
            img (numpy.ndarray): pre-processed image
            padded_labels (torch.Tensor): pre-processed label data.
                The shape is :math:`[max_labels, 5]`.
                each label consists of [class, xc, yc, w, h]:
                    class (float): class index.
                    xc, yc (float) : center of bbox whose values range from 0 to 1.
                    w, h (float) : size of bbox whose values range from 0 to 1.
            info_img : tuple of h, w.
                h, w (int): original shape of the image
            img_id (int): same as the input index. Used for evaluation.
```
* Modify ```__init__``` parameters
* By analzing the init variables, Json parameters which needed are easily known.

 ## 3. Result
 
 Aihub data format should be changed to the format below.
 
 ```
 image{
"id": int, 
"width": int, 
"height": int, 
"file_name": str, 
}
annotation{
"id": int, 
"image_id": int, 
"category_id": int, 
"area": float, 
"bbox": [x,y,width,height], 
}
categories[{
"id": int, 
"name": str, 
}]
```
### What we have to do
 1. Make categories according to the competition announcement.
 2. Move annotation classes into categories.
 3. Set image_id, annotation_id, categoties_id.
 4. Combine all josn files into one json file.
 5. Seperate imsize into width and height.
 
 =>Make program that performs this procedure
 
## Reference
Dataset: https://aihub.or.kr/aidata/27678

YOLO_X: https://github.com/Megvii-BaseDetection/YOLOX

COCO data format: https://cocodataset.org/#format-data

COCO data format example: https://towardsdatascience.com/how-to-work-with-object-detection-datasets-in-coco-format-9bf4fb5848a4

