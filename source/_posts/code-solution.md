---
title: 刷题记录
date: 2023-03-05 13:55:20
categories: 
  - [tech]
  - [cpp]
  - [job]
tags: [tech,cpp,job]
category_bar: true
---

记录从3.5号开始的刷题记录，正在连载

最近还要干nebula的活，只能抽小部分时间先找到做题的感觉，等nebula的事情忙完了，就专心找工作吧，加油。

# Leetcode 1599

```c++
class Solution {
public:
    int minOperationsMaxProfit(vector<int>& customers, int boardingCost, int runningCost) {
        int maxProfit=-1;
        int res=0;
        int profitSum=0;
        for(int i=0;i<customers.size();i++){
            int tmp=0;
            if(customers[i]>4&&i!=(customers.size()-1)){
                
                customers[i+1] +=(customers[i]-4);
                tmp=4;
            }else{
                tmp=customers[i];
            }
            int round =0;
            if(tmp<4||i+1==customers.size()){
                round = tmp/4;
                profitSum+=((boardingCost*(tmp-tmp%4))-(round)*runningCost);
                // printf("stop round %d tmp = %d ceil = %d profit = %d\n",i,tmp,round,profitSum);
                if(maxProfit<profitSum){
                    maxProfit=profitSum;
                    res=i+round;
                }
                profitSum-=((boardingCost*(tmp-tmp%4))-(round)*runningCost);
            }
            round = (int)ceil(tmp/4.0);
            if(round==0 ) round=1;
            profitSum+=((boardingCost*tmp)-(round)*runningCost);
            // printf("continue round %d tmp = %d ceil = %d profit = %d\n",i,tmp,round,profitSum);
            if(maxProfit<profitSum){
                maxProfit=profitSum;
                res=i+round;
            }
        }
        if(maxProfit<=0) return -1;
        return res;
    }
};
```

简单的模拟，每轮最多4个人，能坐满就继续运行，大于4个人就把剩下的人放在后一个轮次直到最后一轮，不能坐满就有两种情况，一种停止，一种继续，中间记录最大收益以及所在轮次即可。

**优化空间**：本身题目数据量较小，如果数据大，那么可以使用bool数组标记每个轮次的人是否到达，不再累加，因为可能溢出。一个剪枝，坐满了也不能盈利就直接返回-1。时间复杂度都为O(n)

# LeetCode 1653

```c++
class Solution {
public:
    int minimumDeletions(string s) {
        int len = s.length();
        printf("len = %d\n",len);
        int a[len+1],b[len+1];//i前面的b和i后面的A
        int Afront=0,behindB=0;
        for(int i=0 ;i<len ;i++){
             a[i]=Afront;
            if(s[i]=='b'){
                Afront++;
            }
        }
        for(int i=len-1 ;i>=0 ;i--){
            b[i]=behindB;
            if(s[i]=='a'){
                behindB++;
            }
        }
        int minn = len;
        for(int i=0 ;i<len ;i++){
            minn = min(a[i]+b[i],minn);
        }
        return minn;
    }
};
```

平衡的条件是a前面没b，b后面没a。那么思路很简单，删除该位置前面的b和后面的a就可以让字符串平衡。先跑两遍数组，得到位置i前面的b和i后面的A，然后找两个之和最小的就行。

**优化思路** 就是个最长上升子序列问题，用两个变量维护就行，O(n)复杂度，维护一个前面的b数量，再维护一个目前的操作数量就行，因为每一步只有保留和删除两个选择，即当是a时，op = min(op+ 1, countB)，是b时countB+1

# 剑指offer 09

```cpp
class CQueue {
public:
    std::stack<int> stIn,stOut;
    CQueue() {
    }
    void appendTail(int value) {
        stIn.push(value);
    }
    int deleteHead() {
        if(stOut.empty()){
            while(!stIn.empty()){
                int val = stIn.top();
                stIn.pop();
                stOut.push(val);
            }
        }
        if(stOut.empty()){
            return -1;
        }
        int res = stOut.top();
        stOut.pop();
        return res;

    }
};
```

两个栈实现队列，利用栈的特性，后进先出，制造一个输入栈一个输出栈即可，当输出栈为空时，将输入栈全部倒入输出栈，这样就可以有一个先进先出的效果，从而实现队列。

# 剑指offer 30

```cpp
class MinStack {
public:
    /** initialize your data structure here. */
    stack<int> stk;
    stack<int> minNum;
    int minn;
    MinStack() {
        minn=0x7fffffff;
    }
    
    void push(int x) {
        stk.push(x);
        minn = minNum.empty()?x:std::min(x,minNum.top());
        minNum.push(minn);
    }
    
    void pop() {
        stk.pop();
        minNum.pop();
    }
    
    int top() {
        return stk.top();
    }
    
    int min() {
        return minNum.top();
    }
};
```

多加个最小值，那么就直接维护一个最小栈就行，每次进栈时，加入一个当前值和最小栈栈顶元素更小的元素。想要查看栈最小值时直接输出最小栈的栈顶。还有一个思路就是每次push两个值进栈，用一个minn变量保存当前栈的最小值，pop时pop两次即可。

# LeetCode 1096

```cpp
class Solution
{
public:
    map<int, int> mp; //{} pos
    bool isLetter(char s)
    {
        if ('a' <= s && s <= 'z')
            return true;
        return false;
    }
    vector<string> multiString(vector<string> const &first, vector<string> const &second)
    {
        vector<string> res;
        for (int i = 0; i < first.size(); i++)
        {
            for (int j = 0; j < second.size(); j++)
            {
                res.push_back(first[i] + second[j]);
            }
        }
        return res;
    }
    vector<string> Process(string expression, int st)
    {

        vector<string> res;
        string tmp;
        bool flag = false;
        vector<string> strTmp;
        for (int i = 0; i < expression.length(); i++)
        {
            if (isLetter(expression[i]))
            {
                tmp += expression[i];
                if (strTmp.size() != 0)
                {
                    if (tmp != "")
                    {
                        for (int j = 0; j < strTmp.size(); j++)
                        {
                            strTmp[j] = strTmp[j] + tmp;
                        }
                        tmp = "";
                    }
                }
            }
            else
            {
                if (expression[i] == ',')
                {
                    if (strTmp.size() == 0)
                    {
                        if (tmp != "")
                        {
                            res.push_back(tmp);
                            tmp = "";
                        }
                    }
                    else
                    {
                        for (int j = 0; j < strTmp.size(); j++)
                        {
                            strTmp[j] = strTmp[j] + tmp;
                            res.push_back(strTmp[j]);
                        }
                        tmp = "";
                        strTmp.clear();
                    }
                }
                else if (expression[i] == '{')
                {
                    if (strTmp.size() != 0)
                    {
                        if (tmp != "")
                        {
                            for (int j = 0; j < strTmp.size(); j++)
                            {
                                strTmp[j] = strTmp[j] + tmp;
                            }
                            tmp = "";
                        }

                        strTmp = multiString(strTmp, Process(expression.substr(i + 1, mp[st + i] - st - i - 1), st + i + 1));
                    }
                    else
                    {
                        strTmp = Process(expression.substr(i + 1, mp[st + i] - st - i - 1), st + i + 1);
                    }
                    i = mp[st + i] - st;
                    if (tmp != "")
                    {
                        for (int j = 0; j < strTmp.size(); j++)
                        {
                            strTmp[j] = tmp + strTmp[j];
                        }
                        tmp = "";
                    }
                }
            }
        }
        if (strTmp.size() != 0)
        {
            for (int j = 0; j < strTmp.size(); j++)
            {
                strTmp[j] = tmp + strTmp[j];
                res.push_back(strTmp[j]);
            }
            strTmp.clear();
            tmp = "";
        }
        if (tmp != "")
        {
            res.push_back(tmp);
            tmp = "";
        }
        return res;
    }
    vector<string> braceExpansionII(string expression)
    {
        stack<int> st;
        for (int i = 0; i < expression.length(); i++)
        {
            if (expression[i] == '{')
            {
                st.push(i);
            }
            if (expression[i] == '}')
            {
                int left = st.top();
                st.pop();
                mp[left] = i;
            }
        }
        vector<string> res = Process(expression, 0), ans;
        set<string> myset;
        for (int i = 0; i < res.size(); i++)
        {
            myset.insert(res[i]);
        }
        for (auto iter = myset.begin(); iter != myset.end(); iter++)
        {
            ans.push_back(*iter);
        }
        return ans;
    }
};
```

大体思路很好想，就是递归展开括号内容进行处理，我首先用map保存了每对括号的位置，然后在递归处理，写的时候思路没理清楚，写的比较乱。后续代码其实可以优化的更清晰，就是找到第一个'}'位置j，如果没找到那就把其中所有字符串输出即可。如果找到了，那么找左边第一个'{'位置i，那么此时字符被分成三部分，第一部分是exp[0,i-1]，第二部分是该大括号内容exp[i+1,j-1]，第三部分是exp[j+1,n-1]。那么此时再按照逗号分隔第二部分，再进行重新组合内容，进行递归Process(expA,expB[i],expC)即可得到结果。

