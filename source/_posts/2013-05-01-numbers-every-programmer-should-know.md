---
layout: post
title: "每个程序员都应该知道的14个数字（关于算法运行时间的）"
date: 2013-05-01
comments: true
tags: Algorithm
---
<p><a href="http://www.infoarena.ro/blog/numbers-everyone-should-know">14 numbers every developer should know</a>列举了14个数字，可以作为大家设计算法时的一个参考。下表列出了秒数量级下各种算法复杂度能接受的输入个数。</p>
<table style="width: 1002px; height: 801px;" border="1" cellpadding="0">
<tbody>
<tr>
<td width="176">
<p><strong>maximum n</strong></p>
</td>
<td width="143">
<p><strong>complexity</strong></p>
</td>
<td width="363">
<p><strong>algorithms</strong></p>
</td>
<td width="476">
<p><strong>data structures</strong></p>
</td>
</tr>
<tr>
<td width="176">
<p>1,000,000,000 and higher</p>
</td>
<td width="143">
<p>log n, sqrt n</p>
</td>
<td width="363">
<p>binary search, ternary search, fast exponentiation, euclid algorithm</p>
</td>
<td width="476">&nbsp;</td>
</tr>
<tr>
<td width="176">
<p>10,000,000</p>
</td>
<td width="143">
<p>n, n log log n, n log* n</p>
</td>
<td width="363">
<p>set intersection, Eratosthenes sieve, radix sort, KMP, topological sort, Euler tour, strongly connected components, 2sat</p>
</td>
<td width="476">
<p>disjoint sets, tries, hash_map, <a href="http://www.infoarena.ro/blog/rolling-hash">rolling hash</a> deque</p>
</td>
</tr>
<tr>
<td width="176">
<p>1,000,000</p>
</td>
<td width="143">
<p>n log n</p>
</td>
<td width="363">
<p>sorting, divide and conquer, sweep line, Kruskal, Dijkstra</p>
</td>
<td width="476">
<p>segment trees, range trees, heaps, treaps, binary indexed trees, suffix arrays</p>
</td>
</tr>
<tr>
<td width="176">
<p>100,000</p>
</td>
<td width="143">
<p>n log<sup>2</sup> n</p>
</td>
<td width="363">
<p>divide and conquer</p>
</td>
<td width="476">
<p>2d range trees</p>
</td>
</tr>
<tr>
<td width="176">
<p>50,000</p>
</td>
<td width="143">
<p>n<sup>1.585</sup>, n sqrt n</p>
</td>
<td width="363">
<p>Karatsuba, <a href="http://www.infoarena.ro/blog/square-root-trick">square root trick</a></p>
</td>
<td width="476">
<p>two level tree</p>
</td>
</tr>
<tr>
<td width="176">
<p>1000 - 10,000</p>
</td>
<td width="143">
<p>n<sup>2</sup></p>
</td>
<td width="363">
<p>largest empty rectangle, Dijkstra, Prim (on dense graphs)</p>
</td>
<td width="476">&nbsp;</td>
</tr>
<tr>
<td width="176">
<p>300-500</p>
</td>
<td width="143">
<p>n<sup>3</sup></p>
</td>
<td width="363">
<p>all pairs shortest paths, largest sum submatrix, naive matrix multiplication, matrix chain multiplication, gaussian elimination, network flow</p>
</td>
<td width="476">&nbsp;</td>
</tr>
<tr>
<td width="176">
<p>30-50</p>
</td>
<td width="143">
<p>n<sup>4</sup>, n<sup>5</sup>, n<sup>6</sup></p>
</td>
<td width="363">&nbsp;</td>
<td width="476">&nbsp;</td>
</tr>
<tr>
<td width="176">
<p>25 - 40</p>
</td>
<td width="143">
<p>3<sup>n/2</sup>, 2<sup>n/2</sup></p>
</td>
<td width="363">
<p><a href="http://www.infoarena.ro/blog/meet-in-the-middle">meet in the middle</a></p>
</td>
<td width="476">
<p>hash tables (for set intersection)</p>
</td>
</tr>
<tr>
<td width="176">
<p>15 - 24</p>
</td>
<td width="143">
<p>2<sup>n</sup></p>
</td>
<td width="363">
<p>subset enumeration, brute force, dynamic programming with exponential states</p>
</td>
<td width="476">&nbsp;</td>
</tr>
<tr>
<td width="176">
<p>15 - 20</p>
</td>
<td width="143">
<p>n<sup>2</sup> 2<sup>n</sup></p>
</td>
<td width="363">
<p>dynamic programming with exponential states</p>
</td>
<td width="476">
<p>bitsets, hash_map</p>
</td>
</tr>
<tr>
<td width="176">
<p>13-17</p>
</td>
<td width="143">
<p>3<sup>n</sup></p>
</td>
<td width="363">
<p>dynamic programming with exponential states</p>
</td>
<td width="476">
<p>hash_map (to store the states)</p>
</td>
</tr>
<tr>
<td width="176">
<p>11</p>
</td>
<td width="143">
<p>n!</p>
</td>
<td width="363">
<p>brute force, backtracking, next_permutation</p>
</td>
<td width="476">&nbsp;</td>
</tr>
<tr>
<td width="176">
<p>8</p>
</td>
<td width="143">
<p>n<sup>n</sup></p>
</td>
<td width="363">
<p>brute force, cartesian product</p>
</td>
<td width="476">&nbsp;</td>
</tr>
</tbody>
</table>