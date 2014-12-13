---
layout: post
title: "C#操作.rtf文档"
date: 2010-12-16
comments: true
tags: CSharp
---
前段时间做过一些用C#操作.rtf文档的工作，当时就是直接根据RTF的规范用修改文本文件的方式修改.rtf文件。比如要插入一段文字，并且用红色表示，就自己弄一个colortable的定义，然后直接把下面的内容插入到.rtf文件中， {\b\cf1Bold Red!} \b表示黑体，\cf1表示用colortable定义中的第一种颜色。（假设我们的这个rtf文件中定义的第一种颜色是红色）。但是这种办法很不直观，而且有限制，需要编码时知道colortable是怎么定义的。<br /><br />今天重新用System.Windows.Forms.RichTextBox来实现。下面的代码实现了在原来的rtf文件的最开始插入一行红色黑体的“Alert”，然后在rtf的commens：后面加上“new comment”，最后在末尾加一句“Bye!”。<br />

```c#
private static void RtfTest(string inpath, string outpath)
{
   // load rtf file
   RichTextBox rtf = new RichTextBox();
   using (StreamReader sr = new StreamReader(inpath))
   {
       rtf.Rtf = sr.ReadToEnd();
   }

	// add alert the begining of the rtf file
   rtf.SelectionStart = 0;
   rtf.SelectionLength = 1;
   string alert = "Alert!" + Environment.NewLine;
   rtf.SelectedText = alert + rtf.SelectedText;
   rtf.SelectionStart = 0;
   // length of new line in string is 2, but in rtf is 1, so we need to minus the line count
   rtf.SelectionLength = alert.Length - 1;
   rtf.SelectionColor = Color.Red;
   rtf.SelectionFont = new Font(rtf.SelectionFont, System.Drawing.FontStyle.Bold);

    // add comment after "Comments:"
   string commentStart = "Comments:";
   string newComment = "new comment";
   rtf.Find(commentStart);
   rtf.SelectedText = rtf.SelectedText + Environment.NewLine + newComment;
   rtf.Find(commentStart);
   rtf.SelectionStart += commentStart.Length;
   rtf.SelectionLength += newComment.Length;
   rtf.SelectionColor = Color.Black;
   rtf.SelectionFont = new Font(rtf.SelectionFont, System.Drawing.FontStyle.Regular);

  // add "Bye!" to the end
   rtf.AppendText(Environment.NewLine + "Bye!");

  // save the rtf file
   rtf.SaveFile(outpath);
   rtf.Dispose();
   rtf = null;
}
```

其中有几个需要注意的地方：<br />1.不能直接给rtf.Text赋值，给Text赋值会导致整个rtf文档的格式丢失。这个示例中使用了selection来实现。<br />2.string中的换行的length是2，但是rtf中的换行的length是1，根据这个长度设置选择区域是要注意。<br />3.往rtf文档的最后以行加东西可以直接调用AppendText，会保留原来的格式。<br /><br /><br />