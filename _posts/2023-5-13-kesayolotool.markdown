---
layout: "post"
title:  "kesa -  a tool for converting and augmenting labelme data for YOLO training"
date:   2023-5-13 23:11:37 +0700
categories: python stuff 
permalink: "/kesa1/"
author: "heabeoun himself"
---
# kesa -  a tool for converting and augmenting labelme data for YOLO training

![image](\assets\img\house1.jpg){: width="600"}
for the past few months , i've been doing a few project with the YOLO framework, in my free time. Now, there is a good platform for labeling, and generating datasets called [roboflow](https://app.roboflow.com), but due to the amount of data that i've accumulated, it supasses the amount allowed on the free tier of roboflow , which is 50000 generations.
So i've decided to write my own conversion and augmentation tool called kesa.
Currently kesa has four commands.
- convert2yolo:
convert labelme annotations to YOLO 
- convert2yoloaug:
convert labelme annotations to YOLO format and then create augmentations of each image 
- autolabel:
automatically labels images in specified folder in labelme JSON format
- end2end:
auto-labels and then converts to YOLO format in one go

### you can currently checkout this project [here](https://github.com/heabeounMKTO/kesa)
# future plans: 
of course there will probably be some changes, but i've planned the following:
- web interface for looking at the dataset's health and the model's performance
- web interface labeling and generating datasets for training, as currently it is only cli only
- web interface for training and CI/CD of models

..that's all for now , until next time! 
