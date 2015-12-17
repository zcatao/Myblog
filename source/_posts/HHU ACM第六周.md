title: ACM训练第六周-万恶的最短路
date: 2015-10-01 18:31:24
tags:
---


## 1001 Problem B 


>POJ 3615 [Cow Hurdles](http://poj.org/problem?id=3615)

### 题意翻译
&#160; &#160; &#160; &#160;Farmer John 想让奶牛准备乡村弹跳比赛,所以Bessie 和 gang 准备练习跳过障碍。然而他们累了，所以他们想尽可能的用更少的能量去跳过这些障碍。
&#160; &#160; &#160; &#160;很显然,让奶牛去跳过一连串矮小的障碍并不是很困难，但是一个高的障碍可能就会有很大的压力。因此，奶牛只关心他们要跳过的最高的高度。
&#160; &#160; &#160; &#160;奶牛的练习房间有`N(1<=N<=300)`站，为方便起见，将他们标记为1……N。在M`(1<=M<=25,000)`种方式中每一种都连接着一对站点；这些路径同样被标记为1……M。路径*i*从S*i*站连接到E*i*站，并且包含一个确定高度H*i*`(1<=Hi<=1,000,000)`。奶牛必须要跳过他们走过任何他们走过道路上的障碍。
&#160; &#160; &#160; &#160;奶牛们有**T**`(1<= T <= 40,000)` 个任务去完成。任务*i*包含两个目的数字，A*i*和B*i*`(1<= Ai <= N ;1<= Bi <= N)`,这意味着奶牛需要从Ai到Bi（通过翻越路线上的一个或多个路径）。奶牛们想去选择一个在他们从A到B的时候需要翻越的最高障碍物的高度最小的路径。你的任务就是写一个决策出最高障碍物高度最小的路径并且输出这个高度。


`（麻痹，终于翻译完了）`
### 输入
- **第一行**：有三个分离的整数，*N* , *M* , *T*;
- **第2……M+1行**：每行包含三个分离的整数，*Si* , *Ei* , *Hi*;
- **第M+1……M+T+1**：每一行都包含两个整数，*Ai* , *Bi*;

### 输出
- 第*i*包含第i个任务的结果，输出这条路经的最高的高度，如果不能通过这两个车站那么输出**-1**.

### Sample Input
5 6 3
1 2 12
3 2 8
1 3 5
2 5 3
3 4 4
2 4 8
3 4
1 2
5 1

### Sample Output
4
8
-1
### 思路
 这个题没有什么好说的，基本就是裸的Floyd，注意这个题点到点之间的权值依旧是有向的。
 还有注意用scanf 白白TLE掉两发
 
```cpp

#include <iostream>
#include "stdio.h"
using namespace std;
#define intmax 0x7fffffff 
#define maxn 305
#define max(x,y)    ((x) > (y) ? (x) : (y))
#define min(x,y)	((x) < (y) ? (x) : (y))
int Distance[maxn][maxn];
int N,M,T;
void Out()
{
	for(int i = 1; i <= N; i++)
	{
		for(int j = 1; j <= N ; j ++)
			if(Distance[i][j] == intmax)
				cout << -1 << '\t';
			else 
				cout << Distance[i][j] << '\t';
		cout << endl;
	}
}
void floyd()
{
	for(int i = 1; i <= N; i ++)
		for(int j = 1; j <= N; j ++)
			for(int k = 1; k <= N; k++)
				Distance[j][k] = 
                min(Distance[j][k],max(Distance[j][i],Distance[i][k]));
}
int main()
{
	
		cin >> N >> M >> T;
		for(int i = 1; i <= N; i++)
			for(int j = 1; j <= N ; j ++)
				Distance[i][j] = intmax;
		for(int i = 1; i <= M; i++)
		{
			int x,y,temp;
			scanf("%d %d %d", &x,&y,&temp);
			Distance[x][y] = temp;
			// buyao renwei shi  shuangxiang de !!!
			//Distance[y][x] = temp;
		}
		//Out();
		floyd();
		int Start,End;
		for (int i = 0; i < T; ++i)
		{
			scanf("%d %d",&Start,&End);
	
			if(Distance[Start][End] == intmax)
				cout << -1 << endl;
			else
				cout << Distance[Start][End] <<endl;
		}
}
```


## 1002 Problem C 
>POJ 1847 [Tram](http://poj.org/problem?id=1847)
wakakkaka