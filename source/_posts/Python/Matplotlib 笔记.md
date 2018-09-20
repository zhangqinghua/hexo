---
title: Matplotlib 笔记

categories:
- Python

date: 2017-09-12
---

Matplotlib 可能是 Python 2D-绘图领域使用最广泛的套件。它能让使用者很轻松地将数据图形化，并且提供多样化的输出格式。

<!-- more -->

- cv2库和matplotlib库色彩空间排布不一致
	cv2中的色彩排列是(b,g,r)，而matplotlib库中的排列方式是(r,g,b)。如果使用matplotlib中的pyplot.imshow()，颜色则反过来了，即r空间和b空间的色彩对调了。如果将cv2.imread()读取的图像的r通道和b通道的值对换一下，就正确了。
	```python
	# shape [200, 150, 3]
	imat = cv2.imread('data/images_data/00001.jpg')
	img_rgb = np.zeros(imat.shape, imat.dtype)
	img_rgb[:, :, 0] = imat[:, :, 2]
	img_rgb[:, :, 1] = imat[:, :, 1]
	img_rgb[:, :, 2] = imat[:, :, 0]

	cv2.imshow('image', imat)
	plt.subplot('121').imshow(imat)
	plt.subplot('122').imshow(img_rgb)

	plt.show()
	cv2.waitKey(0)
	```
	![](01.png)

	

- 显示黑白照片
	```python
	plt.imshow(amat, cmap='Greys_r')
	plt.show()
	```
	![](02.png)

- 清除缓存
	```python
	clf() # 清图。
	cla() # 清坐标轴。
	close() # 关窗口
	```

- 不显示坐标轴
	```python
	plt.axis('off')  # 不显示坐标轴
	```