---
title: 堆
date: 2016-04-30 15:44:58
categories: 数据结构
tags: 二叉树
---
一、堆
 1.堆数据结构是一种数组对象，它可以被视为一棵完全二叉树结构。
 2.大堆与小堆:
   最大堆：每个父节点的都大于孩子节点。
   最小堆：每个父节点的都小于孩子节点。
 3.堆的每一节点与其两个孩子节点下标的关系:
  父节点:parent，左孩子:left，右孩子:right(parent，left，right均为下标)
  left = parent*2+1，right = parent*2+2

二、大堆和小堆的建立
 ![1](http://o6lb63nu0.bkt.clouddn.com/Heap1.png)
  
三、堆的代码实现
 我实现堆使用的是库里的顺序表，没有再自己建立数组

	#pragma once
	#include<vector>
	#include<assert.h>
	
	#pragma warning(disable:4018)
	
	//使用仿函数决定是建大堆还是小堆
	template<class T>
	struct Less
	{
		bool operator()(const T& a, const T& b)
		{
			return a < b;
		}
	};
	
	template<class T>
	struct Greater
	{
		bool operator()(const T& a, const T& b)
		{
			return a > b;
		}
	};
	
	template<class T, class Compare = Less<T>>
	class Heap
	{
	public:
		Heap()                            //无参的构造函数
			:_array(NULL)
		{}
		Heap(const T* a, size_t size)     //带参的构造函数
		{
			for (size_t i = 0; i < size; ++i)
			{
				_array.push_back(a[i]);
			}
		}
		void Sort()
		{
			//向上调整
			AdUp(_array.size() - 1);
		}
		void Push(const T& x)                //插入数据
		{
			_array.push_back(x);
			AdUp(_array.size() - 1);
		}
		T& Top()                             //返回堆顶的数据
		{
			assert(_array.size() > 0);
			return _array[0];
		}
		size_t Size()
		{
			return _array.size();
		}
		void Pop()                          //删除堆顶数据
		{
			assert(_array.size() > 0);
			swap(_array[0], _array[_array.size() - 1]);
			_array.pop_back();
			AdDown(0);                      //从堆顶向下进行调整
		}
		bool IsEmpty()                      //判断堆是否为空
		{
			return _array.empty();
		}
	protected:
		void AdUp(int child)
		{
			int parent = (child - 1) / 2;    //找到堆的第一个非叶子节点
			while (parent >= 0)              //从该节点依次向上进行调整
			{
				if (child < _array.size() - 1 && Compare()(_array[child + 1], _array[child]))
				{
					//找出两个叶子节点中小的那一个(建小堆)
                    //找出两个叶子节点中大的那一个(建大堆)
					++child;
				}
				if (Compare()(_array[child], _array[parent]))
				{
					//不符合大堆或小堆的要求，将两个节点交换
					swap(_array[child], _array[parent]);
					//交换后可能会出现之前调整过的又不符合要求，所以要再向下调整
					AdDown(child);
				}
				--parent;
				child = parent * 2 + 1;
			}
		}
		void AdDown(int root)
		{
			int child = root * 2 + 1;
			while (child < _array.size())
			{
				if (child < _array.size() - 1 && Compare()(_array[child + 1], _array[child]))
				{
					++child;
				}
				if (Compare()(_array[child], _array[root]))
				{
					swap(_array[child], _array[root]);
					root = child;
					child = root * 2 + 1;
				}
				else
				{
					//此时说明后面的都已经符合要求了，不需要再进行调整
					break;
				}
			}
		}
	protected:
		vector<T> _array;
	};
	
	void Test()
	{
		int array[10] = { 10, 16, 18, 12, 11, 13, 15, 17, 14, 15 };
		Heap<int,Greater<int>> h1(array, 10);
	
		h1.Sort();
		h1.Push(9);
		h1.Pop();
	}

在下一篇博客里我们将讨论与堆有关的一个排序——堆排序。
