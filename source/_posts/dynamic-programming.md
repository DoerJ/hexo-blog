---
title: Some Insights of Dynamic Programming Problems
date: 2021-06-16
tags: algorithms, dynamic programming
index_img: /images/thumbnail/dp.png
---
### Approaches to solving dynamic programming problems 
##### What is dynamic programming?
When we face a problem, one of the most common mindsets is to break the problem down to subproblems, solve each subproblem individually, and finally solve the whole problem by aggregating the solution of each subproblem. Dynamic programming shares the very similar philosophy. Dynamic programming is a method which solves problem by combining the solutions to the subproblems. The key idea of this method is to use a data structure(usually a tabular structure such as array) to cache the optimal solution to each subproblem on-the-fly, known as `memorization`. The optimal values stored in the table will then be used for building up the solution to the next subproblem. When construct a dynamic programming algorithm, there are four steps to follow: 
1. Characterize the the subproblem and its optimal solution, that is, how should we formulate the subproblem, and what are the representations of optimal value.
2. Define the update function for the optimal solution to each subproblem. 
3. Fulfill the memorization table recursively or using bottom-up fashion.
4. Construct the optimal solution to the entire problem based on the memorization table.

##### Top-down approach
Top-down approach is one of the two flows to construct a dynamic progarmming algorithm. And known as it's name, top-down approach reaches to the overall optimal solution `recursively`, that is, starting with the problem itself, recursively breaks down into smaller and smaller subproblems, until reaches down to the base case. Since the top-down approach caches the optimal result to each subproblem, those optimal solutions will be memorized and used on-the-fly to avoid the duplicated computation, and boost up the runtime. Therefore, top-down approaches are also referred as memorization for dynamic programming. 

##### Bottom-up approach
Opposed to top-down approach, bttom-up approach solves the problem `iteratively` by gradually building up the optimal solutions and caching them in the table. To develop a bottom-up approach, we need to sort all the subproblems by size and solve the smallest subproblems first. In that way, whichever subproblem to solve, we already have all it's depending smaller subproblems solved and their optimal values saved in the table. Once all the subproblems have been solved, we already have the prerequisites for constructing the final optimal solution to the problem itself. 

Top-down and bottom-up approaches usually have the same runtime. And due to their characteristics of memorization, both approaches trim out the re-computation for the same subproblem, which is a huge factor for optimizing the time complexity. However, the bottom-up approach usually has less time overhead of procedure calls since it's core implementation relies on the iterative method. 

##### Recursion vs dynamic programming
One of the key characteristics for dynamic programming is memorization. The optimal solution to each subproblem is cached in the table for the later use, which helps reduce the runtime comparing the pure recusive implementation. We'll use a simple example to illustrate how the time complexity is minimized behind the scene using dynamic programming. 

When it comes to recursion, the most classic problem that comes to mind, of course, is Fibonacci sequence. Given the pre-conditions $F(0)=0$, $F(1)=1$, and the formula expression $F(n)=F(n-1)+F(n-2)$, what is the value of $n^{th}$ term. The following is the implentation using straight-up recursion:
```javascript 
function fibonacci(n) {
  if (n === 0 || n === 1) {
    return n;
  }
  return fibonacci(n-1) + fibonacci(n-2);
} 
```
This is a working solution, but let's take a look at it's runtime. To analyze the the time complexity, we lay out the computation process in recursion tree as the following (suppose we are computing the $5^{th}$ term of fibonacci sequence):
```text
                          F(5)
                       /        \
                  F(4)            F(3)
                /      \        /      \
            F(3)        F(2)  F(2)     F(1)
          /     \       /  \  /  \
        F(2)    F(1)  F(0)F(1)F(1)F(0)
        /  \
      F(1) F(0)
```
To estimate the time complexity, we need to count how many nodes are in this recursion tree since each node represents a recursive call. The recursion tree is similar to binary tree, except each node has exact;y two nodes. The total number of nodes is $2^0+2^1+...+2^n$, which forms the time complexity of $O(2^n)$. Hence the recursive method runs in exponential time. If we look at the nodes of each level, we can notice that there are re-computation of nodes such as $F(2)$ and $F(3)$ which can be trimmed out using caching table. 

We now take a look at the implentation of top-down and bottom-up approach to dynamic programming respectively:
```javascript 
function fibonacci(n) { // top-down approach
  var memo = new Array(n+1).fill(-1); // memorization table
  return fb(n, memo);
}

function fb(i, m) {
  if (i === 0 || i === 1) { // base cases
    return i;
  }
  if (m[i] === -1) {
    m[i] = fb(i-1, m) + fb(i-2, m); // update function 
  }
  return m[i];
}
```
```javascript
function fibonacci(n) { // bottom-up approach
  var memo = new Array(n+1);  // memorization table
  memo[0] = 0;  // initialization
  memo[1] = 1;
  for (let i = 2; i <= n; i++) {
    memo[i] = memo[i-1] + memo[i-2];  // update function
  }
  return memo[n];
}
```
With the memorization table, the recursion tree now becomes: 
```text
                          F(5)
                       /        \
                  F(4)            F(3)*
                /      \       
            F(3)        F(2)*  
          /     \       
        F(2)    F(1)  
        /  \
      F(1) F(0)
```
Instead of proceeding recursive call for each node of the recursion tree, the tree now goes down the path in left sub-tree, and the nodes which need the re-computation have been trimmed out. The nodes with asterisks are the values cached in the memorization table and can be returned immediately. The total number of nodes in this recursion tree is roughly $2n$ where $n$ is also the height of the tree. Hence the time complexity is $O(n)$. Now the dynamic programming algorithm runs in linear time, which is a huge improvement on the runtime, comparing to the exponential runtime with recursive method.

### Some other classic problems 
##### Longest palindrome substring 
This is one of the most typical problems solved by dynamic programming algorithm. Given a non-empty string with length of $n$, return the longest substring that is palindrome(i.e., a string that reads the same backward as forward). For example:
```text
input: babad
output: bab or aba
```
To build a solution with dynamic programming approach, we first need to define the subproblem and the representation of the optimal value. The subproblems can be built up from the bases cases where the length of string is $1$ and $2$. When the string length is $1$, the string(or letter) itself is a palindrome. For the string of length $2$, if two letters are identical, then the string is a palindrome, otherwise, the longest palindrome strings are the letters of the string. 

Once we have the base cases for the subproblem, we have better idea how to represent the optimal value and the structure of the memorization table. We can create the table as a two-dimensional array $memo$ where $memo[i][j]$ indicates whether the substring $S_{ij}$(i.e., the substring that starts at index $i$, and ends at index $j$) is a palindrome, thus the optimal solution to each table cell will be a boolean value. Based on the characteristic of palindrome, we can then formulate the update function $memo[i][j] = memo[i+1][j-1]$ && $s[i] === s[j]$. 

After the memorization table and update function are defined, the next step will be filling optimal solution into the table. This time we will use bottom-up approach, which starts with the bases cases the length of substring $s=1$ and $s=2$, then gradually increases $s$ until the length of the whole string $n$. Meanwhile, we keep track of the palindrome substring of the maximum length, and returns the value in the end. The following is the code implementation:
```javascript
var longestPalindrome = function(s) {
    var res;  
    var a = new Array(s.length).fill(0);  // create memorization table
    var memo = a.map(() => new Array(s.length).fill(0));
    for (let i = 1; i <= n; i++) {  // iterate over the length of substring
        for (let j = 0; j <= n-i; j++) {
            if (i === 1) {  // base case 1
                memo[j][j] = true;
            }
            else if (i === 2) { // base case 2
                memo[j][j+1] = (s[j] === s[j+1]);
            } 
            else {  // fill in the rest of table cells
                memo[j][j+i-1] = (memo[j+1][j+i-2] && s[j] === s[j+i-1]);
            }
            if (memo[j][j+i-1]) { // keep track of palindrome substrings
                res = s.substr(j, i);
            }
        }
    }
    return res ? res : s[0];

```

### Conclusion 
Similar to divide-and-conquer algorithm, the paradigm of dynamic programming is to break the problem into subproblems that can be easily solved. Then use some data structure to memorize or cache that optimal solution to the subproblems which have been solved, and use those values to build up the solution to the bigger subproblems. Boil it down, the key parts of designing a robust dynamic programming algorithm is to optimize the structure of the memorization table and the update function for the optimal solution to each subproblem.

I hope you find this artical useful, and I'll see you in the next blog!


