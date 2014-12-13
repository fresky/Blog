---
layout: post
title: "定制C# combobox的下拉框"
date: 2010-11-08
comments: true
tags: CSharp
---
<p>今天做了个小东西，需要定制combobox的下拉框，打开下拉框时如下图。</p>

{% limg coloredcombobox.jpg %}

选择一个后，如下图。

{% limg coloredcombobox2.jpg %}

实现的方法是需要把combobox的DrawoMode设置成OwnerDrawVariable，然后处理DrawItem事件，详见<a href="http://msdn.microsoft.com/en-us/library/system.windows.forms.combobox.drawitem.aspx">ComboBox.DrawItem Event (System.Windows.Forms)</a>代码如下：</p>

```c#
private void cb_Risk_DrawItem(object sender, DrawItemEventArgs e)
{
    if (e.Index < 0) return;

    switch (e.Index)
    {
        case 0:
            e.Graphics.FillRectangle(Brushes.Red, e.Bounds);
            break;
        case 1:
            e.Graphics.FillRectangle(Brushes.Yellow, e.Bounds);
            break;
        case 2:
            e.Graphics.FillRectangle(Brushes.Blue, e.Bounds);
            break;
        default:
            break;
    }
    e.Graphics.DrawString(cb_Risk.Items[e.Index].ToString(), cb_Risk.Font, Brushes.Black, (RectangleF)e.Bounds);
}
```

<p><br />对了，我这里的代码都是用<a href="http://copysourceashtml.codeplex.com/">CopySourceAsHtml</a>这个VS的addin粘进来的，对于VS2010，这篇文章<a href="http://blogs.microsoft.co.il/blogs/applisec/archive/2010/02/25/copyashtml-in-visual-studio-2010.aspx">CopyAsHtml in Visual Studio 2010 - AppliSec</a>有一个workaround。</p>
<p><br /><br /></p>
