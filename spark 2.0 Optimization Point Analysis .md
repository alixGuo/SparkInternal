## Spark 2.0 Optimization Point Test
------------------
#### **1、cache内存列存储优化**
**解读**：在Spark2.0中，使用DataSet.cache()方式缓存数据比使用RDD.cache()方式节省内存。

不管是rdd的cache()还是DataSet的cache(),调用的都是各自的persist()函数，cache和persist函数的区别是使用cache时存储级别默认是使用内存的，而且不能更改；persist函数是可以指定存储级别的，使用内存还是磁盘或者两者都使用由用户设置。

但是到了底层（内存级）的存储上面，RDD是以rdd对象的方式存储，DataSet则是以列存储字节buffer的方式存储。

- **Dataset的persist()：**

```
def persist(): this.type = {
 sparkSession.sharedState.cacheManager.cacheQuery(this)
 this
  }
```
其中cacheQuery里面使用cacheData结构来缓存数据，cacheData是CacheData的一个可变列表缓存：

```
private val cachedData = new scala.collection.mutable.ArrayBuffer[CachedData]
```
进一步，看CacheData：

```
case class CachedData(plan: LogicalPlan, cachedRepresentation: InMemoryRelation)
```
在进一步，看数据在内存中的表示，就是`cachedRepresentation`，它是`InMemoryRelation`的实例，深入`InMemoryRelation`：

```
/**
 * CachedBatch is a cached batch of rows.
 *
 * @param numRows The total number of rows in this batch
 * @param buffers The buffers for serialized columns
 * @param stats The stat of columns
 */
private[columnar]
case class CachedBatch(numRows: Int, buffers: Array[Array[Byte]], stats: InternalRow)

case class InMemoryRelation(
    output: Seq[Attribute],
    useCompression: Boolean,
    batchSize: Int,
    storageLevel: StorageLevel,
    @transient child: SparkPlan,
    tableName: Option[String])(
    @transient var _cachedColumnBuffers: RDD[CachedBatch] = null,
    val batchStats: LongAccumulator = child.sqlContext.sparkContext.longAccumulator)
  extends logical.LeafNode with MultiInstanceRelation {
```
代码还是比较清晰的，`InMemoryRelation`中接收 `RDD[CachedBatch]`格式的数据入内存，`CachedBatch`缓存了一批列存储的数据，缓存数据的形式是`Array[Array[Byte]]`。

综合上面的分析可以看到，Spark2.0的DataSet的cache()在内存中将每一批次的数据使用字节数组存储，使Dataset数据在内存缓存空间方面得到比较大的优化。

- **RDD的persist():** 

RDD的persist()最终调用的是SparkContext的`persistRDD()`：

```
/**
   * Register an RDD to be persisted in memory and/or disk storage
   */
  private[spark] def persistRDD(rdd: RDD[_]) {
    persistentRdds(rdd.id) = rdd
  }
```
  它将需要缓存的rdd注册到SparkContext的`persistentRdds`列表里：
 

```
 // Keeps track of all persisted RDDs
  private[spark] val persistentRdds = {
    val map: ConcurrentMap[Int, RDD[_]] = new MapMaker().weakValues().makeMap[Int, RDD[_]]()
    map.asScala
  }
```
在内存中存储的是RDD对象的ConcurrentMap。

- **测试数据：**
![Alt text](https://github.com/alixGuo/Resources/blob/master/2016112301.png)
从测试结果看到，同一份数据，分别使用RDD.cache()和DataSet.cache()的内存占用情况。这就是为啥通过RDD.cache()很容易就OOM了。	


#### **2、Parquet向量读优化**
**参考**：https://issues.apache.org/jira/browse/PARQUET-131
**解读**：实现Parquet一次多行读/写方式






#### **3、whole-stage code generation性能优化**
