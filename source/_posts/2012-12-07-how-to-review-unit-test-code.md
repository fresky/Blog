---
layout: post
title: "如何做test review"
date: 2012-12-07
comments: true
tags: Testing
---
<p><a href="http://artofunittesting.com/unit-testing-review-guidelines/">Test Review Guidelines</a>给出了一些review unit test时的一些准则，我简单翻译一下。</p>  <h2>可读性：</h2>  <ol>   <li>确保setup和teardown方法没有被滥用。最好用factory method来提高可读性。</li>    <li>确保每个test只测试了一件事。</li>    <li>检查是否符合好的，一致的命名规范。</li>    <li>确保只有有意义的assert message才被用到，用有意义的test名称更好。</li>    <li>确保assert和action分开。</li>    <li>确保测试没有使用magic string/value作为输入，用可能的最简单的输入来验证。</li>    <li>确保测试放置的位置具有一致性，可以方便的找到一个方法，一个类，一个项目相关的测试。</li> </ol>  <h2>可维护性：</h2>  <ol>   <li>确保test之间没有依赖关系，并且是可以重复的。</li>    <li>确保测试private或者protected地测试是个例，测试public的总是更好。</li>    <li>确保测试没有过分详细。</li>    <li>确保总是优先考虑基于状态的测试，其次才是基于交互的测试。</li>    <li>确保尽量少使用精确的mock（否则会导致过分详细和脆弱的测试）。</li>    <li>确保每个测试中只是用了一个mock。</li>    <li>确保在同一个测试中没有混淆mock和正常的assert。</li>    <li>确保测试只验证一个mock的函数调用。（否则会导致过分详细和脆弱的测试）。</li>    <li>确保测试只验证一个mock的一个函数调用。（否则会导致过分详细和脆弱的测试）。</li>    <li>确保只有在非常少的情况下，一个mock被同时当作一个stub来使用，在验证的同时返回值。</li> </ol>  <h2>可信性：</h2>  <ol>   <li>确保测试不包含逻辑或者动态的数值。</li>    <li>通过改变数值（布尔或者常量）来检查测试覆盖率。</li>    <li>确保单元测试和集成测试分开。</li>    <li>确保测试没有使用一直在变化的值（比如系统当前时间），应该使用固定的值。</li>    <li>确定测试没有assert动态产生的期望值。（你可能只是在重复的你的production code，没有起到测试的作用）。</li> </ol>