题面可以在 [link](https://www.cometoj.com/contest/14/problem/E?problem_id=207) 看到，但是貌似交不了？

大力手玩题！场切了！

首先看到这种题，我们一定是先想给定一个树怎么求他的最大独立集。我**忘记**怎么贪心了，于是考虑 DP,设 $f_{u,0/1}$ 表示以 $u$ 为根的子树中独立集包含或不包含 $u$ 这个点的最大独立集大小。转移是显然的，为了下文讲解方便还是在这里写出：

$$
\begin{aligned}

f_{u,0} &= \sum\limits_{v\in son_u}max(f_{v,0}, f_{v,1})\\

 f_{u,1} &= \sum\limits_{v\in son_u}f_{v,1}
\end{aligned}
$$

现在可以来考虑这道题了。考虑你把那么多的树接到同一个树上，这种题要么是矩阵快速幂维护转移，要么就是有点规律。显然的，通过上面的转移式可以看出一个点的答案只更他的儿子那一层有关系。这启发了我们只保留 $T(i-1)$ 的根的那两个 dp 值，然后在每一个点加上相应的贡献。这样子做复杂度 $O(n^2)$，你就有了 60 分了！（这么水？）

下一步貌似没有头绪了。但是这时通过观察对拍的小数据的答案你惊讶地发现:

$$ Ans_i= nAns_{i-1}+[unknown](Ans_0)$$

所以现在你需要解决的问题在于中间 $+Ans_0$ 的条件是什么，然后你就可以愉快地 $O(1)$ 递推了！

首先你需要从本质上想想上面那玩意的正确性，如果答案是 $nAns_{i-1} + Ans_0$ 是非常符合常理的。原因是猜测原 dp 值加上某个值后大小关系不变，所以所有的转移是不变的，而每一个 $f$ 值初值一定被加上了 $Ans_{i-1}$，所以就是 $nAns_{i-1} + Ans_0$。

细心的读者或许发现了上面那一段话的问题，就是不是所有的初值都被加上了 $Ans_{i-1}$，如果设 $Ans_{i,0/1}$ 表示选不选 $k_i$ 作为根时候的答案，那么根据上面的定义, $f$ 的初值应当设为：

$$
\begin{aligned}

f_{u,0} &= max(Ans_{i-1,0},Ans_{i-1,1})\\

 f_{u,1} &= Ans_{i-1,0} + 1
\end{aligned}
$$

如果 $Ans_{i-1,0} < Ans_{i-1,1}$ 那么显然有的值相对于我们原来的估算就被少加了 $Ans_{i-1,1} - Ans_{i-1,0}$。而显然有一个结论是 $Ans_{i-1,1} - Ans_{i-1,0} \leq 1$。所以贡献来攻先去毛估估一下刚好是少了 $Ans_0$。

现在我们搞清楚了条件:


$$ Ans_i= nAns_{i-1}+[Ans_{i-1,0} \geq Ans_{i-1,1}](Ans_0)$$

所以我们现在需要弄明白的问题就只剩下了怎么样维护 $Ans_{i,0}, Ans_{i,1}$ 了。

再次观察一下答案的规律(什么鬼，居然是这么做的吗？)，发现貌似答案不大可能连续两次减少 $Ans_0$。仔细想想，是因为你不可能连续两次不选根节点，不然可以调整成更优解？而 $0\leq Ans_{i-1,1} - Ans_{i-1,0} \leq 1 $，所以你考虑如果你 $Ans_{i,0} = Ans_{i,1}$，那么其实就是相当于在原树上直接做，然后答案加一个偏移量而已。这启发了你去考虑原树上的情况。所以结论就是如果在原树上以 $u$ 为根做一遍 DP，这是如果什么 $f_{i,0} = f_{i,1}$,那么那个 $Ans_0$ 的贡献是一定会加上去的。否则如果上一次加了贡献就不加了，否则不加 $Ans_0$。然后你就做完了！

非常厉害，好玩的题目。（我居然可以场切？）

```cpp
#include<bits/stdc++.h>
using namespace std;
const long long inf = 1e18;
const int mininf = 1e9 + 7;
#define int long long
#define pb emplace_back
inline int read(){int x=0,f=1;char ch=getchar();while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}while(ch>='0'&&ch<='9'){x=(x<<1)+(x<<3)+(ch^48);ch=getchar();}return x*f;}
inline void write(int x){if(x<0){x=~(x-1);putchar('-');}if(x>9)write(x/10);putchar(x%10+'0');}
#define put() putchar(' ')
#define endl puts("")
const int MAX = 1e5 + 10;
const int mod = 998244353;
vector <int> g[MAX];
int a[MAX];
int f[MAX][2];
int g2[MAX][2];
int g3[MAX][2];
bool fl[MAX];

void dfs(int u, int fa){
	f[u][1] = 1, f[u][0] = 0;
	for(int v : g[u]){
		if(v == fa)	continue;
		dfs(v, u);
		f[u][0] += max(f[v][0], f[v][1]);
		f[u][1] += f[v][0];
	}
}

void dfs3(int u, int fa){
	for(int v : g[u]){
		if(v == fa)	continue;
		int tmpfk = f[u][1], tmpfk2 = f[u][0];
		f[u][0] -= max(f[v][0], f[v][1]);
		f[u][1] -= f[v][0];
		int tmp = f[v][0], tmp2 = f[v][1];
		f[v][0] += max(f[u][0], f[u][1]);
		f[v][1] += f[u][0];
		g2[v][0] = f[v][0], g2[v][1] = f[v][1];
		dfs3(v, u);
		f[u][0] = tmpfk2, f[u][1] = tmpfk;
		f[v][0] = tmp, f[v][1] = tmp2;
	}
}


void solve(){
	int n = read(), m = read();
	for(int i = 1; i < n; i++){
		int u = read(), v = read();
		g[u].pb(v), g[v].pb(u);
	}
	for(int i = 1; i <= m; i++){
		a[i] = read();
	}
	dfs(1, 1);
	g2[1][1] = f[1][1], g2[1][0] = f[1][0];
	dfs3(1, 1);
	write(max(f[1][1], f[1][0])), endl;
	int prert0 = f[1][0], prert1 = f[1][1];
	int preans = max(f[1][1], f[1][0]);
	int tmp = preans;
	bool fl = g2[1][0] < g2[1][1];
	for(int i = 1; i <= m; i++){
		preans *= n;
		preans %= mod;
		preans += tmp;
		preans %= mod;
		if(fl)	preans -= tmp;
		preans %= mod;
		if(preans < 0)	preans += mod;
		if(g2[a[i]][0] < g2[a[i]][1])	fl ^= 1;
		else fl = 0;
		write(preans), endl;
	}
}

signed main(){
	int t = 1;
	while(t--)	solve();
	return 0;
}
```