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
void rotate(int x) { // 以x的父亲节点为根节点旋转 x 是父节点左儿子就右旋，否则左旋
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
2. zig-zig：在$p$不是根节点且$x$和$p$都是右侧子节点或都是左侧子节点时操作，下方例图显示了$x$和$p$都是左侧子节点时的情况。Splay树首先按照连接$p$与其父节点$g$边旋转，然后按照连接$x$和$p$的边旋转。
![alt text](image-9.png)
即首先将$gp$左旋或右旋，然后将$x$左旋或右旋。
![alt text](image-10.png)
![alt text](image-11.png)
3. zig-zag：在$p$不是根节点且$x$和$p$一个是右侧子节点一个是左侧子节点时操作。Splay树首先按$p$和$x$之间的边旋转，然后按$x$和$g$新生成的结果边旋转。
![alt text](image-12.png)
即将$x$先左旋再右旋，或先右旋再左旋。
![alt text](image-13.png)
![alt text](image-14.png)

```c++
void splay(int x) {
    for (int f = fa[x]; f = fa[x], f; rotate(x)) {
        if (fa[f]) rotate(get(x) == get(f) ? f : x);
    }
    rt = x;
}
```

对于$n$个节点的splay树，做一次splay操作的均摊复杂度为$O(logn)$。因此基于splay的插入、查询、删除等操作的时间复杂度也为均摊$O(logn)$。

### 插入操作

过程

插入操作是一个比较复杂的过程，具体步骤如下（假设插入的值为$k$）：
- 如果树空了，则直接插入根并退出。
- 如果当前节点的权值等于$k$则增加当前节点的大小并更新节点和父亲的信息，将当前节点进行Splay操作。
- 否则按照二叉查找树的性质向下找，找到空节点就插入即可（然后Splay操作）。

```c++
void ins(int k) {
    if (!rt) { //树为空
        val[++tot] = k;
        cnt[tot]++;
        rt = tot;
        maintain(rt);
        return;
    }
    int cur = rt, f = 0; //从根节点开始找
    while (1) {
        if (val[cur] == k) {
            cnt[cur]++;
            maintain(cur);
            maintain(f);
            splay(cur);
            break;
        }
        f = cur;
        cur = ch[cur][val[cur] < k]; // 判断 走左还是右
        if (!cur) {
            val[++tot] = k;
            cnt[tot]++;
            fa[tot] = f;
            ch[f][val[f] < k] = tot;
            maintain(tot);
            maintain(f);
            splay(tot);
            break;
        }
    }
}
```

### 查询x的排名

过程

根据二叉查找树的定义和性质，显然可以按照以下步骤查询$x$的排名：
- 如果$x$比当前节点的权值小，向其左子树查找。
- 如果$x$比当前节点的权值大，将答案加上左子树($size$)和当前节点($cnt$)的大小，向其右子树查找。
- 如果$x$与当前节点的权值相同，将答案加$1$并返回。
注意最后需要进行Splay操作。
```c++
int rk(int k) {
    int res = 0, cur = rt;
    while (1) {
        if (k < val[cur]) {
            cur = ch[cur][0];
        } else { // k >= val[cur]
            res += sz[ch[cur][0]];
            if (k == val[cur]) {
                splay(cur);
                return res + 1;
            }
            res += cnt[cur];
            cur = ch[cur][1];
        }
    }
}
```

### 查询排名x的数

过程

设$k$为剩余排名，具体步骤如下：
- 如果左子树非空且剩余排名$k$不大于左子树的大小$size$，那么向左子树查找。
- 否则将$k$减去左子树和根的大小，如果此时$k$的值小于等于$0$，则返回根节点的权值，否则继续向右子树查找。

```c++
int kth(int k) {
    int cur = rt;
    while (1) {
        if (ch[cur][0] && k <= sz[ch[cur][0]]) {
            cur = ch[cur][0];
        } else {
            k -= cnt[cur] + sz[ch[cur][0]];
            if (k <= 0) {
                splay(cur);
                return val[cur];
            }
            cur = ch[cur][1];
        }
    }
}
```






