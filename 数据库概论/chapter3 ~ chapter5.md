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

### where
![[images/Pasted image 20260314171153.png]]
![[images/Pasted image 20260314171239.png]]

### 连接
![[images/Pasted image 20260314171441.png]]
![[images/Pasted image 20260314171511.png]]
![[images/Pasted image 20260314171533.png]]
![[images/Pasted image 20260314195249.png]]
==**注意：不同表中的两个没关系的同名属性容易被错误的等同**==

### 重命名
![[images/Pasted image 20260314200315.png]]

### 字符匹配
![[images/Pasted image 20260314201227.png]]
![[images/Pasted image 20260314201246.png]]

### order by
![[images/Pasted image 20260314201326.png]]
- ASC：升序（默认）
- DESC：降序
- 永远最后执行

### between
![[images/Pasted image 20260314201434.png]]

### set operation
![[images/Pasted image 20260314201448.png]]
三个操作都会自动去重
如果想要保留重复的数据，可以使用：
- `union all`
- `intersect all`
- `except all`
![[images/Pasted image 20260314201820.png]]

### null value
![[images/Pasted image 20260314201936.png]]

### 聚合函数
- 有group by时：按分组进行计算，返回多行结果
- 无group by时：整张表当一组，返回一行结果

==无分组时，SELECT **只能放聚合函数**，不能放普通字段==
❌ 错误：
```SQL
SELECT name, AVG(salary) FROM instructor;
```
报错！因为整张表只有一个平均值，但 name 有很多个，数据库不知道显示谁。

✅ 正确：
```SQL
SELECT AVG(salary) FROM instructor;
```

![[images/Pasted image 20260314202019.png]]
![[images/Pasted image 20260314202034.png]]

### group by
分组：把相同内容的行，合并为一组，然后对每组做统计计算
```SQL
SELECT 分组字段, 聚合函数(字段) 
FROM 表 
GROUP BY 分组字段;
```
**必须配合聚合函数使用：**
`AVG() / MAX() / MIN() / SUM() / COUNT()`

==**SELECT 里只能出现两种东西：**==
1. **GROUP BY 后面的分组字段**
2. **聚合函数**


### having
![[images/Pasted image 20260314202117.png]]
![[images/Pasted image 20260314202352.png]]

### 关键字执行顺序：
![[images/Pasted image 20260314202513.png]]
![[images/Pasted image 20260314202523.png|560]]
![[images/Pasted image 20260314203501.png]]

### with
![[images/Pasted image 20260314202609.png]]
![[images/Pasted image 20260314202734.png]]
==注意点：==
1. with子句只能被select查询块引用
2. with子句的返回结果存到用户的临时表空间中，只做一次查询，反复使用,提高效率。
3. 在同级select前有多个查询定义的时候，第1个用with，后面的不用with，并且用逗号隔开。
4. 最后一个with 子句与下面的查询之间不能有逗号，只通过右括号分割,with 子句的查询必须用括号括起来

### 子查询
标量子查询（scalar subquery）：**标量子查询**是**返回结果只有 1 行 1 列**（单个值）的子查询，是子查询中最简单、最常用的类型。可当作普通字段来使用
![[images/Pasted image 20260314203010.png]]
![[images/Pasted image 20260314203246.png]]
- 错误原因：SQL的执行顺序是固定的：
	- 先执行WHERE
	- 后执行聚合函数
	- 所以在执行where时，还没算出平均工资，数据库根本不知道`avg(salary)`是多少，所以直接报错

### 嵌套子查询
![[images/Pasted image 20260314203731.png]]
![[images/Pasted image 20260314203807.png]]
![[images/Pasted image 20260314203854.png]]
![[images/Pasted image 20260314203922.png]]

主查询：直接呈现结果的代码
内查询：额外筛选条件的代码

![[images/Pasted image 20260314204148.png]]

### some
![[images/Pasted image 20260314204305.png]]
![[images/Pasted image 20260314204230.png]]
![[images/Pasted image 20260314204242.png]]

### all
![[images/Pasted image 20260314204346.png]]

### exists & not exists
![[images/Pasted image 20260314204511.png]]
![[images/Pasted image 20260314204524.png]]
==**EXISTS (子查询) = 判断子查询有没有结果**==
- 有结果 → **TRUE**（满足条件）
- 没结果 → **FALSE**（不满足）

==**NOT EXISTS (子查询) = 子查询没结果才满足**==
- 有结果 → **FALSE**
- 没结果 → **TRUE**（满足条件）

![[images/Pasted image 20260314204828.png]]
查询（选了生物系所有课程的）学生
- 从student表中获取学生id、学生名字。使用distinct去重，确保一个学生只出现一次
- `where not exists(子查询)`：子查询结果为空时，这条学生记录才被记录
- `(生物系的所有课) except (这个学生选择的所有课程)`
- 总结：不存在生物系有，但是这个学生没选的课（这个学生选择了生物系的所有课）

### unique
![[images/Pasted image 20260314205622.png]]
unique关键字判断子查询结果中是否有重复的元组
- 对于空集的unique为“true”

### from
![[images/Pasted image 20260314205743.png]]

### delete
![[images/Pasted image 20260314205826.png]]
![[images/Pasted image 20260314205850.png]]
![[images/Pasted image 20260314205948.png]]
![[images/Pasted image 20260314210038.png]]

### insert
![[images/Pasted image 20260314210104.png]]
![[images/Pasted image 20260314210224.png]]

### update
![[images/Pasted image 20260314210315.png]]

### case
![[images/Pasted image 20260314210324.png]]
case关键字：就是SQL语句中的if else，用于做条件判断、字段分类
- 简单case（等值判断）：只判断等于某个值：
```SQL
CASE 字段 
	WHEN 值1 THEN 结果1 
	WHEN 值2 THEN 结果2 
	ELSE 其他结果 
END
```
```SQL
SELECT 
	score, 
	CASE score 
		WHEN 90 THEN '优秀' 
		WHEN 80 THEN '良好' 
		ELSE '其他' 
	END AS level 
FROM test;
```
- 搜索case（范围判断）：可以判断>、<、>=、<=、between等范围：
```SQL
CASE 
	WHEN 条件1 THEN 结果1 
	WHEN 条件2 THEN 结果2 
	ELSE 其他结果 
END
```
```SQL
SELECT 
	name, 
	score, 
	CASE 
		WHEN score >= 90 THEN 'A' 
		WHEN score >= 80 THEN 'B' 
		WHEN score >= 60 THEN 'C' 
		ELSE 'D' 
	END AS grade 
FROM student;
```

## 中级SQL（Intermediate SQL）
### joined relations
![[images/Pasted image 20260315141036.png]]
![[images/Pasted image 20260315141324.png]]
![[images/Pasted image 20260315141506.png]]

#### outer join（外连接）
![[images/Pasted image 20260315141304.png]]
![[images/Pasted image 20260315141337.png]]
![[images/Pasted image 20260315141347.png]]
![[images/Pasted image 20260315141517.png]]

![[images/Pasted image 20260315141618.png]]
### on
给Join指定连接规则：哪两个字段相等，表就怎么连
特点：
- 不管字段名是否相同都能使用
- 可以加额外条件
- 必须写表别名/表名。字段

### using
当两个表的连接字段完全一样时