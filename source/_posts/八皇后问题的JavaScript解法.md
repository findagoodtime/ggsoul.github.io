---
title: 八皇后问题的JavaScript解法
date: 2016-05-13 15:28:24
tags: javascript
categories: think
---

关于八皇后问题的 JavaScript 解法，总觉得是需要学习一下算法的，哪天要用到的时候发现真不会就尴尬了

<!--more-->

## 背景

八皇后问题是一个以国际象棋为背景的问题：如何能够在 8×8 的国际象棋棋盘上放置八个皇后，使得任何一个皇后都无法直接吃掉其他的皇后？为了达到此目的，任两个皇后都不能处于同一条横行、纵行或斜线上

八皇后问题可以推广为更一般的n皇后摆放问题：这时棋盘的大小变为 n×n ，而皇后个数也变成n 。当且仅当`n = 1`或`n ≥ 4`时问题有解

## 盲目的枚举算法

通过N重循环，枚举满足约束条件的解（八重循环代码好多，这里进行四重循环），找到四个皇后的所有可能位置，然后再整个棋盘里判断这四个皇后是否会直接吃掉彼此，程序思想比较简单

```javascript
function check1(arr, n) {
    for(var i = 0; i < n; i++) {
        for(var j = i + 1; j < n; j++) {
            if((arr[i] == arr[j]) || Math.abs(arr[i] - arr[j]) == j - i) {
                return false;
            }
        }
    }
    return true;
}
function queen1() {
    var arr = [];

    for(arr[0] = 1; arr[0] <= 4; arr[0]++) {
        for(arr[1] = 1; arr[1] <= 4; arr[1]++) {
            for(arr[2] = 1; arr[2] <= 4; arr[2]++) {
                for(arr[3] = 1; arr[3] <= 4; arr[3]++) {
                    if(!check1(arr, 4)) {
                        continue;
                    } else {
                        console.log(arr);
                    }
                }
            }
        }
    }
}

queen1();
//[ 2, 4, 1, 3 ]
//[ 3, 1, 4, 2 ]
```

关于结果，在 4*4 的棋盘里，四个皇后都不可能是在一排， arr[0] 到 arr[3] 分别对应四个皇后，数组的下标与下标对应的值即皇后在棋盘中的位置

## 回溯法

『走不通，就回头』，在适当节点判断是否符合，不符合就不再进行这条支路上的探索

```javascript
function check2(arr, n) {
    for(var i = 0; i <= n - 1; i++) {
        if((Math.abs(arr[i] - arr[n]) == n - i) || (arr[i] == arr[n])) {
            return false;
        }
    }
    return true;
}

function queen2() {
    var arr = [];

    for(arr[0] = 1; arr[0] <= 4; arr[0]++) {
        for(arr[1] = 1; arr[1] <= 4; arr[1]++) {
            if(!check2(arr, 1)) continue; //摆两个皇后产生冲突的情况
            for(arr[2] = 1; arr[2] <= 4; arr[2]++) {
                if(!check2(arr, 2)) continue; //摆三个皇后产生冲突的情况
                for(arr[3] = 1; arr[3] <= 4; arr[3]++) {
                    if(!check2(arr, 3)) {
                    	continue;
                    } else {
                    	console.log(arr);
                    }
                }
            }
        }
    }
}

queen2();
//[ 2, 4, 1, 3 ]
//[ 3, 1, 4, 2 ]
```

## 非递归回溯法

算法框架

```javascript
while(k > 0 『有路可走』 and 『未达到目标』) { // k > 0 有路可走
    if(k > n) { // 搜索到叶子节点
        // 搜索到一个解，输出
    } else {
        //a[k]第一个可能的值
        while(『a[k]在不满足约束条件且在搜索空间内』) {
            // a[k]下一个可能的值
        }
        if(『a[k]在搜索空间内』) {
            // 标示占用的资源
            // k = k + 1;
        } else {
            // 清理所占的状态空间
            // k = k - 1;
        }
    }
}
```

具体代码如下，最外层while下面包含两部分，一部分是对当前皇后可能值的遍历，另一部分是决定是进入下一层还是回溯上一层

```javascript
function backdate(n) {
    var arr = [];

    var k = 1; // 第n的皇后
    arr[0] = 1;

    while(k > 0) {

        arr[k-1] = arr[k-1] + 1;
        while((arr[k-1] <= n) && (!check2(arr, k-1))) {
            arr[k-1] = arr[k-1] + 1;
        }
        // 这个皇后满足了约束条件，进行下一步判断

        if(arr[k-1] <= n) {
            if(k == n) { // 第n个皇后
                console.log(arr);
            } else {
                k = k + 1; // 下一个皇后
                arr[k-1] = 0;
            }
        } else {
            k = k - 1; // 回溯，上一个皇后
        }
    }
}

backdate(4);
//[ 2, 4, 1, 3 ]
//[ 3, 1, 4, 2 ]
```

## 递归回溯法

递归调用大大减少了代码量，也增加了程序的可读性

```javascript
var arr = [], n = 4;
function backtrack(k) {
    if(k > n) {
        console.log(arr);
    } else {
        for(var i = 1;i <= n; i++) {
            arr[k-1] = i;
            if(check2(arr, k-1)) {
                backtrack(k + 1);
            }
        }
    }
}

backtrack(1);
//[ 2, 4, 1, 3 ]
//[ 3, 1, 4, 2 ]
```

## 华而不实的amb

什么是 amb ？给它一个数据列表，它能返回满足约束条件的成功情况的一种方式，没有成功情况就会失败，当然，它可以返回所有的成功情况。笔者写了上面那么多的重点，就是为了在这里推荐这个amb算法，它适合处理简单的回溯场景，很有趣，让我们来看看它是怎么工作的

首先来处理一个小问题，寻找相邻字符串：拿到几组字符串数组，每个数组拿出一个字符串，前一个字符串的末位字符与后一个字符串的首位字符相同，满足条件则输出这组新取出来的字符串

```javascript
ambRun(function(amb, fail) {

    // 约束条件方法
    function linked(s1, s2) {
        return s1.slice(-1) == s2.slice(0, 1);
    }

    // 注入数据列表
    var w1 = amb(["the", "that", "a"]);
    var w2 = amb(["frog", "elephant", "thing"]);
    var w3 = amb(["walked", "treaded", "grows"]);
    var w4 = amb(["slowly", "quickly"]);

    // 执行程序
    if (!(linked(w1, w2) && linked(w2, w3) && linked(w3, w4))) fail();

    console.log([w1, w2, w3, w4].join(' '));
    // "that thing grows slowly"
});
```

看起来超级简洁有没有！不过使用的前提是，你不在乎性能，它真的是很浪费时间！

下面是它的 javascript 实现，有兴趣可以研究研究它是怎么把回溯抽出来的

```javascript
function ambRun(func) {
    var choices = [];
    var index;

    function amb(values) {
        if (values.length == 0) {
            fail();
        }
        if (index == choices.length) {
            choices.push({i: 0,
                count: values.length});
        }
        var choice = choices[index++];
        return values[choice.i];
    }

    function fail() { throw fail; }

    while (true) {
        try {
            index = 0;
            return func(amb, fail);
        } catch (e) {
            if (e != fail) {
                throw e;
            }
            var choice;

            while ((choice = choices.pop()) && ++choice.i == choice.count) {}
            if (choice == undefined) {
                return undefined;
            }
            choices.push(choice);
        }
    }
}
```

以及使用 amb 实现的八皇后问题的具体代码

```javascript
ambRun(function(amb, fail){
    var N = 4;
    var arr = [];
    var turn = [];
    for(var n = 0; n < N; n++) {
        turn[turn.length] = n + 1;
    }
    while(n--) {
        arr[arr.length] = amb(turn);
    }
    for (var i = 0; i < N; ++i) {
        for (var j = i + 1; j < N; ++j) {
            var a = arr[i], b = arr[j];
            if (a == b || Math.abs(a - b) == j - i) fail();
        }
    }
    console.log(arr);
    fail();
});
```

## 八皇后问题的JavaScript解法

这是八皇后问题的JavaScript解法，整个程序都没用for循环，都是靠递归来实现的，充分运用了Array对象的map, reduce, filter, concat, slice方法

```javascript
'use strict';
var queens = function (boarderSize) {
  // 用递归生成一个start到end的Array
  var interval = function (start, end) {
    if (start > end) { return []; }
    return interval(start, end - 1).concat(end);
  };
  // 检查一个组合是否有效
  var isValid = function (queenCol) {
    // 检查两个位置是否有冲突
    var isSafe = function (pointA, pointB) {
      var slope = (pointA.row - pointB.row) / (pointA.col - pointB.col);
      if ((0 === slope) || (1 === slope) || (-1 === slope)) { return false; }
      return true;
    };
    var len = queenCol.length;
    var pointToCompare = {
      row: queenCol[len - 1],
      col: len
    };
    // 先slice出除了最后一列的数组，然后依次测试每列的点和待测点是否有冲突，最后合并测试结果
    return queenCol
      .slice(0, len - 1)
      .map(function (row, index) {
        return isSafe({row: row, col: index + 1}, pointToCompare);
      })
      .reduce(function (a, b) {
        return a && b;
      });
  };
  // 递归地去一列一列生成符合规则的组合
  var queenCols = function (size) {
    if (1 === size) {
      return interval(1, boarderSize).map(function (i) { return [i]; });
    }
    // 先把之前所有符合规则的列组成的集合再扩展一列，然后用reduce降维，最后用isValid过滤掉不符合规则的组合
    return queenCols(size - 1)
      .map(function (queenCol) {
        return interval(1, boarderSize).map(function (row) {
          return queenCol.concat(row);
        });
      })
      .reduce(function (a, b) {
        return a.concat(b);
      })
      .filter(isValid);
  };
  // queens函数入口
  return queenCols(boarderSize);
};

console.log(queens(8));
// 输出结果:
// [ [ 1, 5, 8, 6, 3, 7, 2, 4 ],
//   [ 1, 6, 8, 3, 7, 4, 2, 5 ],
//   ...
//   [ 8, 3, 1, 6, 2, 5, 7, 4 ],
//   [ 8, 4, 1, 3, 6, 2, 7, 5 ] ]
```

## 总结

回溯算法是很常用的基本算法，认真掌握是没有错的，笔者也是一边学习一边写下本篇，学习内容来源

[八皇后问题](https://zh.wikipedia.org/wiki/%E5%85%AB%E7%9A%87%E5%90%8E%E9%97%AE%E9%A2%98)
[五大常用算法之四：回溯法](http://www.cnblogs.com/steven_oyj/archive/2010/05/22/1741376.html)
[回溯法——八皇后问题](http://www.cnblogs.com/houkai/p/3480940.html)
[Amb() in JavaScript](http://mihai.bazon.net/blog/amb-in-javascript)
[八皇后问题的 JavaScript 解法](https://yyqian.com/post/1442113857711/)