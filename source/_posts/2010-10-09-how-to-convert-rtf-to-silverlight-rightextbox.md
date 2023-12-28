---
layout: post
title: "silverlight中的richtextbox和rtf格式的转换"
date: 2010-10-09
comments: true
tags: Programming
---
<p>silvergliht 4.0中有了richtextbox控件，但是它只接受xaml格式的输入，下面这个函数实现了rtf和xaml的转换，用到了flowdocument。<br /><br />需要注意的是直接从rtf转换过来的话有些attribute silverlight不认。我也不知道找全了没有，目前发现的是 section 里面的一堆attribute，然后就是margin和textindent。另外中文字体也会出错。<br /><br /></p>

```csharp
/// <summary>
/// convert the string "from" from sourceFormat to targetFormat
/// </summary>
/// <param name="from"></param>
/// <param name="sourceFormat">System.Windows.DataFormats.Rtf</param>
/// <param name="targetFormat">System.Windows.DataFormats.Xaml </param>
/// <returns></returns>
private string convertFormat(string from, string sourceFormat, string targetFormat)
{
    string xaml = String.Empty;
    System.Windows.Documents.FlowDocument doc = new System.Windows.Documents.FlowDocument();
    System.Windows.Documents.TextRange range = new System.Windows.Documents.TextRange(doc.ContentStart, doc.ContentEnd);

    using (MemoryStream ms = new MemoryStream())
    {
        using (StreamWriter sw = new StreamWriter(ms))
        {
            sw.Write(from);
            sw.Flush();
            ms.Seek(0, SeekOrigin.Begin);
            range.Load(ms, sourceFormat);
        }
    }


    using (MemoryStream ms = new MemoryStream())
    {
        range = new System.Windows.Documents.TextRange(doc.ContentStart, doc.ContentEnd);

        range.Save(ms, targetFormat);
        ms.Seek(0, SeekOrigin.Begin);
        using (StreamReader sr = new StreamReader(ms))
        {
            xaml = sr.ReadToEnd();
        }
    }

    // remove all attribuites in section and remove attribute margin
    if (targetFormat.Equals(System.Windows.DataFormats.Xaml))
    {
        int start = xaml.IndexOf("<Section");
        int stop = xaml.IndexOf(">") + 1;

        string section = xaml.Substring(start, stop);

        xaml = xaml.Replace(section, "<Section xml:space=\"preserve\" HasTrailingParagraphBreakOnPaste=\"False\" xmlns=\"http://schemas.microsoft.com/winfx/2006/xaml/presentation\">");

        // remove Margin attribute
        Regex reg = new Regex("Margin=\"[^\"]*\"");
        xaml = reg.Replace(xaml, string.Empty);

       // remove TextIndent attribute
        reg = new Regex("TextIndent=\"[^\"]*\"");
        xaml = reg.Replace(xaml, string.Empty);

        // change the non english font family
        reg = new Regex("FontFamily=\"[^a-zA-Z]*\"");
        xaml = reg.Replace(xaml, "FontFamily=\"Arial\"");

    }
    return xaml;
}
```