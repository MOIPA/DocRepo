title: Yolo视觉识别
author: tr
tags:
  - AI
categories:
  - AI
date: 2023-02-03 05:24:00
---
# Yolo算法

> Yolo是一个非常棒的视觉识别算法，项目地址：https://github.com/ultralytics/yolov5
>
> 个人地址：`/home/tr/Documents/JupyterNotebook/YOLOv5`
>
> 在myTrain.ipynb中写了使用训练的demo


![upload successful](/images/pasted-191.png)

## 文件目录介绍


![upload successful](/images/pasted-192.png)

1. data目录：存放训练数据集，数据标记，训练配置等（yaml文件）

2. model 模型算法核心（不用关注）

3. runs 每执行一次detect，或者训练都会在目录下生成新的结果目录

4. weights 存放模型结果

## 标记图片工具-生成训练工具

```
docker pull heartexlabs/label-studio:latest\n
docker run -it -p 6002:8080 -v ~/Documents/LabelStudio/mydata:/label-studio/data heartexlabs/label-studio:latest label-studio --log-level DEBUG
```

docker 拉取完毕后访问6002端口即可，注册账户，上传图片进行标记，具体使用方法很简单，标记完毕后导出数据格式为yolo


![upload successful](/images/pasted-193.png)

## 编写训练配置

> 在label-studio中标记好导出的数据有两个目录，images和labels，labels内每个txt文件对应各自的图片文件，第一列为物品类别，后面分别是方框的xy坐标
>
> 需要注意的是，第一列的类别标签一会要作为配置文件内的类别，不能弄混
>
> 假设导出的数据目录为：tr03(内有images和labels目录)，放入YOLOV5/datasets内，创建文件：YOLOV5/yolov5/data/tr03.yaml，内容如下

```yaml
# Train/val/test sets as 1) dir: path/to/imgs, 2) file: path/to/imgs.txt, or 3) list: [path/to/imgs1, path/to/imgs2, ..]
path: ../datasets/tr03  # dataset root dir
train: images/  # train images (relative to 'path') 128 images
val: images/  # val images (relative to 'path') 128 images
test:  # test images (optional)


# Classes
names:
  0: TR
  1: person
  2: chair
```

## Yolo算法的训练

> Yolo已经存在了一些训练好的模型，我们再次训练的时候，可以基于已有的模型，也可以自己从头训练
>
> `!python train.py --img 640 --batch 16 --epochs 600 --data tr03.yaml --weights ./weights/yolov5s.pt ` 这个命令就是基于 weights目录下的yolov5s.pt模型训练，训练600次

![upload successful](/images/pasted-194.png)

> 执行完毕以后，会在runs目录下生成最新结果，我们的模型文件也在其中

![upload successful](/images/pasted-195.png)

## Yolo算法的使用

> 有了模型文件后可以利用模型识别各种图片

```python
# YOLOv5 PyTorch HUB Inference (DetectionModels only)
import torch

# model = torch.hub.load('ultralytics/yolov5', 'yolov5s.pt')  # yolov5n - yolov5x6 or custom
model = torch.hub.load('./','custom', './runs/train/exp8/weights/best.pt', source='local')  # yolov5n - yolov5x6 or custom
# model = torch.hub.load('./','custom', './weights/yolov5s.pt', source='local')  # yolov5n - yolov5x6 or custom
im = '../datasets/tr03/images/test.jpg'  # file, Path, PIL.Image, OpenCV, nparray, list
results = model(im)  # inference
results.print()  # or .show(), .save(), .crop(), .pandas(), etc.

model('../datasets/tr02/images/train01/01.jpg').show()

```
![upload successful](/images/pasted-196.png)