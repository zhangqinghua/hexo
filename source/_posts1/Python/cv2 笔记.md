---
title: cv2 笔记

categories:
- Python

date: 2017-09-23
---

- 读入图片
	```python
	# 读入图片(图片文件名，如何读取图片)
	# cv2.IMREAD_COLOR：读入一副彩色图片
	# cv2.IMREAD_GRAYSCALE：以灰度模式读入图片
	# cv2.IMREAD_UNCHANGED：读入一幅图片，并包括其alpha通道。
	img=cv2.imread('1.jpg',cv2.IMREAD_COLOR)# 读入彩色图片
	```

<!-- more -->

- 缩放图片
	```python
	# 缩放图片(源，目标(宽，高)，变换方法)
	# CV_INTER_NN - 最近邻插值,  
	# CV_INTER_LINEAR - 双线性插值 (缺省使用)  
	# CV_INTER_AREA - 使用象素关系重采样。当图像缩小时候，该方法可以避免波纹出现。当图像放大时，类似于 CV_INTER_NN 方法..  
	# CV_INTER_CUBIC - 立方插值.  
	res = cv2.resize(image, (32, 32), interpolation=cv2.INTER_CUBIC)
	```

- 旋转图片
	```python
	# rotate(): rotate image
	# return: rotated image object
	def rotate(img, angle=30):
	    height = img.shape[0]
	    width = img.shape[1]

	    if angle % 180 == 0:
	        scale = 1
	    elif angle % 90 == 0:
	        scale = float(max(height, width)) / min(height, width)
	    else:
	        scale = math.sqrt(pow(height, 2) + pow(width, 2)) / min(height, width)

	        # print 'scale %f\n' %scale

	    rotateMat = cv2.getRotationMatrix2D((width / 2, height / 2), angle, scale)
	    rotateImg = cv2.warpAffine(img, rotateMat, (width, height))
	    # cv2.imshow('rotateImg',rotateImg)
	    # cv2.waitKey(0)

	    return rotateImg  # rotated image
	```


- 显示图片
	有时显示不了
	```python
	cv2.imshow('images', image)
	cv2.waitKey(0) # 无限期等待输入
	```


- 保存图片
	```python
	cv2.imwrite('test.png',img)
	```

