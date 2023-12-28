---
layout: post
title: "C++的vector::push_back()和vector::resize()比较"
date: 2011-06-24
comments: true
tags: Programming
---
今天看别的code写着下面几行来往vector里面添加一个元素。<br />

```cpp
vector<Type> v;
long size = (long)v.size();
v.resize(size+1);
v[size] = newValue;
```

觉得挺奇怪，就去查了查resize()和push_back()的区别，可以看到他们的源代码如下：<br />

```cpp
	void resize(size_type _Newsize)
	{	// determine new length, padding with _Ty() elements as needed
		resize(_Newsize, _Ty());
	}
	void resize(size_type _Newsize, _Ty _Val)
	{	// determine new length, padding with _Val elements as needed
		if (size() < _Newsize)
			_Insert_n(end(), _Newsize - size(), _Val);
		else if (_Newsize < size())
			erase(begin() + _Newsize, end());
	}

	void push_back(const _Ty& _Val)
	{	// insert element at end
		if (size() < capacity())
			_Mylast = _Ufill(_Mylast, 1, _Val);
		else
			insert(end(), _Val);
	}
	iterator insert(const_iterator _Where, const _Ty& _Val)
	{	// insert _Val at _Where
		size_type _Off = size() == 0 ? 0 : _Where - begin();
		_Insert_n(_Where, (size_type)1, _Val);
		return (begin() + _Off);
	}
```

<br />从代码可以看出来，如果size()小于capasity()肯定是push_back()比resize()要好。如果size()大于等于capacity()，那么push_back()和resize()差不多，但是怎么着都比resize()然后再赋值好。所以最开始的代码应该用push_back()。