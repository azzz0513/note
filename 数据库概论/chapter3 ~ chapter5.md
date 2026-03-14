## Data Definition Language
![[images/Pasted image 20260314163135.png]]![[images/Pasted image 20260314165741.png]]

定义域类型：
![[images/Pasted image 20260314165758.png]]

![[images/Pasted image 20260314165958.png]]

![[images/Pasted image 20260314170213.png]]

## Data Manipulation Language（DML）
![[images/Pasted image 20260314170708.png]]
==**一个SQL查询的结果是一个关系（relation）**==

![[images/Pasted image 20260314170823.png]]
SQL名称大小写不敏感
![[images/Pasted image 20260314170924.png]]
![[images/Pasted image 20260314171108.png]]

![[images/Pasted image 20260314171153.png]]
![[images/Pasted image 20260314171239.png]]

![[images/Pasted image 20260314171441.png]]
![[images/Pasted image 20260314171511.png]]
![[images/Pasted image 20260314171533.png]]
![[images/Pasted image 20260314195249.png]]
==**注意：不同表中的两个没关系的同名属性容易被错误的等同**==

![[images/Pasted image 20260314200315.png]]

![[images/Pasted image 20260314201227.png]]
![[images/Pasted image 20260314201246.png]]

![[images/Pasted image 20260314201326.png]]

![[images/Pasted image 20260314201434.png]]

![[images/Pasted image 20260314201448.png]]
三个操作都会自动去重
如果想要保留重复的数据，可以使用：
- `union all`
- `intersect all`
- `except all`
![[images/Pasted image 20260314201820.png]]

![[images/Pasted image 20260314201936.png]]


![[images/Pasted image 20260314202019.png]]
![[images/Pasted image 20260314202034.png]]

![[images/Pasted image 20260314202117.png]]
![[images/Pasted image 20260314202352.png]]

关键字执行顺序：
![[images/Pasted image 20260314202513.png]]
![[images/Pasted image 20260314202523.png|560]]
![[images/Pasted image 20260314203501.png]]


![[images/Pasted image 20260314202609.png]]
![[images/Pasted image 20260314202734.png]]
==注意点：==
1. with子句只能被select查询块引用
2. with子句的返回结果存到用户的临时表空间中，只做一次查询，反复使用,提高效率。
3. 在同级select前有多个查询定义的时候，第1个用with，后面的不用with，并且用逗号隔开。
4. 最后一个with 子句与下面的查询之间不能有逗号，只通过右括号分割,with 子句的查询必须用括号括起来

标量子查询（scalar subquery）：**标量子查询**是**返回结果只有 1 行 1 列**（单个值）的子查询，是子查询中最简单、最常用的类型。可当作普通字段来使用
![[images/Pasted image 20260314203010.png]]
![[images/Pasted image 20260314203246.png]]
- 错误原因：SQL的执行顺序是固定的：
	- 先执行WHERE
	- 后执行聚合函数
	- 所以在执行where时，还没算出平均工资，数据库根本不知道`avg(salary)`是多少，所以直接报错

![[images/Pasted image 20260314203731.png]]
![[images/Pasted image 20260314203807.png]]
![[images/Pasted image 20260314203854.png]]
![[images/Pasted image 20260314203922.png]]

主查询：直接呈现结果的代码
内查询：额外筛选条件的代码

![[images/Pasted image 20260314204148.png]]

![[images/Pasted image 20260314204305.png]]
![[images/Pasted image 20260314204230.png]]
![[images/Pasted image 20260314204242.png]]

![[images/Pasted image 20260314204346.png]]

![[images/Pasted image 20260314204511.png]]
![[images/Pasted image 20260314204524.png]]

