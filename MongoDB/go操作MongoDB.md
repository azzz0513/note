### 获取依赖
```Go
go get go.mongodb.org/mongo-driver/mongo
```

### 连接操作
```Go
package main  
  
import (  
    "context"  
    "fmt"  
    "go.mongodb.org/mongo-driver/mongo"    "go.mongodb.org/mongo-driver/mongo/options")  
  
func main() {  
    // 设置客户端连接配置  
    clientOptions := options.Client().  
       ApplyURI("mongodb://root1:123456@localhost:27017/test?authSource=test"). // 用户名和密码  
       SetMaxPoolSize(100).                                                     // 最大连接数  
       SetMinPoolSize(10)                                                       // 最小连接数  
  
    // 连接到MongoDB  
    client, err := mongo.Connect(context.TODO(), clientOptions)  
    if err != nil {  
       panic(err)  
    }  
    fmt.Println("Connected to MongoDB!")  
  
    // 断开连接  
    err = client.Disconnect(context.TODO())  
    if err != nil {  
       panic(err)  
    }  
    fmt.Println("Connection to MongoDB closed.")  
}
```

### 插入操作
```Go
func insert() {  
    collection := client.Database("test").Collection("users")  
    // 定义要插入的文档  
    document := bson.D{  
       {"name", "Alice"},  
       {"age", 30},  
       {"email", "New York"},  
    }  
    // 插入文档  
    insertResult, err := collection.InsertOne(context.Background(), document)  
    if err != nil {  
       log.Fatal(err)  
    }  
    fmt.Println("Inserted document ID:", insertResult.InsertedID)  
}
```

不同的bson类型：
```Go
bson.D(有序文档)
字段顺序严格按照添加顺序存储和传输，查询时也会保持顺序。
适合需要精确控制字段顺序的场景（如MongoDB的聚合管道、排序规则等）

bson.E(文档字段元素)
作为bson.D的组成单元，用于定义有序文档中的每个字段

bson.M(无序文档)
语法更简洁，使用方式类似JSON对象，适合快速构建简单文档
字段顺序在编码/解码时可能被打乱，不适合依赖顺序的场景

bson.A(数组)
元素可以是任意类型（字符串、数字、bson.D、bson.M、甚至另一个bson.A）
保留元素的插入顺序，与MongoDB中的数组行为一致
```
