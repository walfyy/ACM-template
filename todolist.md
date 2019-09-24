min25
差分约束系统
~~模意义下高斯消元~~
set维护凸壳(CHT)
cdq优化dp(平行四边形等式)
类欧几里得
~~卢卡斯定理~~
持久化treap
~~子集卷积~~
dp套dp,动态dp?
博弈?
扩展bsgs
~~nsqrt(n)二分图匹配~~
~~两端加点pam~~
吉司机线段树
trie树扩展sam
闵可夫斯基和
rsa解密
整体二分
斐波那契循环节
数论整理!
```
#include<bits/stdc++.h>
#define ll long long
#define mod 1000000007
using namespace std;

const int N=(1<<20)+10,inf=0x3f3f3f3f;

int d[20][N],dp[20][N],a[N];
void fwt_or(int *a,int n,int dft)
{
    for(int i=1;i<n;i<<=1)
        for(int j=0;j<n;j+=i<<1)
            for(int k=j;k<j+i;k++)
            {
                if(dft==1)a[i+k]=(a[i+k]+a[k])%mod;
                else a[i+k]=(a[i+k]-a[k]+mod)%mod;
            }
}
int main()
{
    int n=17;
    for(int i=0;i<(1<<n);i++)d[__builtin_popcount(i)][i]=a[i];
    for(int i=0;i<=n;i++)fwt_or(d[i],(1<<n),1);
    for(int i=0;i<=n;i++)for(int j=0;j<=i;j++)
    for(int k=0;k<(1<<n);k++)
    {
        dp[i][k]+=1ll*d[j][k]*d[i-j][k]%mod;
        if(dp[i][k]>=mod)dp[i][k]-=mod;
    }
    for(int i=0;i<=n;i++)fwt_or(dp[i],(1<<n),-1);
    for(int i=0;i<(1<<n);i++)a[i]=dp[__builtin_popcount(i)][i];
    return 0;
}
```
