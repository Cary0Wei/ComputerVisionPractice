# 计算机视觉实践-练习1



##### 算法

本实验要使用的算法类似于 Brown 和 Lowe 在 2007 年论文[Automatic Panoramic Image Stitching with Invariant Features](https://www.doc88.com/p-9062359287397.html)中提出的方法。其中算法已被opencv实现且可以直接调用`cv2.Stitcher_create()`接口得到 。



##### 原始图片

本实验使用图片实例：

<img src=".\README.assets\1.png" alt="1" style="zoom:25%;" />

由五张图片组成，`./picture6`文件下保存，`main.ipynb`所用示例也是这五张图片



##### 直接调用

直接调用 `cv2.Stitcher_create()` 对图片处理得到：

<img src=".\README.assets\2.png" alt="2" style="zoom:25%;" />

可以见图片周围很多黑边，该输出目录为`.\output\orioutput6.jpg`

##### 改进：

黑边处理全景图拼接完成后，会出现图像边界外的**黑色像素(0)**，使全景图不完美。可采取如下方法去除黑边：**全景图轮廓提取**、**轮廓最小正矩形**、**腐蚀处理**。

```stitched = cv2.copyMakeBorder(pano, 10, 10, 10, 10, cv2.BORDER_CONSTANT, (0, 0, 0))

# 全景图轮廓提取
stitched = cv2.copyMakeBorder(pano, 10, 10, 10, 10, 		 cv2.BORDER_CONSTANT, (0, 0, 0))
gray = cv2.cvtColor(stitched, cv2.COLOR_BGR2GRAY)
thresh = cv2.threshold(gray, 0, 255, cv2.THRESH_BINARY)[1]
cnts = cv2.findContours(thresh, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)[0

```

可以得到 `.\output\modoutput6.jpg`

<img src=".\README.assets\3.png" alt="3" style="zoom:25%;" />

可以看见图片变小很多，所以改进仍然存在缺点

