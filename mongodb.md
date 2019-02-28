所有单document的写操作是原子的，多document的写操作中，每个单document写是原子的，整体不是原子的  

### Write Concern
`{ w: <value>, j: <boolean>, wtimeout: <number> }`

* 恢复日志（Journal）MongoDB会先把数据更新写入到Journal Buffer里面然后再更新内存数据，然后再返回给应用端。Journal会以100ms的间隔批量刷到盘上。默认开启。
* 写关注（Write Concern)可以让你灵活地指定你写操作的持久化设定。这是一个在性能和可靠性之间的一个权衡。 有以下几个级别：  
> 1. {w: 0} Unacknowledged，指的是对每一个写入操作，MongoDB并不会返回一个是否成功的状态值。这个级别是写入性能最好但也是最不安全的级别。  
> 2. {w: 1} Acknowledged，意思就是对每一个写入MongoDB都会确认操作的完成状态，不管是成功还是失败。当然这个确认只是基于主节点的内存写入。可以侦测到重复主键， 网络错误，系统故障或者是无效数据等错误。MongoDB的默认写安全设置就是 {w:1} Acknowledged。在这种情况下，出现因为系统故障掉电原因而导致的数据丢失只会是我们早些提到的日志没有及时刷盘的情况。如果你不能接受因为系统崩溃而引起的可能的100ms的数据损失，那么你可以选用下一个级别： {j:1} Journaled
> 3. {j:true} Journaled，每一次的写操作会在MongoDB实实在在的把journal落盘以后才会返回。MongoDB会等最多30ms，把30ms内的写操作集中到一起，采用顺序追加的方式写入到盘里。在这30ms内客户端线程会处于等待状态。如果是单机版本，这个基本上就是可以确保100% 安全的了（除非硬盘损坏）。
> 4. {w: “majority”} 写到多数节点，MongoDB只有在数据已经被复制到多数个节点的情况下才会向客户端返回确认。



### db.collection.insertOne()
插入单个document，syntax：
```
db.collection.insertOne(
   <document>,
   {
      writeConcern: <document>
   }
)
document	要插入的文档.
writeConcern	写安全级别，不要在事务中明确设置writeConcern
Returns:
{
    "acknowledged" : true,
    "insertedId" : ObjectId("5c748f6b47622f641cd2b1ad")
}
```
collection不存在时会创建一个collection，如果没指定_id，会自动添加一个ObjectId作为_id,不要在事务中使用可能会创建collection的insert操作，不要在事务中设置insert的writeCOncern

### db.collection.insertMany()
插入多个document, syntax:
```
db.collection.insertMany(
   [ <document 1> , <document 2>, ... ],
   {
      writeConcern: <document>,
      ordered: <boolean>  // 是否按顺序插入，如果不按顺序mongo可以对插入进行优化
   }
)
```

### db.collection.insert()
插入单个或多个document
```
db.collection.insert(
   <document or array of documents>,
   {
     writeConcern: <document>,
     ordered: <boolean>
   }
)
```

### db.collection.find()
```
db.collection.find( {} ) 查询所有
db.collection.find( { <field>: <value>, ... }， { <pfield>: [0|1], ... })
  and条件查询, 如果field是数组元素的域，则表示数组元素的域能分别满足所有条件
  如果field是数组，则只要含有value元素就算匹配
  value可以是document，value必须完全匹配包括顺序
  field可以是嵌套域"field.nestedField"，必须有引号,如果value是document数组，则只要有一个document数组元素匹配就算匹配
  field可以是数组元素，比如"dim_cm.0"，"dim_cm.1"，"dim_cm.2"
  field可以是数组元素的域，比如'instock.0.qty'，'instock.1.qty'
  value是null时，表示不含有field，或者field的值是null，用{ item : { $exists: false } }表示不含有item，用{ item : { $type: "null" } }表示item是null


db.collection.find( { { <field1>: { <operator1>: <value1> }, ... } )
db.collection.find( { $or: [ { status: "A" }, { qty: { $lt: 30 } } ] } ) or条件查询
db.collection.findOne() == db.collection.find().limit(1)
db.collection.find( { "field.nestedField": <value1>, ... } ) 子域查询，注意引号
db.collection.find( { tags: ["red", "blank"] } ) 完全匹配数组包括顺序
db.collection.find( { tags: { $all: ["red", "blank"] } } ) 包含所有，无关顺序
db.collection.find( { tags: "red" } ) 如果tags是数组，只要含有red就算匹配
db.collection.find( { dim_cm: { $gt: 25 } } ) dim_cm有一个元素大于25就算匹配
db.collection.find( { dim_cm: { $gt: 15, $lt: 20 } } ) dim_cm有一个元素大于15，有另一个元素小于20就算匹配
db.collection.find( { dim_cm: { $elemMatch: { $gt: 22, $lt: 30 } } } ) elemMatch表示数组dim_cm的任一元素同时匹配
db.collection.find( { "instock": { $elemMatch: { qty: 5, warehouse: "A" } } } ) elemMatch表示数组instock的任一元素同时匹配，不区分顺序
db.collection.find( { "tags": { $size: 3 } } ) 查找tags数组长度为3的元素
```

### Query operators
```
{ <field>: { $eq: <value> } } field == values，等同于{ <field>:<value>}如果value是一个document，则必须完全匹配，包括字段顺序，如果field是一个数组，则只需要有一个数组元素匹配value
{field: {$gt: value} } field > value
{field: {$gte: value} } field >= value
{field: {$lt: value} } field < value
{ field: { $lte: value} } field <= value
{field: {$ne: value} } field != value
{ field: { $in: [<value1>, <value2>, ... <valueN> ] } } field在集合中，如果field是数组，则只需要数组中有一个元素在集合中就算匹配
{ field: { $nin: [ <value1>, <value2> ... <valueN> ]} } field不存在或者不在集合中
{ $and: [ { <expression1> }, { <expression2> } , ... , { <expressionN> } ] } and短路计算
{ $or: [ { <expression1> }, { <expression2> }, ... , { <expressionN> } ] }  or短路计算
{ field: { $not: { <operator-expression> } } }
{ $nor: [ { <expression1> }, { <expression2> }, ...  { <expressionN> } ] }
{ field: { $exists: <boolean> } }
{ field: { $type: <BSON type> } }、{ field: { $type: [ <BSON type1> , <BSON type2>, ... ] } } type(field) == type

{ field: { $mod: [ divisor, remainder ] } } field % divisor == remainder
{ <field>: { $all: [ <value1> , <value2> ... ] } } field是个数组，并且包含所有的value
{ <field>: { $elemMatch: { <query1>, <query2>, ... } } } field是个数组，并且数组的一个元素同时满足所有的query
{ field: { $size:  num} } field是个长度为num的数组

```

### Projection operators
```
```

### Update operators
```
db.collection.updateOne(<filter>, <update>, <options>)
db.collection.updateMany(<filter>, <update>, <options>)
db.collection.replaceOne(<filter>, <update>, <options>)
update：
{
  <update operator>: { <field1>: <value1>, ... },
  <update operator>: { <field2>: <value2>, ... },
  ...
}
<update operator>:
{ $currentDate: { <field1>: <typeSpecification1>, ... } }设置field为当前时间，typeSpecification可以是true（{ $type: "date" }）或者时间戳类型{ $type: "timestamp" } 或者 可读类型{ $type: "date" }
{ $inc: { <field1>: <amount1>, <field2>: <amount2>, ... } }增加field的值，amount可以是正值或负值
{ $min: { <field1>: <value1>, ... } } 如果field的值大于value则设置为value
{ $max: { <field1>: <value1>, ... } } 如果field的值小于value则设置为value
{ $mul: { <field1>: <number1>, ... } } field的值乘以number
{ $rename: { <field1>: <newName1>, <field2>: <newName2>, ... } } 变更field的名字，先$unset老名字和新名字，再$set新名字，不能用于数组
{ $set: { <field1>: <value1>, ... } } 设置field的值为value
{ $setOnInsert: { <field1>: <value1>, ... } } 在upsert: true时触发insert操作时变为$set操作
{ $unset: { <field1>: "", ... } } 删除field，如果field是数组元素，则设置数组元素为null而不是删除
{ $bit: { <field>: { <and|or|xor>: <int> } } } 对field进行位运行,只支持整数field如NumberInt、NumberLong

{ <update operator>: { "<array>.$" : value } } $是数组第一个匹配元素的占位符
{ <update operator>: { "<array>.$[]" : value } } $[]是数组所有元素的占位符
{ <update operator>: { "<array>.$[<identifier>]" : value } },{ arrayFilters: [ { <identifier>: <condition> } } ] } 条件匹配占位符，满足匹配条件的数组元素
{ $addToSet: { <field1>: <value1>, ... } }  向数组field添加不重复元素value
{ $pop: { <field>: <-1 | 1>, ... } }  删除数组field第一个（-1）或者最后一个（1）元素
{ $pull: { <field1>: <value|condition>, <field2>: <value|condition>, ... } } 删除数组field中满足条件的所有元素，value是一个document时只需要包含document中所有的元素，不需要顺序相同
{ $push: { <field1>: <value1>, ... } } 向数组field中添加元素value
{ $push: { <field1>: { <modifier1>: <value1>, ... }, ... } }
{ $pullAll: { <field1>: [ <value1>, <value2> ... ], ... } } 移除field中所有的value1、value2等元素
{ $addToSet: { <field>: { $each: [ <value1>, <value2> ... ] } } }、{ $push: { <field>: { $each: [ <value1>, <value2> ... ] } } } each迭代value元素
{ $push: { <field>: { $each: [ <value1>, <value2>, ... ], $position: <num> } } }  position表示push中插入的位置，必须和each联合使用
{ $push: { <field>: { $each: [ <value1>, <value2>, ... ], $slice: <num> }}} slice限制push之后的元素总数量，必须和each联合使用
{ $push: { <field>: { $each: [ <value1>, <value2>, ... ], $sort: <sort specification>}}} sort对push之后的元素排序
```

### db.collection.updateOne(filter, update, options)
```
db.collection.updateOne(
   <filter>,
   <update>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ]
   }
)
```
upsert如果为true,updateOne如果没有找到任何document就会创建一个，此时collection必须存在


### db.collection.replaceOne(filter, replacement, options)
```
db.collection.replaceOne(
   <filter>,
   <replacement>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>
   }
)
```
替换第一个匹配的document


### Delete Documents
db.collection.deleteMany()  
db.collection.deleteOne()


### db.collection.findAndModify()
```
db.collection.findAndModify({
    query: <document>,  // 查询
    sort: <document>,   // 如果查询到多个document，先通过sort排序，然后操作排序后的第一个document
    remove: <boolean>,  // true表示移除document，默认是false，remove和update必须二选一
    update: <document>, // 更新document，remove和update必须二选一
    new: <boolean>,     // true返回修改后的document，false返回修改前的文档，默认false，update时有效
    fields: <document>, // 返回的域 projection
    upsert: <boolean>,
    bypassDocumentValidation: <boolean>,
    writeConcern: <document>,
    collation: <document>,
    arrayFilters: [ <filterdocument1>, ... ]
});
```
get-and-set 式的操作，修改并返回一个document，如果匹配多个，只修改第一个document
如果upsert:true并且查询的域不是唯一索引，并发操作时，可能会插入一个文档多次
如果在分片环境使用，必须包含所有的分片key

### index
```
db.collection.createIndex( { <field1>: <type>, <field2>: <type2>, ... } ) 创建索引，field:1升序，field:-1降序
db.collection.getIndexes() 查询索引
db.collection.dropIndex({ <field1>: <type>}) 删除索引
db.collection.dropIndexes() 删除collection上的所有索引
```
