---
layout: post
title: "C#中5种timer的比较"
date: 2013-07-09
comments: true
tags: CSharp
---
<p>C#中有5个timer，它们的主要区别如下：</p>
<ol>
<li>System.Threading.Timer&nbsp; 在线程池启动一个后台任务。我前段时间写过一个关于timer的垃圾回收的需要注意一下，参见<a href="/2013/06/20/where-is-my-timer-csharp-gc/">谁动了我的timer？</a>。</li>
<li>System.Windows.Forms.Timer&nbsp; 告诉windows把一个计时器和调用它的线程(UI线程)关联起来，通过往UI线程的消息队列里放一个WM_TIMER的消息来实现，所以它的callback一定是在UI线程调用的，不存在多线程调用的问题。</li>
<li>System.Windows.Threading.DispatcherTimer 用在WPF和Silverlight中，对应于System.Windows.Forms.Timer。</li>
<li>Windows.UI.Xaml.Dispatchertimer 用在windows store app中，对应于System.Windows.Forms.Timer。</li>
<li>System.Timers.Timer 包装了一下System.Threading.Timer，使它有了和System.Windows.Forms.Timer类似的接口，而且也能在visual studio的toolbox designer里找到。它也是在线程池中执行，但是如果你是在visual studio的designer中使用它，visual studio会自动把它所在的control设为这个timer的SynchronizingObject，这样就会保证callback会在UI线程调用了。Jeffrey Richter不建议使用它，建议直接用System.Threading.Timer。这个timer也有个坑，参见<a href="/2011/06/23/dot-net-2-timer-elapsed-event-will-catch-all-exception-for-you/">.NET 2.0的Timer elapsed event 会自动catch住所有的exception</a>。</li>
</ol>
<p><a href="http://msdn.microsoft.com/en-us/magazine/cc164015.aspx">Comparing the Timer Classes in the .NET Framework Class Library</a>也比较了3中timer（System.Threading.Timer ，System.Windows.Forms.Timer和System.Timers.Timer），并且画了个如下的表格。</p>
<table style="width: 686px;" border="0" cellspacing="0" cellpadding="0"><colgroup><col style="mso-width-source: userset; mso-width-alt: 9216; width: 189pt;" width="252" /> <col style="mso-width-source: userset; mso-width-alt: 6290; width: 129pt;" width="172" /> <col style="mso-width-source: userset; mso-width-alt: 4717; width: 97pt;" width="129" /> <col style="mso-width-source: userset; mso-width-alt: 4864; width: 100pt;" width="133" /> </colgroup>
<tbody>
<tr style="height: 15.0pt;">
<td id="th113923680000" class="xl65" style="height: 15.0pt; width: 189pt; font-size: 11.0pt; color: #cce8cf; font-weight: bold; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: .5pt solid #9BC2E6; background: #5B9BD5; mso-pattern: #5B9BD5 none;" width="252" height="20">&nbsp;</td>
<td id="th113923680001" class="xl65" style="width: 129pt; font-size: 11.0pt; color: #cce8cf; font-weight: bold; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: none; background: #5B9BD5; mso-pattern: #5B9BD5 none;" width="172">System.Windows.Forms</td>
<td id="th113923680002" class="xl65" style="width: 97pt; font-size: 11.0pt; color: #cce8cf; font-weight: bold; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: none; background: #5B9BD5; mso-pattern: #5B9BD5 none;" width="129">System.Timers</td>
<td id="th113923680003" class="xl65" style="width: 100pt; font-size: 11.0pt; color: #cce8cf; font-weight: bold; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: .5pt solid #9BC2E6; border-bottom: .5pt solid #9BC2E6; border-left: none; background: #5B9BD5; mso-pattern: #5B9BD5 none;" width="133">System.Threading</td>
</tr>
<tr style="height: 15.0pt;">
<td class="xl66" style="height: 15.0pt; width: 189pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: .5pt solid #9BC2E6; background: #DDEBF7; mso-pattern: #DDEBF7 none;" width="252" height="20">Timer   event runs on what thread?</td>
<td class="xl66" style="width: 129pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: none; background: #DDEBF7; mso-pattern: #DDEBF7 none;" width="172">UI thread</td>
<td class="xl66" style="width: 97pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: none; background: #DDEBF7; mso-pattern: #DDEBF7 none;" width="129">UI or worker thread</td>
<td class="xl66" style="width: 100pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: .5pt solid #9BC2E6; border-bottom: .5pt solid #9BC2E6; border-left: none; background: #DDEBF7; mso-pattern: #DDEBF7 none;" width="133">Worker thread</td>
</tr>
<tr style="height: 15.0pt;">
<td class="xl66" style="height: 15.0pt; width: 189pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: .5pt solid #9BC2E6;" width="252" height="20">Instances are thread safe?</td>
<td class="xl66" style="width: 129pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: none;" width="172">No</td>
<td class="xl66" style="width: 97pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: none;" width="129">Yes</td>
<td class="xl66" style="width: 100pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: .5pt solid #9BC2E6; border-bottom: .5pt solid #9BC2E6; border-left: none;" width="133">No</td>
</tr>
<tr style="height: 15.0pt;">
<td class="xl66" style="height: 15.0pt; width: 189pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: .5pt solid #9BC2E6; background: #DDEBF7; mso-pattern: #DDEBF7 none;" width="252" height="20">Familiar/intuitive   object model?</td>
<td class="xl66" style="width: 129pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: none; background: #DDEBF7; mso-pattern: #DDEBF7 none;" width="172">Yes</td>
<td class="xl66" style="width: 97pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: none; background: #DDEBF7; mso-pattern: #DDEBF7 none;" width="129">Yes</td>
<td class="xl66" style="width: 100pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: .5pt solid #9BC2E6; border-bottom: .5pt solid #9BC2E6; border-left: none; background: #DDEBF7; mso-pattern: #DDEBF7 none;" width="133">No</td>
</tr>
<tr style="height: 15.0pt;">
<td class="xl66" style="height: 15.0pt; width: 189pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: .5pt solid #9BC2E6;" width="252" height="20">Requires Windows Forms?</td>
<td class="xl66" style="width: 129pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: none;" width="172">Yes</td>
<td class="xl66" style="width: 97pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: none;" width="129">No</td>
<td class="xl66" style="width: 100pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: .5pt solid #9BC2E6; border-bottom: .5pt solid #9BC2E6; border-left: none;" width="133">No</td>
</tr>
<tr style="height: 15.0pt;">
<td class="xl66" style="height: 15.0pt; width: 189pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: .5pt solid #9BC2E6; background: #DDEBF7; mso-pattern: #DDEBF7 none;" width="252" height="20">Metronome-quality   beat?</td>
<td class="xl66" style="width: 129pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: none; background: #DDEBF7; mso-pattern: #DDEBF7 none;" width="172">No</td>
<td class="xl66" style="width: 97pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: none; background: #DDEBF7; mso-pattern: #DDEBF7 none;" width="129">Yes*</td>
<td class="xl66" style="width: 100pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: .5pt solid #9BC2E6; border-bottom: .5pt solid #9BC2E6; border-left: none; background: #DDEBF7; mso-pattern: #DDEBF7 none;" width="133">Yes*</td>
</tr>
<tr style="height: 15.0pt;">
<td class="xl66" style="height: 15.0pt; width: 189pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: .5pt solid #9BC2E6;" width="252" height="20">Timer event supports state object?</td>
<td class="xl66" style="width: 129pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: none;" width="172">No</td>
<td class="xl66" style="width: 97pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: none;" width="129">No</td>
<td class="xl66" style="width: 100pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: .5pt solid #9BC2E6; border-bottom: .5pt solid #9BC2E6; border-left: none;" width="133">Yes</td>
</tr>
<tr style="height: 15.0pt;">
<td class="xl66" style="height: 15.0pt; width: 189pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: .5pt solid #9BC2E6; background: #DDEBF7; mso-pattern: #DDEBF7 none;" width="252" height="20">Initial   timer event can be scheduled?</td>
<td class="xl66" style="width: 129pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: none; background: #DDEBF7; mso-pattern: #DDEBF7 none;" width="172">No</td>
<td class="xl66" style="width: 97pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: none; background: #DDEBF7; mso-pattern: #DDEBF7 none;" width="129">No</td>
<td class="xl66" style="width: 100pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: .5pt solid #9BC2E6; border-bottom: .5pt solid #9BC2E6; border-left: none; background: #DDEBF7; mso-pattern: #DDEBF7 none;" width="133">Yes</td>
</tr>
<tr style="height: 15.0pt;">
<td class="xl66" style="height: 15.0pt; width: 189pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: .5pt solid #9BC2E6;" width="252" height="20">Class supports inheritance?</td>
<td class="xl66" style="width: 129pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: none;" width="172">Yes</td>
<td class="xl66" style="width: 97pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: none; border-bottom: .5pt solid #9BC2E6; border-left: none;" width="129">Yes</td>
<td class="xl66" style="width: 100pt; font-size: 11.0pt; color: black; font-weight: 400; text-decoration: none; text-underline-style: none; text-line-through: none; font-family: Calibri; border-top: .5pt solid #9BC2E6; border-right: .5pt solid #9BC2E6; border-bottom: .5pt solid #9BC2E6; border-left: none;" width="133">No</td>
</tr>
<tr style="height: 15.0pt;">
<td class="xl66" style="height: 15.0pt; width: 515pt;" colspan="4" width="686" height="20">* Depending on the availability of system resources   (for example, worker threads)</td>
</tr>
</tbody>
</table>
<table style="width: 1px; height: 1px;" border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr style="height: 15.0pt;">
<td class="xl66" style="height: 15.0pt; width: 495pt;" colspan="4" width="660" height="20">&nbsp;</td>
</tr>
</tbody>
</table>