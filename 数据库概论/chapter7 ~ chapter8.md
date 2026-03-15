### modeling
![[images/Pasted image 20260315150318.png]]

![[images/Pasted image 20260315150507.png]]
- relationship（关系）：两个（或多个）实体之间的一次关联
	- 是具体的、个别的关系
- relationship set（关系集）：同一类型的所有关系的集合
	- 是整体的、全部的关系
	- 对应数据库的一张表

![[images/Pasted image 20260315151223.png]]
![[images/Pasted image 20260315151449.png]]

### binary relationship（二分关系）
![[images/Pasted image 20260315151537.png]]

### attribute
![[images/Pasted image 20260315151647.png]]
![[images/Pasted image 20260315151801.png]]

### mapping cardinality constraints
![[images/Pasted image 20260315151853.png]]
![[images/Pasted image 20260315151905.png]]
![[images/Pasted image 20260315151926.png]]

### ==keys==
![[images/Pasted image 20260315151957.png]]
![[images/Pasted image 20260315152617.png]]
关系集的key：用来唯一标识一条“关系”的属性集

### redundant attribute（冗余属性）
可以通过其他属性计算/推导出来的属性，不是必须存在的属性
![[images/Pasted image 20260315153140.png]]

### E-R Diagrams
![[images/Pasted image 20260315153241.png]]
![[images/Pasted image 20260315153331.png]]

### roles（角色）
关系中，实体所扮演的“身份/角色”
通常用于：同一个实体自己和自己关联
![[images/Pasted image 20260315153626.png]]

### cardinality constraints
![[images/Pasted image 20260315153714.png]]
![[images/Pasted image 20260315153747.png]]
![[images/Pasted image 20260315153756.png]]
![[images/Pasted image 20260315153925.png]]
![[images/Pasted image 20260315154000.png]]
![[images/Pasted image 20260315154043.png]]
![[images/Pasted image 20260315154105.png]]

![[images/Pasted image 20260315154128.png]]
![[images/Pasted image 20260315154357.png]]

### weak entity sets（弱实体集）
![[images/Pasted image 20260315154435.png]]
不能单独存在，必须依赖另一个实体才能存在的实体集，叫弱实体集
> 自己没有足够的属性形成唯一主键，必须依附于强实体

![[images/Pasted image 20260315155116.png]]
![[images/Pasted image 20260315155844.png]]

### reduction to relation schemas
![[images/Pasted image 20260315155958.png]]![[images/Pasted image 20260315160020.png]]

### entities → schemas
![[images/Pasted image 20260315160108.png]]
![[images/Pasted image 20260315160329.png]]

#### relationships → schemas
![[images/Pasted image 20260315160509.png]]
![[images/Pasted image 20260315160527.png]]

### optimization，remove redundancy schemas
![[images/Pasted image 20260315160810.png]]
![[images/Pasted image 20260315161521.png]]

![[images/Pasted image 20260315161535.png]]

![[images/Pasted image 20260315161601.png]]

![[images/Pasted image 20260315161644.png]]
![[images/Pasted image 20260315161727.png]]
![[images/Pasted image 20260315161804.png]]
![[images/Pasted image 20260315161816.png]]

![[images/Pasted image 20260315162502.png]]

![[images/Pasted image 20260315162545.png]]
### Design Issues
![[images/Pasted image 20260315163529.png]]
![[images/Pasted image 20260315163542.png]]
![[images/Pasted image 20260315163553.png]]
![[images/Pasted image 20260315163621.png]]
![[images/Pasted image 20260315163641.png]]
![[images/Pasted image 20260315163703.png]]
![[images/Pasted image 20260315164129.png]]
