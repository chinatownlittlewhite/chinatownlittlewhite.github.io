# K. Permutation Counting

题意：给你n个数，m个限制，每一个限制都是第i个数小于第j个数，求满足这m个条件的长度为n的排列有多少。

简单地说，这题可以简化成 每一个限制都是从j到i连一条有向边，再求整个图合法拓扑排序的数量。

但是这题有些问题，我用标程跑有横叉边的树的答案为0，故对此题进行补充：连边后的图不为有横插边的树。

如下图，我们假设所连成的图为一棵树，此时显然节点1已经固定为最大数8，我们先考虑分子树2，相当于从剩下7个点中选3个点，即$$C73$$，之后考虑分子树3，剩下4个点选3个即$$C43$$，再分子树6，2个点选1个即$$C21$$，再分子树7,1个点选1个即$$C11$$，最后分子树4，1个点选1个即$$C11$$，答案即为上面各式相乘。

![无标题](C:\Users\lenovo\Desktop\无标题.png)

看懂上面的做法，这题就很简单了，我们只需处理每个子树节点的个数，然后利用组合数即可得出答案。

另外，若连成的为森林，我们只需要将所有入度为0的节点和一个额外的节点0相连，这样就可以将森林变成一颗树，节省代码量。

若有环则结果为0。

```c++
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
typedef unsigned long long ull;
int n,m,mod=998244353;
vector<int>g[2000010];
int vis[2000010];
int d[2000010],s[2000010];
ll in[2000010],inf[2000010];
ll ans=1;
ll qmi(ll a,ll b)
{
    ll res=1;
    while(b)
    {
        if(b&1) res=res*a%mod;
        a=a*a%mod;
        b>>=1;
    }
    return res;
}
ll c(ll a,ll b)
{
    if(a<b) return 0;
    return in[a]*inf[b]%mod*inf[a-b]%mod;
}
void dfs1(int u)
{
    vis[u]++;
    s[u]=1;
    for(int i:g[u])
    {
        dfs1(i);
        s[u]+=s[i];
    }
}
void dfs(int u)
{
    ll res=s[u]-1;
    for(int i:g[u])
    {
        dfs(i);
        ans=(ans*c(res,s[i]))%mod;
        res-=s[i];
    }
}
void solve()
{
    cin>>n>>m;
    in[0]=inf[0]=1;
    for(int i=1;i<=n;i++)
    {
        in[i]=in[i-1]*i%mod;
        inf[i]=qmi(in[i],mod-2);
    }
    for(int i=0;i<m;i++)
    {
        int a,b;
        scanf("%d%d",&a,&b);
        g[b].push_back(a);
        d[a]++;
    }
    for(int i=1;i<=n;i++)
    {
    	if(!d[i]) g[0].push_back(i);
	}
	dfs1(0);
	dfs(0);
    for(int i=1;i<=n;i++)
        if(!vis[i])
        {
            cout<<0<<'\n';
            return ;
        }
    cout<<ans<<'\n';
}
int main()
{
    solve();
}

```

