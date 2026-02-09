---
title: scaleOIJ
date: 2023-07-24
categories:
  - Paper
tags:
  - Streaming
---

# Scalable OIJ on Modern Multicore Processors in OpenMLDB

<!-- more -->

### 动机

● 该研究基于OpenMLDB平台，旨在提高现代多核服务器上进行OIJ的延迟和吞吐量。

● Online Interval Join（OIJ）是一种常见的基于时间数据库的区间连接方法，用于在数据流中连接具有重叠时间区间的元组。然而，现有的OIJ解决方案（key-OIJ）在处理大规模数据时存在性能瓶颈，因此需要进行优化。

● 本研究通过实验分析现有解决方案的设计空间和关键问题，并提出了一种新的解决方案——Scale-OIJ，以提高OIJ的性能。

● Dataset：Both real-world and synthetic

● Indicator: Throughput(mean value); Lateness(CDF); Unbalancedness(standard deviation of workloads); LCC miss(last-level cache miss)

### 简单对比

● 在文本中，作者提出了一种名为Scale-OIJ的新方法，用于实现可扩展的在线区间连接。与现有的OIJ解决方案Key-OIJ相比，Scale-OIJ具有以下区别：

● Scale-OIJ：
● - 使用SWMR(Single-Writer-Multiple-Reader) Time-Travel数据结构，可以在不锁定数据的情况下进行并发读写操作。
● - 使用动态平衡调度，可以在多个线程之间动态分配工作负载，以实现更好的负载均衡。
● - 使用增量窗口聚合，可以避免重复计算重叠窗口，从而提高计算效率。

● Key-OIJ：
● - 使用基于键的分区并行化策略，可以将输入元组并发地与相同键的缓冲区连接。
● - 为了处理潜在的乱序流到达，元组只能在一段时间后才能被删除。

### 传统方法的缺点—key-OIJ

● 1. 处理无序数据的成本高昂
● 2. 负载不平衡
● 3. 冗余计算

### 新方法— scale-OIJ

● 整体上，该设计遵循基于键的分区模型（Key-OIJ），但允许根据工作负载分布和相同分区的共享处理来动态重新分区数据。

● Time-Travel Data Structure

为了实现高效的窗口数据检索，文章设计了一种基于double-layered skip-list的数据结构。

时间复杂度：O(logNkey) + O(logNts)

SWMR Concurrency Property：具有相同键的元组可以由多个连接器（joiners）共同处理

● Dynamic Schedule

shared Processing：通过共享处理框架和虚拟团队的设计，连接器可以共享具有相同键的元组。

Dynamic Schedule：共享处理框架允许在不迁移数据的情况下动态重新分区数据。

● Incremental Online Interval Join

重叠窗口问题：在进行区间连接时，特别是当窗口较大时，存在邻近窗口可能重叠的高概率。

### 算法

##### Algorithm1:Search A Tuple

```plaintext
Input: <key, ts>
Output: the node containing the matched tuple
1 node ← HEAD
2 level ← HEIGHT
3 while true do
4 next = load_acquire (node[level].next)
5 if next == NULL || next.key > key then
6 if level <= 0 then
7 return node
8 level = level − 1
9 else if next.key == key then
10 return node
11 else
12 node = next
13 return node
```

skiplist.h中使用 FindEqual 方法来寻找 skiplist 中的一个 tuple：

```c++
Node<K, V>* FindEqual(const K& key) {
    Node<K, V>* node = head_;
    uint8_t level = GetMaxHeight() - 1;
    while (true) {
        Node<K, V>* next = node->GetNext(level);
        if (next == NULL || compare_(next->GetKey(), key) > 0) {
            if (level <= 0) {
                return node;
            }
            level--;
        } else {
            node = next;
        }
    }
}
```

##### Algorithm2: Put A New Tuple

```plaintext
Input: <key, ts, x>
Output: N.A.
// search the position to insert
1 node ← HEAD
2 level ← HEIGHT
3 while true do
4 next = node[level].next
5 if next == NULL || next.key >= key then
6 pre[level] = node
7 if level <= 0 then
8 break
9 level = level − 1
10 else
11 node = next
// insert into the skiplist
12 new_node = NewNode(x, random_height)
13 for i ← 0 to height do
14 store_relaxed(new_node[i].next, pre[i].next)
15 for i ← 0 to height do
16 store_release(pre[i].next, new_node)
```

##### Algorithm3: Dynamic Schedule

```plaintext
Input: Load distribution of all the keys, current key partition schedule S
Output: optimized key partition schedule
1 Snew = S
2 while true do
3 calculate the workload Wi for every joiner Ji according to Equation 3
4 select the maximum and minimum joiners:
  Jmax = arg maxi Wi
  Jmin = arg mini Wi
5 add all key partitions ∀pj ∈ Jmax → priority queue PQJmax
6 for pi ← PQJmax.top() do
7 replicate pi to Jmin in the new schedule Snew
8 if last_unbalancedness − unbalancedness > δ then
9 break
10 PQJmax.pop()
11 if Snew does not change then
12 break
13 ∀k |xk| = λ × |xk|
14 return the new schedule Snew
```

![Scale-OIJ架构](968b9ceba0c63a5cf96ee3ea0e8ae65.png)

### Dynamic schedule

Dynamic Schedule算法通过收集运行时的数据分布统计信息，可以推导出所有Joiner的工作负载分布情况，从而可以定期重新调度分区分配。

#### Shared Processing

Shared Processing框架使用了一种虚拟团队的概念，将具有相同键的元组分配给多个Joiner共同处理。

#### Virtual Team

Virtual Team是由具有相同键的元组随机分配给多个Joiner组成的。Joiner向虚拟团队中的所有成员公开读取(R)权限，而将写入(W)权限保留给自己。

#### Dynamic schedule

动态调度框架中重新调度的目标是减少Joiner之间工作负载调度的不平衡性。

![动态调度公式](30f8251ce7c6b4b55a27993e63183e2.png)

算法3的启发式解决方案的基本步骤是：

```plaintext
1）计算每个Joiner的工作负载，并选择具有最大工作负载Jmax和最小工作负载Jmin的Joiner（第3-4行）。
2）尝试将Jmax中具有最大工作负载的分区复制到Jmin中（第5-7行）。
3）如果不平衡度降低了一个阈值，就退出（第8-9行），并重复步骤1）-2）。
4）如果在迭代后新计划中没有变化，则停止探索（第11-12行）。
5）最后衰减统计数据（第13行）。
```
