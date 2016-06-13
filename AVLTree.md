---
title: 二叉平衡搜索树
date: 2016-04-22 20:26:25
categories: 数据结构
tags: 二叉树
---
一、AVL树
 AVL树又称为高度平衡的二叉搜索树，是1962年有俄罗斯的数学家G.M.Adel'son-Vel'skii和E.M.Landis提出来的。它能保持二叉树的高度平衡，尽量降低二叉树的高度，减少树的平均搜索长度。

二、AVL树的性质
 1.左子树和右子树的高度之差的绝对值不超过1
 2.树中的每个左子树和右子树都是AVL树
 3.每个节点都有一个平衡因子(balance factor--bf),任一节点的平衡因子是-1/0/1。(每个节点的 平衡因子等于右子树的高度减去左子树的高度)

三、AVL树的效率
  一棵AVL树有N个节点，其高度可以保持在log2N，插入/删除/查找的时间复杂度也是log2N。
 (ps：log2N是表示log以2为底N的对数)

四、AVL树的插入
 这里我们只讨论AVL树的插入，每插入一个节点，从该节点的父节点向上改变平衡因子，在左子树插入，平衡因子减一，在右子树插入，平衡因子加一:
 ![1](http://o6lb63nu0.bkt.clouddn.com/AVL1.png)
 当某节点的平衡因子改变后变为2或-2时，则要调整平衡因子。
 当某节点的平衡因子改变后变为2或-2时，我们可以分为四种情况来讨论平衡因子的调整
 1.情况一:改变后平衡因子情况为2，1，0时
 ![2](http://o6lb63nu0.bkt.clouddn.com/AVL2.png)
 **注意在parent的父节点变为subR之前要将subR链在parent的原父节点之后。**
 2.情况二:改变后平衡因子情况为-2，-1，0时
 ![3](http://o6lb63nu0.bkt.clouddn.com/AVL3.png)
 **注意在parent的父节点变为subL之前要将subL链在parent的原父节点之后。**
 3.情况三:改变后平衡因子情况为2，-1，0时
 ![4](http://o6lb63nu0.bkt.clouddn.com/AVL4.png)
 **注意在parent的父节点变为subRL之前要将subRL链在parent的原父节点之后。**
 旋转之后需要调整平衡因子
 ![5](http://o6lb63nu0.bkt.clouddn.com/AVL5.png)
 4.情况四:改变后平衡因子情况为-2，1，0时
 ![6](http://o6lb63nu0.bkt.clouddn.com/AVL6.png)
 **注意在parent的父节点变为subLR之前要将subLR链在parent的原父节点之后。**
 旋转之后需要调整平衡因子
 ![7](http://o6lb63nu0.bkt.clouddn.com/AVL7.png)

五、AVL树的代码实现
 我实现AVL树使用的是非递归的方法
	
    #pragma once
    //AVL树节点的实现，这里使用的是三叉链的结构
	template<class K,class V>
	struct AVLTreeNode
	{
		AVLTreeNode<K, V>* _left;     //左孩子
		AVLTreeNode<K, V>* _right;    //右孩子
		AVLTreeNode<K, V>* _parent;   //父节点
		K _key;                       //关键字
		V _value;                     //与关键字有关的信息
		int _bf;                      //平衡因子

		AVLTreeNode(const K& key,const V& value) //构造函数
			:_left(NULL)
			, _right(NULL)
			, _parent(NULL)
			, _key(key)
			, _value(value)
			, _bf(0)
		{}
	};

	template<class K,class V>
	class AVLTree
	{
		typedef AVLTreeNode<K, V> Node;
	public:
		AVLTree()                           //构造函数
			:_root(NULL)
		{}
	public:
		bool Insert(const K& key,const V& value)  //AVL树的插入
		{
			if (_root == NULL)
			{
				_root = new Node(key, value);
				return true;
			}
				
			Node* parent = NULL;
			Node* cur = _root;
			bool Isadjust = false;
			while (cur)                         //找到插入的位置
			{
				if (cur->_key < key)
				{
					parent = cur;
					cur = cur->_right;
				}
				else if (cur->_key > key)
				{
					parent = cur;
					cur = cur->_left;
				}
				else
				{
					return false;
				}	
			}

			cur = new Node(key,value);         //创建新节点
			cur->_parent = parent;             //插入新节点
			if (cur->_key < parent->_key)
			{
				parent->_left = cur;
			}
			else
			{
				parent->_right = cur;
			}

			while (parent)                    //由父节点向上修改平衡因子
			{
				
				if (cur == parent->_left)
				{
					parent->_bf--;
					if (parent->_bf == 0)
						break;
				}
				else if (cur == parent->_right)
				{
					parent->_bf++;
					if (parent->_bf == 0)
						break;
				}
                //当某一节点的平衡因子为2或-2时，调整平衡因子
				if (parent->_bf > 1 || parent->_bf < -1)
				{
					Isadjust = true;
					if (parent->_bf > 1)
					{
						if (parent->_right->_bf == 1)
						{
							//情况为2,1,0;为左单旋
							_RotateL(parent);
						}
						else if (parent->_right->_bf == -1)
						{
							//情况为2,-1,0;为右左双旋
							_RotateRL(parent);
						}
					}
					else
					{
						if (parent->_left->_bf == -1)
						{
							//情况为-2,-1,0;为右单旋
							_RotateR(parent);
						}
						else if (parent->_left->_bf == 1)
						{
							//情况为-2,1,0;为为左右双旋
							_RotateLR(parent);
						}
					}
					if (parent->_parent == NULL)
					{
						_root = parent;
						return true;
					}
				}

				Node* ppNode = parent->_parent;
				if (Isadjust && ppNode)
				{
					if (parent->_key < ppNode->_key)
					{
						ppNode->_left = parent;
					}
					else
					{
						ppNode->_right = parent;
					}
					return true;
				}
				cur = parent;
				parent = cur->_parent;
			}
			return true;
		}
	public:
		void _RotateL(Node*& parent)
		{
			Node* subR = parent->_right;    //parent为_bf为2的节点,subR为_bf为1的节点
			Node* subRL = subR->_left;      //subRL为subR的左孩子

			parent->_right = subRL;         //将subRL链为parent的右孩子
			if (subRL)
				subRL->_parent = parent;

			subR->_parent = parent->_parent;//先将parent的父节点链为subR的父节点
			parent->_parent = subR;         //再将subR作为parent的父节点
			subR->_left = parent;

			parent->_bf = subR->_bf = 0;
			parent = subR;                  //将subR赋为parent(此处用的是引用,此处变,上面的cur也会变)
			
		}
		void _RotateR(Node*& parent)
		{
			//右单旋与左单旋相似
			Node* subL = parent->_left;
			Node* subLR = subL->_right;

			parent->_left = subLR;
			if (subLR)
				subLR->_parent = parent;

			subL->_parent = parent->_parent;
			parent->_parent = subL;
			subL->_right = parent;

			parent->_bf = subL->_bf = 0;
			parent = subL;
		}
		void _RotateRL(Node*& parent)
		{
			/*_RotateR(parent->_right);
			_RotateL(parent);*/
			Node* subR = parent->_right;
			Node* subRL = subR->_left;
			//右旋
			subR->_left = subRL->_right;
			if (subRL->_right)
				subRL->_right->_parent = subR;
			parent->_right = subRL;
			subRL->_right = subR;
			//改变平衡因子
			if (subRL->_bf == 0 || subRL->_bf == 1)
			{
				subR->_bf = 0;
			}
			else
			{
				subR->_bf = 1;
			}
			//左旋
			parent->_right = subRL->_left;
			if (subRL->_left)
				subRL->_left->_parent = parent;
			subRL->_parent = parent->_parent;
			subRL->_left = parent;
			parent->_parent = subRL;
			//改变平衡因子
			if (subRL->_bf == 0 || subRL->_bf == -1)
			{
				parent->_bf = 0;
			}
			else
			{
				parent->_bf = -1;
			}

			subRL->_bf = 0;
			parent = subRL;

		}
		void _RotateLR(Node*& parent)
		{
			/*_RotateL(parent->_left);
			_RotateR(parent);*/
			Node* subL = parent->_left;
			Node* subLR = subL->_right;
			//左旋
			subL->_right = subLR->_left;
			if (subLR->_left)
				subLR->_left->_parent = subL;
			parent->_left = subLR;
			subLR->_left = subL;
			//改变平衡因子
			if (subLR->_bf == 0 || subLR->_bf == -1)
			{
				subL->_bf = 0;
			}
			else
			{
				subL->_bf = -1;
			}
			//右旋
			parent->_left = subLR->_right;
			if (subLR->_right)
				subLR->_right->_parent = parent;
			subLR->_parent = parent->_parent;
			subLR->_right = parent;
			parent->_parent = subLR;
			//改变平衡因子
			if (subLR->_bf == 0 || subLR->_bf == 1)
			{
				parent->_bf = 0;
			}
			else
			{
				parent->_bf = 1;
			}

			subLR->_bf = 0;
			parent = subLR;
		}
		int Hight()                   //求深度
		{
			return _Hight(_root);
		}
		bool IsBalance()              //判断是否平衡
		{
			return _IsBalance(_root);
		}
		void InOrder()
		{
			_InOrder(_root);
			cout << endl;
		}
	protected:
		void _InOrder(Node* root)
		{
			if (root == NULL)
				return;

			_InOrder(root->_left);
			cout << root->_key << " ";
			_InOrder(root->_right);
		}
		bool _IsBalance(Node* root)
		{
			if (root == NULL)
				return true;

			int left = _Hight(root->_left);
			int right = _Hight(root->_right);
			int poor = right-left;
			bool IsBalance = abs(poor) < 2 && root->_bf == poor;

			return _IsBalance(root->_left) && _IsBalance(root->_right) && IsBalance;
		}
		int _Hight(Node* root)
		{
			if (root == NULL)
				return 0;

			int left = _Hight(root->_left)+1;
			int right = _Hight(root->_right)+1;

			return left > right ? left : right;
		}
	protected:
		Node* _root;
	};

	void Test1()
	{
		int a[] = { 16, 3, 7, 11, 9, 26, 18, 14, 15 };

		AVLTree<int, int> t;
		for (int i = 0; i < sizeof(a) / sizeof(int);++i)
		{
			t.Insert(a[i],i);
		}

		t.InOrder();
		cout<<"IsBalance:"<<t.IsBalance()<<endl;
	}

	void Test2()
	{
		int a[] = { 4, 2, 6, 1, 3, 5, 15, 7, 16, 14 };

		AVLTree<int, int> t;
		for (int i = 0; i < sizeof(a) / sizeof(int); ++i)
		{
			t.Insert(a[i], i);
		}

		t.InOrder();
		cout << "IsBalance:" << t.IsBalance() << endl;
	}
