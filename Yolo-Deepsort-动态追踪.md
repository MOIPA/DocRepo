title: Yolo+Deepsort 动态追踪
author: tr
date: 2023-02-03 06:12:04
tags:
---
# 动态追踪算法

算法地址：https://github.com/mikel-brostrom/Yolov7_StrongSORT_OSNet。

`pip install -r requirements.txt`

## 运行

> 项目下载好后，根据github的使用文档，可以在项目目录下执行：`python track.py --source test.mp4 --strong-sort-weights osnet_x0_25_market1501.pt` 
>
> 如果执行 `python track.py --source test.mp4 ` 项目会自动去google网盘下载osnet模型，所以可以提前下载StrongSort模型[模型下载区](https://kaiyangzhou.github.io/deep-person-reid/MODEL_ZOO)，然后用`--strong-sort-weights`指定即可
>
>这里 source 可以是视频文件、摄像头 ID 或者网络视频(rtsp、http、https 都支持)


## Yolo 模型

> 这个项目使用的Yolo算法，所以在里面的Yolo目录中可以像往常一样使用yolo算法生成自己的训练模型，最后在追踪算法内指定YOLO模型即可`--yolo-weights yolov7.pt --img 640`


## StrongSort模型

模型可以到 https://kaiyangzhou.github.io/deep-person-reid/MODEL_ZOO 下载，这里的模型后缀是 pth，可以直接重命名为 pt

## 代码解析

> 打开track.py
>
> 从109行开始加载数据流，dataset为要识别的视频流
>
> 148行开始便利dataset视频流，将每一帧作为输入，im,im0s,im0都是视频的一帧，赋值给了curr_frames, prev_frames，当前帧和上一帧
>
> 162行pred = model(im)，pred为帧图片识别结果
>
> 198行将当前帧图片和上一帧图片作为入参输入到strongSort内

## 利用算法

> 算法在：track.py 中，算法执行可以选择` --show-vid` 从而开启一个识别视频流，同时后台会实时的打印识别到的物体
>
> 算法所有的入参可以在 track.py 第 287行中看到
>
> 算法在第248行打印识别结果，可以在这里将识别结果发送到kafka或者写入redis操作，从而配合其他系统使用
>
> 算法在171行开始处理图形识别

![upload successful](/images/pasted-197.png)
