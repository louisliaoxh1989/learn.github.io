#一、 索引类型

## （一）、单键索引
>在一个键上创建的索引就是单键索引，单键索引是最常见的索引，如MongoDB默认创建的_id的索引就是单键索引。

## （二）、复合索引
>在多个键上建立的索引就是复合索引

## （三）、多建索引
>如果在一个值为数组的字段上面创建索引， MongoDB会自己决定，是否要把这个索引建成多键索引 

## （四）、地理空间索引
>MongoDB支持几种类型的地理空间索引。其中最常用的是 2dsphere 索引（用于地球表面类型的地图）和 2d 索引（用于平面地图和时间连续的数据）

## （五）、全文索引
>全文索引用于在文档中搜索文本，我们也可以使用正则表达式来查询字符串，但是当文本块比较大的时候，正则表达式搜索会非常慢，而且无法处理语言理解的问题（如 entry 和 entries 应该算是匹配的）。使用全文索引可以非常快地进行文本搜索，就如同内置了多种语言分词机制的支持一样。创建索引的开销都比较大，全文索引的开销更大。创建索引时，需后台或离线创建

## （六）、哈希索引
>哈希 索引可以支持相等查询，但是 哈希 索引不支持范围查询。您可能无法创建一个带有 哈希 索引键的复合索引或者对 哈希 索引施加唯一性的限制。但是，您可以在同一个键上同时创建一个 哈希 索引和一个递增/递减(例如，非哈希)的索引，这样MongoDB对于范围查询就会自动使用非哈希的索引

#二、 索引属性

## (一)、唯一索引
>唯一索引可以拒绝保存那些被索引键的值已经重复的文档。
>默认情况下，MongoDB索引的 unique 属性是  false 。如果对复合索引施加唯一性的限制，那么MongoDB就会强制要求 复合 值的唯一性，而不是分别对每个单独的值要求唯一。 
>**唯一性的限制是针对一个集合中不同文档的。也即，唯一索引可以防止 不同 文档的被索引键上存储相同值，但是它不禁止同一篇文档在被索引键存储的数组里存储的元素或者内嵌文档是相同的值。**
>在同一篇文档存储重复数据的情况下，重复的值只会被存入索引一次。
>如果一篇文档不包含唯一索引的被索引键，那么索引默认会为该文档存储一个null值。由于唯一性的限制，MongoDB将只允许有一篇可以不包含被索引键。如果超过一篇文档不包含被索引键或没有值，那么会抛出键重复(duplicate key)错误导致索引创建失败。可以组合使用唯一性和稀疏索引的特性来过滤那些包含null值的文档以避免这个错误。

## （二）、稀疏索引(Sparse Indexes)
>1. 稀疏索引会跳过所有不包含被索引键的文档。这个索引之所以称为 “稀疏” 是因为它并不包括集合中的所有文档。与之相反，非稀疏的索引会索引每一篇文档，如果一篇文档不含被索引键则为它存储一个null值。
>2. 如果一个索引会导致查询或者排序的结果集是不完整的，那么MongoDB将不会使用这个索引，除非用户使用 hint() 方法来显示指定索引。例如，查询 { x: { $exists: false } } 将不会使用 x 键上的稀疏索引，除非显示的hint。
>3. 2dsphere (version 2), 2d 和 text 这些索引总是稀疏的。
>4. 只要一个文档里有至少一个被索引键，稀疏且只包含有递增/递减索引键的复合索引就会索引这篇文档。
>5. 至于稀疏且包含有地理索引键(例如 2dsphere, 2d)以及递增/递减索引键的复合索引，只有地理索引键的存在与否能决定一篇文档是否被索引。
>5. 至于稀疏且包含了全文索引键和其他递增/递减索引键的复合索引，只有全文索引键的存在与否能决定是否索引该文档。
>6. 一个稀疏且唯一的索引，可以防止集合中的文档被索引键中出现重复值，同时也允许多个文档里不包含被索引键
##（三）、 局部索引(Partial Indexes)
> 1. 这是3.2.0版本以后新增的
> 2. 只会对Collections满足条件[partialFilterExpression]((https://docs.mongodb.com/manual/core/index-partial/))的文档进行索引
> 3. 使用该索引，会在一定程度上减少存储空间和创建索引和维护性能降低的成本
## （四）、 TTL索引
>1. TTL 集合支持失效时间设置，当超过指定时间后，集合自动清除超时的文档，这用来保存一些诸如session会话信息的时候非常有用，或者存储缓存数据使用。**但删除会有延时 **
>2. 索引的字段必须是一个日期的 bson 类型 
>3. 你不能创建 TTL 索引，如果要索引的字段已经在其他索引中使用。否则超时后文档不会被自动清除。
>4. 索引不能包含多个字段


#三、 索引属性选项配置

## （一）、options全局选项（适用于所有索引）
 
参数名| 类型| 描述
----|------|----
background | Boolean | 建索引过程会阻塞其它数据库操作，background可指定以后台方式创建索引，即增加 "background" 可选参数。 "background" 默认值为**false**。 
unique | Boolean | 建立的索引是否唯一。指定为true创建唯一索引。默认值为**false**. 
name | string | 索引的名称。如果未指定，MongoDB的通过连接索引的字段名和排序顺序生成一个索引名称。**不能超过128字符** 
partialFilterExpression| Document| 设定索引只对满足条件的文档起作用 具体见[官网对Partial Indexes](https://docs.mongodb.com/manual/core/index-partial/)
sparse | Boolean | 对文档中不存在的字段数据不启用索引；这个参数需要特别注意，如果设置为true的话，在索引字段中不会查询出不包含对应字段的文档.。默认值为 **false**. 
expireAfterSeconds | integer | 指定一个以秒为单位的数值，完成 TTL设定，设定集合的生存时间。 
storageEngine| Document| 允许用户在创建索引时指定每一个索引的配置

##（二）、 全文（文本）索引选项

参数名| 类型| 描述
----|------|----
weights | document | 设置文本索引字段的权重，权重值1- 99,999
default_language | 设置文本分词的语言，默认为english，[其他支持语言](https://docs.mongodb.com/manual/reference/text-search-languages/#text-search-languages)
language_override | string | 使用文档中的一个字段的值作为设置文本分词的语言，默认为language,[例子](https://docs.mongodb.com/manual/tutorial/specify-language-for-text-index/#specify-language-field-text-index-example)
textIndexVersion | integer | 版本号,可以是1或2

> *备注：其他索引属性见[官网](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#db.collection.createIndex)*



#四、 查看索引


>**语法：db.collection.getIndexes()**

```
示例：
db.index_test.getIndexes()
输出：
[
	{
		"v" : 1,
		"key" : {
			"_id" : 1
		},
		"name" : "_id_",
		"ns" : "usercenter_test.index_test"
	}
]
```
>**可以看出_id字段是默认建立了索引的**

# 五、建立索引

> 从3.0版本后使用 db.collection.createIndex()代替db.collection.ensureIndex()

>**语法：db.collection.createIndex(keys, options) **
>> 参数说明：
> 1.  keys:  {字段名1：ascending,... 字段名n：ascending}: ascending 设为1 标识索引升序，-1降序
> 2.  options : 设置[索引选项](http://blog.csdn.net/louisliaoxh/article/details/51543552#t13)，如设置名称、设置成为唯一索引

**准备数据**

```
db.index_test.insert({"name":"2","age":53,"sex":1})
db.index_test.insert({"name":"3","age":19,"sex":0})
db.index_test.insert({"name":"4","age":20,"sex":2})
db.index_test.insert({"name":"5","age":63,"sex":5})
db.index_test.insert({"name":"6","age":18,"sex":0})
db.index_test.insert({"name":"7","age":98,"sex":1})
db.index_test.insert({"name":"8","age":76,"sex":1})
db.index_test.insert({"name":"9","age":7,"sex":2})
db.index_test.insert({"name":"l0","age":15,"sex":1})
db.index_test.insert({"name":"l","age":23,"sex":1})
```
##（一）、 单一键上创建普通索引

> 语法： db.collections.createIndex({"字段名":1或-1})
```
示例：
db.index_test.createIndex({"age":1})

执行成功后可以看到返回的numIndexesAfter比numIndexesBefore大
```
###（二）、单键索引与查询字段设置对查询性能的影响

**示例1：查询字段不包含索引字段 ** 


```
db.index_test.find({"sex":1}).explain("executionStats")
```
> 可以看到其 winningPlan.stage=COLLSCAN是全表扫描

**示例2：查询字段同时包含索引字段和非索引字段 ** 

```
db.index_test.find({"age":{"$gte":10},"sex":1}).explain("executionStats")
```
>虽然 winningPlan.stage=FETCH以及winningPlan.inputStage.stage =IXSCAN，但是其totalKeysExamined和totalDocsExamined都比nReturned大，说明在查询的时候进行了一些没有必要的扫描。

**示例3：查询字段同时只包含索引字段 ** 
```
db.index_test.find({"age":{"$gte":10}}).explain("executionStats")
```
>可以看到返回中的

>winningPlan.stage=FETCH(根据索引去检索指定document )
>winningPlan.inputStage.stage =IXSCAN(索引扫描)
>executionStats.nReturned=totalKeysExamined=totalDocsExamined=9表示该查询使用的根据索引去查询指定文档
(nReturned:查询返回的条目,totalKeysExamined:索引扫描条目,totalDocsExamined:文档扫描条目) 

***<font color="red">总结：在设置查询字段时应尽量只设置建立了索引的字段 </font>***


###（三）、单键索引与查询时排序字段的设置对查询性能的影响


**示例1：排序字段不包含索引字段 ** 
```
 db.index_test.find({"age":20}).sort({"sex":-1}).explain()
```
> 返回中的winningPlan.stage=SORT 即查询后需要在内存中排序再返回

**示例2：排序字段同时包含索引字段和非索引字段 ** 

```
db.index_test.find({"age":20}).sort({"age":1,"sex":1}).explain()
```
> 结果与上面一样

**示例3：排序字段只包含一个单个索引字段 ** 

```
db.index_test.find({"age":20}).sort({"age":1}).explain()
```
> 可以看到winningPlan.stage变为了FETCH(使用索引)

**示例4：排序字段包含多个单个索引字段 **

```
db.index_test.find({}).sort({"sex":1,"age":1}).explain("executionStats")
db.index_test.find({}).sort({"age":1,"sex":1).explain("executionStats")
```
> 可以看到这种情况winningPlan.stage为sort即索引在排序中没有起到作用，这说明单键索引在多个字段排序时没有作用。

***<font color="red">总结:</font>***
>1. <font color="red">排序操作可以通过从索引中按照索引顺序获取文档的方式来保证结果的有序性。如果查询计划器(planner)无法从索引中得到排序顺序，那么它将需要在内存中排序（winningPlan.stage=SORT）结果。</font>
>2. <font color="red">在多个字段上做排序时需要使用复合索引</font>


 



