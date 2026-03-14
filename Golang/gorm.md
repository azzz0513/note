### 连接
需要下载mysql的驱动：
```Go
go get -u gorm.io/driver/mysql
go get -u gorm.io/gorm
```

```Go
package main

import (
	"fmt"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
)

var DB *gorm.DB

func init() {
	username := "root"   // 账号
	password := "123456" // 密码
	host := "127.0.0.1"  // 数据库地址，可以是IP或域名
	port := "3306"       // 数据库端口
	dbname := "gorm"     // 数据库名
	timeout := "10s"     // 连接超时，10秒

	// root:123456@tcp(127.0.0.1:3306)/gorm?
	dsn := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?charset=utf8&parseTime=True&loc=Local&timeout=%s", username, password, host, port, dbname, timeout)
	// 连接MySQL，获得DB类型实例，用于后面的数据库读写操作
	db, err := gorm.Open(mysql.Open(dsn))
	// 连接失败
	if err != nil {
		panic("数据库连接失败, error: " + err.Error())
	}
	// 连接成功
	DB = db
}

func main() {
	fmt.Println(DB)
}
```

#### 高级配置
##### 跳过默认事务
为了确保数据一致性，GORM会在事务里执行写入操作（创建，更新，删除）。如果没有这方面的要求，就可以在初始化时禁用它，这样可以获得60%的性能提升。
```Go
db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{
	SkipDefaultTransaction: true,
})
```

##### 命名策略
gorm采用的命名策略是，表名是蛇形复数，字段名是蛇形单数
```Go
type Student struct {
	Name string
	Age  int
	ID   uint
}
```
gorm会为我们生成表结构：
```Go
CREATE TABLE `students` (`name` longtext, `age` bigint, `id` bigint unsigned)
```

我们也可以修改这些策略：
```Go
db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{
	//SkipDefaultTransaction: true,
	NamingStrategy: schema.NamingStrategy{
		TablePrefix:   "f_",  // 表名前缀
		SingularTable: false, // 单数表名
		NoLowerCase:   false, // 关闭小写切换
	},
})
```

##### 显示日志
gorm的默认日志是只打印错误和慢SQL
```Go
package main

import (
	"fmt"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
	"gorm.io/gorm/logger"
)

var DB *gorm.DB
var mysqlLogger logger.Interface

func init() {
	username := "root"   // 账号
	password := "123456" // 密码
	host := "127.0.0.1"  // 数据库地址，可以是IP或域名
	port := "3306"       // 数据库端口
	dbname := "gorm"     // 数据库名
	timeout := "10s"     // 连接超时，10秒

	mysqlLogger = logger.Default.LogMode(logger.Info)

	// root:123456@tcp(127.0.0.1:3306)/gorm?
	dsn := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?charset=utf8&parseTime=True&loc=Local&timeout=%s", username, password, host, port, dbname, timeout)
	// 连接MySQL，获得DB类型实例，用于后面的数据库读写操作
	db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{
		//SkipDefaultTransaction: true,
		//NamingStrategy: schema.NamingStrategy{
		//	TablePrefix:   "f_",  // 表名前缀
		//	SingularTable: false, // 单数表名
		//	NoLowerCase:   false, // 关闭小写切换
		//},
		//Logger: mysqlLogger,
	})
	// 连接失败
	if err != nil {
		panic("数据库连接失败, error: " + err.Error())
	}
	// 连接成功
	DB = db
}

type Student struct {
	Name string
	Age  int
	ID   uint
}

func main() {
	//DB = DB.Session(&gorm.Session{
	//	Logger: mysqlLogger,
	//})
	// 表的创建
	// 自动生成表结构
	DB.Debug().AutoMigrate(&Student{})
}
```

### gorm.DB
`gorm.DB`是GORM的核心类型，它代表了与数据库的连接和交互。所有与数据库的交互均需要通过`gorm.DB`对象来实现。
```Go
type DB struct {
	*Config
	Error        error
	RowsAffected int64
	Statement    *Statement
	// contains filtered or unexported fields
}
```

#### 初始化
使用`gorm.Open`函数来初始化`gorm.DB`
```Go
func Open(dialector Dialector, opts ...Option) (db *DB, err error)
```

#### Model
```Go
func (db *DB) Model(value interface{}) (tx *DB)
```
Model方法的主要作用是将指定的模型（结构体或实例）与当前数据库操作关联
```Go
// update all users's name to `hello`
db.Model(&User{}).Update("name", "hello")
// if user's primary key is non-blank, will use it as condition, then will only update that user's name to `hello`
db.Model(&user).Update("name", "hello")
```

#### Table
```Go
func (db *DB) Table(name string, args ...interface{}) (tx *DB)
```
Table方法的主要作用是指定当前数据库操作的目标表名

### 事务
要在事务中执行一系列操作，一般流程如下：
`Transaction`方法，形参是一个函数类型`func(tx *gorm.DB) error`，可以传入匿名函数。
`Transaction`方法简单理解就是将一系列操作封装在一个事务中。如果其中一个操作失败，则事务会回滚，所有之前的操作都会被撤销，并将错误传递给调用方。如果所有操作都成功，则事务会被提交。会自动处理事务的提交和回滚，无需手动调用`Begin`、`Commit`、`Rollback`
```Go
db.Transaction(func(tx *gorm.DB) error {
  // 在事务中执行一些 db 操作（从这里开始，您应该使用 'tx' 而不是 'db'）
  if err := tx.Create(&Animal{Name: "Giraffe"}).Error; err != nil {
    // 返回任何错误都会回滚事务
    return err
  }

  if err := tx.Create(&Animal{Name: "Lion"}).Error; err != nil {
    return err
  }

  // 返回 nil 提交事务
  return nil
})
```

#### 禁用默认事务
为了确保数据一致性，GORM会在事务里执行写入操作（创建、更新、删除）。如果没有这方面的要求，可以在初始化时禁用它，这将获得大约30%性能提升。
```Go
// 全局禁用
db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
  SkipDefaultTransaction: true,
})

// 持续会话模式
tx := db.Session(&Session{SkipDefaultTransaction: true})
tx.First(&user, 1)
tx.Find(&users)
tx.Model(&user).Update("Age", 18)
```

#### 嵌套事务
GORM支持嵌套事务，可以回滚较大事务内执行的一部分操作：
```Go
db.Transaction(func(tx *gorm.DB) error {
  tx.Create(&user1)

  tx.Transaction(func(tx2 *gorm.DB) error {
    tx2.Create(&user2)
    return errors.New("rollback user2") // Rollback user2
  })

  tx.Transaction(func(tx2 *gorm.DB) error {
    tx2.Create(&user3)
    return nil
  })

  return nil
})

// Commit user1, user3
```

#### 手动事务
```Go
// 开始事务
tx := db.Begin()

// 在事务中执行一些 db 操作（从这里开始，您应该使用 'tx' 而不是 'db'）
tx.Create(...)

// ...

// 遇到错误时回滚事务
tx.Rollback()

// 否则，提交事务
tx.Commit()
```

#### SavePoint、RollbackTo
GORM提供了`SavePoint`、`RollbackTo`来提供保存点以及回滚至保存点：
```Go
tx := db.Begin()
tx.Create(&user1)

tx.SavePoint("sp1")
tx.Create(&user2)
tx.RollbackTo("sp1") // Rollback user2

tx.Commit() // Commit user1
```

### Hook
#### 对象声明周期
Hook是在创建、查询、更新、删除等操作之前、之后调用的函数。
如果已经为模型定义了指定的方法，它会在创建、更新、查询、删除时自动被调用。如果任何回调返回错误，GORM将停止后续的操作并回滚事务。

- 函数签名：`func(*gorm.DB) error`

#### 创建对象
创建时可用的hook
```Go
// 开始事务
BeforeSave
BeforeCreate
// 关联前的 save
// 插入记录至 db
// 关联后的 save
AfterCreate
AfterSave
// 提交或回滚事务
```

#### 更新对象
更新时可用的hook
```Go
// 开始事务
BeforeSave
BeforeUpdate
// 关联前的 save
// 更新 db
// 关联后的 save
AfterUpdate
AfterSave
// 提交或回滚事务
```

#### 删除对象
删除时可用的hook
```Go
// 开始事务
BeforeDelete
// 删除 db 中的数据
AfterDelete
// 提交或回滚事务
```

#### 查询对象
查询时可用的hook
```Go
// 从 db 中加载数据
// Preloading (eager loading)
AfterFind
```

==在GORM中保存、删除操作会默认运行在事务上，因此在事务完成之前该事务中所作的更改是不可见的，如果您的钩子返回了任何错误，则修改将被回滚。==

### 会话


### 模型定义
模型是使用普通结构体定义的。 这些结构体可以包含具有基本Go类型、指针或这些类型的别名，甚至是自定义类型（只需要实现`database/sql`包中的`Scanner`和`Valuer`接口）。

#### 约定
- 主键：GORM使用名为ID的字段作为每个模型的默认主键。
- 表名：默认情况下，GORM将结构体名称转换为`snake_case`并为表名加上复数形式，如一个`User`结构体在数据库中的表名变为`users`。
- 列名：GORM自动将结构体字段名称转换为`snake_case`作为数据库中的别名。
- 时间戳字段：GORM使用字段`CreatedAt`和`UpdateAt`来自动跟踪记录的创建和更新时间。

==小写属性不会生成字段==
定义一张表：
```Go
type Student struct {
	ID 		uint // 默认使用ID作为主键
	Name 	string
	Email 	*stirng // 使用指针是为了存空值
}
```

#### 自动生成表结构
```Go
DB.AutoMigrate(&Student{})
```
`AutoMigrate`的逻辑是只新增，不删除，不修改（大小会修改）

#### gorm.Model
GORM提供了一个预定义的结构体，名为`gorm.Model`：
```Go
// gorm.Model 的定义
type Model struct {
	ID        uint           `gorm:"primaryKey"`
	CreatedAt time.Time
	UpdatedAt time.Time
	DeletedAt gorm.DeletedAt `gorm:"index"`
}
```
- 将其嵌入在您的结构体中: 我们可以直接在我们的结构体中嵌入`gorm.Model` ，以便自动包含这些字段。 这对于在不同模型之间保持一致性并利用GORM内置的约定非常有用。
- 包含的字段：
    - ID ：每个记录的唯一标识符（主键）。
	- CreatedAt ：在创建记录时自动设置为当前时间。
	- UpdatedAt：每当记录更新时，自动更新为当前时间。
	- DeletedAt：用于软删除（将记录标记为已删除，而实际上并未从数据库中删除）。

#### 嵌入结构体
对于匿名字段，GORM会将其字段包含在父结构体中，如：
```Go
type Author struct {
	Name  string
	Email string
}

type Blog struct {
	Author
	ID      int
	Upvotes int32
}
// 相等于
type Blog struct {
	ID      int64
	Name    string
	Email   string
	Upvotes int32
}
```

对于正常的结构体字段，也可以通过标签`embedded`将其嵌入，例如：
```Go
type Author struct {
    Name  string
    Email string
}

type Blog struct {
	ID      int
	Author  Author `gorm:"embedded"`
	Upvotes int32
}
// 等效于
type Blog struct {
	ID    int64
	Name  string
	Email string
	Upvotes  int32
}
```

#### 字段标签

| 标签名 | 说明 |
| :--: | :--: |
|column	|指定`db`列名|
| type | 列数据类型，推荐使用兼容性好的通用类型，例如：所有数据库都支持bool、int、uint、float、string、time、bytes并且可以和其他标签一起使用，例如：not null、size, autoIncrement… 像`varbinary(8)`这样指定数据库数据类型也是支持的。在使用指定数据库数据类型时，它需要是完整的数据库数据类型，如：`MEDIUMINT UNSIGNED not NULL AUTO_INCREMENT` |
| serializer | 指定将数据序列化或反序列化到数据库中的序列化器, 例如: serializer:json/gob/unixtime |
| size | 定义列数据类型的大小或长度，例如：`size: 256` |
| primaryKey | 将列定义为主键 |
| unique | 将列定义为唯一键 |
| default | 定义列的默认值 |
| precision | 指定列的精度 |
| scale | 指定列大小 |
| not null | 指定列为`NOT NULL` |
| autoIncrement	| 指定列为自动增长 |
| autoIncrementIncrement | 自动步长，控制连续记录之间的间隔 |
| embedded | 嵌套字段 |
| embeddedPrefix | 嵌入字段的列名前缀 |
| autoCreateTime | 创建时追踪当前时间，对于int字段，它会追踪时间戳秒数，您可以使用`nano/milli`来追踪纳秒、毫秒时间戳，例如：`autoCreateTime:nano` |
| autoUpdateTime | 创建/更新时追踪当前时间，对于int字段，它会追踪时间戳秒数，您可以使用`nano/milli`来追踪纳秒、毫秒时间戳，例如：`autoUpdateTime:milli` |
| index | 根据参数创建索引，多个字段使用相同的名称则创建复合索引
| uniqueIndex | 与index相同，但创建的是唯一索引 |
| check | 创建检查约束，例如：`check:age > 13` |
| <- | 设置字段写入的权限，`<-:create`只创建、`<-:update`只更新、`<-:false`无写入权限、`<-`创建和更新权限 |
| -> | 设置字段读的权限，`->:false`无读权限 |
| - | 忽略该字段，`-`表示无读写，`-:migration`表示无迁移权限，`-:all`表示无读写迁移权限 |
| comment | 迁移时为字段添 |




### 创建
#### 创建记录
==无法向`create`传递结构体，所以要传入数据的指针==
```Go
func main() {
	// 通过结构体进行创建
	student := Student{
		Name: "李四",
		Age:  22,
		ID:   456,
	}
	result := DB.Create(&student) // 通过数据的指针来创建

	fmt.Println(student.ID) // 返回插入数据的主键
	if result.Error != nil {
		fmt.Println(result.Error) // 返回error
		return
	}
	fmt.Println(result.RowsAffected) // 返回插入记录的条数
	
	//// 创建一条记录
	//DB.Create(&Student{
	//	Name: "李四",
	//	Age:  22,
	//	ID:   456,
	//})

	//// 创建多项记录
	// users := []Student{
	// 	{Name: "小明", Age: 18, ID: 1},
	// 	{Name: "小红", Age: 19, ID: 2},
	// }
	// result := DB.Create(&users)
	// if result.Error != nil {
	// 	fmt.Println(result.Error)
	// 	return
	// }
}
```

#### 用指定的字段创建记录
创建记录并为指定字段赋值：
```Go
// 创建记录并为指定字段赋值
users := []Student{
	{Name: "小明", Age: 18, ID: 1},
	{Name: "小红", Age: 19, ID: 2},
}

DB.Select("*").Create(&users)
```

创建记录并忽略传递给“Omit”的字段值（不会将Name的值传递给创建的记录的指定字段）：
```Go
DB.Omit("Name").Create(&users)
```

#### 批量插入
要高效地插入大量数据，可以将slice传递给`Create`方法。gorm将生成一条SQL来插入所有数据，以返回所有的主键值，并触发`Hook`方法。当这些记录可以被分割为多个批次时，GORM会开启一个事务来处理它们。
```Go
var users = []User{ {Name: "jinzhu1"}, {Name: "jinzhu2"}, {Name: "jinzhu3"}}
db.Create(&users)

for _, user := range users {
	user.ID // 1,2,3
}
```
也可以通过db.CreateInBatches方法来指定批量插入的批次大小。
```Go
var users = []User{ {Name: "jinzhu_1"}, ...., {Name: "jinzhu_10000"}}

// batch size 100
db.CreateInBatches(users, 100)
```

==使用`CreateBatchSize`选项初始化GORM实例后，此后进行创建&关联操作时所有的INSERT行为都会遵循初始化时的配置。==

#### 创建钩子
GORM允许用户通过实现这些接口`BeforeSave`，`BeforeCreate`，`AfterSave`，`AfterCreate`来自定义钩子。 这些钩子方法会在创建一条记录时被调用。
例：
```Go
func (u *User) BeforeCreate(tx *gorm.DB) (err error) {
  u.UUID = uuid.New()

    if u.Role == "admin" {
        return errors.New("invalid role")
    }
    return
}
```

如果想要跳过`Hooks`方法，可以使用`SkipHooks`会话模式，例：
```Go
DB.Session(&gorm.Session{SkipHooks: true}).Create(&user)

DB.Session(&gorm.Session{SkipHooks: true}).Create(&users)

DB.Session(&gorm.Session{SkipHooks: true}).CreateInBatches(users, 100)
```

#### 根据Map创建
GORM支持通过`map[string]interface{}`与`[]map[string]interface{}{}`来创建记录。
```Go
db.Model(&User{}).Create(map[string]interface{}{
	"Name": "jinzhu", "Age": 18,
})

// batch insert from `[]map[string]interface{}{}`
db.Model(&User{}).Create([]map[string]interface{}{
	{"Name": "jinzhu_1", "Age": 18},
	{"Name": "jinzhu_2", "Age": 20},
})
```
==当使用map来创建时，钩子方法不会执行，关联不会被保存且不会回写主键（即无法获取生成的主键值）。==


#### 默认值
可以通过结构体Tag`default`来定义字段的默认值：
```Go
type User struct {
	ID   int64
	Name string `gorm:"default:galeone"`
	Age  int64  `gorm:"default:18"`
}
```
这些默认值会被当作结构体字段的零值插入到数据库中

==当结构体的字段默认值是零值的时候比如0，''，false，这些字段值将不会被保存到数据库中，你可以使用指针类型或者Scanner/Valuer来避免这种情况。==
```Go
type User struct {
	gorm.Model
	Name string
	Age  *int           `gorm:"default:18"`
	Active sql.NullBool `gorm:"default:true"`
}
```


### 查询
#### 检索单个对象
GORM供了`First`、`Take`、`Last`方法，以便从数据库中检索单个对象。当查询数据库时它添加了LIMIT 1条件，且没有找到记录时，它会返回`ErrRecordNotFound`错误
```Go
// 获取第一条记录（主键升序）
db.First(&user)
// SELECT * FROM users ORDER BY id LIMIT 1;

// 获取一条记录，没有指定排序字段
db.Take(&user)
// SELECT * FROM users LIMIT 1;

// 获取最后一条记录（主键降序）
db.Last(&user)
// SELECT * FROM users ORDER BY id DESC LIMIT 1;

result := db.First(&user)
result.RowsAffected // 返回找到的记录数
result.Error        // returns error or nil

// 检查 ErrRecordNotFound 错误
errors.Is(result.Error, gorm.ErrRecordNotFound)
```
如果想避免`ErrRecordNotFound`错误，可以使用`Find`，如`db.Limit(1).Find(&user)`，`Find`方法会接受struct和slice的数据。

对单个对象使用`Find`而不带limit，`db.Find(&user)`将会查询整个表并且只返回第一个对象，只是性能不高而且结果不确定。

`First`和`Last`方法会按主键排序找到第一条记录和最后一条记录 (分别)。只有在目标`struct`是指针或者通过`db.Model()`指定model时，该方法才有效。此外，如果相关model没有定义主键，那么将按model的第一个字段进行排序。例如：
```Go
var user User
var users []User

// works because destination struct is passed in
db.First(&user)
// SELECT * FROM `users` ORDER BY `users`.`id` LIMIT 1

// works because model is specified using `db.Model()`
result := map[string]interface{}{}
db.Model(&User{}).First(&result)
// SELECT * FROM `users` ORDER BY `users`.`id` LIMIT 1

// doesn't work
result := map[string]interface{}{}
db.Table("users").First(&result)

// 但可以配合 Take 使用
result := map[string]interface{}{}
db.Table("users").Take(&result)

// 根据第一个字段排序
type Language struct {
  Code string
  Name string
}
db.First(&Language{})
// SELECT * FROM `languages` ORDER BY `languages`.`code` LIMIT 1
```

##### 根据主键检索
如果主键是数字类型，可以使用内联条件来检索对象。当使用字符串时，需要额外的注意来避免SQL注入。
```Go
db.First(&user, 10)
// SELECT * FROM users WHERE id = 10;

db.First(&user, "10")
// SELECT * FROM users WHERE id = 10;

db.Find(&users, []int{1,2,3})
// SELECT * FROM users WHERE id IN (1,2,3);
```

如果主键是字符串（如uuid），查询将被写成：
```Go
db.First(&user, "id = ?", "1b74413f-f3b8-409f-ac47-e8c062e3472a")
// SELECT * FROM users WHERE id = "1b74413f-f3b8-409f-ac47-e8c062e3472"
```

当目标对象有一个主键值时，将使用主键构建查询条件，例如：
```Go
var user = User{ID: 10}
db.First(&user)
// SELECT * FROM users WHERE id = 10;

var result User
db.Model(User{ID: 10}).First(&result)
// SELECT * FROM users WHERE id = 10;
```

#### 检索全部对象
```Go
// Get all records
result := db.Find(&users)
// SELECT * FROM users;

result.RowsAffected // 返回找到的记录数，相当于 `len(users)`
result.Error        // returns error
```

#### 条件
##### String条件
```Go
// 获取第一条匹配的记录
db.Where("name = ?", "jinzhu").First(&user)
// SELECT * FROM users WHERE name = 'jinzhu' ORDER BY id LIMIT 1;

// 获取全部匹配的记录
db.Where("name <> ?", "jinzhu").Find(&users)
// SELECT * FROM users WHERE name <> 'jinzhu';

// IN
db.Where("name IN ?", []string{"jinzhu", "jinzhu 2"}).Find(&users)
// SELECT * FROM users WHERE name IN ('jinzhu','jinzhu 2');

// LIKE
db.Where("name LIKE ?", "%jin%").Find(&users)
// SELECT * FROM users WHERE name LIKE '%jin%';

// AND
db.Where("name = ? AND age >= ?", "jinzhu", "22").Find(&users)
// SELECT * FROM users WHERE name = 'jinzhu' AND age >= 22;

// Time
db.Where("updated_at > ?", lastWeek).Find(&users)
// SELECT * FROM users WHERE updated_at > '2000-01-01 00:00:00';

// BETWEEN
db.Where("created_at BETWEEN ? AND ?", lastWeek, today).Find(&users)
// SELECT * FROM users WHERE created_at BETWEEN '2000-01-01 00:00:00' AND '2000-01-08 00:00:00';
```

如果对象设置了主键，条件查询将不会覆盖主键的值，而是用And连接条件：
```Go
var user = User{ID: 10}
db.Where("id = ?", 20).First(&user)
// SELECT * FROM users WHERE id = 10 and id = 20 ORDER BY id ASC LIMIT 1
```
这个查询将会给出`record not found`错误 所以，在你想要使用例如`user`这样的变量从数据库中获取新值前，需要将例如`id`这样的主键设置为nil。

##### Struct&Map条件
```Go
// Struct
db.Where(&User{Name: "jinzhu", Age: 20}).First(&user)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 20 ORDER BY id LIMIT 1;

// Map
db.Where(map[string]interface{}{"name": "jinzhu", "age": 20}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 20;

// 主键切片条件
db.Where([]int64{20, 21, 22}).Find(&users)
// SELECT * FROM users WHERE id IN (20, 21, 22);
```
当使用结构作为条件查询时，GORM只会查询非零值字段。这意味着如果您的字段值为0、''、false 或其他零值，该字段不会被用于构建查询条件，例如：
```Go
db.Where(&User{Name: "jinzhu", Age: 0}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu";
```
假如需要查询零值字段可以使用map来构建查询条件，例如：
```Go
db.Where(map[string]interface{}{"Name": "jinzhu", "Age": 0}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 0;
```

##### Not条件
用法与`Where`类似
```Go
db.Not("name = ?", "jinzhu").First(&user)
// SELECT * FROM users WHERE NOT name = "jinzhu" ORDER BY id LIMIT 1;

// Not In
db.Not(map[string]interface{}{"name": []string{"jinzhu", "jinzhu 2"}}).Find(&users)
// SELECT * FROM users WHERE name NOT IN ("jinzhu", "jinzhu 2");

// Struct
db.Not(User{Name: "jinzhu", Age: 18}).First(&user)
// SELECT * FROM users WHERE name <> "jinzhu" AND age <> 18 ORDER BY id LIMIT 1;

// 不在主键切片中的记录
db.Not([]int64{1,2,3}).First(&user)
// SELECT * FROM users WHERE id NOT IN (1,2,3) ORDER BY id LIMIT 1;
```

##### Or条件
```Go
db.Where("role = ?", "admin").Or("role = ?", "super_admin").Find(&users)
// SELECT * FROM users WHERE role = 'admin' OR role = 'super_admin';

// Struct
db.Where("name = 'jinzhu'").Or(User{Name: "jinzhu 2", Age: 18}).Find(&users)
// SELECT * FROM users WHERE name = 'jinzhu' OR (name = 'jinzhu 2' AND age = 18);

// Map
db.Where("name = 'jinzhu'").Or(map[string]interface{}{"name": "jinzhu 2", "age": 18}).Find(&users)
// SELECT * FROM users WHERE name = 'jinzhu' OR (name = 'jinzhu 2' AND age = 18);
```

##### 选择特定字段
```Go
db.Select("name", "age").Find(&users)
// SELECT name, age FROM users;

db.Select([]string{"name", "age"}).Find(&users)
// SELECT name, age FROM users;

db.Table("users").Select("COALESCE(age,?)", 42).Rows()
// SELECT COALESCE(age,'42') FROM users;
```

##### Order
指定从数据库检索记录时的排序方式
```Go
db.Order("age desc, name").Find(&users)
// SELECT * FROM users ORDER BY age desc, name;

// 多个 order
db.Order("age desc").Order("name").Find(&users)
// SELECT * FROM users ORDER BY age desc, name;

db.Clauses(clause.OrderBy{
  Expression: clause.Expr{SQL: "FIELD(id,?)", Vars: []interface{}{[]int{1, 2, 3}}, WithoutParentheses: true},
}).Find(&User{})
// SELECT * FROM users ORDER BY FIELD(id,1,2,3)
```

##### Limit&Offset
`Limit`指定获取记录的最大数量`Offset`指定在开始返回记录之前要跳过的记录数量
```Go
db.Limit(3).Find(&users)
// SELECT * FROM users LIMIT 3;

// 通过 -1 消除 Limit 条件
db.Limit(10).Find(&users1).Limit(-1).Find(&users2)
// SELECT * FROM users LIMIT 10; (users1)
// SELECT * FROM users; (users2)

db.Offset(3).Find(&users)
// SELECT * FROM users OFFSET 3;

db.Limit(10).Offset(5).Find(&users)
// SELECT * FROM users OFFSET 5 LIMIT 10;

// 通过 -1 消除 Offset 条件
db.Offset(10).Find(&users1).Offset(-1).Find(&users2)
// SELECT * FROM users OFFSET 10; (users1)
// SELECT * FROM users; (users2)
```

#### Scan
Scan结果到struct，用法与Find类似
```Go
type Result struct {
	Name string
	Age  int
}

var result Result
db.Table("users").Select("name", "age").Where("name = ?", "Antonio").Scan(&result)

// 原生 SQL
db.Raw("SELECT name, age FROM users WHERE name = ?", "Antonio").Scan(&result)
```


### 更新
#### 保存所有字段
`Save`会保存所有的字段，即使字段是零值
```Go
db.First(&user)

// user的id是111
user.Name = "jinzhu 2"
user.Age = 100
db.Save(&user)
// UPDATE users SET name='jinzhu 2', age=100, birthday='2016-01-01', updated_at = '2013-11-17 21:34:10' WHERE id=111;
```

#### 更新单个列
当使用`Update`更新单个列时，你需要指定条件，否则会返回`ErrMissingWhereClause`错误。当使用了`Model`方法，且该对象主键有值，该值会被用于构建条件，例如：
```Go
// 条件更新
db.Model(&User{}).Where("active = ?", true).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE active=true;

// User 的 ID 是 `111`
db.Model(&user).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111;

// 根据条件和 model 的值进行更新
db.Model(&user).Where("active = ?", true).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111 AND active=true;
```

#### 更新多列
`Updates`方法支持`struct`和`map[string]interface{}`参数。即使用`struct`更新时，默认情况下，GORM只会更新非零值的字段。
```Go
// 根据 `struct` 更新属性，只会更新非零值的字段
db.Model(&user).Updates(User{Name: "hello", Age: 18, Active: false})
// UPDATE users SET name='hello', age=18, updated_at = '2013-11-17 21:34:10' WHERE id = 111;

// 根据 `map` 更新属性
db.Model(&user).Updates(map[string]interface{}{"name": "hello", "age": 18, "actived": false})
// UPDATE users SET name='hello', age=18, actived=false, updated_at='2013-11-17 21:34:10' WHERE id=111;
```
==当通过struct更新时，GORM只会更新非零字段。如果想确保指定字段被更新，应该使用Select更新选定字段，或使用map来完成更新操作。==

#### 更新选定字段
如果想要在更新时选定、忽略某些字段，可以使用`Select`、`Omit`
```Go
// Select 和 Map
// User's ID is `111`:
db.Model(&user).Select("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "actived": false})
// UPDATE users SET name='hello' WHERE id=111;

db.Model(&user).Omit("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "actived": false})
// UPDATE users SET age=18, actived=false, updated_at='2013-11-17 21:34:10' WHERE id=111;

// Select 和 Struct （可以选中更新零值字段）
db.Model(&result).Select("Name", "Age").Updates(User{Name: "new_name", Age: 0})
// UPDATE users SET name='new_name', age=0 WHERE id=111;
```

#### 更新钩子
对于更新操作，GORM支持`BeforeSave`、`BeforeUpdate`、`AfterSave`、`AfterUpdate`钩子，
这些方法将在更新记录时被调用。
```Go
func (u *User) BeforeUpdate(tx *gorm.DB) (err error) {
    if u.Role == "admin" {
        return errors.New("admin user not allowed to update")
    }
    return
}
```

#### 批量更新
如果尚未通过`Model`指定记录的主键，则GORM会执行批量操作。
```Go
// 根据 struct 更新
db.Model(User{}).Where("role = ?", "admin").Updates(User{Name: "hello", Age: 18})
// UPDATE users SET name='hello', age=18 WHERE role = 'admin;

// 根据 map 更新
db.Table("users").Where("id IN ?", []int{10, 11}).Updates(map[string]interface{}{"name": "hello", "age": 18})
// UPDATE users SET name='hello', age=18 WHERE id IN (10, 11);
```

#### 更新的记录数
获取受更新影响的行数
```Go
// 通过 `RowsAffected` 得到更新的记录数
result := db.Model(User{}).Where("role = ?", "admin").Updates(User{Name: "hello", Age: 18})
// UPDATE users SET name='hello', age=18 WHERE role = 'admin;

result.RowsAffected // 更新的记录数
result.Error        // 更新的错误
```

#### 检查字段是否变更
GORM提供了`Changed`方法，它可以被用在`Before Update Hook`里，它会返回字段是否有变更的布尔值
`Changed`方法只能与`Update`、`Updates`方法一起使用，并且它只是检查Model对象字段的值与`Update`、`Updates`的值是否相等，如果值有变更，且字段没有被忽略，则返回true。
```Go
func (u *User) BeforeUpdate(tx *gorm.DB) (err error) {
  // 如果 Role 字段有变更
    if tx.Statement.Changed("Role") {
    return errors.New("role not allowed to change")
    }

	if tx.Statement.Changed("Name", "Admin") { // 如果 Name 或 Role 字段有变更
		tx.Statement.SetColumn("Age", 18)
	}

	// 如果任意字段有变更
    if tx.Statement.Changed() {
        tx.Statement.SetColumn("RefreshedAt", time.Now())
    }
    return nil
}

db.Model(&User{ID: 1, Name: "jinzhu"}).Updates(map[string]interface{"name": "jinzhu2"})
// Changed("Name") => true
db.Model(&User{ID: 1, Name: "jinzhu"}).Updates(map[string]interface{"name": "jinzhu"})
// Changed("Name") => false, 因为 `Name` 没有变更
db.Model(&User{ID: 1, Name: "jinzhu"}).Select("Admin").Updates(map[string]interface{
  "name": "jinzhu2", "admin": false,
})
// Changed("Name") => false, 因为 `Name` 没有被 Select 选中并更新

db.Model(&User{ID: 1, Name: "jinzhu"}).Updates(User{Name: "jinzhu2"})
// Changed("Name") => true
db.Model(&User{ID: 1, Name: "jinzhu"}).Updates(User{Name: "jinzhu"})
// Changed("Name") => false, 因为 `Name` 没有变更
db.Model(&User{ID: 1, Name: "jinzhu"}).Select("Admin").Updates(User{Name: "jinzhu2"})
// Changed("Name") => false, 因为 `Name` 没有被 Select 选中并更新
```

#### 在更新时修改值
若要在Before钩子中改变要更新的值，如果它是一个完整的更新，可以使用`Save`，否则，应该使用`SetColumn`：
```Go
func (user *User) BeforeSave(tx *gorm.DB) (err error) {
	if pw, err := bcrypt.GenerateFromPassword(user.Password, 0); err == nil {
    	tx.Statement.SetColumn("EncryptedPassword", pw)
	}

	if tx.Statement.Changed("Code") {
    	s.Age += 20
    	tx.Statement.SetColumn("Age", s.Age+20)
	}
}

db.Model(&user).Update("Name", "jinzhu")
```


### 删除
#### 删除一条记录
删除一条记录时，删除对象需要指定主键，否则会触发批量删除：
```Go
// Email 的 ID 是 `10`
db.Delete(&email)
// DELETE from emails where id = 10;

// 带额外条件的删除
db.Where("name = ?", "jinzhu").Delete(&email)
// DELETE from emails where id = 10 AND name = "jinzhu";
```

#### 根据主键删除
GORM支持通过内联条件指定主键来检索对象，但==只支持整型数值==，因为string可能导致SQL注入。
```Go
db.Delete(&User{}, 10)
// DELETE FROM users WHERE id = 10;

db.Delete(&User{}, "10")
// DELETE FROM users WHERE id = 10;

db.Delete(&users, []int{1,2,3})
// DELETE FROM users WHERE id IN (1,2,3);
```

#### 删除钩子
对于删除操作，GORM支持`BoforeDelete`、`AfterDelete`钩子，在删除记录时会调用这些方法。
```Go
func (u *User) BeforeDelete(tx *gorm.DB) (err error) {
    if u.Role == "admin" {
        return errors.New("admin user not allowed to delete")
    }
    return
}
```

#### 批量删除
如果指定的值不包括主属性，那么GORM会执行批量删除，它将删除所有匹配的记录
```Go
db.Where("email LIKE ?", "%jinzhu%").Delete(Email{})
// DELETE from emails where email LIKE "%jinzhu%";

db.Delete(&Email{}, "email LIKE ?", "%jinzhu%")
// DELETE from emails where email LIKE "%jinzhu%";
```

#### 阻止全局删除
如果在没有任何条件的情况下执行批量删除，GORM不会执行该操作，并返回`ErrMissingWhereClause`错误
所以，必须要加一些条件，或者使用原生SQL，或者启用`AllowGLobalUpdate`模式，如：
```Go
db.Delete(&User{}).Error // gorm.ErrMissingWhereClause

db.Where("1 = 1").Delete(&User{})
// DELETE FROM `users` WHERE 1=1

db.Exec("DELETE FROM users")
// DELETE FROM users

db.Session(&gorm.Session{AllowGlobalUpdate: true}).Delete(&User{})
// DELETE FROM users
```

#### 软删除
如果我们的模型包含了一个`gorm.deletedat`字段（`gorm.Model`已经包含了该字段），它将自动获得乱删除的能力。
拥有软删除的模型调用`Delete`时，记录不会被从数据库中真正删除。但GORM会将`DeleteAt`置为当前时间，并且不能再通过正常的查询方法找到该记录。
```Go
// user 的 ID 是 `111`
db.Delete(&user)
// UPDATE users SET deleted_at="2013-10-29 10:23" WHERE id = 111;

// 批量删除
db.Where("age = ?", 20).Delete(&User{})
// UPDATE users SET deleted_at="2013-10-29 10:23" WHERE age = 20;

// 在查询时会忽略被软删除的记录
db.Where("age = 20").Find(&user)
// SELECT * FROM users WHERE age = 20 AND deleted_at IS NULL;
```

##### 查找被乱删除的记录
可以使用`Unscoped`找到被软删除的记录
```Go
db.Unscoped().Where("age = 20").Find(&users)
// SELECT * FROM users WHERE age = 20;
```

##### 永久删除
可以使用`Unscoped`永久删除匹配的记录
```Go
db.Unscoped().Delete(&order)
// DELETE FROM orders WHERE id=10;
```


### 原生SQL
#### Scan
`DB.Raw`在需要查询数据，映射到结构体时使用，通常用于执行原生SQL查询语句或可执行的命令。它可以执行任意的SQL语句，并返回查询结果或影响的行数。返回的是`*sql.Rows`结果集对象，通过调用`Scan`方法可以将查询结果映射到相应的结构体中。
```Go
type Result struct {
	ID   int
	Name string
	Age  int
}

var result Result
db.Raw("SELECT id, name, age FROM users WHERE id = ?", 3).Scan(&result)

var age int
db.Raw("select sum(age) from users where role = ?", "admin").Scan(&age)
```

#### Exec
当增删改表中数据的时候使用，不映射到结构体中
用于执行原生SQL语句或可执行的命令。它不返回查询结果，而是返回一个`*sql.Result`对象，其中包含了执行结果的信息。
```Go
db.Exec("DROP TABLE users")
db.Exec("UPDATE orders SET shipped_at=? WHERE id IN ?", time.Now(), []int64{1,2,3})

// Exec SQL 表达式
db.Exec("update users set money=? where name = ?", gorm.Expr("money * ? + ?", 10000, 1), "jinzhu")
```

### DryRun模式
在不执行的情况下生成`SQL`，可以用于准备或准备生成的SQL。
```Go
stmt := db.Session(&Session{DryRun: true}).First(&user, 1).Statement
stmt.SQL.String() //=> SELECT * FROM `users` WHERE `id` = $1 ORDER BY `id`
stmt.Vars         //=> []interface{}{1}
```

#### Row&Rows
用于执行原生SQL查询并返回结果
获取`*sql.Row`结果
```Go
// 使用 GORM API 构建 SQL
row := db.Table("users").Where("name = ?", "jinzhu").Select("name", "age").Row()
row.Scan(&name, &age)

// 使用原生 SQL
row := db.Raw("select name, age, email from users where name = ?", "jinzhu").Row()
row.Scan(&name, &age, &email)
```

获取`*sql.Rows`结果
```Go
// 使用 GORM API 构建 SQL
rows, err := db.Model(&User{}).Where("name = ?", "jinzhu").Select("name, age, email").Rows()
defer rows.Close()
for rows.Next() {
	rows.Scan(&name, &age, &email)

	// 业务逻辑...
}

// 原生 SQL
rows, err := db.Raw("select name, age, email from users where name = ?", "jinzhu").Rows()
defer rows.Close()
for rows.Next() {
	rows.Scan(&name, &age, &email)

	// 业务逻辑...
}
```

#### 将sql.Rows扫描至model
使用`ScanRows`将一行记录扫描到struct：
```Go
rows, err := db.Model(&User{}).Where("name = ?", "jinzhu").Select("name, age, email").Rows() // (*sql.Rows, error)
defer rows.Close()

var user User
for rows.Next() {
	// ScanRows 将一行扫描至 user
	db.ScanRows(rows, &user)

	// 业务逻辑...
}
```