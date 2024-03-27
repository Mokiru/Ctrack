# Splay树

## 定义

Splay树，或伸展树，是一种平衡二叉查找树，它通过Splay/伸展操作不断将某个节点旋转到根节点，使得整棵树仍然满足二叉查找树的性质，能够在均摊$O(logN)$时间内完成插入、查找和删除操作，并且保持平衡而不至于退化为链。

## 结构

### 二叉查找树的性质

Splay树是一棵二叉搜索树，查找某个值时满足性质：左子树任意节点的值<根节点的值<右子树任意节点的值。

### 节点维护信息

|rt|tot|fa[i]|ch[i][0/1]|val[i]|cnt[i]|sz[i]|
|--|--|--|--|--|--|--|
|根节点编号|节点个数|父亲|左右儿子编号|节点权值|权值出现次数|子树大小|

## 操作

### 基本操作

- `maintain(x)`：在改变节点位置后，将节点$x$的`size`更新。
- `get(x)`：判断节点$x$是父亲节点的左儿子还是右儿子。
- `clear(x)`：销毁节点$x$。

```c++
void maintain(int x) {
    sz[x] = sz[ch[x][0]] + sz[ch[x][1]] + cnt[x];
}

bool get(int x) {
    return x == ch[fa[x]][1]; // 1 右 0 左
}

void clear(int x) {
    ch[x][0] = ch[x][1] = fa[x] = val[x] = sz[x] = cnt[x] = 0;
}
```

### 旋转操作

为了使Splay保持平衡而进行旋转操作，旋转的本质是将某个节点上移一个位置。

旋转需要保证：
- 整棵Splay的中序遍历不变（不能破坏二叉查找树的性质）。
- 受影响的节点维护的信息依然正确有效。
- `root`必须指向旋转后的根节点。

在Splay中旋转分左旋和右旋。

![alt text](image-5.png)

过程

具体分析旋转步骤（假设需要旋转的节点为$x$，其父亲为$y$，以右旋为例）
1. 将$y$的左儿子指向$x$的右儿子，且$x$的右儿子（如果$x$有右儿子的话）的父亲指向$y$;`ch[y][0]=ch[x][1];fa[ch[x][1]]=y;`
2. 将$x$的右二子指向$y$，且$y$的父亲指向$x$;`ch[x][chk^1]=y;fa[y]=x;`
3. 如果原来的$y$还有父亲$z$,那么把$z$的某个儿子（原来$y$所在的儿子位置）指向$x$，且$x$的父亲指向$z$。`fa[x]=z;if(z) ch[z][y==ch[z][1]]=x;`

```c++
void rotate(int x) { // 以x的父亲节点为根节点旋转
    int y = fa[x], z = fa[y], chk = get(x);

    //1
    ch[y][chk] = ch[x][chk ^ 1]; 
    if (ch[x][chk ^ 1]) fa[ch[x][chk ^ 1]] = y;
    //2
    ch[x][chk ^ 1] = y;
    fa[y] = x;
    //3
    fa[x] = z;
    if (z) ch[z][y == ch[z][1]] = x;
    maintain(x);
    maintain(y);
}
```

### Splay操作

Splay操作规定：每访问一个节点$x$后都要强制将其旋转到根节点。

Splay操作即对$x$做一系列splay步骤，每次对$x$做一次splay步骤，$x$到根节点的距离就会更近，定义$p$为$x$的父节点，Splay步骤有三种，具体分为六种情况：

1. zig：在$p$是根节点时操作，Splay树会根据$x$和$p$间的边旋转，zig存在是用于处理奇偶校验问题，仅当$x$在splay操作开始时具有奇数深度时作为splay操作的最后一步执行。
![alt text](image-6.png)
即直接将$x$左旋或右旋。
![alt text](image-7.png)
![alt text](image-8.png)
2. 


