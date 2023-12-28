---
layout: post
title: "使用C++的开源序列化(Serialization)库cereal"
date: 2014-04-04 10:26
comments: true
tags: [Programming, Tool]
keywords: c++, cereal, serialization, open source
description: cereal是一个开源的（BSD License），轻量级的C++序列化库。它只有头文件，支持序列化成binary，xml，JSON。
---
[cereal](http://uscilab.github.io/cereal/index.html)是一个开源的（BSD License），轻量级的C++序列化库。它只有头文件，支持序列化成binary，xml，JSON。

下面是一个小例子：
```c++
void Serialization()
{
    {
        std::ofstream os("data.xml");
        cereal::XMLOutputArchive archive(os);

        int Age = 30;
        string str = "Dawei";

        archive(CEREAL_NVP(Age), cereal::make_nvp("Name", str));
    }

    {
        std::ifstream is("data.xml");
        cereal::XMLInputArchive archive(is);

        int age;
        string name;

        archive(age, name);
        cout << "Age: " << age << endl << "Name: " << name << endl;
    }
}
```
生成的xml文件如下：
```xml
<?xml version="1.0" encoding="utf-8"?>
<cereal>
	<Age>30</Age>
	<Name>Dawei</Name>
</cereal>
```