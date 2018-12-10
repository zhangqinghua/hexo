---
title: scipy 笔记

categories:
- Python

date: 2017-10-12
---

SciPy是一款方便、易于使用、专为科学和工程设计的Python工具包。它包括统计、优化、整合、线性代数模块、傅里叶变换、信号和图像处理、常微分方程求解器等等。

<!-- more -->

- 读写.mat文件
	```python
	data = {}  
	data['x'] = x  
	scipy.io.savemat('test.mat',data)
	data = scipy.io.loadmat('test.mat') 
	```
- 以图像形式保存数组
	```python
	rmat = np.reshape(mat, (w, h))
	# 4通道，最后一个通道表示透明度，0透明，255不透明
	amat = np.zeros((w, h, 4), dtype=np.int)
	amat[:, :, 3] = 1 * 255
	amat[:, :, 0:3] = org
	misc.imsave(name + '.png', amat)
	```