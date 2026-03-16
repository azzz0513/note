## Basic Concepts
索引机制是用于加速对所需数据的访问
![[images/Pasted image 20260316213521.png]]
Search Key：用于查找文件中记录的属性集的属性

## Ordered Indices
在有序索引中，索引条目按搜索键值排序

==**primary index**==：在按顺序排序的文件中，其搜索键指定文件的顺序的索引
- 也称为聚簇索引（clustering index）
- 主索引的搜索键通常但不一定是主键
![[images/Pasted image 20260316214141.png]]

==**secondary index**==：其搜索键指定不同于文件顺序的顺序的索引。
- 也成为非聚簇索引（non-clustering indexo）
![[images/Pasted image 20260316214155.png]]

==**dense index**==（密集索引）：数据文件中每一个不同的搜索码值，在索引文件中都有且仅有一个对应的索引项
![[images/Pasted image 20260316214256.png]]
![[images/Pasted image 20260316214429.png]]

==**sparse index**==（稀疏索引）：只为数据块 / 数据页建立索引项，不是每条数据都建索引
![[images/Pasted image 20260316214550.png]]![[images/Pasted image 20260316214633.png]]

![[images/Pasted image 20260316214716.png]]
![[images/Pasted image 20260316214725.png]]

## index update
### delete
- 密集索引：删除搜索键类似于删除文件记录
- 稀疏索引：如果索引中存在搜索键的条目，则通过用文件中的下一个搜索键替换索引中的条目来删除该条目。
	- 如果下一个搜索键已经有了索引项，则该项将被删除而不是被替换

### insert
- 单层索引插入：
	- 密集索引：如果索引中没有搜索键，就插入它‘
	- 稀疏索引：如果索引为文件中的每个块存储一个条目，除非创建新块，否则无需对索引做任何修改
		- 如果创建新块，则将该块中出现的第一个搜索键插入索引中
- 多层插入与删除：是单层的简单扩展

## secondary index
![[images/Pasted image 20260316215427.png]]
![[images/Pasted image 20260316215445.png]]
==二级索引必须是密集索引==

## B+树索引文件
![[images/Pasted image 20260316215610.png]]
![[images/Pasted image 20260316215621.png]]
![[images/Pasted image 20260316215643.png]]
![[images/Pasted image 20260316215750.png]]
![[images/Pasted image 20260316220040.png]]
![[images/Pasted image 20260316220133.png]]
![[images/Pasted image 20260316220149.png]]
![[images/Pasted image 20260316220716.png]]
![[images/Pasted image 20260316220811.png]]
![[images/Pasted image 20260316220927.png]]


![[images/Pasted image 20260316221041.png]]

## multiple-key access
![[images/Pasted image 20260316221059.png]]![[images/Pasted image 20260316221201.png]]
![[images/Pasted image 20260316221214.png]]

## B+-Tree update
![[images/Pasted image 20260316221359.png]]
![[images/Pasted image 20260316221412.png]]