---
title: "Python-PIL库的使用"
description: 读书时的碎碎念
date: 2017-12-01T21:14:23+08:00
tags:
    - Python
Categories:
    - school_notes
---

只记录下我用到的，

```
from PIL import Image,ImageDraw,ImageChops
```

1. image = Image.open(“test.jpg”)

   > 打开图片.

2. image = image.convert(“L”)

   > 转成灰度图片，和黑白图片有区别

   ![img](http://p6jpvwsnk.bkt.clouddn.com/18-5-21/27060551.jpg)

   左边是convert的结果，右边是黑白（黑0/白255）图片。

3. image.save(“result.png”)

   > 存储成图片文件

4. image = image.point( array[] , ‘1’ )

   这个array[] 实际上要看你的图片是什么类型的。如果不是PNG最好先转成PNG，或者使用convert(“L”) 把 image 转成灰度图片。

   因此，这里的array[] 其实是一个长度为256的数组，也就是代表了0-255灰度值。

   这个函数的作用就是图片根据这个数组里每一位灰度值，比如array[27] 为0，那么图片的像素里所有灰度值等于array[27]的像素值都变成0（即变黑），否则就变成1。和二值（0/255）黑白图片很像，但实际上是（0/1）二值图片。

   `example:

   table = []

   for i in range(256):

   ```
   if i &lt; 180: # 所有灰度值小于180的都置0
   
       table.append(0) 
   
   else:
       table.append(1)
   ```

   image = image.point(table,’1’)`

   这个例子的结果就是将一张图片里所有灰度值小于180的都变成了黑色（0），其它都变成了像素值为1的颜色。即成了一张灰度图片。

5. image.size

   > 返回image图像的大小，比如 122 X 54 ，返回格式是 [122,54] 一个list。

6. image.getpixel( (x,y) )

   > 返回image图像坐标(x,y)的灰度值。

7. image = ImageChops.invert( image )

   > 运行结果是将黑白灰度值反转了，结果是 黑白变成了原来的白黑。

8. image = image.crop( part )

   > 此处的part是一个含四个元素的tuple元组。

   part = （ 左上角点的x坐标，左上角点的y坐标，右下角点的x坐标，右下角点的y坐标 ）

   返回的是一个原图经过切割，变成part设定的坐标 大小的图片。

9. image.show()

   > 显示图片

10. image = ImageChops.difference( imbw1 , imbw2 )

    > 返回一张image图片，imbw1和imbw2也是图片。

    > 此函数作用就是将有相同灰度值的像素点 的 灰度值 变为 0（黑） ， 不同灰度值的像素点的 灰度值 变为 255（白）。形成一张image图片返回。