---
layout: post
title: "C# XMLSerialize 去掉xml声明和xml namespace"
date: 2012-09-28
comments: true
tags: Programming
---
下面的代码可以在C# XMLSerialize 时去掉xml声明和xml namespace。

```csharp
	private static void OutputXml(string xmlFilePath, ObjectToSerialize  objectToSerialize )
	{
		XmlSerializer xmlSerializer = new XmlSerializer(typeof(ObjectToSerialize ));
		XmlSerializerNamespaces ns = new XmlSerializerNamespaces();
		ns.Add("", "");
		XmlWriterSettings settings = new XmlWriterSettings {OmitXmlDeclaration = true, Indent = true};

		using (XmlWriter xmlWriter = XmlWriter.Create(xmlFilePath, settings))
		{
			xmlSerializer.Serialize(xmlWriter, objectToSerialize, ns);
		}
	}
```