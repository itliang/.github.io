---
layout: post
title:  "算法练习"
date:   2018-08-30 17:00:05
categories: Algorithm
excerpt: DP算法练习。
---

* content
{:toc}

动态规划(Dynamic Programming)是运筹学的一个分支，是求解决策过程(decision process)最优化的数学方法。


##Dynamic Program基本概念及思想
分解后的子问题不是相互独立的，下阶段的求解建立在上一个子阶段问题的解上。<br>
最优子结构：问题的最优解所包含的子问题的解也是最优的。<br>
重叠子问题：每次计算的子问题并不总是新问题，每个子问题只计算一次，将结果保存在一个表中。

###LCS最长公共子序列

最长公共子序列就是寻找两个给定序列的子序列，该子序列在两个序列中以相同的顺序出现，但是不必要是连续的。
例如序列X=ABCBDAB，Y=BDCABA。序列BCBA是X和Y的一个LCS，序列BDAB也是。<br>
LCS(X，Y)表示X和Y的一个最长公共子序列。<br>
如果Xm = Yn, 那么LCS(X, Y) = Xm +　LCS(Xm-1, Yn-1);<br>
如果Xm != Yn, 那么LCS(X, Y) = Max(LCS(Xm-1, Y), LCS(X, Yn-1)); 其中<br>
LCS(Xm-1, Y)表示在数组X中，m-1位之前的数据和在数组Y中n位之前的数据的最长公共子序列的长度<br>
LCS(X, Yn-1)表示在数组X中，m位之前的数据和在数组Y中n-1位之前的数据的最长公共子序列的长度<br>

1. 求公共子序列的最大长度：

		string x = "ABCBDAB";//"Hello, How are you";
		string y = "BDCABA";//"llo, re";
		int x_len = x.length();
		int y_len = y.length();
		int dp[100][100];
		char k[100];
		for (int k = 0; k < 100; k++){
			dp[k][0] = 0;
			dp[0][k] = 0;
		}
	
		for (int i = 1; i <= x_len; i++){
			for (int j = 1; j <= y_len; j++){
				if (x[i-1] == y[j-1]){
					dp[i][j] = dp[i - 1][j - 1] + 1;
				}
				else {
					dp[i][j] = dp[i - 1][j] > dp[i][j - 1] ? dp[i - 1][j] : dp[i][j - 1];
				}
			}
		}
		int maxLen = dp[x_len][y_len];
		cout << "LCS Max Len:%d" << maxLen <<endl;

2. 打印公共子序列：

		//Store the LCS in K array
		int temp = maxLen;
		int ii = x_len;
		int jj = y_len;
		
		while (ii && jj){
			if (dp[ii][jj] == dp[ii - 1][jj - 1] + 1){			
				k[--temp] = x[ii - 1];
				ii--;
				jj--;
			}
			else if(dp[ii-1][jj] > dp[ii][jj-1]){
				ii--;
			}
			else{
				jj--;
			}
		}

###LCS最长公共子串