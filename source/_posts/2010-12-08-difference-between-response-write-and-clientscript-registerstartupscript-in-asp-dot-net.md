---
layout: post
title: "ASP .NET调用javascript中Response.Write和ClientScript.RegisterStartupScript的区别"
date: 2010-12-08
comments: true
tags: CSharp
---
最近在用ASP .NET的code behind 调用javascript中发现Response.Write不能拿到form的值，而ClientScript.RegisterStartupScript可以。例如下面的代码<br />

```c#
StringBuilder sb = new StringBuilder();
sb.Append("<script language=javascript>");
sb.Append("alert(document.forms.length);");
sb.Append("</script>");
Response.Write(sb.ToString());
ClientScript.RegisterStartupScript(this.GetType(), "test", sb.ToString());
```

可以明显的看到，Response.Write得到的是0，ClientScript.RegisterStartupScript得到的是1。<br /><br />

另外，Response.Write不能调用aspx里面定义的javascript函数，ClientScript.RegisterStartupScrip可以，示例如下。<br />.cs代码<br />

```c#
StringBuilder sb = new StringBuilder();
sb.Append("<script language=javascript>");
sb.Append("TestAlert();");
sb.Append("</script>");
//Response.Write(sb.ToString());
ClientScript.RegisterStartupScript(this.GetType(), "test", sb.ToString());
```

.aspx代码<br />

```c#
<script type="text/javascript">
function TestAlert() {
	alert('just a test');
}
</script>
```

可以看到Response.Write会出错，firebug里面提示TestAlert没有定义，而ClientScript.RegisterStartupScript可以正确执行。<br /><br />