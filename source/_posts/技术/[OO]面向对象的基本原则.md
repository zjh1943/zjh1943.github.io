---
title: '[OO]面向对象的基本原则'
date: 2016-03-10 16:40
tags: 
- ood 
- priciple 
- design 
categories: 
- 技术
---



The first five principles are principles of class design. They are:

<table border="1" cellspacing="0">
<tbody><tr><td><b>SRP</b></td>
<td><a href="https://docs.google.com/open?id=0ByOwmqah_nuGNHEtcU5OekdDMkk">The Single Responsibility Principle</a></td>
<td><i>A class should have one, and only one, reason to change.</i></td>
</tr>
<tr><td><b>OCP</b></td>
<td><a href="http://docs.google.com/a/cleancoder.com/viewer?a=v&amp;pid=explorer&amp;chrome=true&amp;srcid=0BwhCYaYDn8EgN2M5MTkwM2EtNWFkZC00ZTI3LWFjZTUtNTFhZGZiYmUzODc1&amp;hl=en">The Open Closed Principle</a></td>
<td><i>You should be able to extend a classes behavior, without modifying it.</i></td>
</tr>
<tr><td><b>LSP</b></td>
<td><a href="http://docs.google.com/a/cleancoder.com/viewer?a=v&amp;pid=explorer&amp;chrome=true&amp;srcid=0BwhCYaYDn8EgNzAzZjA5ZmItNjU3NS00MzQ5LTkwYjMtMDJhNDU5ZTM0MTlh&amp;hl=en">The Liskov Substitution Principle</a></td>
<td><i>Derived classes must be substitutable for their base classes.</i></td>
</tr>
<tr><td><b>ISP</b></td>
<td><a href="http://docs.google.com/a/cleancoder.com/viewer?a=v&amp;pid=explorer&amp;chrome=true&amp;srcid=0BwhCYaYDn8EgOTViYjJhYzMtMzYxMC00MzFjLWJjMzYtOGJiMDc5N2JkYmJi&amp;hl=en">The Interface Segregation Principle</a></td>
<td><i>Make fine grained interfaces that are client specific.</i></td>
</tr>
<tr><td><b>DIP</b></td>
<td><a href="http://docs.google.com/a/cleancoder.com/viewer?a=v&amp;pid=explorer&amp;chrome=true&amp;srcid=0BwhCYaYDn8EgMjdlMWIzNGUtZTQ0NC00ZjQ5LTkwYzQtZjRhMDRlNTQ3ZGMz&amp;hl=en">The Dependency Inversion Principle</a></td>
<td><i>Depend on abstractions, not on concretions.</i></td>
</tr>
</tbody></table>

The next six principles are about packages. In this context a package is a binary deliverable like a .jar file, or a dll as opposed to a namespace like a java package or a C++ namespace.

The first three package principles are about package cohesion, they tell us what to put inside packages:

<table border="1" cellspacing="0">
<tbody><tr><td><b>REP</b></td>
<td><a href="http://docs.google.com/a/cleancoder.com/viewer?a=v&amp;pid=explorer&amp;chrome=true&amp;srcid=0BwhCYaYDn8EgOGM2ZGFhNmYtNmE4ZS00OGY5LWFkZTYtMjE0ZGNjODQ0MjEx&amp;hl=en">The Release Reuse Equivalency Principle</a></td>
<td><i>The granule of reuse is the granule of release.</i></td>
</tr>
<tr><td><b>CCP</b></td>
<td><a href="http://docs.google.com/a/cleancoder.com/viewer?a=v&amp;pid=explorer&amp;chrome=true&amp;srcid=0BwhCYaYDn8EgOGM2ZGFhNmYtNmE4ZS00OGY5LWFkZTYtMjE0ZGNjODQ0MjEx&amp;hl=en">The Common Closure Principle</a></td>
<td><i>Classes that change together are packaged together.</i></td>
</tr>
<tr><td><b>CRP</b></td>
<td><a href="http://docs.google.com/a/cleancoder.com/viewer?a=v&amp;pid=explorer&amp;chrome=true&amp;srcid=0BwhCYaYDn8EgOGM2ZGFhNmYtNmE4ZS00OGY5LWFkZTYtMjE0ZGNjODQ0MjEx&amp;hl=en">The Common Reuse Principle</a></td>
<td><i>Classes that are used together are packaged together.</i></td>
</tr>
</tbody></table>


The last three principles are about the couplings between packages, and talk about metrics that evaluate the package structure of a system.


<table border="1" cellspacing="0">
<tbody><tr><td><b>ADP</b></td>
<td><a href="http://docs.google.com/a/cleancoder.com/viewer?a=v&amp;pid=explorer&amp;chrome=true&amp;srcid=0BwhCYaYDn8EgOGM2ZGFhNmYtNmE4ZS00OGY5LWFkZTYtMjE0ZGNjODQ0MjEx&amp;hl=en">The Acyclic Dependencies Principle</a></td>
<td><i>The dependency graph of packages must have no cycles.</i></td>
</tr>
<tr><td><b>SDP</b></td>
<td><a href="http://docs.google.com/a/cleancoder.com/viewer?a=v&amp;pid=explorer&amp;chrome=true&amp;srcid=0BwhCYaYDn8EgZjI3OTU4ZTAtYmM4Mi00MWMyLTgxN2YtMzk5YTY1NTViNTBh&amp;hl=en">The Stable Dependencies Principle</a></td>
<td><i>Depend in the direction of stability.</i></td>
</tr>
<tr><td><b>SAP</b></td>
<td><a href="http://docs.google.com/a/cleancoder.com/viewer?a=v&amp;pid=explorer&amp;chrome=true&amp;srcid=0BwhCYaYDn8EgZjI3OTU4ZTAtYmM4Mi00MWMyLTgxN2YtMzk5YTY1NTViNTBh&amp;hl=en">The Stable Abstractions Principle</a></td>
<td><i>Abstractness increases with stability.</i></td>
</tr>
</tbody></table>



中文版：

<table class="wikitable" style="width: auto; font-size: 95%; table-layout: fixed; line-height:1.25; margin-left: auto; margin-right: auto;">
<tbody><tr>
<th>首字母</th>
<th>指代</th>
<th>概念</th>
</tr>
<tr>
<th>S</th>
<td><a href="https://zh.wikipedia.org/wiki/%E5%8D%95%E4%B8%80%E5%8A%9F%E8%83%BD%E5%8E%9F%E5%88%99" title="单一功能原则">单一功能原则</a></td>
<td>
<dl>
<dt><a href="https://zh.wikipedia.org/wiki/%E5%8D%95%E4%B8%80%E5%8A%9F%E8%83%BD%E5%8E%9F%E5%88%99" title="单一功能原则">单一功能原则</a></dt>
<dd>认为<a href="https://zh.wikipedia.org/wiki/%E5%AF%B9%E8%B1%A1_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6)" title="对象 (计算机科学)">对象</a>应该仅具有一种单一功能的概念。</dd>
</dl>
</td>
</tr>
<tr>
<th>O</th>
<td><a href="https://zh.wikipedia.org/wiki/%E5%BC%80%E9%97%AD%E5%8E%9F%E5%88%99" title="开闭原则">开闭原则</a></td>
<td>
<dl>
<dt><a href="https://zh.wikipedia.org/wiki/%E5%BC%80%E9%97%AD%E5%8E%9F%E5%88%99" title="开闭原则">开闭原则</a></dt>
<dd>认为“软件体应该是对于扩展开放的，但是对于修改封闭的”的概念。</dd>
</dl>
</td>
</tr>
<tr>
<th>L</th>
<td><a href="https://zh.wikipedia.org/wiki/%E9%87%8C%E6%B0%8F%E6%9B%BF%E6%8D%A2%E5%8E%9F%E5%88%99" title="里氏替换原则">里氏替换原则</a></td>
<td>
<dl>
<dt><a href="https://zh.wikipedia.org/wiki/%E9%87%8C%E6%B0%8F%E6%9B%BF%E6%8D%A2%E5%8E%9F%E5%88%99" title="里氏替换原则">里氏替换原则</a></dt>
<dd>认为“程序中的对象应该是可以在不改变程序正确性的前提下被它的子类所替换的”的概念。参考 <a href="https://zh.wikipedia.org/wiki/%E5%A5%91%E7%BA%A6%E5%BC%8F%E8%AE%BE%E8%AE%A1" title="契约式设计">契约式设计</a>。</dd>
</dl>
</td>
</tr>
<tr>
<th>I</th>
<td><a href="https://zh.wikipedia.org/wiki/%E6%8E%A5%E5%8F%A3%E9%9A%94%E7%A6%BB%E5%8E%9F%E5%88%99" title="接口隔离原则">接口隔离原则</a></td>
<td>
<dl>
<dt><a href="https://zh.wikipedia.org/wiki/%E6%8E%A5%E5%8F%A3%E9%9A%94%E7%A6%BB%E5%8E%9F%E5%88%99" title="接口隔离原则">接口隔离原则</a></dt>
<dd>认为“多个特定客户端接口要好于一个宽泛用途的接口”<sup id="cite_ref-martin-design-principles_5-0" class="reference"><a href="#cite_note-martin-design-principles-5">[5]</a></sup> 的概念。</dd>
</dl>
</td>
</tr>
<tr>
<th>D</th>
<td><a href="https://zh.wikipedia.org/wiki/%E4%BE%9D%E8%B5%96%E5%8F%8D%E8%BD%AC%E5%8E%9F%E5%88%99" title="依赖反转原则">依赖反转原则</a></td>
<td>
<dl>
<dt><a href="https://zh.wikipedia.org/wiki/%E4%BE%9D%E8%B5%96%E5%8F%8D%E8%BD%AC%E5%8E%9F%E5%88%99" title="依赖反转原则">依赖反转原则</a></dt>
<dd>认为一个方法应该遵从“依赖于抽象而不是一个实例”<sup id="cite_ref-martin-design-principles_5-1" class="reference"><a href="#cite_note-martin-design-principles-5">[5]</a></sup> 的概念。<br>
<a href="https://zh.wikipedia.org/wiki/%E4%BE%9D%E8%B5%96%E6%B3%A8%E5%85%A5" title="依赖注入" class="mw-redirect">依赖注入</a>是该原则的一种实现方式。</dd>
</dl>
</td>
</tr>
</tbody></table>

