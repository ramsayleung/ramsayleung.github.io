+++
title = "区块链的完整性校验方案: Merkle Tree"
date = 2024-10-12T18:22:00-07:00
lastmod = 2024-10-14T18:55:16-07:00
tags = ["blockchain"]
categories = ["blockchain"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 前言 {#前言}

最近通过 [Solidity-103](https://www.wtf.academy/docs/solidity-103) 课程在学习 Solidity, 看到[第36课](https://www.wtf.academy/docs/solidity-103/MerkleTree/) Merkle Tree 的时候着实头疼, 即使我已经了解 Merkle Tree 这数据结构，但是课程还是看得不明所以。 <br/>

所以写下这篇文章，梳理我所理解的 Merkle Tree 及其用途，既加深自己的理解，又践行了[费曼学习法](https://ramsayleung.github.io/zh/post/2022/feynman_technique/) <br/>


## <span class="section-num">2</span> 区块链与交易 {#区块链与交易}

关于区块链的资料有非常多，我也不赘述了. <br/>

简单理解，区块链是由一个一个区块构成的有序链表，每一个区块都记录了一系列交易，并且，每个区块都指向前一个区块，从而形成一个链条。区块链听起来很高级，其实就是个单链表。 <br/>

{{< figure src="/ox-hugo/blockchain.jpg" >}} <br/>

每个区块都会保存对应的交易信息，也会包含元数据信息在头部，包括前一个区块的 hash, 包含的交易数，merkle tree 的根节点 hash,时间戳等信息. <br/>
![](/ox-hugo/block_detail.jpg) <br/>

我们总说区块链是不可窜改，那么它究竟是怎么不可窜改的? <br/>

如果用户想要验证某个区块的某笔交易是否被窜改，他要怎么做？ <br/>
最简单的方式自然是把整个区块的交易都下载下来，平均每个区块有1M的数据，验证起来肯定很费时间. <br/>

是否有一个验证方案，可以使用很小的数据集就完成验证? <br/>

有的，那就是 Merkle Tree. <br/>


## <span class="section-num">3</span> Merkle Tree {#merkle-tree}

那什么是 Merkle Tree? <br/>

假如我们有8笔交易被包含在区块中, 每笔交易都可以通过 hash 函数计算出一个 hash 值: <br/>

{{< figure src="/ox-hugo/8transactions.jpg" >}} <br/>

哈希值也可以看做数据，所以可以把 `h1` 和 `h2` 拼起来， `h3` 和 `h4` 拼起来, 依此类推，再计算出哈希值 `b1` 和 `b2` <br/>

{{< figure src="/ox-hugo/merkle_tree_layer_two.jpg" >}} <br/>

递归计算下去，直到计算结果只有一个 hash 值，这个就是所谓的 merkle root, 而 h1-h8 就是所谓的 `leaf node`, 两者之间的就是 `non-leaf node`. <br/>

交易数量恰好是偶数能这么算，如果是奇数，那要怎么算呢？这个时侯，只需要把最后一个 hash 值复制一份，也能算出最终的 merkle root： <br/>

{{< figure src="/ox-hugo/odd_leaf_merkle_tree.jpg" >}} <br/>


## <span class="section-num">4</span> Merkle tree validation {#merkle-tree-validation}

现在有了 Merkle Tree, 如果我们要验证区块中的交易是否被修改，要怎么算呢？ <br/>

最简单粗暴的方式肯定是把区块所有的交易下载下来，从头重组整棵 Merkle Tree, 8笔交易计算起来还可以，如果是几千笔呢？几百万笔呢？甚至几亿笔交易呢？ <br/>

重组 Merkle Tree 的时间复杂度是 O(N), 如果是1亿笔交易，这意味着你要计算1亿次，太慢了。 <br/>

但是，如果我们利用 Merkle Tree 的特性，从数学的角度，我们只需要少量的Merkle Proof(你可以理解成需要提供的验证数据集), 就可以完成验证. <br/>

回到上文的 Merkle Tree, 假如我们要验证 `tx2` 是否被窜改，我们需要有 Merkle Tree Root 和 Merkle Proof: <br/>

{{< figure src="/ox-hugo/merkle_proof.jpg" >}} <br/>

假设现在我们有 `tx2` 的交易数据，我们只需要 Merkle Proof 提供3个hash 值(图中的绿色部分)，然后我们只计算4次（橙色部分），就会算出 Merkle Root Tree 的值，用来和区块头部的 Merkle root 值进行比对。 <br/>

通过 Merkle Proof 提供的数据集，我们就可以把下载8笔交易，计算15次hash，优化成只需3个 hash 值，以及计算4次hash，需要的 Merkle Proof 数量等于Merkle Tree的高度，而Merkle Tree就是一棵完美的平衡二叉树, 树的高度就是 `log₂(N)`, 所以时间复杂度从O(N)降低成O(logN). <br/>

这个比对似乎不明显，但是以1亿交易为例的话，log(1_000_000_000) ~= 27, 也就是只需要 Merkle Proof 提供27个 hash 值即可, 巨大的性能提升. <br/>


## <span class="section-num">5</span> 区块链的不可窜改性 {#区块链的不可窜改性}

通过Merkle tree root可以保证交易的不可窜改性，而区块 hash 又能保证区块头部的元数据不被窜改. <br/>

因为每个区块都有区块 hash, 区块hash是通过计算头部元数据信息计算出来的: <br/>

{{< figure src="/ox-hugo/block_hash.jpg" >}} <br/>

只要修改了其中一个元数据值，那么 block hash 就会发生变化，而区块链就是一个单链表，通过后一个区块通过 `prev_hash` 指向前一个区块，如果 block hash 发生变化，那么后一个区块就无法正确指向前一个区块了，这个链就断了. <br/>

如果一个恶意的攻击者修改了一个区块中的某个交易，那么Merkle Hash验证就不会通过。 <br/>

所以，他只能重新计算Merkle Hash，然后把区块头的Merkle Hash也修改了。 <br/>

这时，我们就会发现，这个区块本身的Block Hash就变了，所以，下一个区块指向它的链接就断掉了, 他就要把后续所有区块全部重新计算并且伪造出来，才能够修改整个区块链； <br/>

而要修改后续所有区块，这个攻击者必须掌握全网51%以上的算力才行。 <br/>

理论上可行，但是实操难度非常非常非常大. <br/>


## <span class="section-num">6</span> Merkle Tree 版本管理中的应用 {#merkle-tree-版本管理中的应用}

除去区块链，Merkle Tree还被应用于类似 Git 和 Mercurial 这样的版本管理系统中，以Git为例, 假如我们Git项目内有4个文件: <br/>

{{< figure src="/ox-hugo/git_merkle_tree.jpg" >}} <br/>

当你push 代码到远程分支或者从远程分支 pull 代码的时候，Git就计算你的Merkle Tree Root 的值, 比较远程分支的Merkle Tree Root和本地分支的Merkle Tree Root 是否相同: <br/>

如果相同，那就不用更新了；如果不同的话它就会检查左节点或者右节点，并且递归下去， <br/>
直到找到是哪些文件发生了修改，只通过网络传输修改部分的内容, 以提高传输效率. <br/>

不过Git实际用的是Merkle Tree的变体，并不是直接使用Merkle Tree. <br/>

除些之外, Merkle Tree 还在 Cassandra, DynamoDB 这样的NoSQL数据库中被用于检查不同节点数据的一致性, 细节可以看下这个 [Stackoverflow 问题](https://stackoverflow.com/questions/5486304/explain-merkle-trees-for-use-in-eventual-consistency)。 <br/>


## <span class="section-num">7</span> 参考 {#参考}

-   [Blockchain for Test Engineers: Merkle Trees](https://alexromanov.github.io/2022/06/19/bchain-test-7-merkle-tree/) <br/>
-   [Understanding Merkle Trees](https://medium.com/geekculture/understanding-merkle-trees-f48732772199) <br/>
-   [Explain Merkle Trees for use in Eventual Consistency](https://stackoverflow.com/questions/5486304/explain-merkle-trees-for-use-in-eventual-consistency) <br/>

