[TOC]



### Redis 基本指令操作示例

通用操作`GenericCmdable` 位于 generic_commands.go文件中

```go

type GenericCmdable interface {
	Del(ctx context.Context, keys ...string) *IntCmd
	Dump(ctx context.Context, key string) *StringCmd
	Exists(ctx context.Context, keys ...string) *IntCmd
	Expire(ctx context.Context, key string, expiration time.Duration) *BoolCmd
	ExpireAt(ctx context.Context, key string, tm time.Time) *BoolCmd
	ExpireTime(ctx context.Context, key string) *DurationCmd
	ExpireNX(ctx context.Context, key string, expiration time.Duration) *BoolCmd
	ExpireXX(ctx context.Context, key string, expiration time.Duration) *BoolCmd
	ExpireGT(ctx context.Context, key string, expiration time.Duration) *BoolCmd
	ExpireLT(ctx context.Context, key string, expiration time.Duration) *BoolCmd
	Keys(ctx context.Context, pattern string) *StringSliceCmd
	Migrate(ctx context.Context, host, port, key string, db int, timeout time.Duration) *StatusCmd
	Move(ctx context.Context, key string, db int) *BoolCmd
	ObjectRefCount(ctx context.Context, key string) *IntCmd
	ObjectEncoding(ctx context.Context, key string) *StringCmd
	ObjectIdleTime(ctx context.Context, key string) *DurationCmd
	Persist(ctx context.Context, key string) *BoolCmd
	PExpire(ctx context.Context, key string, expiration time.Duration) *BoolCmd
	PExpireAt(ctx context.Context, key string, tm time.Time) *BoolCmd
	PExpireTime(ctx context.Context, key string) *DurationCmd
	PTTL(ctx context.Context, key string) *DurationCmd
	RandomKey(ctx context.Context) *StringCmd
	Rename(ctx context.Context, key, newkey string) *StatusCmd
	RenameNX(ctx context.Context, key, newkey string) *BoolCmd
	Restore(ctx context.Context, key string, ttl time.Duration, value string) *StatusCmd
	RestoreReplace(ctx context.Context, key string, ttl time.Duration, value string) *StatusCmd
	Sort(ctx context.Context, key string, sort *Sort) *StringSliceCmd
	SortRO(ctx context.Context, key string, sort *Sort) *StringSliceCmd
	SortStore(ctx context.Context, key, store string, sort *Sort) *IntCmd
	SortInterfaces(ctx context.Context, key string, sort *Sort) *SliceCmd
	Touch(ctx context.Context, keys ...string) *IntCmd
	TTL(ctx context.Context, key string) *DurationCmd
	Type(ctx context.Context, key string) *StatusCmd
	Copy(ctx context.Context, sourceKey string, destKey string, db int, replace bool) *IntCmd

	Scan(ctx context.Context, cursor uint64, match string, count int64) *ScanCmd
	ScanType(ctx context.Context, cursor uint64, match string, count int64, keyType string) *ScanCmd
}

```

上面接口解释：

- **Del**: 删除一个或多个键。

  ```go
  client.Del(context.Background(), "mykey")
  ```

- **Dump**: 序列化给定键的值，并返回序列化结果。

- **Exists**: 检查给定的一个或多个键是否存在。

  ```go
  exists := client.Exists(context.Background(), "mykey")
  fmt.Println("Does key exist:", exists.Val())
  ```

- **Expire**: 设置键的过期时间。

- **ExpireAt**: 设置键在指定的时间点过期。

- **ExpireTime**: 返回键的剩余过期时间。

- **ExpireNX**: 仅当键不存在时，设置键的过期时间。

- **ExpireXX**: 仅当键存在时，设置键的过期时间。

- **ExpireGT**: 仅当键的过期时间大于指定的时间时，设置键的过期时间。

- **ExpireLT**: 仅当键的过期时间小于指定的时间时，设置键的过期时间。

- **Keys**: 查找所有符合给定模式（正则匹配）的键。

  ```go
  // 根据正则获取keys, .
  keys, _ := client.Keys(ctx, "*").Result()
  // 根据前缀获取keys
  keys2, _ := rdb.Keys(ctx, "user*").Result()
  // DBSize() 查看当前数据库key的数量
  num, err := client.DBSize(ctx).Result()
  ```

- **Migrate**: 迁移键到其他的 Redis 实例。

- **Move**: 将键移动到另一个数据库。

- **ObjectRefCount**: 获取对象的引用计数。

- **ObjectEncoding**: 获取存储在键值中对象的编码格式。

- **ObjectIdleTime**: 获取键的空闲时间。

- **Persist**: 移除键的过期时间，让键持久化。

- **PExpire**: 设置键的过期时间，以毫秒为单位。

- **PExpireAt**: 设置键在指定的时间点过期，以毫秒为单位。

- **PExpireTime**: 返回键的剩余过期时间，以毫秒为单位。

- **PTTL**: 获取键的剩余生存时间，以毫秒为单位。

- **RandomKey**: 返回随机的键。

- **Rename**: 重命名键。

- **RenameNX**: 重命名键，如果新键名不存在。

- **Restore**: 从序列化的字符串中恢复值到键。

- **RestoreReplace**: 从序列化的字符串中恢复值到键，可以替换已存在的键。

- **Sort**: 对列表、集合或有序集合进行排序。

- **SortRO**: 对只读的列表、集合或有序集合进行排序。

- **SortStore**: 对列表、集合或有序集合进行排序，并将结果保存到一个新的键中。

- **SortInterfaces**: 对列表、集合或有序集合进行排序，返回结果作为接口类型的切片。

- **Touch**: 更新键的最后访问时间。

- **TTL**: 获取键的剩余生存时间。

- **Type**: 返回键的数据类型。

- **Copy**: 复制一个键到另一个键。

- **Scan**: 迭代数据库中的键。

  ```go
  // 如果key的数量非常多的时候，我们可以搭配使用Scan命令和Del命令完成删除。
    iter := client.Scan(ctx, 0, "user*", 0).Iterator()
    for iter.Next(ctx) {
      err := rdb.Del(ctx, iter.Val()).Err()
      if err != nil {
        panic(err)
      }
    }
    if err := iter.Err(); err != nil {
      panic(err)
    }
  ```

  

- **ScanType**: 迭代数据库中的指定类型的键。

### 字符串操作

`StringCmdable` 位于 `string_commands.go` 文件中

```go
type StringCmdable interface {
	Append(ctx context.Context, key, value string) *IntCmd
	Decr(ctx context.Context, key string) *IntCmd
	DecrBy(ctx context.Context, key string, decrement int64) *IntCmd
	Get(ctx context.Context, key string) *StringCmd
	GetRange(ctx context.Context, key string, start, end int64) *StringCmd
	GetSet(ctx context.Context, key string, value interface{}) *StringCmd
	GetEx(ctx context.Context, key string, expiration time.Duration) *StringCmd
	GetDel(ctx context.Context, key string) *StringCmd
	Incr(ctx context.Context, key string) *IntCmd
	IncrBy(ctx context.Context, key string, value int64) *IntCmd
	IncrByFloat(ctx context.Context, key string, value float64) *FloatCmd
	LCS(ctx context.Context, q *LCSQuery) *LCSCmd
	MGet(ctx context.Context, keys ...string) *SliceCmd
	MSet(ctx context.Context, values ...interface{}) *StatusCmd
	MSetNX(ctx context.Context, values ...interface{}) *BoolCmd
	Set(ctx context.Context, key string, value interface{}, expiration time.Duration) *StatusCmd
	SetArgs(ctx context.Context, key string, value interface{}, a SetArgs) *StatusCmd
	SetEx(ctx context.Context, key string, value interface{}, expiration time.Duration) *StatusCmd
	SetNX(ctx context.Context, key string, value interface{}, expiration time.Duration) *BoolCmd
	SetXX(ctx context.Context, key string, value interface{}, expiration time.Duration) *BoolCmd
	SetRange(ctx context.Context, key string, offset int64, value string) *IntCmd
	StrLen(ctx context.Context, key string) *IntCmd
}
```

简要说明及常见用法示例：

- **Append**: 将指定的字符串值追加到指定键的字符串值末尾。返回值是追加后的字符串长度。

  ```go
  newValLen := client.Append(ctx, "mykey", " some more data").Val()
  ```

- **Decr**: 键的整数值减一。

- **DecrBy**: 键的整数值减去指定整数。

  ```go
  newIntVal := client.DecrBy(ctx, "mykey", 5).Val()
  ```

- **Get**: 获取指定键的字符串值。

  ```go
  val := client.Get(ctx, "mykey").Val()	
  ```

- **GetRange**: 字符串截取，获取指定键的字符串值的指定区间。

  ```go
  	// 注：即使key不存在，调用GetRange()也不会报错，只是返回的截取结果是空"",可以使用fmt.Printf("%q\n", val)来打印测试	
  val1, _ := rdb.GetRange(ctx, "mykey", 1, 4).Result()
  ```

- **GetSet**: 设置指定键的字符串值，并返回之前的字符串值。

- **GetEx**: 获取指定键的字符串值，并设置键的过期时间。

- **GetDel**: 获取指定键的字符串值，并删除该键。

- **Incr**: 键的整数值加一。

- **IncrBy**: 键的整数值增加指定整数。

  ```go
  newIntVal := client.IncrBy(ctx, "mykey", 10).Val()
  ```

  

- **IncrByFloat**: 键的浮点数值增加指定浮点数。

- **LCS**: 执行最长公共子序列算法。

- **MGet**: 获取多个键的字符串值。

  ```go
  vals, err := client.MGet(ctx, "key1", "key2", "key3").Result()
  ```

- **MSet**: 设置多个键值对。

  ```go
  err := client.MSet(ctx, "key1", "value1", "key2", "value2").Err()
  ```

- **MSetNX**: 设置多个键值对，如果键不存在。

- **Set**: 设置指定键的字符串值，并可设置过期时间，如果过期时间设置为-1则代表不过期 。

  ```go
  err := client.Set(ctx, "mykey", "myvalue", 10 * time.Second).Err()
  ```

- **SetArgs**: 设置指定键的字符串值，可以附加参数。

- **SetEx**: 设置指定键的字符串值，并设置过期时间。

- **SetNX**: 仅当键不存在时，设置键的字符串值。

  ```go
  	val, err := client.SetNX(ctx, "username", "weiyigeek", 0).Result()
  ```

- **SetXX**: 仅当键存在时，设置键的字符串值。

- **SetRange**: 从指定偏移量开始，替换键值的一部分。

- **StrLen**: 获取指定键的字符串长度。

  ```go
  length := client.StrLen(ctx, "mykey").Val()
  ```


每个方法提供了对 Redis 中字符串类型数据进行操作的不同方式。您可以根据需要选择适当的方法，并使用提供的参数执行操作。每个方法返回的命令对象可用于检索操作结果。

### 列表 list 类型操作

接口`ListCmdable`定义，源码位于 `list_commands.go`

```go
type ListCmdable interface {
	BLPop(ctx context.Context, timeout time.Duration, keys ...string) *StringSliceCmd
	BLMPop(ctx context.Context, timeout time.Duration, direction string, count int64, keys ...string) *KeyValuesCmd
	BRPop(ctx context.Context, timeout time.Duration, keys ...string) *StringSliceCmd
	BRPopLPush(ctx context.Context, source, destination string, timeout time.Duration) *StringCmd
	LIndex(ctx context.Context, key string, index int64) *StringCmd
	LInsert(ctx context.Context, key, op string, pivot, value interface{}) *IntCmd
	LInsertBefore(ctx context.Context, key string, pivot, value interface{}) *IntCmd
	LInsertAfter(ctx context.Context, key string, pivot, value interface{}) *IntCmd
	LLen(ctx context.Context, key string) *IntCmd
	LMPop(ctx context.Context, direction string, count int64, keys ...string) *KeyValuesCmd
	LPop(ctx context.Context, key string) *StringCmd
	LPopCount(ctx context.Context, key string, count int) *StringSliceCmd
	LPos(ctx context.Context, key string, value string, args LPosArgs) *IntCmd
	LPosCount(ctx context.Context, key string, value string, count int64, args LPosArgs) *IntSliceCmd
	LPush(ctx context.Context, key string, values ...interface{}) *IntCmd
	LPushX(ctx context.Context, key string, values ...interface{}) *IntCmd
	LRange(ctx context.Context, key string, start, stop int64) *StringSliceCmd
	LRem(ctx context.Context, key string, count int64, value interface{}) *IntCmd
	LSet(ctx context.Context, key string, index int64, value interface{}) *StatusCmd
	LTrim(ctx context.Context, key string, start, stop int64) *StatusCmd
	RPop(ctx context.Context, key string) *StringCmd
	RPopCount(ctx context.Context, key string, count int) *StringSliceCmd
	RPopLPush(ctx context.Context, source, destination string) *StringCmd
	RPush(ctx context.Context, key string, values ...interface{}) *IntCmd
	RPushX(ctx context.Context, key string, values ...interface{}) *IntCmd
	LMove(ctx context.Context, source, destination, srcpos, destpos string) *StringCmd
	BLMove(ctx context.Context, source, destination, srcpos, destpos string, timeout time.Duration) *StringCmd
}

```

方法解释：

这些 Redis 列表操作方法提供了对列表数据结构的不同操作：

- `BLPop`: 从一个或多个列表的左侧移除并获取元素，如果列表没有元素则会阻塞一段时间直到超时。
- `BLMPop`: 从多个列表的左侧移除并获取元素，返回的是多个列表键值对。
- `BRPop`: 从一个或多个列表的右侧移除并获取元素，如果列表没有元素则会阻塞一段时间直到超时。
- `BRPopLPush`: 从一个列表的右侧弹出元素并将其添加到另一个列表的左侧。
- `LIndex`: 获取列表中指定位置的元素。

  ```go
  val2, _ := client.LIndex(ctx, "list", 2).Result()
  	fmt.Printf("下标为2的值为: %v\n", val2)
  ```
- `LInsert`: 在列表中某个元素的前或后插入新元素。

  ```go
  // LInsert() 在某个位置插入新元素
  // 在名为key的缓存项值为2的元素前面插入一个值，值为123 ， 注意只会执行一次
  _ = client.LInsert(ctx, "list", "before", "2", 123).Err()
  // 在名为key的缓存项值为2的元素后面插入一个值，值为321
  _ = client.LInsert(ctx, "list", "after", "2", 321).Err()
  ```
- `LLen`: 获取列表的长度。

  ```go
  	// LLen() 获取列表表元素个数
  	length, _ := client.LLen(ctx, "list").Result()
  	fmt.Printf("当前链表的长度为: %v\n", length)
  ```
- `LMPop`: 从多个列表的任意位置移除并获取元素，返回的是多个列表键值对。
- `LPop`: 移除并获取列表的第一个元素。

  ```go
  // 从链表左侧弹出数据
  	val3, _ := client.LPop(ctx, "list").Result()
  	fmt.Printf("弹出下标为0的值为: %v\n", val3)
  ```
- `LPopCount`: 移除并获取列表的前 N 个元素。
- `LPos`: 获取列表中某个值的索引位置。
- `LPosCount`: 获取列表中某个值的前 N 个索引位置。
- `LPush`: 将一个或多个值插入到列表的左侧（头部）。

  ```go
  // 插入指定值到list列表中，返回值是当前列表元素的数量
  	// 使用LPush()方法将数据从左侧压入链表（后进先出）,也可以从右侧压如链表对应的方法是RPush()
  	count, _ := client.LPush(ctx, "list", 1, 2, 3).Result()
  	fmt.Println("插入到list集合中元素的数量: ", count)
  ```
- `LPushX`: 仅当列表存在时，在列表左侧添加一个值。
- `LRange`: 获取列表指定范围内的元素。
- `LRem`: 从列表中删除指定数量的匹配元素。

  ```go
  // LRem() 根据值移除元素 lrem key count value
  	n, _ := client.LRem(ctx, "list", 2, "256").Result()
  	fmt.Printf("移除了: %v 个\n", n)
  ```
- `LSet`: 设置列表中指定位置的元素值。

  ```go
  // LSet() 设置某个元素的值
  	//下标是从0开始的
  	val1, _ := client.LSet(ctx, "list", 2, 256).Result()
  	fmt.Println("是否成功将下标为2的元素值改成256: ", val1)
  ```
- `LTrim`: 对列表进行修剪，保留指定范围内的元素。
- `RPop`: 移除并获取列表的最后一个元素。
- `RPopCount`: 移除并获取列表的后 N 个元素。
- `RPopLPush`: 移除列表 source 的最后一个元素，并将该元素添加到列表 destination 的头部。
- `RPush`: 将一个或多个值插入到列表的右侧（尾部）。
- `RPushX`: 仅当列表存在时，在列表右侧添加一个值。
- `LMove`: 将列表中的元素移动到另一个列表中的指定位置。
- `BLMove`: 类似 LMove 但支持超时参数。

### 集合Set类型操作

接口`SetCmdable`定义，Tips：集合数据的特征，元素不能重复保持唯一性, 元素无序不能使用索引(下标)操作

```go
type SetCmdable interface {
	SAdd(ctx context.Context, key string, members ...interface{}) *IntCmd
	SCard(ctx context.Context, key string) *IntCmd
	SDiff(ctx context.Context, keys ...string) *StringSliceCmd
	SDiffStore(ctx context.Context, destination string, keys ...string) *IntCmd
	SInter(ctx context.Context, keys ...string) *StringSliceCmd
	SInterCard(ctx context.Context, limit int64, keys ...string) *IntCmd
	SInterStore(ctx context.Context, destination string, keys ...string) *IntCmd
	SIsMember(ctx context.Context, key string, member interface{}) *BoolCmd
	SMIsMember(ctx context.Context, key string, members ...interface{}) *BoolSliceCmd
	SMembers(ctx context.Context, key string) *StringSliceCmd
	SMembersMap(ctx context.Context, key string) *StringStructMapCmd
	SMove(ctx context.Context, source, destination string, member interface{}) *BoolCmd
	SPop(ctx context.Context, key string) *StringCmd
	SPopN(ctx context.Context, key string, count int64) *StringSliceCmd
	SRandMember(ctx context.Context, key string) *StringCmd
	SRandMemberN(ctx context.Context, key string, count int64) *StringSliceCmd
	SRem(ctx context.Context, key string, members ...interface{}) *IntCmd
	SScan(ctx context.Context, key string, cursor uint64, match string, count int64) *ScanCmd
	SUnion(ctx context.Context, keys ...string) *StringSliceCmd
	SUnionStore(ctx context.Context, destination string, keys ...string) *IntCmd
}

```

方法解释：

这些方法用于 Redis 的集合类型进行操作：

- `SAdd`: 向集合添加一个或多个成员。

  ```go
  // 集合元素缓存设置
  	keyname := "Program"
  	mem := []string{"C", "Golang", "C++", "C#", "Java", "Delphi", "Python", "Golang"}
  	// //由于Golang已经被添加到Program集合中，所以重复添加时无效的
  	for _, v := range mem {
  		client.SAdd(ctx, keyname, v)
  	}
  ```

- `SCard`: 获取集合的成员数。

  ```go
  // SCard() 获取集合元素个数
  	total, _ := client.SCard(ctx, keyname).Result()
  	fmt.Println("golang集合成员个数: ", total)
  ```

- `SDiffStore`: 将多个集合的差集存储到指定的目标集合中。

- `SInterCard`: 返回多个集合的交集的基数（元素个数）。

- `SInterStore`: 将多个集合的交集存储到指定的目标集合中。

- `SIsMember`: 判断指定成员是否是集合的成员。

  ```go
  // SIsMember() 判断元素是否在集合中
  	exists, _ := client.SIsMember(ctx, keyname, "golang").Result()
  	if exists {
  		fmt.Println("golang 存在 Program 集合中.") // 注意:我们存入的是Golang而非golang
  	} else {
  		fmt.Println("golang 不存在 Program 集合中.")
  	}
  ```

- `SMIsMember`: 判断多个成员是否是集合的成员，返回每个成员的判断结果。

- `SMembers`: 获取集合中的所有成员。

  ```go
  // SSMembers() 获取所有成员
  	val3, _ := client.SMembers(ctx, keyname).Result()
  	fmt.Printf("随机获取一个元素: %v , 随机获取多个元素: %v \n所有成员: %v\n", val1, val2, val3)
  ```

- `SMembersMap`: 获取集合中的所有成员，返回 map 形式。

- `SMove`: 将一个成员从一个集合移动到另一个集合。

- `SPop`: 移除并返回集合中的一个随机元素。

  ```go
  // SPop() 随机获取一个元素 （无序性，是随机的）
  	val1, _ := client.SPop(ctx, keyname).Result()
  ```

- `SPopN`: 移除并返回集合中的多个随机元素。

  ```go
  // SPopN()  随机获取多个元素.
  	val2, _ := client.SPopN(ctx, keyname, 2).Result()
  ```

- `SRandMember`: 返回集合中的一个随机元素。

- `SRandMemberN`: 返回集合中的多个随机元素。

- `SRem`: 移除集合中一个或多个成员。

  ```go
  // 删除集合中指定元素(返回成功)
    n, _ := client.SRem(ctx, "setB", "a", "f").Result()
    fmt.Println("已成功删除元素的个数: ",n)
  ```

- `SScan`: 迭代集合中的元素，可用于实现遍历。

- `SUnion`: 返回多个集合的并集。

  ```go
  // SUnion():并集, SDiff():差集, SInter():交集
  	client.SAdd(ctx, "setA", "a", "b", "c", "d")
  	client.SAdd(ctx, "setB", "a", "d", "e", "f")
  
  	//并集
  	union, _ := rdb.SUnion(ctx, "setA", "setB").Result()
  	fmt.Println("并集", union)
  ```

- `SDiff`: 返回多个集合的差集。

  ```go
  //差集
  diff, _ := rdb.SDiff(ctx, "setA", "setB").Result()
  fmt.Println("差集", diff)
  ```

- `SInter`: 返回多个集合的交集。

  ```go
  //交集
  inter, _ := rdb.SInter(ctx, "setA", "setB").Result()
  fmt.Println("交集", inter)
  ```

- `SUnionStore`: 将多个集合的并集存储到指定的目标集合中。

### 有序集合(zset)类型操作

```go
type SortedSetCmdable interface {
    BZPopMax(ctx context.Context, timeout time.Duration, keys ...string) *ZWithKeyCmd
    BZPopMin(ctx context.Context, timeout time.Duration, keys ...string) *ZWithKeyCmd
    BZMPop(ctx context.Context, timeout time.Duration, order string, count int64, keys ...string) *ZSliceWithKeyCmd
    ZAdd(ctx context.Context, key string, members ...Z) *IntCmd
    ZAddLT(ctx context.Context, key string, members ...Z) *IntCmd
    ZAddGT(ctx context.Context, key string, members ...Z) *IntCmd
    ZAddNX(ctx context.Context, key string, members ...Z) *IntCmd
    ZAddXX(ctx context.Context, key string, members ...Z) *IntCmd
    ZAddArgs(ctx context.Context, key string, args ZAddArgs) *IntCmd
    ZAddArgsIncr(ctx context.Context, key string, args ZAddArgs) *FloatCmd
    ZCard(ctx context.Context, key string) *IntCmd
    ZCount(ctx context.Context, key, min, max string) *IntCmd
    ZLexCount(ctx context.Context, key, min, max string) *IntCmd
    ZIncrBy(ctx context.Context, key string, increment float64, member string) *FloatCmd
    ZInter(ctx context.Context, store *ZStore) *StringSliceCmd
    ZInterWithScores(ctx context.Context, store *ZStore) *ZSliceCmd
    ZInterCard(ctx context.Context, limit int64, keys ...string) *IntCmd
    ZInterStore(ctx context.Context, destination string, store *ZStore) *IntCmd
    ZMPop(ctx context.Context, order string, count int64, keys ...string) *ZSliceWithKeyCmd
    ZMScore(ctx context.Context, key string, members ...string) *FloatSliceCmd
    ZPopMax(ctx context.Context, key string, count ...int64) *ZSliceCmd
    ZPopMin(ctx context.Context, key string, count ...int64) *ZSliceCmd
    ZRange(ctx context.Context, key string, start, stop int64) *StringSliceCmd
    ZRangeWithScores(ctx context.Context, key string, start, stop int64) *ZSliceCmd
    ZRangeByScore(ctx context.Context, key string, opt *ZRangeBy) *StringSliceCmd
    ZRangeByLex(ctx context.Context, key string, opt *ZRangeBy) *StringSliceCmd
    ZRangeByScoreWithScores(ctx context.Context, key string, opt *ZRangeBy) *ZSliceCmd
    ZRangeArgs(ctx context.Context, z ZRangeArgs) *StringSliceCmd
    ZRangeArgsWithScores(ctx context.Context, z ZRangeArgs) *ZSliceCmd
    ZRangeStore(ctx context.Context, dst string, z ZRangeArgs) *IntCmd
    ZRank(ctx context.Context, key, member string) *IntCmd
    ZRankWithScore(ctx context.Context, key, member string) *RankWithScoreCmd
    ZRem(ctx context.Context, key string, members ...interface{}) *IntCmd
    ZRemRangeByRank(ctx context.Context, key string, start, stop int64) *IntCmd
    ZRemRangeByScore(ctx context.Context, key, min, max string) *IntCmd
    ZRemRangeByLex(ctx context.Context, key, min, max string) *IntCmd
    ZRevRange(ctx context.Context, key string, start, stop int64) *StringSliceCmd
    ZRevRangeWithScores(ctx context.Context, key string, start, stop int64) *ZSliceCmd
    ZRevRangeByScore(ctx context.Context, key string, opt *ZRangeBy) *StringSliceCmd
    ZRevRangeByLex(ctx context.Context, key string, opt *ZRangeBy) *StringSliceCmd
    ZRevRangeByScoreWithScores(ctx context.Context, key string, opt *ZRangeBy) *ZSliceCmd
    ZRevRank(ctx context.Context, key, member string) *IntCmd
    ZRevRankWithScore(ctx context.Context, key, member string) *RankWithScoreCmd
    ZScore(ctx context.Context, key, member string) *FloatCmd
    ZUnionStore(ctx context.Context, dest string, store *ZStore) *IntCmd
    ZRandMember(ctx context.Context, key string, count int) *StringSliceCmd
    ZRandMemberWithScores(ctx context.Context, key string, count int) *ZSliceCmd
    ZUnion(ctx context.Context, store ZStore) *StringSliceCmd
    ZUnionWithScores(ctx context.Context, store ZStore) *ZSliceCmd
    ZDiff(ctx context.Context, keys ...string) *StringSliceCmd
    ZDiffWithScores(ctx context.Context, keys ...string) *ZSliceCmd
    ZDiffStore(ctx context.Context, destination string, keys ...string) *IntCmd
    ZScan(ctx context.Context, key string, cursor uint64, match string, count int64) *ScanCmd
}
```

常用方法:

```go
/ 有序集合成员与分数设置
	// zSet类型需要使用特定的类型值*redis.Z，以便作为排序使用
	lang := []*redis.Z{
		&redis.Z{Score: 90.0, Member: "Golang"},
		&redis.Z{Score: 98.0, Member: "Java"},
		&redis.Z{Score: 95.0, Member: "Python"},
		&redis.Z{Score: 97.0, Member: "JavaScript"},
		&redis.Z{Score: 99.0, Member: "C/C++"},
	}
```

- `BZPopMax`: 从多个有序集合中弹出包含最大分值成员的命令。

- `BZPopMin`: 从多个有序集合中弹出包含最小分值成员的命令。

- `BZMPop`: 从多个有序集合中弹出具有最大或最小值的指定个数的成员。

- `ZAdd`: 向有序集合添加一个或多个成员。

  ```go
  //插入ZSet类型
  	num, err := rdb.ZAdd(ctx, "language_rank", lang...).Result()
  	if err != nil {
  		fmt.Printf("zadd failed, err:%v\n", err)
  		return
  	}
  	fmt.Printf("zadd %d succ.\n", num)
  ```

- `ZAddLT`: 向有序集合中分数小于或等于指定分数的成员。

- `ZAddGT`: 向有序集合中分数大于或等于指定分数的成员。

- `ZAddNX`: 仅在成员不存在时，向有序集合添加成员。

- `ZAddXX`: 仅在成员存在时，向有序集合添加成员。

- `ZAddArgs`: 向有序集合添加成员，支持更多参数。

- `ZAddArgsIncr`: 向有序集合添加成员并返回分数值。

- `ZCard`: 获取有序集合的成员数。

- `ZCount`: 获取有序集合中分数范围内的成员数。

- `ZLexCount`: 获取有序集合中分数范围内的成员数，根据字典排序。

- `ZIncrBy`: 有序集合中成员的分数增加指定数量。

  ```go
  // 将ZSet中的某一个元素顺序值增加: 把Golang的分数加10
  	newScore, err := client.ZIncrBy(ctx, "language_rank", 10.0, "Golang").Result()
  	if err != nil {
  		fmt.Printf("zincrby failed, err:%v\n", err)
  		return
  	}
  	fmt.Printf("Golang's score is %f now.\n", newScore)
  ```

  

- `ZInter`: 计算多个有序集合的交集。

- `ZInterWithScores`: 计算多个有序集合的交集并返回成员和分数。

- `ZRevRangeWithScores`：根据分数排名，获取排序后的数据段

  ```go
  // 根据分数排名取出元素:取分数最高的3个
  	ret, err := client.ZRevRangeWithScores(ctx, "language_rank", 0, 2).Result()
  	if err != nil {
  		fmt.Printf("zrevrange failed, err:%v\n", err)
  		return
  	}
  	fmt.Printf("zsetKey前3名热度的是: %v\n,Top 3 的 Memeber 与 Score 是:\n", ret)
  	for _, z := range ret {
  		fmt.Println(z.Member, z.Score)
  	}
  ```

- `ZRangeByScore、ZRevRangeByScore`： 获取score过滤后排序的数据段

  ```go
  // ZRangeByScore()、ZRevRangeByScore():获取score过滤后排序的数据段
  	// 此处表示取95~100分的
  	op := redis.ZRangeBy{
  		Min: "95",
  		Max: "100",
  	}
  	ret, err = client.ZRangeByScoreWithScores(ctx, "language_rank", &op).Result()
  	if err != nil {
  		fmt.Printf("zrangebyscore failed, err:%v\n", err)
  		return
  	}
  	// 输出全部成员及其score分数
  	fmt.Println("language_rank 键存储的全部元素:")
  	for _, z := range ret {
  		fmt.Println(z.Member, z.Score)
  	}
  }
  ```

- 其余方法涉及有序集合的各种操作，包括获取、移除、排序等。

### 哈希(hash)类型操作

```go
type HashCmdable interface {
    HDel(ctx context.Context, key string, fields ...string) *IntCmd
    HExists(ctx context.Context, key, field string) *BoolCmd
    HGet(ctx context.Context, key, field string) *StringCmd
    HGetAll(ctx context.Context, key string) *MapStringStringCmd
    HIncrBy(ctx context.Context, key, field string, incr int64) *IntCmd
    HIncrByFloat(ctx context.Context, key, field string, incr float64) *FloatCmd
    HKeys(ctx context.Context, key string) *StringSliceCmd
    HLen(ctx context.Context, key string) *IntCmd
    HMGet(ctx context.Context, key string, fields ...string) *SliceCmd
    HSet(ctx context.Context, key string, values ...interface{}) *IntCmd
    HMSet(ctx context.Context, key string, values ...interface{}) *BoolCmd
    HSetNX(ctx context.Context, key, field string, value interface{}) *BoolCmd
    HScan(ctx context.Context, key string, cursor uint64, match string, count int64) *ScanCmd
    HVals(ctx context.Context, key string) *StringSliceCmd
    HRandField(ctx context.Context, key string, count int) *StringSliceCmd
    HRandFieldWithValues(ctx context.Context, key string, count int) *KeyValueSliceCmd
}
```

hash 是一个 string 类型的 field（字段） 和 value（值） 的映射表，hash 特别适合用于存储对象。* 

上述接口方法用于操作 Redis 中的哈希（Hash）数据结构。

- `HDel`: 删除哈希中一个或多个字段。

  ```go
  count, _ := rdb.HDel(ctx, "huser", "key3", "key4").Result()
  	fmt.Println("删除元素的个数: ", count)
  ```

- `HExists`: 检查哈希中是否存在指定字段。

  ```go
  	flag, _ := rdb.HExists(ctx, "hmuser", "address").Result()
  	fmt.Println("address 是否存在 hmuser 中: ", flag)
  ```

- `HGet`: 获取哈希中指定字段的值。

  ```go
  address, _ := rdb.HGet(ctx, "hmuser", "address").Result()
  	fmt.Println("hmuser.address -> ", address)
  ```

- `HGetAll`: 获取哈希中所有字段和值。

  ```go
  hmuser, _ := rdb.HGetAll(ctx, "hmuser").Result()
  	fmt.Println("hmuser :=> ", hmuser)
  ```

- `HIncrBy`: 哈希中指定字段的整数值增加。

- `HIncrByFloat`: 哈希中指定字段的浮点数值增加。

- `HKeys`: 获取哈希中所有字段的键。

- `HLen`: 获取哈希中字段的数量。

  ```go
  length, _ := rdb.HLen(ctx, "hmuser").Result()
  	fmt.Println("hmuser hash 键长度: ", length)
  ```

- `HMGet`: 获取哈希中一个或多个字段的值。

- `HSet`: 设置哈希中指定字段的值。

  ```go
   // HSet() 设置字段和值
  	rdb.HSet(ctx, "huser", "key1", "value1", "key2", "value2")
  	rdb.HSet(ctx, "huser", []string{"key3", "value3", "key4", "value4"})
  	rdb.HSet(ctx, "huser", map[string]interface{}{"key5": "value5", "key6": "value6"})
  ```

- `HMSet`: 设置哈希中多个字段的值。

  ```go
  	rdb.HMSet(ctx, "hmuser", map[string]interface{}{"name": "WeiyiGeek", "age": 88, "address": "重庆"})
  ```

- `HSetNX`: 设置哈希中字段的值，仅在字段不存在时设置成功。

- `HScan`: 迭代哈希中的字段。

- `HVals`: 获取哈希中所有字段的值。

- `HRandField`: 从哈希中随机获取指定数量的字段。

- `HRandFieldWithValues`: 从哈希中随机获取指定数量的字段和值。

### 基数统计 HyperLogLog 类型操作

#### 描述

HyperLogLog（HLL）是一种数据结构，用于估计集合中不重复元素的数量。它可以使用很少的内存空间来存储一个大型集合的近似基数，即集合中不同元素的数量，且具有很高的精度。

HLL 使用了概率性算法来估计基数，因此虽然可以提供一个接近真实基数的值，但并不保证精确的计数。它的优点在于占用的空间相对较小，适合于大规模数据集合的去重统计或者近似统计。

通过一系列的哈希函数和位操作，HLL 将元素的值映射到一个固定长度的位数组中，然后利用这些位来计算近似的基数。即使处理大型数据集合，HLL 也可以以较小的内存开销来估计唯一元素的数量。

要注意的是，HLL 的估计值是统计意义上的近似值，对于相同数据集合，在不同的运行中可能会有些许差异。

#### 应用场景

HyperLogLog（HLL）适用于需要估计大型数据集合的唯一元素数量的场景，但不需要精确计数。一些典型的应用场景包括：

1. **大数据集合去重统计：** 当处理大规模数据集合时，需要快速估算其唯一元素的数量，例如统计网站的独立访客数或用户 ID 数量。
2. **近似分析：** 适用于对数据集合进行近似分析而不需要确切的计数，比如对流量数据、日志数据或其他大数据集的统计分析。
3. **数据库优化：** 在数据库中，可以使用 HLL 来估算某些列或属性的唯一值数量，这样可以节省内存空间并提高查询效率。
4. **实时统计和监控：** 对于需要实时监控数据并快速估算唯一值数量的应用，如实时系统的性能监控、用户行为分析等。

HLL 虽然能够提供较为精确的近似值，但并不适用于需要完全精确计数的场景。在这些情况下，应当使用确保准确性的数据结构和算法。

Tips: 每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基数.

#### 示例

```go
type HyperLogLogCmdable interface {
	PFAdd(ctx context.Context, key string, els ...interface{}) *IntCmd
	PFCount(ctx context.Context, keys ...string) *IntCmd
	PFMerge(ctx context.Context, dest string, keys ...string) *StatusCmd
}

```

接口解释：

- `PFAdd`: 将一个或多个元素添加到 HyperLogLog 结构中，用于估计唯一元素的数量。它接受一个键和一个或多个元素作为参数，并返回添加的元素数量。
- `PFCount`: 估计一个或多个 HyperLogLog 的唯一元素数量。通过传入一个或多个键，可以估算所有这些键关联的 HyperLogLog 的总元素数量。
- `PFMerge`: 将多个 HyperLogLog 结构合并为一个新的 HyperLogLog。这可以用于将多个统计结果汇总，从而得到一个整体的唯一元素数量估算。

示例：模拟统计一组用户访问网站的唯一 IP 地址数量：

```go
import (
	"context"
	"fmt"
	"github.com/go-redis/redis/v9"
)

func main() {
	ctx := context.Background()
	client := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379", // Redis地址
		Password: "",               // Redis密码，如果有的话
		DB:       0,                // Redis数据库索引
	})

	key := "unique_ips" // Redis键

	// 添加一些假设的IP地址到HyperLogLog中
	ips := []string{"192.168.1.1", "192.168.1.2", "192.168.1.3", "192.168.1.1"}
	result := client.PFAdd(ctx, key, ips...)
	fmt.Printf("Added %d unique IPs\n", result.Val())

	// 估计唯一IP数量
	count := client.PFCount(ctx, key)
	fmt.Printf("Estimated unique IP count: %d\n", count.Val())

	// 演示合并不同HyperLogLogs
	key2 := "unique_ips_2"
	ips2 := []string{"192.168.1.4", "192.168.1.5", "192.168.1.6"}
	result2 := client.PFAdd(ctx, key2, ips2...)
	fmt.Printf("Added %d unique IPs\n", result2.Val())

	// 合并两个HyperLogLog并获取估算的唯一IP总数
	destKey := "merged_unique_ips"
	mergeResult := client.PFMerge(ctx, destKey, key, key2)
	fmt.Printf("Merged %d unique IPs\n", mergeResult.Val())

	finalCount := client.PFCount(ctx, destKey)
	fmt.Printf("Total estimated unique IP count after merge: %d\n", finalCount.Val())
}

```

### 自定义redis指令操作

描述: 我们可以采用go-redis提供的Do方法，可以让我们直接执行redis-cli中执行的相关指令, 可以极大的便于使用者上手。

```go
func ExampleClient_CMD(rdb *redis.Client, ctx context.Context) {
	log.Println("Start ExampleClient_CMD")
	defer log.Println("End ExampleClient_CMD")

	// 1.执行redis指令 Set 设置缓存
	v := rdb.Do(ctx, "set", "NewStringCmd", "redis-cli").String()
	log.Println(">", v)

	// 2.执行redis指令 Get 设置缓存
	v = rdb.Do(ctx, "get", "NewStringCmd").String()
	log.Println("Method1 >", v)

	// 3.匿名方式执行自定义redis命令
	// Set
	Set := func(client *redis.Client, ctx context.Context, key, value string) *redis.StringCmd {
		cmd := redis.NewStringCmd(ctx, "set", key, value) // 关键点
		client.Process(ctx, cmd)
		return cmd
	}
	v, _ = Set(rdb, ctx, "NewCmd", "go-redis").Result()
	log.Println("> set NewCmd go-redis:", v)

	// Get
	Get := func(client *redis.Client, ctx context.Context, key string) *redis.StringCmd {
		cmd := redis.NewStringCmd(ctx, "get", key) // 关键点
		client.Process(ctx, cmd)
		return cmd
	}
	v, _ = Get(rdb, ctx, "NewCmd").Result()
	log.Println("Method2 > get NewCmd:", v)

	// 4.执行redis指令 hset 设置哈希缓存 (实践以下方式不行)
	// kv := map[string]interface{}{"key5": "value5", "key6": "value6"}
	// v, _ = rdb.Do(ctx, "hmset", "NewHashCmd", kv)
	// log.Println("> ", v)
}

```

#### Redis Pipeline 通道操作

#### 描述

Redis Pipeline 是一种批量执行多个 Redis 命令的机制。它允许在一个网络往返中发送多个命令，并在收到回复时一次性接收它们，而不是等待每个命令的回复。

这个机制通常用于提高 Redis 命令的执行效率，特别是在需要执行大量独立命令的情况下。Pipeline 能够减少网络延迟对性能的影响，因为它将多个命令打包成一个请求一次性发送到 Redis 服务器，并收集所有回复。

应用场景包括但不限于：

1. **批量写入**：当需要执行大量 SET、HSET 等命令时，Pipeline 能够将这些命令批量发送给 Redis，减少网络往返时间。
2. **数据计算**：用于执行一系列 Redis 命令的原子操作，如 INCR、DECR 等，Pipeline 能够提高计算效率。
3. **数据预加载**：在启动时预加载大量数据到 Redis 中，可以使用 Pipeline 提高数据加载速度。

使用 Pipeline 需要注意以下几点：

- Pipeline 操作不是原子的。如果中途出现错误，比如网络故障，只有部分命令可能被执行。此时需要谨慎处理错误和回滚。
- 在使用 Pipeline 时，尽量避免长时间占用连接，以免影响其他客户端的操作。

### 功能接口：

```go
type Pipeliner interface {
	StatefulCmdable

	// Len is to obtain the number of commands in the pipeline that have not yet been executed.
	Len() int

	// Do is an API for executing any command.
	// If a certain Redis command is not yet supported, you can use Do to execute it.
	Do(ctx context.Context, args ...interface{}) *Cmd

	// Process is to put the commands to be executed into the pipeline buffer.
	Process(ctx context.Context, cmd Cmder) error

	// Discard is to discard all commands in the cache that have not yet been executed.
	Discard()

	// Exec is to send all the commands buffered in the pipeline to the redis-server.
	Exec(ctx context.Context) ([]Cmder, error)
}

```

#### 接口解释

- **StatefulCmdable**: `Pipeliner` 继承了 `StatefulCmdable` 接口，这个接口定义了执行 Redis 命令的方法，通常是与 Redis 状态有关的命令的执行。
- **Len() int**: 这个方法用于获取当前 Pipeline 中未执行的命令数量。
- **Do(ctx context.Context, args ...interface{}) \*Cmd**: `Do` 方法用于执行任何 Redis 命令。如果某个 Redis 命令还没有被 Go Redis 库支持，你可以使用 `Do` 方法来执行它。
- **Process(ctx context.Context, cmd Cmder) error**: `Process` 方法将要执行的命令放入 Pipeline 的缓冲区中。
- **Discard()**: `Discard` 方法用于丢弃 Pipeline 中尚未执行的所有命令。
- **Exec(ctx context.Context) ([]Cmder, error)**: `Exec` 方法用于将 Pipeline 缓冲区中的所有命令发送给 Redis 服务器执行，并返回一个 `[]Cmder`，代表已执行的命令集合。

### 示例：

使用 Redis 的 Pipeline 机制执行了两个命令：INCR 和 EXPIRE，它们被封装在 Pipeline 中，并通过 `pipe.Exec()` 一次性发送到 Redis 服务器。

```go
package main

import (
	"context"
	"fmt"
	"github.com/go-redis/redis/v9"
)

func main() {
	ctx := context.Background()
	client := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "", // 如果有密码的话
		DB:       0,  // 使用默认的数据库索引
	})

	// 创建一个 Pipeline
	pipe := client.Pipeline()

	// 向 Pipeline 中添加一系列命令
	incrCmd := pipe.Incr(ctx, "pipeline_counter")
	pipe.Expire(ctx, "pipeline_counter", 24*3600*time.Second)

	// 执行 Pipeline 中的命令
	_, err := pipe.Exec(ctx)
	if err != nil {
		fmt.Println("Pipeline error:", err)
		return
	}

	// 获取 INCR 命令的结果
	counterVal, err := incrCmd.Result()
	if err != nil {
		fmt.Println("Get result error:", err)
		return
	}

	fmt.Println("Pipeline executed successfully. Counter value:", counterVal)
}

```

### MULTI/EXEC 事务处理操作

描述: Redis是单线程的，因此单个命令始终是原子的，但是来自不同客户端的两个给定命令可以依次执行，例如在它们之间交替执行。但是`Multi/exec`
能够确保在其两个语句之间的命令之间没有其他客户端正在执行命令。

在这种场景我们需要使用TxPipeline, 它总体上类似于上面的Pipeline, 但是它内部会使用`MULTI/EXEC`
包裹排队的命令。例如：

```go
pipe := client.TxPipeline()
incr := pipe.Incr("tx_pipeline_counter")
pipe.Expire("tx_pipeline_counter", time.Hour)
_, err := pipe.Exec()
fmt.Println(incr.Val(), err)

// # 上面代码相当于在一个RTT下执行了下面的redis命令：
MULTI
INCR pipeline_counter
EXPIRE pipeline_counts 3600
EXEC

// # 还有一个与上文类似的TxPipelined方法，使用方法如下：
var incr *redis.IntCmd
_, err := rdb.TxPipelined(func(pipe redis.Pipeliner) error {
	incr = pipe.Incr("tx_pipelined_counter")
	pipe.Expire("tx_pipelined_counter", time.Hour)
	return nil
})
fmt.Println(incr.Val(), err)

```

**简单示例:**

```go
func TxPipelineExample(rdb *redis.Client, ctx context.Context) {
	// 开pipeline与事务
	pipe := client.TxPipeline()
	// 设置TxPipeline键缓存
	v, _ := client.Do(ctx, "set", "TxPipeline", 1023.0).Result()
	log.Println(v)
	// 自增+1.0
	incr := pipe.IncrByFloat(ctx, "TxPipeline", 1026.0)
	log.Println(incr) // 未提交时  incr.Val() 值 为 0
	// 设置键过期时间
	pipe.Expire(ctx, "TxPipeline", time.Hour)
	// 提交事务
	_, err := pipe.Exec(ctx)
	if err != nil {
		log.Println("执行失败, 进行回滚操作!")
		return
	}
	fmt.Println("事务执行成功,已提交!")
	log.Println("TxPipeline :", incr.Val()) // 提交后值 为 2049
}

```

### Watch 监听操作

描述: 在某些场景下我们除了要使用`MULTI/EXEC`命令外，还需要配合使用WATCH命令, 用户使用WATCH命令监视某个键之后，直到该用户执行EXEC命令的这段时间里，如果有其他用户抢先对被监视的键进行了替换、更新、删除等操作，那么当用户尝试执行EXEC的时候，事务将失败并返回一个错误，用户可以根据这个错误选择重试事务或者放弃事务。

Watch方法接收一个函数和一个或多个key作为参数,其函数原型:

```go
Watch(fn func(*Tx) error, keys ...string) error 
```

基本示例：

```go
// 监视watch_count的值，并在值不变的前提下将其值+1
key := "watch_count"
err = client.Watch(func(tx *redis.Tx) error {
	n, err := tx.Get(key).Int()
	if err != nil && err != redis.Nil {
		return err
	}
	_, err = tx.Pipelined(func(pipe redis.Pipeliner) error {
		pipe.Set(key, n+1, 0)
		return nil
	})
	return err
}, key)

```

命令以事务方式递增Key的值的示例，仅当Key的值不发生变化时提交一个事务。

```go
// 监视watch_count的值，并在值不变的前提下将其值+1
key := "watch_count"
err = client.Watch(func(tx *redis.Tx) error {
	n, err := tx.Get(key).Int()
	if err != nil && err != redis.Nil {
		return err
	}
	_, err = tx.Pipelined(func(pipe redis.Pipeliner) error {
		pipe.Set(key, n+1, 0)
		return nil
	})
	return err
}, key)

```

### Script 脚本操作

描述: 从 Redis 2.6.0 版本开始的，使用内置的 Lua 解释器，可以对 Lua 脚本进行求值, 所以我们可直接在redis客户端中执行一些脚本。

redis Eval 命令基本语法如下：`EVAL script numkeys key [key ...] arg [arg ...]`

- script: 参数是一段 Lua 5.1 脚本程序。脚本不必(也不应该)定义为一个 Lua 函数。
- numkeys: 用于指定键名参数的个数。
- key [key ...]: 从 EVAL 的第三个参数开始算起，表示在脚本中所用到的那些 Redis 键(key)，这些键名参数可以在 Lua 中通过全局变量 KEYS 数组，用 1 为基址的形式访问`( KEYS[1] ， KEYS[2] ，以此类推)`。
- arg [arg ...]: 附加参数，在 Lua 中通过全局变量 ARGV 数组访问，访问的形式和 KEYS 变量类似`( ARGV[1] 、 ARGV[2] ，诸如此类)`。

`redis.call()`与 `redis.pcall()`唯一的区别是当redis命令执行结果返回错误时 redis.call() 将返回给调用者一个错误，而redis.pcall()会将捕获的错误以Lua表的形式返回

```go
# 利用eval执行脚本
127.0.0.1:6379> set name weiyigeek
OK
127.0.0.1:6379> eval "return redis.call('get','name')" 0
"weiyigeek"
127.0.0.1:6379> eval "return redis.call('set','foo','bar')" 0
OK
127.0.0.1:6379> eval "return redis.pcall('get','foo')" 0
"bar"
127.0.0.1:6379> eval "return {KEYS[1],ARGV[1],KEYS[2],ARGV[2]}" 2 name age weiyigeek 25
1) "name"
2) "weiyigeek"
3) "age"
4) "25"


# Lua 数据类型和 Redis 数据类型之间转换
> eval "return 10" 0
(integer) 10

> eval "return {1,2,{3,'Hello World!'}}" 0
1) (integer) 1
2) (integer) 2
3) 1) (integer) 3
   2) "Hello World!"

> eval "return redis.call('get','foo')" 0
"bar"

```

那在`go-redis`客户端中如何执行脚本操作?

```go
func ScriptExample(rdb *redis.Client, ctx context.Context) {
	// Lua脚本定义1. 传递key输出指定格式的结果
	EchoKey := redis.NewScript(`
		if redis.call("GET", KEYS[1]) ~= false then
			return {KEYS[1],"==>",redis.call("get", KEYS[1])}
		end
		return false
	`)

	err := rdb.Set(ctx, "xx_name", "WeiyiGeek", 0).Err()
	if err != nil {
		panic(err)
	}
	val1, err := EchoKey.Run(ctx, rdb, []string{"xx_name"}).Result()
	log.Println(val1, err)

	// Lua脚本定义2. 传递key与step使得，key值等于`键值+step`
	IncrByXX := redis.NewScript(`
		if redis.call("GET", KEYS[1]) ~= false then
			return redis.call("INCRBY", KEYS[1], ARGV[1])
		end
		return false
	`)

	// 判断键是否存在，存在就删除该键
	exist, err := rdb.Exists(ctx, "xx_counter").Result()
	if exist > 0 {
		res, err := rdb.Del(ctx, "xx_counter").Result()
		log.Printf("is Exists?: %v, del xx_counter: %v, err: %v \n", exist, res, err)
	}

	// 首次调用
	val2, err := IncrByXX.Run(ctx, rdb, []string{"xx_counter"}, 2).Result()
	log.Println("首次调用 IncrByXX.Run ->", val2, err)

	// 写入 xx_counter 键
	err = rdb.Set(ctx, "xx_counter", 40, 0).Err()
	if err != nil {
		panic(err)
	}
	// 二次调用
	val3, err := IncrByXX.Run(ctx, rdb, []string{"xx_counter"}, 2).Result()
	log.Println("二次调用 IncrByXX.Run ->", val3, err)
}

```

