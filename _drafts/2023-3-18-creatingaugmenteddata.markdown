---
layout: "post"
title:  "transfering labelme masked image to a random background"
date:   2023-3-18 10:11:37 +0700
categories: python stuff 
permalink: "/augmentedData1/"
author: "heabeoun himself"
---
# transfering labelme masked images to a random background 
![image](\assets\img\eden.jpg){: width="600"}
it's been awhile since my last post *(it wasn't even complete lol)* , but during that time i've been tinkering around with various deep learning models and frameworks, and what i've found out is that augmented data is important in order to train accurate ML models, while there are libraries made for augmentations like [albumentations](https://albumentations.ai) readily available , the type of augmentation that i want to do is not avialable anywhere (if there is a library that can do this ,please do tell).
what im trying to do is taking a labeled image from labelme and then putting it in a different background, so let's get to it!

---
```python
# import the required libraries
import cv2
import json
import numpy as np
from pathlib import pathlib
import os

#first we look for labelme json files in a directory

def lookForJson(folder):
  folder = Path(folder)
  jsons = []
  for roots, dir, files in os.walk(folder):
    print("searching in: ", folder)
    for file in files:
      if file.endswith(".json"):
        jsons.append(Path(os.path.join(folder, file))) # get the full path of the json file
  return jsons # returns a list of all the labelme json files
```
---
the code above returns a list of json from a specified input folder path ,i.e your labelme image folder.

then , we cut the labels from the actual image

---


```

```
