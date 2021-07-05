# 大O表示法（Big O）
一般用大O表示法来描述复杂度，它表示的是数据规模 n 对应的复杂度
大O表示法仅仅是一种粗略的分析模型，是一种估算，能帮助我们短时间内了解一个算法的执行效率

> 忽略常数、系数、低阶

1. 9 >> O(1)
2. 2n + 3 >> O(n)
3. n^2+ 2n + 6 >> O(n^2)
4. 4n^3+ 3n^2+ 22n + 100 >> O(n^3)
5. 对数阶一般省略底数,所以 log2(n) 、log9(n) 统称为 logn

## 常见的复杂度
|执行次数|复杂度|非正式术语|
|--|--|--|
|123|O(1)|常数阶|
|3n + 8|O(n)|线性阶|
|5n^2 + 8n + 9|O(n^2)|平方阶|
|5log2(n) + 250|O(logn)|对数阶|
|30n + 29nlog3(n) + 159|O(nlogn)|nlogn阶|
|12n^3 + 31n^2 + 220n + 1000|O(n^3)|立方阶|
|2^n|O(2^n)|指数阶|

O(1) < O(logn) < O(n) < O(nlogn) < O(n^2) < O(n^3) < O(2^n) < O(n!) < O(n^n)

## 借助函数生成工具对比复杂度的大小
[numberempire](https://zh.numberempire.com/graphingcalculator.php)

> 1、数据比较少的时候

![KekBVm](https://gitee.com/Liuccccc/oss/raw/master/2021/06/09/KekBVm.png)

> 2、数据量大的时候

![6LT9jC](https://gitee.com/Liuccccc/oss/raw/master/2021/06/09/6LT9jC.png)