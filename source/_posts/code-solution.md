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

# 剑指Offer 47

```cpp
class Solution {
public:
    int maxValue(vector<vector<int>>& grid) {
        int dp[grid.size()+1][grid[0].size()+1];
        memset(dp,0,sizeof(dp));
        dp[0][0]=grid[0][0];
        for(int i=0;i<grid.size();i++){
            for(int j=0;j<grid[0].size();j++){
                if(i>0&&j>0){
                    dp[i][j]=std::max(dp[i-1][j],dp[i][j-1])+grid[i][j];
                    
                }else if(i>0){
                    dp[i][j]=dp[i-1][j]+grid[i][j];
                }else if(j>0){
                     dp[i][j]=dp[i][j-1]+grid[i][j];
                }
                //printf("dp[%d][%d] = %d\n",i,j,dp[i][j]);
            }
        }
        return dp[grid.size()-1][grid[0].size()-1];
    }
};
```

简单dp,$dp[i][j]$代表的含义为在坐标(i,j)下，所能拿到的最大值，由题目可知，这个状态只能从$dp[i-1][j],dp[i][j-1]$得到，那么很容易推出dp公式$dp[i][j]=max(dp[i-1][j],dp[i][j-1])+grid[i][j]$

# LeetCode 2379

```cpp
class Solution {
public:
    int minimumRecolors(string blocks, int k) {
        int white[blocks.length()+5];
        int num = 0;
        memset(white,0,sizeof(white));
        for(int i=0;i<blocks.length();i++){
            if(blocks[i]=='W') num++;
            white[i]=num;
        }
        int minn = k;
        for(int i=k-1;i<blocks.length();i++){
            int now =0;
            if(i==k-1) now = white[i];
            else now = white[i]-white[i-k];
            minn = std::min(minn,now);
        }
        return minn;
    }
};
```

滑动窗口，或者前缀和，简单思路就是滑动窗口。我写的空间复杂度是O(n)，可以**优化**成O(1)的空间复杂度，把white数组变成两个cnt即可，记录最左边位置之前的白色数量和最右边位置之前的白色数量，每次滑动都维护一下就行。

# LeetCode 704

```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int mid = nums.size()/2;
        int l=0,r=nums.size()-1;
        while(nums[mid]!=target){
            if(l==r) return -1;
            if(nums[mid]<target) l = mid+1;
            else if(nums[mid]>target) r = mid-1;
            mid = (l+r)/2;
        }
        return mid;
    }
};
```

简单有序数组二分查找，记得lr每次需要收缩即可

# LeetCode 203

```cpp
class Solution {
public:
    ListNode* removeElements(ListNode* head, int val) {
        ListNode *now = head;
        ListNode *pre = NULL,*next = NULL;
        while(now!=NULL){
            next = now->next;
            if(now->val == val){
                if(pre!=NULL) pre->next = next;
                else head = next;
            }else{
                pre = now;
            }
            now = next;
        }
        return head;
    }
};
```

简单链表删除

# LeetCode 27

```cpp
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int slow=0;
        for(int i=0;i<nums.size();i++){
            if(nums[i]==val){
                continue;
            }else {
                nums[slow]=nums[i];
                slow++;
            }
        }
        return slow;
    }
};
```

快慢指针简单题，慢指针为新数组的下标值

# LeetCode 977

```cpp
class Solution {
public:
    int squares(int num){
        return num*num;
    }
    vector<int> sortedSquares(vector<int>& nums) {
        vector<int>res;
        int negPos=0;
        int pre=0,next=0;
        for(int i =0;i<nums.size();i++){
            negPos=i;
            if(nums[i]>=0){
                break;
            }
        }
        pre = negPos-1;
        next = negPos;
        while(pre>=0&&next<nums.size()){
            if(squares(nums[pre])>squares(nums[next])){
                res.push_back(squares(nums[next]));
                next++;
            }else{
                res.push_back(squares(nums[pre]));
                pre--;
            }
        }
        if(pre>=0){
            for(;pre>=0;pre--){
                res.push_back(squares(nums[pre]));
            }
        }
        if(next<nums.size()){
            for(;next<nums.size();next++){
                res.push_back(squares(nums[next]));
            }
        }
        return res;
    }
};
```

简单题，暴力做法O($n^2$)，先全平方后排序。还有个比较好想的做法，还是双指针，找到第一个大于0的数的位置，然后两个指针从中间往两边扫，每次把平方数小的放入数组即可。

都想到双指针了，怎么没从两边往中间扫。。。。尬住了。代码还更简单

# LeetCode 1605

```cpp
class Solution {
public:
    vector<vector<int>> restoreMatrix(vector<int>& rowSum, vector<int>& colSum) {
        int row_len = rowSum.size(),col_len = colSum.size();
        vector<vector<int>> res(row_len,vector<int>(col_len,0));
        for(int row = 0;row<row_len;row++){
            for(int col = 0;col<col_len;col++){
                if(rowSum[row]<colSum[col]){
                    res[row][col] = rowSum[row];
                    rowSum[row]=0;
                    colSum[col]-=res[row][col];
                }
                else{
                    res[row][col] = colSum[col];
                    colSum[col]=0;
                    rowSum[row]-=res[row][col];
                }
            }
        }
        return res;
    }
};
```

就是个贪心，我直接选该行该列更小的那个就行了，每次更新一下rowsum和colsum即可。能填就填

有个小优化思路，即当rowsum[row]=0的时候，直接break就行，因为后面的全是0

# LeetCode 059

```cpp
class Solution {
public:
    bool check(int p,int n){
        if(p==n||p==-1) return true;
        return false;
    }
    vector<vector<int>> generateMatrix(int n) {
        vector<vector<int>> res(n,vector<int>(n,0));
        int dir[4][2]={{0,1},{1,0},{0,-1},{-1,0}};
        int x=0,y=-1;
        int cnt=0;
        int flag=0;
        while(cnt<n*n){
            cnt++;
            x+=dir[flag][0];
            y+=dir[flag][1];
            res[x][y]=cnt;
            if((check(x+dir[flag][0],n)||check(y+dir[flag][1],n))||res[x+dir[flag][0]][y+dir[flag][1]]!=0){
                flag++;
                flag%=4;
            }
            
        }
        return res;
    }
};
```

每次撞到边界或已经存在数的位置就换方向，简单模拟题。注意越界问题即可

# LeetCode 1616

```cpp
class Solution {
public:
    bool checkPalindromeFormation(string a, string b) {
        bool front=true,end=true;
        bool resafront=true,resbend=true;
        bool resaend=true,resbfront=true;
        int mid = a.length()/2;
        for(int i=0;i<mid;i++){
            if(front){
                if(a[i]!=b[a.length()-1-i]){
                    front=false;
                    if(b[i]!=b[a.length()-1-i])
                        resafront=false;
                    if(a[i]!=a[a.length()-1-i])
                        resaend=false;
                }
            }else{
                if(b[i]!=b[a.length()-1-i])
                    resafront=false;
                if(a[i]!=a[a.length()-1-i])
                    resaend=false;
            }
            if(end){
                if(b[i]!=a[a.length()-1-i]){
                    end=false;
                    if(a[i]!=a[a.length()-1-i])
                        resbend=false;
                    if(b[i]!=b[a.length()-1-i])
                        resbfront=false;
                }
            }else{
                if(a[i]!=a[a.length()-1-i])
                    resbend=false;
                if(b[i]!=b[a.length()-1-i])
                    resbfront=false;
            }
        }
        return resafront||resbend||resbfront||resaend;
    }
};
```

思路简单，题目要求aprefix + bsuffix或者bprefix + asuffix为回文串，那么可以确定的就是a的前面部分和b的后面部分或者b的前部分和a的后部分一定是相同的，然后中间那块可能是a的也可能是b的，分类讨论，共4种情况，apre+amid+bsuf,apre+bmid+bsuf,bpre+amid+asuf,bpre+bmid+asuf，都判断下就好了

代码优化，其实代码重复部分过多，可以写成函数

# LeetCode 1625

```cpp
class Solution {
public:
    string addSum(string now,int a){
        for(int i=0;i<now.length();i++){
            if(i&1) {
                now[i]=((now[i]-'0'+a)%10) + '0';
            }   
        }
        return now;
    }
    string shuffledString(string now,int b){
        string res="";
        for(int i=now.length()-b;i<2*now.length()-b;i++){
            res+=now[i%now.length()];
        }
        return res;
    }
    string findLexSmallestString(string s, int a, int b) {
        queue<string> q;
        set<string> res;
        unordered_map<string,bool> mp;
        q.push(s);
        mp[s]=true;
        while(!q.empty()){
            string now = q.front();
            res.insert(now);
            q.pop();
            string tmp = addSum(now,a);
            if(mp.find(tmp)==mp.end()){
                mp[tmp]=true;
                q.push(tmp);
            }
            tmp = shuffledString(now,b);
            if(mp.find(tmp)==mp.end()){
                mp[tmp]=true;
                q.push(tmp);
            }
        }
        return *res.begin();
    }
};
```

想不到O(n)的做法，但是感觉可能存在，因为如果b是奇数那么字符串每个都可以为头，如果b是偶数，那么只有偶数位可以为头。可能可以根据序列差值处理出结果。

暴力模拟，BFS最好写，还有个模拟方法就是纯暴力，很容易知道，a最多加10次就加到原来的值，b最多挪n次，也回到原来的位置，如果b是奇数就是$n*10*10$种情况，偶数就为$n*10$种情况，时间复杂度就是在这基础上乘个n，因为需要处理字符串
