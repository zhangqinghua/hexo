---
title: python 笔记

categories:
- Python

date: 2017-09-02
---
Python是一种计算机程序设计语言。你可能已经听说过很多种流行的编程语言，比如非常难学的C语言，非常流行的Java语言，适合初学者的Basic语言，适合网页编程的JavaScript语言等等。

<!-- -->

- 随机数
	```python
	import random
	# 生成 0 ~ 9 之间的随机数
	print(random.randint(0,9)) # 2
	```

- 数组
	```python
	# 追加
	[1, 2, 3] + [4, 5, 6] # [1, 2, 3, 4, 5, 6]
	[1, 2, 3].append(2) # [1, 2, 3, 2]
	# 重复
	['Hi!'] * 4 # 	['Hi!', 'Hi!', 'Hi!', 'Hi!']
	# 元素是否存在于列表中
	3 in [1, 2, 3] # True
	```

- 打包
	```python
	x = [1, 2, 3]
	y = [2, 4, 6]
	z = zip(x, y)
	for i in z:
	    print(i) 
	# (1, 2) 
	# (2, 4) 
	# (3, 6)
	```


- 查看数据类型
	```python
	id = 1L
	type(id) # <type 'long'>
	```


- 读取文件
	```python
	file = open('alldata_urls.txt').read()
	for i in file.split('\n'):
	    print(i.split(' ')[0])
	```


- 下载保存文件
	```pythno
	import urllib.request
	open('test.png', 'wb').write(urllib.request.urlopen(url).read())
	```
