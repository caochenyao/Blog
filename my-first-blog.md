---
title: 哈希表与哈希冲突
date: 2016-04-7 15:58:15
categories: 数据结构
tags: 哈希表
---
一、什么是哈希表？
 哈希表是根据关键字（key）而直接访问在内存存储位置的数据结构。它通过一个关键值的函数将所需的数据映射到表中的位置来访问数据，这个映射函数叫做散列函数，存放记录的数组叫做散列表。

二、什么是哈希冲突？
 不同的Key值经过哈希函数Hash(Key)处理以后可能产生相同的值——哈希地址，我们称这种情况为哈希冲突。任意的散列函数都不能避免产生冲突。 
**何为荷载因子:荷载因子 = 填入表中的元素个数/散列表的长度
 因为散列表的长度为定值，所以当荷载因子越大时，表中元素就越多则产生冲突的可能性越大，所以荷载因子一般被限定在0.7-0.8。

三、解决哈希冲突的办法：
1、闭散列法
(1)线性探测
描述：线性探测是解决哈希冲突的一种方法，key值经过哈希函数处理就会得到对应的哈希地址，当这个地址已经存放了其他的key值时，向后寻找该地址的下一个地址，如果仍有其他的key值那么接着向后找，直到找到空闲的位置为止。
![1](http://o6lb63nu0.bkt.clouddn.com/Hash1.png)
线性探测的代码实现

	#pragma once
	#include<string>

	enum Status        //使用状态机来表示散列表每个位置的状态
	{                 
		EXITS,         //存在
		DELETED,       //删除
		EMPTY,         //空
	};

	template<class T>                      //使用仿函数获得不同类型关键字的哈希值
	struct HashFuner
	{
		size_t operator()(const T& key)
		{
			return key;
		}
	};

	static size_t BKDRHash(const char* str)//字符串的哈希算法
	{
		size_t seed = 131;
		size_t hash = 0;
		while (*str)
		{
			hash = hash * seed + (*str++);
		}
		return (hash & 0x7FFFFFFF);
	}

	template<>
	struct HashFuner<string>               //获得string类型的哈希值
	{
		size_t operator()(const string& key)
		{
			return BKDRHash(key.c_str());    
		}	
	};


	template<class T,class HashFun = HashFuner<T>>
	class HashTable
	{
	public:
		HashTable()                        //无参的构造函数
			:_array(NULL)
			, _status(0)
			, _size(0)
			, _capacity(0)
		{}
		HashTable(const size_t size)       //有参的构造函数
			:_array(new T[size])
			, _status(new Status[size])
			, _size(0)
			, _capacity(size)
		{
			for (size_t i = 0; i < size;++i)
			{
				_status[i] = EMPTY;       //初始状态全部置为空
			}
		}
		~HashTable()                      //析构函数
		{
			if (_array)
			{
				delete[] _array;
				delete[] _status;
			}
		}
	public:
		bool Insert(const T& key)
		{
			if (_size*10/_capacity == 8)     //荷载因子一般控制在0.7到0.8之间，当荷载因子达到0.8时需要增容
			{
				HashTable<T,HashFun> newHT(_capacity * 2); //开辟一个更长长度的新空间
				for (size_t i = 0; i < _capacity;++i)
				{
					if (_status[i] == EXITS)
					{
						newHT.Insert(_array[i]);           //将原有空间里的所有数据插入到新的空间
					}
				}
				Swap(newHT);                               //与原有空间交换
			}
			size_t index = Location(key);                  //获取关键字的位置
			while (_status[index] == EXITS)                //如果该位置已有数据则向后寻找
			{
				++index;
				if (index == _capacity)                    //若已经走到表的尾部，则回到表头
				{
					index = 0;
				}	
			}
			_array[index] = key;                           //插入数据
			_status[index] = EXITS;                        //状态置为存在状态
			++_size;                                       //表中数据个数加一
			return true;
		}
		size_t Location(const T& key)                      //得到关键字的位置
		{
			HashFun hash;                                  //用仿函数获取不同类型关键字的哈希值
			return hash(key)%_capacity;                    //计算得到关键字的位置
		}
		bool Remove(const T& key)
		{
			size_t index = Location(key);                  //找到关键字的起始位置
			while (_status[index] != EMPTY)                //如果位置不为空则寻找
			{
				if (_array[index] == key && _status[index] == EXITS)
                                                           //若该位置的关键字与要找的关键字相同并且状态为存在，说明已找到，
                                                           //状态置为删除状态，并且数据个数减一(若状态为删除状态则表示已经删除)
				{
					_status[index] = DELETED;
					_size--;
				}
				++index;                                   //否则的话向后继续寻找
				if (index == _capacity)
				{
					index = 0;
				}
			}
			return false;
		}
		size_t Find(const T& key)
		{
			size_t index = Location(key);                  //获得关键字的初始位置
			while (_status[index] != EMPTY)                //如果位置不为空则寻找
			{
				if (_array[index] == key && _status[index] == EXITS)
                                                           //如果该关键字与要找的关键字相同且状态为存在，则返回位置
                                                           //(若状态为删除状态则表示已经删除)
				{
					return index;
				}
				++index;                                   //否则的话继续向后寻找
				if (index == _capacity)
				{
					index = 0;
				}
			}
			return -1;
		}
		void Print()
		{
			for (size_t i = 0; i < _capacity;++i)
			{
				cout << "[" << i << "]" << ":";
				cout << _array[i] << ":";
				cout << _status[i] << endl;
			}
			cout << "----------------------------------"<<endl;
		}
		void PrintStr()
		{
			for (size_t i = 0; i < _capacity; ++i)
			{
				cout << "[" << i << "]" << ":";
				cout << _array[i].c_str() << ":";
				cout << _status[i] << endl;
			}
			cout << "----------------------------------" << endl;
		}
	protected:
		void Swap(HashTable<T,HashFun>& ht)                //交换函数，用于扩容后两个空间的交换
		{
			swap(ht._array,_array);
			swap(ht._status,_status);
			swap(ht._size,_size);
			swap(ht._capacity,_capacity);
		}
	protected:
		T* _array;              //散列表
		Status* _status;        //状态机
		size_t _size;           //数据个数
		size_t _capacity;       //删列表长度
	};

	void TestLiner()
	{
		HashTable<int> h1(10);
		h1.Insert(89);
		h1.Insert(19);
		h1.Insert(49);
		h1.Insert(18);
		h1.Insert(9);
		h1.Insert(89);
		h1.Insert(19);
		h1.Insert(49);
		h1.Insert(18); 
		h1.Insert(9);

		h1.Print();

		h1.Remove(9);
		h1.Remove(29);

		h1.Print();

		size_t pos = h1.Find(18);
		cout << "Find():" << pos << endl;

		HashTable<string> h2(5);
		h2.Insert("abcd");
		h2.Insert("dbca");

		h2.PrintStr();
	}
不难看出，线性探测解决哈希冲突的效率并不理想，尤其是当数据比较多的时候由此我们就可以接着来进行讨论线性探测的优化算法——二次探测。
(2)二次探测
描述：二次探测是线性探测的优化算法，在线性探测的基础上，查找方式略有不同线性探测在遇到对应的哈希地址有其他key值的情况下，查找的下一个地址用函数Hash(key)+n^2(n>=0)进行查找，这样使得查找起来比线性探测更加的高效，但这样仍然不是很优化，所以接下来我们推出开链法，也就是俗称的哈希桶。
![2](http://o6lb63nu0.bkt.clouddn.com/Hash2.png)
二次探测的代码实现
    //二次探测的实现与线性探测大致相同，只有一些细微的差别
	#pragma once
    #include<string>

    enum Status
    {
	    EXITS,
	    DELETED,
	    EMPTY,
    };

    template<class K>
    struct HashFuner
    {
	    size_t operator()(const K& key)
	    {
		    return key;
	    }
    };

    static size_t BKDRHash(const char* str)
    {
	    size_t seed = 131;
	    size_t hash = 0;
	    while (*str)
	    {
		    hash = hash * seed + (*str++);
	    }
	    return (hash & 0x7FFFFFFF);
    }

    template<>
    struct HashFuner<string>
    {
	    size_t operator()(const string& key)
	    {
		    return BKDRHash(key.c_str());
	    }
    };

    template<class K, class V>
    struct KeyValueNode               //定义结构体包含关键字和与关键字有关的信息
    {                                 //例如学生的学号(关键字)与成绩(与关键字有关的信息)
	    K key;
	    V value;
	    KeyValueNode()
	    {}
	    KeyValueNode(const K& key, const V& value)
		   :key(key)
		   ,value(value)
	    {}
    };

	template<class K, class V, class HashFun = HashFuner<K>>
	class HashTable
	{
		typedef KeyValueNode<K, V> KVNode;
	public:
		HashTable()
			:_array(NULL)
			, _status(0)
			, _size(0)
			, _capacity(0)
		{}
		HashTable(const size_t size)
			:_array(new KVNode[size])
			, _status(new Status[size])
			, _size(0)
			, _capacity(size)
		{
			for (size_t i = 0; i < size; ++i)
			{
				_status[i] = EMPTY;
			}
		}
		~HashTable()
		{
			if (_array)
			{
				delete[] _array;
				delete[] _status;
			}
		}
		HashTable(const HashTable<K,V>& ht)
		{
			HashTable<K, V> newHT(ht._capacity);
			for (size_t i = 0; i < ht._capacity; ++i)
			{
				if (ht._status[i] == EXITS)
				{
					newHT.Insert(ht._array[i].key, ht._array[i].value);
				}
			}
			Swap(newHT);
		}
		HashTable<K, V>& operator=(const HashTable<K, V>& ht)
		{
			HashTable<K, V> newTable(ht);
			Swap(newTable);
			return *this;
		}
	public:
		bool Insert(const K& key,const V& value)
		{
			if (_size * 10 / _capacity == 8)
			{
				HashTable<K, V> newHT(_capacity * 2);
				for (size_t i = 0; i < _capacity; ++i)
				{
					if (_status[i] == EXITS)
					{
						newHT.Insert(_array[i].key, _array[i].value);
					}
				}
				Swap(newHT);
			}
			size_t index = Location0(key);
			size_t i = 1;
			while (_status[index] == EXITS)
			{
				index = Location2(index,i++);
				if (index == _capacity)
				{
					index = 0;
				}
			}
			_array[index].key = key;
			_array[index].value = value;
			_status[index] = EXITS;
			++_size;
			return true;
		}
		size_t Location0(const K& key)              //求取的是关键字的初始地址
		{
			HashFun hash;
			return hash(key) % _capacity;
		}
		size_t Location2(size_t prev,size_t i)      //查找下一个地址用函数Hash(key)+n^2(n>=0)进行查找
		{                                           //通过化简得到前一个位置的哈希值与后一个位置的哈希值的关系
			return (prev + 2 * i - 1) % _capacity;  //后一位置的哈希值为前一个位置哈希值的二倍减一
		}
		bool Remove(const K& key,size_t n)
		{
			size_t index = Location0(key);
			size_t i = 1;
			while (_status[index] != EMPTY)
			{
				if (_array[index].key == key && _status[index] == EXITS)
				{
					_status[index] = DELETED;
					_size--;
				}
				index = Location2(index, i++);
				if (index == _capacity)
				{
					index = 0;
				}
			}
			return false;
		}
		size_t Find(const K& key, size_t n)
		{
			size_t index = Location(key);
			size_t i = 1;
			while (_status[index] != EMPTY)
			{
				if (_array[index].key == key && _status[index] == EXITS)
				{
					return index;
				}
				index = Location2(index, i++);
				if (index == _capacity)
				{
					index = 0;
				}
			}
			return -1;
		}
		void Print()
		{
			for (size_t i = 0; i < _capacity; ++i)
			{
				cout << "[" << i << "]" << ":";
				cout << _array[i].key << "-";
				cout << _array[i].value << ":";
				cout << _status[i] << endl;
			}
			cout << "----------------------------------" << endl;
		}
		void PrintStr()
		{
			for (size_t i = 0; i < _capacity; ++i)
			{
				cout << "[" << i << "]" << ":";
				cout << _array[i].key.c_str() << "-";
				cout << _array[i].value.c_str() << ":";
				cout << _status[i] << endl;
			}
			cout << "----------------------------------" << endl;
		}
	protected:
		void Swap(HashTable<K,V>& ht)
		{
			swap(ht._array, _array);
			swap(ht._status, _status);
			swap(ht._size, _size);
			swap(ht._capacity, _capacity);
		}
	protected:
		KVNode* _array;
		Status* _status;
		size_t _size;
		size_t _capacity;
	};

	void TestTwice()
	{
		HashTable<int, int> ht(5);
		ht.Insert(12,14);
		ht.Insert(11,50);
		ht.Insert(24,22);
		ht.Insert(1,18);
		ht.Insert(4,55);
		ht.Print();

		HashTable<int, int> ht1(ht);
		ht1.Print();

		HashTable<int, int> ht2;
		ht2 = ht;
		ht2.Print();

		HashTable<string, string> htstr(10);
		htstr.Insert("字典", "dictionary");
		htstr.Insert("清除", "clear,destroy");
		htstr.Insert("手机", "mobilephone");
		htstr.Insert("电脑", "pc");
		htstr.Insert("吸血鬼", "vampire");
		htstr.PrintStr();

		HashTable<string, string> htstr1(htstr);
		htstr1.PrintStr();
	}
	
2、开链法(哈希桶)
描述：开链法使用的是单链表与顺序表相结合的算法，顺序表的每一个位置链接的的是相应key值对应的单链表的头结点，那么当某个key值找到对应的哈希地址时，将该key值插入到该链表中(我实现时用的是头插)，这样建立起来的散列表无论是存储key值还是查找key值，相比起闭散列法中的两种方法都比较高效。
![3](http://o6lb63nu0.bkt.clouddn.com/Hash3.png)
开链法的代码实现

    开链法与闭散列法的区别在于散列表的每一个位置存放的不是数据，而是每个位置都链有一个单链表，而当出现
    哈希冲突时不用再先后寻找空位置，而是直接将该数据头插到对应位置的单链表中。
    #pragma once
	#include<vector>
	#include<string>

	template<class K>
	struct HashFuner
	{
		size_t operator()(const K& key)
		{
			return key;
		}
	};

	static size_t BKDRHash(const char* str)
	{
		size_t seed = 131;
		size_t hash = 0;
		while (*str)
		{
			hash = hash * seed + (*str++);
		}
		return (hash & 0x7FFFFFFF);
	}

	template<>
	struct HashFuner<string>
	{
		size_t operator()(const string& key)
		{
			return BKDRHash(key.c_str());
		}
	};

	template<class K,class V>
	struct HashTableNode
	{
		K _key;
		V _value;
		HashTableNode<K,V>* _next;
		HashTableNode(const K& key, const V& value)
			:_key(key)
			, _value(value)
			, _next(NULL)
		{}
	};

	template<class K, class V, class HashFun = HashFuner<K>>
	class HashTable
	{
	public:
		typedef HashTableNode<K, V> KVNode;
	public:
		HashTable()
			:_size(0)
		{}
		HashTable(size_t size)
		{
			_tables.resize(size);
		}
		~HashTable()
		{
			Destroy();
		}
		HashTable(const HashTable<K, V>& ht)
		{
			HashTable<K, V> newTable(ht._tables.size());	 
			for (size_t i = 0; i < ht._tables.size();++i)
			{
				//KVNode* & head = newTable._tables[i];
				KVNode* cur = ht._tables[i];
				if (cur == NULL)
				{
					continue;
				}
				while (cur)
				{
					/*KVNode* tmp = cur;
					cur = cur->_next;
					tmp->_next = head;
					head = tmp;*/
					newTable.Insert(cur->_key,cur->_value);
					cur = cur->_next;
				}
				
			}
			Swap(newTable);
		}
		HashTable<K, V>& operator=(const HashTable<K, V>& ht)
		{
			if (&ht != this)
			{
				HashTable<K, V> newTable(ht);
				Swap(newTable);
			}
			return *this;
		}
	public:
		bool Insert(const K& key,const V& value)
		{
			//增容
			_CreateCapacity();
			//插入数据
			size_t index = Location(key);
			KVNode* cur = new KVNode(key,value);
			if (_tables[index] == NULL)
			{
				cur->_next = _tables[index];
				_tables[index] = cur;
				++_size;
			}
			else
			{
				KVNode* tmp = _tables[index];
				while (tmp)
				{
					if (tmp->_key == key)
						return false;
					tmp = tmp->_next;
				}
				cur->_next = _tables[index];
				_tables[index] = cur;
			}
			return true;
		}
		size_t Location(const K& key)
		{
			HashFuner<K> hf;
			return hf(key) % _tables.size();
		}
		void Print()
		{
			for (size_t i = 0; i < _tables.size();++i)
			{
				printf("Table[%d]->", i);
				KVNode* cur = _tables[i];
				while (cur)
				{
					cout << "[" << cur->_key << ":" << cur->_value << "]->";
					cur = cur->_next;
				}
				cout << "NULL" << endl;
			}
			cout << "-------------------------------------" << endl;
		}
		bool Remove(const K& key)
		{
			size_t index = Location(key);
			KVNode* cur = _tables[index];
			if (cur == NULL)
			{
				return true;
			}
			if (cur->_next == NULL)
			{
				delete cur;
				_tables[index] = NULL;
				return true;
			}
			while (cur)
			{
				KVNode* prev = cur;
				KVNode* tmp = cur->_next;
				if (tmp->_key == key)
				{
					prev->_next = tmp->_next;
					delete tmp;
					return true;
				}
				cur = cur->_next;
			}
			return false;
		}
		KVNode* Find(const K& key)
		{
			size_t index = Location(key);
			KVNode* cur = _tables[index];
			while (cur)
			{
				if (cur->_key == key)
				{
					return cur;
				}
				cur = cur->_next;
			}
			return NULL;
		}
	protected:
		void Swap(HashTable<K,V>& ht)
		{
			_tables.swap(ht._tables);
			swap(_size,ht._size);
		}
		void Destroy()
		{
			for (size_t i = 0; i < _tables.size(); ++i)
			{
				KVNode* cur = _tables[i];
				while (cur)
				{
					KVNode* tmp = cur;
					cur = cur->_next;
					delete tmp;
				}
				_tables[i] = NULL;
			}
		}
		void _CreateCapacity()                        //这里的设置散列表的容量时使用一个数组给定的容量值
		{
			static const int _PrimeSize = 28;
			static const unsigned long _PrimeList[_PrimeSize] =
			{
				53ul, 97ul, 193ul, 389ul, 769ul,
				1543ul, 3079ul, 6151ul, 12289ul, 24593ul,
				49157ul, 98317ul, 196613ul, 393241ul, 786433ul,
				1572869ul, 3145739ul, 6291469ul, 12582917ul, 25165843ul,
				50331653ul, 100663319ul, 201326611ul, 402653189ul, 805306457ul,
				1610612741ul, 3221225473ul, 4294967291ul
			};
			if (_size == _tables.size())
			{
				HashTable<K,V> ht;
				for (int i = 0; i < _PrimeSize;++i)
				{
					if (_size < _PrimeList[i])
					{
						ht._tables.resize(_PrimeList[i]);
						break;
					}		
				}
				for (size_t i = 0; i < _size;++i)
				{
					KVNode* cur = _tables[i];
					while (cur)
					{	
						ht.Insert(cur->_key, cur->_value);
						cur = cur->_next;
					}
				}
				_tables.swap(ht._tables);
			}
		}
	protected:
		vector<KVNode*> _tables;
		size_t _size;
	};

	void TestBucket()
	{
		HashTable<int,int> ht(2);
		ht.Insert(1,25);
		ht.Insert(11, 35);
		ht.Insert(12, 21);
		ht.Insert(16, 36);
		ht.Insert(19, 81);
		ht.Insert(54, 0);
		//ht.Print();

		//ht.Remove(12);
		//ht.Print();

		//HashTableNode<int, int>* tmp = ht.Find(54);
		//cout << "Find?" << tmp->_key << ":" << tmp->_value << endl;

		HashTable<int, int> ht1(ht);
		ht1.Print();

		HashTable<int, int> ht2;
		ht2 = ht1;
		ht2.Print();

		HashTable<string, string> htstr(10);
		htstr.Insert("字典","dictionary");
		htstr.Insert("清除", "clear,destroy");
		htstr.Insert("手机", "mobilephone");
		htstr.Insert("电脑", "pc");
		htstr.Insert("吸血鬼", "vampire");
		htstr.Print();

		HashTable<string, string> htstr1(htstr);
		htstr1.Print();
	}