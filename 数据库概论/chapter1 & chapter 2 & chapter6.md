# Chapter1

### 文件系统vs数据库系统
- 文件系统：
	- 数据冗余
	- 信息更改
	- 关联性
	- 修改繁琐
	- 权限更麻烦
	- 多个表格交叉查询
- 数据库系统：
	- 面向全组织的复杂数据结构，数据冗余度小，易扩充较高的数据及程序独立性，功能统一易管理


# Chapter2 & Chapter6
## 关系模式介绍
- 定义域（domain）：每个属性允许的值集合称为该属性的定义域
- 原子值（atomic）：属性值（通常）要求为原子值；也就是说，是不可分割的
- null（空值）：特殊值null是每个域的成员
- 空值会使许多操作的定义变得复杂

A1属性名字；D1属性A1的取值范围；a1，范围D1中的一个元素
`Example : A1  = Name, D1 =  all possible names, a1 = Einstein.`

关系模式（relation schema）：
![[images/Pasted image 20260314142017.png]]
![[images/Pasted image 20260314143228.png]]

关系实例（relation instance）：关系的当前值（关系实例）由表指定
r中的一个元素t是一个元组（tuple），由表中的一行表示
![[images/Pasted image 20260314142240.png]]
元组的顺序无关紧要（元组可以按任意顺序存储）

==**Database**==
数据库（database）由多个关系组成
例：关于大学的信息被划分为多个部分：导师、学生、助教
- 糟糕的设计：univ(instructor-ID, name, dept_name, salary, student_id)
	- 信息重复：多个学生可能有同一个导师
	- 需要控制：一个学生可能没有导师

![[images/Pasted image 20260314142728.png]]
超键（superkey）：若干属性的组合可以唯一确定一个实例
- 只要**能区分开每一条记录**，它就是超键。
- 超键可以一个属性，也可以多个属性组合
- 可以包含多余属性
- 只要能唯一标识就行，不管冗余不冗余

候选键（candidate key）：最小的超键（去掉其中任意一个属性就不唯一了）

主键（primary key）：从候选键中选一个当主键

外键（foreign key）：一张表中的字段，引用另一张表的主键
- 例：`student(stu_id, name)`，`score(score_id, stu_id, socre)`
	- 其中的score表中的stu_id就是外键
- 外键约束到底约束什么东西：
	- 不能引用不存在的东西
	- 不能随便删除被引用的数据

## 关系查询语言（relation query language）
是用户用来从数据库查询获取数据的语言


## 关系代数（Relation Algebra）
![[images/Pasted image 20260314144304.png]]
![[images/Pasted image 20260314145138.png]]

![[images/Pasted image 20260314144349.png]]
![[images/Pasted image 20260314144359.png]]
![[images/Pasted image 20260314144408.png]]
![[images/Pasted image 20260314144424.png]]
![[images/Pasted image 20260314144456.png]]
![[images/Pasted image 20260314144508.png]]

### 自然连接（natural join）
自然连接：两张表中，把 “同名同类型” 的字段自动匹配，只保留相同值的行，并且重复字段只留一个
![[images/Pasted image 20260314145022.png]]

![[images/Pasted image 20260314144833.png]]

![[images/Pasted image 20260314145708.png]]
![[images/Pasted image 20260314150201.png]]

![[images/Pasted image 20260314150226.png]]
![[images/Pasted image 20260314150239.png]]

![[images/Pasted image 20260314150308.png]]
![[images/Pasted image 20260314150332.png]]

![[images/Pasted image 20260314150345.png]]
![[images/Pasted image 20260314150358.png]]

![[images/Pasted image 20260314150447.png]]

![[images/Pasted image 20260314150502.png]]
![[images/Pasted image 20260314150520.png]]

![[images/Pasted image 20260314150552.png]]

![[images/Pasted image 20260314150614.png]]
![[images/Pasted image 20260314150910.png]]
==**如果两个表没有同名列，自然连接会退化为笛卡尔积**==

![[images/Pasted image 20260314151634.png]]
要比较 “同一个表” 里两条不同记录的 salary，
必须给表起两个不同的名字，
否则数据库不知道你在比谁和谁！

![[images/Pasted image 20260314152224.png]]
**theta join**：带任意条件的连接

θ就是比较运算符：
`> < >= <= = !=`

theta join怎么计算：
1. 先做笛卡尔积（所有行两两组合）
2. 再筛选满足θ条件的行
![[images/Pasted image 20260314152714.png]]


==**指派操作（assignment operation）**==
![[images/Pasted image 20260314152857.png]]
把一个关系代数查询的结果，临时存到一个 “新关系 / 变量” 里，方便后面重复使用。

把右边的查询结果，存到左边的名字中

==**外连接（outer join）**==
![[images/Pasted image 20260314153457.png]]
![[images/Pasted image 20260314153522.png]]
![[images/Pasted image 20260314153600.png]]
![[images/Pasted image 20260314153611.png]]

**==空值（null）==**
![[images/Pasted image 20260314153728.png]]
![[images/Pasted image 20260314153804.png]]
==只要表达式里有 **NULL**，结果就是 **unknown**。==
任何值和NULL比较→unknown

判断NULL必须使用：
- `IS NULL`
- `IS NOT NULL`
- 不能使用`=`或`!=`

![[images/Pasted image 20260314154236.png]]
![[images/Pasted image 20260314154300.png]]
**r ÷ s = 在 r 里，找【匹配了 s 中**所有**记录】的那些值。**
![[images/Pasted image 20260314154650.png]]

![[images/Pasted image 20260314155140.png]]
- 普通投影：只列出字段
- 广义投影：不仅列字段，还能做运算、加常量、改值


![[images/Pasted image 20260314155340.png]]
![[images/Pasted image 20260314155410.png]]
聚合操作（aggregate operation）：
G（group by）
G前：G1，G2，……，Gn为用于分组的一系列字段
每个Fi为一个聚合函数，每个Ai为一个属性名，
（1）同一组中所有元组在G1,G2,...,Gn上的值相同；
（2）不同组中元组在G1,G2,...,Gn​上的值不同。
最后得到的关系模式为(G1​,G2​,...,Gn​,F1​(A1​),F2​(A2​),...,Fm​(Am​))。


![[images/Pasted image 20260314161207.png]]
