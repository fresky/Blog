---
layout: post
title: "使用windbg通过vtable找到优化后的this指针"
date: 2012-07-10
comments: true
tags: Debug
---


如果用windbg调试的时候遇到优化过的代码，this指针的地址是不准确的，下面介绍如何通过vtable找到this指针。<br /><br />1. <strong>kbn</strong><br /># ChildEBP? RetAddr? Args to Child<br />00 1d61fad0 7c90d21a 7c8023f1 00000000 1d61fb04 ntdll!KiFastSystemCallRet<br />01 1d61fad4 7c8023f1 00000000 1d61fb04 1a314e78 ntdll!NtDelayExecution+0xc<br />02 1d61fb2c 7c802455 00000042 00000000 1d61fb6c kernel32!SleepEx+0x61<br />03 <span style="color: #ff0000;">1d61fb3c </span><span style="color: #3366ff;">(ChildEBPaftercall</span>) 4c08f466 00000042 6496c8a2 1a3128f0 kernel32!Sleep+0xf<br />04 1d61fb6c 5c2656d4 616c06bc 1d61fbbc 1d61fe74 DllName!NameSpaceName::ClassName::OnProcess+0x106 [source1.cpp @ 5908]<br />05 <span style="color: #ff0000;">1d61fbb0 </span>(<span style="color: #3366ff;">ChildEBPbeforecall</span>) 77520c9a 1a312b0c 1d61fd78 0ef5bd18<br />Dll2Name!Class2Name::Process+0xb4 [source2.cpp @ 104]<br />06 1d61fbc8 77587f67 4c0e4df0 1a312b0c 1d61fcfc ole32!CallFrame::Invoke+0x54<br /><br />2. <strong>dpp </strong>(<span style="color: #3366ff;">ChildEBPaftercall</span>) (<span style="color: #3366ff;">ChildEBPbeforecall</span>) ，来找到vtable </p>
<p>0:022&gt; dpp 1d61fb3c 1d61fbb0<br />1d61fb3c 1d61fb6c 1d61fbb0 &lt;Unloaded_API.DLL&gt;+0x1d61fb7f<br />1d61fb40 4c08f466 d908ec83<br />1d61fb44 00000042<br />1d61fb48 6496c8a2 1e6041d6 &lt;Unloaded_API.DLL&gt;+0x1e6041a5<br />1d61fb4c <span style="color: #ff0000;">1a3128f0 </span><span style="color: #3366ff;">(init this pointer address</span>) <span style="color: #ff0000;">4c1a3e24 </span>(<span style="color: #3366ff;">vtable address</span>) DllName!ATL::CComObject::`vftable'<br /><br />3. <strong>dds </strong>(<span style="color: #3366ff;">vtable address</span>)<strong>-4</strong>，来找到RTTI </p>
<p>0:022&gt; dds 4c1a3e24 -4<br /><span style="color: #000000;">4c1a3e20 </span><span style="color: #ff0000;">4c1b99e0 </span>(<span style="color: #3366ff;">RTTI address</span>) DllName!ATL::CComObject::`RTTI Complete Object Locator'<br /><br />4. <strong>dds </strong>(<span style="color: #3366ff;">RTTI address</span>)，来找到偏移量，是第二个 </p>
<p>0:022&gt; dds 4c1b99e0<br />4c1b99e0 00000000<br />4c1b99e4 <span style="color: #ff0000;">00000000 </span>(<span style="color: #3366ff;">offset</span>)<br />4c1b99e8 00000000<br /><br />5. <strong>dt ModuleName <span style="color: #3366ff;">(init this pointer address</span>) - (<span style="color: #3366ff;">offset</span>)</strong> ，查看this指针，搞定：）<br />dt DllName!NameSpaceName::ClassName 1a3128f0-0 </p>
<div>
