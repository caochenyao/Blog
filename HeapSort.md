---
title: 堆排序
date: 2016-05-5 15:45:15
categories: 数据结构
tags: 排序
---
一、堆排序
 堆排序是通过建立大堆或小堆而实现的一种排序方法。(关于大堆与小堆请参考我的上一篇博客)

二、堆排序的思路
 这里我们以实现一组数的升序来进行说明
 ![1](http://o6lb63nu0.bkt.clouddn.com/HeapSort1.png)
 ![2](http://o6lb63nu0.bkt.clouddn.com/HeapSort2.png)

三、堆排序的代码实现
 
	//堆排序->向下调整
	void AdjustDown(int* a, size_t size, size_t root)
	{
		assert(a);
		size_t child = root * 2 + 1;
		while (child <size)
		{
			if (a[child] < a[child + 1] && (child + 1)<size)
			{
				++child;
			}
			if (a[child] > a[root])
			{
				swap(a[child], a[root]);
				root = child;
				child = root * 2 + 1;
			}
			else
			{
				break;
			}
		}
	}
	//堆排序->向上调整
	void AdjustUp(int* a, size_t size, size_t parent)
	{
		assert(a);
		size_t child = parent * 2 + 1;
		if (a[child] < a[child+1] && (child+1)<size)
		{
			++child;
		}
		if (a[child] > a[parent])
		{
			swap(a[child], a[parent]);
		}
	}
	//堆排序
	void HeapSort(int* a, size_t size)
	{
		assert(a);
		for (size_t parent = (size - 2)/2; parent > 0;--parent)
		{
			AdjustUp(a,size,parent);
		}
		for (size_t index = size; index > 0;--index)
		{
			AdjustDown(a,index,0);
			swap(a[0],a[index - 1]);
		}
	}