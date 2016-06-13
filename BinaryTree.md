---
title: 二叉树
date: 2016-04-11 18:56:35
categories: 数据结构
tags: 二叉树
---
一、一个树包含以下几个部分
  结点：结点包含数据和指向其它结点的指针。
  根节点：树第一个结点称为根节点。
  结点的度：结点拥有的子节点个数。
  叶节点：没有子节点的节点(度为0)。
  父子节点：一个节点father指向另一个节点child，则child为孩子节点，father为父亲结点。
  兄弟节点：具有相同父节点的节点互为兄弟节点。
  节点的祖先：从根节点开始到该节点所经的所有节点都可以称为该节点的祖先。
  子孙：以某节点为根的子树中任一节点都称为该节点的子孙。
  树的高度：树中距离根节点最远结点的路径长度。
  如图1:
  ![1](http://o6lb63nu0.bkt.clouddn.com/binarytree1.png)
  *但树不能有连通的情况
  例如:
  ![2](http://o6lb63nu0.bkt.clouddn.com/binarytree2.png)
  如上图2所示的就不是一个树

  而一个树的存储结构是如图3:
  ![3](http://o6lb63nu0.bkt.clouddn.com/binarytree3.png)

二、二叉树
  二叉树:二叉树是一棵特殊的树，二叉树每个节点最多有两个孩子结点，分别称为左孩子和右孩子。
  满二叉树:高度为N的满二叉树有2^N - 1个节点的二叉树。
  完全二叉树: 若设二叉树的深度为h，除第 h 层外，其它各层 (1～h-1) 的结点数都达到最大个
  数，第 h 层所有的结点都连续集中在最左边，这就是完全二叉树。
  如图4、图5:
  ![4](http://o6lb63nu0.bkt.clouddn.com/binarytree4.png)
  ![5](http://o6lb63nu0.bkt.clouddn.com/binarytree5.png)

三、二叉树的遍历方法
  ![6](http://o6lb63nu0.bkt.clouddn.com/binarytree6.png)
  前序遍历(先根遍历):(1)：先访问根节点；(2)：前序访问左子树；(3)：前序访问右子树。
 【1 2 3 4 5 6】
  中序遍历:(1)：中序访问左子树；(2)：访问根节点；(3)：中序访问右子树。
 【3 2 4 1 6 5】
  后序遍历(后根遍历):(1)：后序访问左子树；(2)：后序访问右子树；(3)：访问根节点。
 【3 4 2 6 5 1】
  层序遍历：一层层节点依次遍历。
 【1 2 5 3 4 6】

四、二叉树的代码实现
  只是进行概念的讲解的话难免会有些枯燥，下面是我实现的代码，中间也有注释进行说明。

	#pragma once
	#include<assert.h>
	#include<queue>

	template<class T>
	struct BinaryTreeNode                   //二叉树结构
	{
		T _data;
		BinaryTreeNode<T>* _left;
		BinaryTreeNode<T>* _right;
		BinaryTreeNode(const T& x)         //节点的构造函数
			:_data(x)
			, _left(NULL)
			, _right(NULL)
		{
		}
	};

	template<class T>
	class BinaryTree
	{
	public:
		BinaryTree()                       //无参的构造函数
			:_root(NULL)
		{
		}
		BinaryTree(const T* a,size_t size) //有参的构造函数
		{
			size_t index = 0;
			_root = CreateTree(a,size,index);
		}
		void PrevOrder()                  //先序遍历
		{
			_PrevOrder(_root);
			cout << endl;
		}
		void InOrder()                    //中序遍历
		{
			_InOrder(_root);
			cout << endl;
		}
		void PostOrder()                  //后序遍历
		{
			_PostOrder(_root);
			cout << endl;
		}
		void LevelOrder()                 //层次遍历(通过队列来实现)
		{
			queue<BinaryTreeNode<T>*> q;
			if (_root == NULL)
			{
				return;
			}
			q.push(_root);
			while (!q.empty())
			{
				BinaryTreeNode<T>* root = q.front();
				q.pop();
				if (root->_left)          //如果出队的节点的左孩子或右孩子不为空,其不为空的孩子入队列 
				{
					q.push(root->_left);
				}
				if (root->_right)
				{
					q.push(root->_right);
				}
				cout << root->_data << " ";//打印出队的节点->相当于遍历出队的节点
			}
			cout << endl;
		}
		size_t Size()
		{ 
			return _Size(_root);
		}
		size_t Depth()                    //计算二叉树的深度
		{
			return _Depth(_root);
		}
		BinaryTreeNode<T>* Find(const T& x)//用层次遍历寻找符合条件的节点
		{
			queue<BinaryTreeNode<T>*> q;
			if (_root == NULL)
			{
				return NULL;
			}
			q.push(_root);
			while (!q.empty())
			{
				BinaryTreeNode<T>* root = q.front();
				q.pop();
				if (root->_left)
				{
					q.push(root->_left);
				}
				if (root->_right)
				{
					q.push(root->_right);
				}
				if (root->_data == x)
				{
					return root;
				}
			}
			return NULL;
			cout << endl;
		}
	protected:
		size_t _Depth(BinaryTreeNode<T>* root)
		{
			if (root == NULL)
			{
				return 0;
			}
			size_t leftdepth = 1;
			size_t rightdepth = 1;
			leftdepth += _Depth(root->_left);
			rightdepth += _Depth(root->_right);
			return (leftdepth > rightdepth) ? leftdepth : rightdepth;
		}
		size_t _Size(BinaryTreeNode<T>* root)
		{
			if (root == NULL)
				return 0;
			return (_Size(root->_left) + _Size(root->_right) + 1);
		}
		void _PostOrder(BinaryTreeNode<T>* root)
		{
			BinaryTreeNode<T>* curroot = root;
			if (curroot == NULL)
			{
				return;
			}
			_PostOrder(curroot->_left);	
			_PostOrder(curroot->_right);
			cout << curroot->_data << " ";
		}
		void _InOrder(BinaryTreeNode<T>* root)
		{
			BinaryTreeNode<T>* curroot = root;
			if (curroot == NULL)
			{
				return;
			}
			_InOrder(curroot->_left);
			cout << curroot->_data << " ";
			_InOrder(curroot->_right);
		}
		void _PrevOrder(BinaryTreeNode<T>* root)
		{
			BinaryTreeNode<T>* curroot = root;
			if (curroot == NULL)
			{
				return;
			}
			cout << curroot->_data << " ";
			_PrevOrder(curroot->_left);
			_PrevOrder(curroot->_right);
		}
		//建立二叉树(index为数组下标)
		BinaryTreeNode<T>* CreateTree(const T*a,size_t size,size_t& index)
		{
			assert(a);
			BinaryTreeNode<T>* newroot = NULL;
			if (index < size && a[index] != '#')
			{
				newroot = new BinaryTreeNode<T>(a[index]);
				newroot->_left = CreateTree(a, size, ++index);
				newroot->_right = CreateTree(a, size, ++index);
			}
			return newroot;
		}
	protected:
		BinaryTreeNode<T>* _root;
	};

	void Test()
	{
		int a[10] = {1,2,3,'#','#',4,'#','#',5,6};
		BinaryTree<int> t1(a,10);
		t1.PrevOrder();
		t1.InOrder();
		t1.PostOrder();
		t1.LevelOrder();
		cout << t1.Size() << endl;
		cout << t1.Depth() << endl;
		cout << t1.Find(3)->_data << endl;
	}
