---
layout: post
title: blog_01:test_01_2023-01-26
date: 2023-01-26
categories: blog
tags: [标签一,标签二]
description: 搭建完成
---

blog搭建完成的第一天。

以下是test字符：

#原文链接 https://zhuanlan.zhihu.com/p/678823367
# 算法学习笔记：树形DP与拓扑排序

## 前言：

最近做了一道不错的基环树相关的题，顺带复习了一下树形DP。

//然后搁置了一个多月才写，真是忙炸了...

本来没打算写拓扑排序的，但今天整理笔记的时候无意中发现一道非常典的题目似乎可以用树形dp+拓扑排序（而不是dfs/bfs的传统方式）解决，遂写。

（后来发现早就有人写过了...原来只是因为我太菜了）

至于基环树的那道题目，把这个月的进度补完之后会考虑写一下。

（一个月的时间没写过题，现在敲代码都恍恍惚惚的，太堕落了）

## 参考文章：

[算法学习笔记(53): 拓扑排序 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/260112913)

[拓扑排序 学习笔记 - dbxxx - 博客园 (cnblogs.com)](https://www.cnblogs.com/crab-in-the-northeast/p/topological-sort.html)

[树形dp part1 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/658792779)

[算法学习笔记：从图论入门到入坟（1）图的储存和遍历 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/666643604)

## 正文：

### （一）树形DP

什么是树形DP？

顾名思义，即在树上进行的 DP。由于树固有的递归性质，树形 DP 一般都是递归进行的

典型的树形DP做法通常是一个二维数组进行DFS。

我们以一道经典例题来切入：[P1352 没有上司的舞会](https://www.luogu.com.cn/problem/P1352)

#### 题面如下：

某大学有 $n$ 个职员，编号为 $1\ldots n$。

他们之间有从属关系，也就是说他们的关系就像一棵以校长为根的树，父结点就是子结点的直接上司。

现在有个周年庆宴会，宴会每邀请来一个职员都会增加一定的快乐指数 $r_i$，但是呢，如果某个职员的直接上司来参加舞会了，那么这个职员就无论如何也不肯来参加舞会了。

所以，请你编程计算，邀请哪些职员可以使快乐指数最大，求最大的快乐指数。

#### 思路切入：

看了一下题目之后，我们发现似乎是一个规划问题：某个节点去或者不去都会对前后造成影响，无法使用简单的线性算法解决，于是我们考虑能否将其转化为动态规划问题。事实上这很简单——我们只需要开一个二维数组，把选择节点和不选择节点的情况都记录下来即可。

那么我们此时发现，似乎只需要最简单的递归就能解决这个问题——假设二维数组`f[u][1]`表示u参加舞会的最大快乐值，`f[u][0]`表示u不参加舞会的最大快乐值，那么假设**总根节点**（即顶头上司）为`u=root`，那么显然，最终得到的最大快乐值`ans=max(f[root][1]+f[root][0])`。那么只需要简单递归就可以了。

**DFS：**

这里我使用了邻接表存图，“上司”作为根节点，对应的直属下级即是其分支。

（不了解邻接表的可以看我以前的文章：[算法学习笔记：从图论入门到入坟（1）图的储存和遍历 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/666643604)）

由于题目中说“给出的关系一定是一棵树”，也就是说有着唯一的头节点（总根节点，即顶头上司）root。

那么我们只需要在读入的时候将所有子节点标记，剩余的没有被标记的那个即是root。

dfs函数应该怎么写呢？

很简单，从root开始进行递归即可。先将`f[x][1]`初始化为`happy[x]`，`f[x][0]`初始化为0，然后向下递归逐级加上前面各个分支的max即可——如果是`f[x][1]`的话，意味着x的下属不能参加，那么只需：`f[x][1]+=f[pre][0]`；如果是`f[x][0]`的话，下属可以来也可以不来，选最大值即可：`f[x][0]+=max(f[pre][0],f[pre][1])`。

DFS核心代码：

```c++
for(int i=0;i<G[x].size();i++){
        int pre=G[x][i];
        dfs(pre);
        f[x][0]+=max(f[pre][0],f[pre][1]);
        f[x][1]+=f[pre][0];
    }
```

完整源码如下：

```c++
#include<iostream>
#include<vector>
using namespace std;

int f[6003][2];
int happy[6003];
vector<int>G[6003];//邻接表,G[i][j]表示以i为顶点的第j条边
bool vis[6003];

void dfs(int x){
    f[x][0]=0;
    f[x][1]=happy[x];
    for(int i=0;i<G[x].size();i++){
        int pre=G[x][i];
        dfs(pre);
        f[x][0]+=max(f[pre][0],f[pre][1]);
        f[x][1]+=f[pre][0];
    }

}

int main(){
    int n;
    cin>>n;
    for(int i=1;i<=n;i++){
        cin>>happy[i];
    }
    int l,k;
    for(int i=1;i<=n-1;i++){
        cin>>l>>k;
        G[k].push_back(l);
        vis[l]=1;
    }
    int root;
    for(int i=1;i<=n;i++){//找只作为根节点的节点,即顶层
        if(!vis[i]){
            root=i;
            break;
        }
    }
    dfs(root);
    cout<<max(f[root][0],f[root][1])<<endl;
    return 0;
}
```



dfs的方法学会了，接下来我们来看看bfs——这种方法在解决树退化成链状的问题时相当有用（当数据量较大时很容易爆栈）。

**BFS：**

BFS写起来要稍微复杂一些。其实观察源码后我们可以知道，这种方法其实相当于**直接自底向上模拟了dfs**（类似于手动栈）。传统意义上的bfs是先入根节点再入子节点，但由于我们要自底向上模拟，因此要先入子节点再入根节点。

原理比较简单，不多赘述。

完整源码如下：

```c++
#include<iostream>
#include<vector>
#include<queue>
using namespace std;

int t=0;
int que[6003],happy[6003];//que记录队列中的元素
bool vis[6003],fa[6003];
int f[6003][2];
vector<int>G[6003];
queue<int> q;

void bfs(int s)//普通的BFS
{
    q.push(s);
    vis[s]=1;
    que[++t]=s;
    while(!q.empty())
    {
        int u=q.front();
        q.pop();
        int x=G[u].size();
        for(int i=0;i<x;i++)
        {
            if(!vis[G[u][i]])
            {
                vis[G[u][i]]=1;
                q.push(G[u][i]);
                que[++t]=G[u][i];
            }
        }
    }
    return ;
}
int main()
{
    int n;
    cin>>n;
    for(int i=1;i<=n;i++){
        cin>>happy[i];
    }
    int l,k;
    for(int i=1;i<=n-1;i++){
        cin>>l>>k;
        fa[l]=1;
        G[k].push_back(l);
    }
    int root;
    for(int i=1;i<=n;i++){
        if(!fa[i]){
            root=i; 
            break;
        }
    }
    //cout<<s<<endl;
    bfs(root);
    for(int i=t;i>0;i--){//先入子节点再入根节点，故倒着来取队列中的元素。
        int u=que[i];
        for(int j=0;j<G[u].size();j++)
        {
            int v=G[u][j];
            f[u][0]+=max(f[v][0],f[v][1]);
            f[u][1]=max(f[u][1],f[u][1]+f[v][0]);
        }
        f[u][1]+=happy[u];
    }
    cout<<max(f[root][0],f[root][1])<<endl;
}
```





### （二）拓扑排序

#### （1）什么是拓扑排序？

单看名字似乎有点抽象，不妨让我们看看他的定义：

 对于一个DAG（有向无环图）$G$，将 $G$ 中所有顶点排序为一个线性序列，使得图中任意一对顶点 $u$ 和 $v$，若 $u$ 和 $v$ 之间存在一条从 $u$ 指向 $v$ 的边，那么 $u$ 在线性序列中一定在 $v$ 前。

以下图为例：

![Image](https://pic4.zhimg.com/80/v2-38c97648e07b47c1d658c2f460c156d1.png)


假设1,2,3,4都是需要完成的工作，且如果某个工作有前置工作，那么必须完成前置工作才能做该工作。

在这个例子中，有向图边的方向就可以表示前置关系：比如1是2，3的前置，2，3又是4的前置，那么似乎我们只能按照1，2，3，4和1，3，2，4的顺序完成工作。这两个顺序就是我们所说的拓扑序。

#### （2）拓扑排序有什么作用？


#### （3）如何实现拓扑排序？
**toposort（DFS）：**

前置准备：
- 首先定义一个标记数组 flag。
- 初始化 $flag_i = 0$
- 循环遍历 $u = 1 \rightarrow n$，**当 $flag_u = 0$ 时，进行dfs**。dfs此处是一个bool函数，**如果返回true则代表运行正常，如果返回false代表发现了环或无向边。那么如果此时的dfs函数返回一个false值，直接返回，无拓扑序。** 至于dfs怎么判环，会在下方的dfs函数处理步骤给出。

接下来是dfs函数处理步骤：

- 对于一个来自参数的节点u，**先令 $flag_u \leftarrow -1$，** 然后遍历其出边。**如果发现$u \rightarrow v \in G$，且 $flag_v = -1$，说明有环，返回false。**然后**如果 $flag_v = 0$（$flag_v = 1$ 代表此点安全，不必再处理），我们就进行dfs(v)的操作。** 如果递归的dfs(v)返回了false，**本体也返回false。**
- 如果没有返回false，说明当前节点处理正常。令 $flag_u \leftarrow 1$，**将 u 节点插入拓扑序，返回true**。

核心代码：（来自[拓扑排序 学习笔记 - dbxxx - 博客园 (cnblogs.com)](https://www.cnblogs.com/crab-in-the-northeast/p/topological-sort.html)，因为和我的码风很像所以直接拿来当板子用了）
```c++
int flag[maxn];
vector <int> topo;
vector <int> G[maxn];
int n, m;

bool dfs(int u) {
    flag[u] = -1;
    for (int i = 0; i < G[u].size(); ++i) {
        int v = G[u][i];
        if (flag[v] == -1) return false;
        else if (flag[v] == 0) {
            if (!dfs(v)) return false;
        }
    }
    flag[u] = 1;
    topo.push_back(u);
    return true;
}

bool toposort() {
    topo.clear();
    for (int i = 1; i <= n; ++i) flag[i] = 0;
    for (int i = 1; i <= n; ++i) {
        if (flag[i] == 0) {
            if (!dfs(i)) return false;
        }
    }
    reverse(topo.begin(), topo.end());
    return true;
}
```
**Kahn算法：**
类似于BFS版的toposort

流程如下:

- 将**入度为0的点**组成一个集合 $S$
- 从 $S$ 中取出一个顶点 $u$，**插入拓扑序列**。
- 遍历顶点 $u$ 的所有出边，并**全部删除**，如果删除这条边后对方的点入度为 0，也就是没删前，$u \rightarrow v$ 这条边已经是$v$ 的最后一条入边，**那么就把  $v$ 插入  $S$**。
- 重复执行上两个操作，直到 $S = \varnothing$。此时检查拓扑序列是否正好有 n 个节点，不多不少。**如果拓扑序列中的节点数比 n 少，说明 $G$ 非DAG，无拓扑序，返回false。如果拓扑序列中恰好有 n 个节点，说明 $G$ 是DAG，返回拓扑序列。**

也就是说，Kahn算法的核心就是**维护一个入度为0的顶点**。

核心代码：


```c++
int ind[maxn];
bool toposort() {
    topo.clear();
    queue <int> q;
    for (int i = 1; i <= n; ++i) {
        if (ind[i] == 0) q.push(i);
    }

    while (!q.empty()) {
        int u = q.front();
        topo.push_back(u);
        q.pop();
        for (int i = 0; i < G[u].size(); ++i) {
            int v = G[u][i];
            --ind[v];
            if (ind[v] == 0) q.push(v);
        }
    }

    if (topo.size() == n) return true;
    return false;
}
```
值得一提的是，Kahn算法和dfs算法的时间复杂度都为 $\operatorname{O}(E+V)$，需要注意。

例题1：[Problem - C - Codeforces](https://codeforces.com/contest/510/problem/C)

（来自[算法学习笔记(53): 拓扑排序 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/260112913)）

简单地说，就是给出一张名字的列表，要找到一张字母表使得这张人名的列表是按字典序排列的。

通过对相邻的两个名字间进行比较，可以得到 $n-1$ 个关系（如样例中 $r<s,s<a$），它们可以构成一张图，而最后线性的字母表也必须要满足这些关系。很明显，如果这张图存在环，必然是无解的；而如果无环，那么拓扑排序即可得到结果。


例题2：[P1352 没有上司的舞会](https://www.luogu.com.cn/problem/P1352)，即我们讲树形dp时的例题。

这道题如何从拓扑排序入手呢？考虑到本题进行dp时是自底向上的——即子节点到根节点，而拓扑排序是从根节点开始到子节点，两者的特质似乎正好相反，一时之间想不到有什么联系。

这意味着你忽略了一个很重要的思维：**反向建边** 。

思路打开后，那么只需要反向建边，就能使原本的根节点成为子节点，从而可以使用拓扑排序，边求拓扑排序边更新答案求解。有一种豁然开朗的感觉。












