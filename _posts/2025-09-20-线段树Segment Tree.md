---
layout: post
title: 线段树(Segment Tree) 
---

{{ page.title }}
================
<p class="meta">20 Sep 2025</p>

**1.简介**

- 是用于维护区间信息的数据结构。
- 可进行动态的区间查询（单点修改，区间修改，区间查询）。
- 可以用树状数组做的题，都可以用线段树；可以用线段树做的题，不一定能用树状数组。

**2.原理及实现**

线段树是一种二叉树，除去最底层是一棵完全二叉树。每一个节点代表一段区间，最底层的节点代表单个元素。父节点通过子节点的运算得来，如相加、求最大值等。

<figure style="text-align: center;">
    <img src="/images/segmenttree/p1.jpg" alt="p1" style="display: block; margin: 0 auto;">
    <figcaption style="text-align: center; margin-top: -20px;">图1</figcaption> 

</figure>

如图1所示，线段树记录了一个为[10, 11, 12, 13, 14]的数组，父节点由子节点求和得到。

不难发现，一个二叉树中，对于第n个节点，其左子节点编号为2n，右子节点编号为 2n+1。

**2.1 建树**

```c
void build(int p, int pl, int pr) { // 当前节点标号，以及其对应区间
        if(pl == pr) {
                    tree[p] = a[pl]; // 抵达最底层，赋值
                    return;
        }
        int mid = (pl + pr) / 2;
        build(2*p, pl, mid);
        build(2*p+1, mid+1, pr);
        tree[p] = tree[2*p] + tree[2*p+1]; // 重要赋值语句
}
```

采用递归的方法，从根节点开始，函数每次都向下一级传入当前节点编号、当前节点对应的区间。因此以如下形式调用。

```c
build(1, 1, n);
```

当递归到二叉树最底层时停止递归，具体的判断条件为当前节点对应的区间长度为1（左端点=右端点）。触发此条件时，还把最底层的节点赋值。

若还未到树的最底层，那么就开始向两个子节点递归，传入两个子节点对应的编号和区间。

巧妙的地方是，在两行递归代码的下面是给当前节点赋值的语句。因为它上面的递归回到当前这一层时，当前的节点的两个子节点已经赋好值了，所以此时可以把两个子节点的值拿来计算。

**2.2 区间询问**

```c
// p是当前节点,[pl,pr]是对应区间,[l,r]为目标区间
int query(int p, int pl, int pr, int l, int r) {
        if(pl >= l && pr <= r) return tree[p];
        int mid = (pl + pr) / 2;	
        int sum = 0;
        if(mid >= l) sum += query(2*p, pl, mid, l, r);
        if(mid+1 <= r) sum += query(2*p+1, mid+1, pr, l, r);
        return sum; 
}
```

询问同样使用递归，并且从根节点开始向下搜索。因此以如下形式调用。

```c
query(1, 1, n, l, r);
```

如果当前节点的区间在目标区间之内，那么就不用再向下搜索了，可以直接返回当前节点对应的值。这就是树状数组区间查询节约时间的地方。

接下来递归即可，但要判断当前区间与目标区间的位置关系，两个if语句起到了这个作用。mid是当前区间的正中间。

若mid在目标区间左端点L的左边，那么证明区间[pl, mid]包含了目标区间的一部分，故需要搜索，如图2所示。

<figure style="text-align: center;">
    <img src="/images/segmenttree/p2.jpg" alt="p2" style="display: block; margin: 0 auto;">
    <figcaption style="text-align: center; margin-top: -10px;">图2</figcaption> 

</figure>

若mid+1在目标区间右端点R的左边，那么证明区间[mid+1, pr]包含了目标区间的一部分，故需要搜索，如图3所示。

<figure style="text-align: center;">
    <img src="/images/segmenttree/p3.jpg" alt="p3" style="display: block; margin: 0 auto;">
    <figcaption style="text-align: center; margin-top: -10px;">图3</figcaption> 

</figure>

**2.3 单点修改**

```c
// p为当前节点，[pl,pr]为当前对应区间，目标是将第x个元素加上k
void update(int p, int pl, int pr, int x, int k) {
        if(pl == pr) {
                tree[p] += k;
                return ;
        }
        int mid = (pl + pr) / 2;
        if(x <= mid) update(p*2, pl, mid, x, k);
        if(x >= mid+1) update(p*2+1, mid+1, pr, x, k);
        tree[p] = tree[p*2] + tree[p*2+1];
}
```

本质上单点修改和建树是一模一样的。

**2.4 区间修改与懒惰标记**

若用利用单点修改来进行区间修改，则需将区间内每个元素遍历一遍，那么时间复杂度将会很高。

所以引入**懒惰标记**。懒惰标记就是延迟对节点信息的更改。对于一段长度为n、需要修改的区间，通过线段树的特性，可以用少于n个的t[i]来表示这段区间。对于要修改的信息，只把它保存在每个t[i]之上，而不更新到每个t[i]的子节点上。直到下一次访问到标记过的节点，再进行实质性的修改。

在树上搜索的同时，完成懒惰标记的实现。

1. 从根节点开始搜索。

2. 如果当前区间为目标区间的某个子集时，这意味着区间修改对当前节点有效，那么修改当前节点的值（p表示当前节点编号）t[p] += (pr - pl + 1) * k。同时更新懒惰标记tag[p] += k。

3. 如上文所说，*直到下一次访问到标记了的节点，再进行实质性的修改*

   每当访问到一个节点时，就要判断是否有标记。如果有标记的话，就把标记的信息传递到它的两个子节点上，并把当前节点的标记删去。

4. 最后再对比区间关系，进行递归。

```c
void update(int p, int pl, int pr, int l, int r, int k) {	
        if(pl >= l && pr <= r) { // 当前区间为目标区间的子集
                tag[p] += k;
                tree[p] += (pr - pl + 1) * k;
                return ;
        }
        int mid = (pl + pr) / 2;
        if(tag[p]) {
                tag[2*p] += tag[p];
                tag[2*p+1] += tag[p]; // 把标记传递给两个子节点
                tree[2*p] += (mid - pl + 1) * tag[p];
                tree[2*p+1] += (pr - mid) * tag[p]; // 修改两个子节点的值
                tag[p] = 0;
        }
        if(l <= mid) update(p*2, pl, mid, l, r, k);
        if(r >= mid+1) update(p*2+1, mid+1, pr, l, r, k); // 递归
        tree[p] = tree[p*2] + tree[p*2+1];
}
```

例如，对于一个 10,11,12,13,14 的数组，在[3,5]的区间内，每个数字都加上5，如图4所示。

<figure style="text-align: center;">
    <img src="/images/segmenttree/p4.jpg" alt="p4" style="display: block; margin: 0 auto;">
    <figcaption style="text-align: center; margin-top: -20px;">图4</figcaption> 
</figure>

1. 从根节点开始搜索，找到了[3,3]和[4,5]，分别对应第5和第3个节点。给这两个节点打上标记，并修改值。
2. 找到之后递归函数还返回前几层，从而更新对应的节点值（如1号节点，2号节点，3号节点的值都会被更新，通过将两个子节点求和）。

修改完成后，线段树如图5所示：

<figure style="text-align: center;">
    <img src="/images/segmenttree/p5.jpg" alt="p5" style="display: block; margin: 0 auto;">
    <figcaption style="text-align: center; margin-top: -30px;">图5</figcaption> 
</figure>

延迟修改节点信息是如何体现的？比如，当前的树状数组，第6、第7个节点的信息还未更新，而他们是属于[3,5]区间内的。他们在下一次被访问到的时候会被更新。

比如：查询[4,4]区间。

1. 在搜索到第三个节点时（区间[4,5]），发现该节点有标记。
2. 标记传递给两个子节点，并修改两个子节点的值。
3. 将当前节点的标记删除。

在全知的角度来看，把标记传递给第6、7节点没有用处，因为它们已经没有子节点了。但是在搜索的过程中，无法知晓是否有子节点，所以还是需要将标记传递给第6、7节点，如图6所示。

<figure style="text-align: center;">
    <img src="/images/segmenttree/p6.jpg" alt="p6" style="display: block; margin: 0 auto;">
    <figcaption style="text-align: center; margin-top: -40px;">图6</figcaption> 
</figure>
若使用懒惰标记，询问的代码也需要进行修改。

```c
int query(int p, int pl, int pr, int l, int r) {
        if(pl >= l && pr <= r) return tree[p]; // 当前区间为目标区间的子集
        int mid = (pl + pr) / 2;
        if(tag[p]) {
                tag[2*p] += tag[p];
                tag[2*p+1] += tag[p]; // 传递标记给两个子节点
                tree[2*p] += (mid - pl + 1) * tag[p];
                tree[2*p+1] += (pr - mid) * tag[p]; // 修改两个子节点的值
                tag[p] = 0;
        }
        int sum = 0;
        if(mid >= l) sum += query(2*p, pl, mid, l, r);
        if(mid+1 <= r) sum += query(2*p+1, mid+1, pr, l, r); // 递归
        return sum; 
}
```

**3.例题**

Example 1 [洛谷P2357](https://www.luogu.com.cn/problem/P2357#submit)

无难度，一道板子题。区间修改和区间询问，需要用到懒惰标记。

对于主墓碑（单点修改），看作[1,1]的区间来调用就行了。

Example 2 [洛谷P2574](https://www.luogu.com.cn/problem/P2574)

对于一个长度为n只包括0或1的数列：修改操作为将0变为1、将1变为0，查询操作为找到目标区间内1的数量。

此题中，t数组表示对应区间的和，即1的个数。

修改操作的实现：t[x]表示区间内1的数量。若将该区间反转，则t[x] = len - t[x]。这应该很好理解。

tag数组的使用：若一个区间反转两次则相当于没有反转。令tag[x] = 1表示需要反转，tag[x] = 0表示不需要。每次进行修改时，令tag[x] = !tag[x]。

最后一点，输入是要注意输入的是一个字符串，需要处理。

```c
//修改和查询的代码
void update_tag() {
  	tag[p*2] = !tag[p*2];
	tag[p*2+1] = !tag[p*2+1]; // 不同之处
	tree[2*p] = (mid - pl + 1) - tree[2*p];
	tree[2*p+1] = (pr - mid) - tree[2*p+1]; // 不同之处
	tag[p] = 0;
}
void update(int p, int pl, int pr, int l, int r) {
	if(pl >= l && pr <= r) {
		tag[p] = !tag[p]; // 不同之处
		tree[p] = (pr - pl + 1) - tree[p]; // 不同之处
		return ;
	}
	int mid = (pl + pr) / 2;
	if(tag[p]) update_tag();
	recursion1();
}

int query(int p, int pl, int pr, int l, int r) {
	if(pl >= l && pr <= r) return tree[p];
	if(tag[p]) update_tag();
	recursion2();
}
```

Example 3 [洛谷P1198](https://www.luogu.com.cn/problem/P1198)

简单变式，将求和改为求最大值即可。
