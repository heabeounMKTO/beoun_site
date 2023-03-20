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


first of all , we need to setup our folders properly!

├── projectFolder 
│   ├── imgs     //images labeled in label me and thier json files
│   │   ├── img1.jpeg
│   │   ├── img1.json
│   ├── random   // random image of your choosing to be used as background
│   │   ├── random1.jpeg
│   │   ├── random2.png
│   ├── test.py
│   ├── test //outputs folder




the code below should cut labels from the **`imgs`** and then put them in the test folder 
with a random background from **`random`** assigned.

---
`test.py`

---

```python
import os
import cv2
import json
from pathlib import Path
import numpy as np
import random


class labelCut:
    def __init__(self, folder,randomImgFolder,randomImageArray=None, jsonArray=None,imageArray=None):
        self.folder = folder
        self.jsonArray = jsonArray
        self.imageArray = imageArray
        self.randomImgFolder = randomImgFolder
        
    def lookForJson(self, folder):
        folder = Path(self.folder)
        jsons = []
        for roots, dir, files in os.walk(folder):
            for file in files:
                if file.endswith(".json"):
                    jsons.append((os.path.join(folder, file)))
        self.jsonArray = jsons
        return self.jsonArray

    def readImageFromLabelme(self):
        imagefiles = []
        
        jsons = self.lookForJson(self.folder)
        for jsonfile in jsons:
            imagePath = ""
            imageName = ""
            extension = os.path.splitext(Path(json.load(open(jsonfile))['imagePath']))[1]       
            localDir = Path(json.load(open(jsonfile))['imagePath']).stem.split('.')[0]
            if localDir != "":
                imagePath = Path(json.load(open(jsonfile))['imagePath']).stem.split('.')[0].split('\\')[1] 
            else:
                continue        
            frontpath = (os.path.join(os.getcwd(),self.folder))
            fullpath = (frontpath + "/" +imagePath + extension)
            imagefiles.append(fullpath)
        self.imageArray = imagefiles
        return imagefiles
     
    def getRandomBackground(self):
        randomimg = []
        imgfmts = (".jpg", ".jpeg", ".png")
        for root, dirs, files in os.walk(self.randomImgFolder):
            for file in files:
                if file.endswith(imgfmts):
                    randomimg.append(os.path.join(os.getcwd(), (self.randomImgFolder + "/" + file)))
        return randomimg


    def randomBgFromLabelme(self):
        self.readImageFromLabelme() #load json
        randomimg = self.getRandomBackground()
        for index, label in enumerate(self.jsonArray): 
            loadjson = json.load(open(label))
            try:
                pts = loadjson["shapes"][index]["points"]
                mask = np.array(pts, np.int32)
                img = cv2.imread(self.imageArray[index])
                blackbg = np.zeros((img.shape[0], img.shape[1], 3))
                gg = cv2.fillPoly(blackbg, pts = [mask], color=(255,255,255))
                gg = gg.astype(np.uint8)
                
                result = cv2.cvtColor(cv2.bitwise_and(img, gg), 1)
                _,alpha = cv2.threshold(result, 0, 255, cv2.THRESH_BINARY)
                alpha = cv2.cvtColor(alpha, cv2.COLOR_BGR2GRAY)
                b,g,r = cv2.split(img)
                rgba = [b,g,r,alpha]
                dst = cv2.merge(rgba,4)
                
                loadrandimg = cv2.imread(randomimg[random.randint(0, len(randomimg))])
                loadrandimg = cv2.resize(loadrandimg, (img.shape[1], img.shape[0]))
                
                alpha = cv2.merge([alpha,alpha,alpha])
                front = dst[:,:,0:3]
                result = np.where(alpha==(0,0,0), loadrandimg, front)
                    
                cv2.imwrite(f"test/t{index}.png", result)
            except:
                continue

test = labelCut(folder="imgs", randomImgFolder="random")
test.randomBgFromLabelme()
```
---
 
**that's it for now, until next time!**
