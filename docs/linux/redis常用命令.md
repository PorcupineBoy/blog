## redis常用命令

​	

```
1. 连接redis服务器
    redis-cli -h hostip -p 6379 -a password
	默认  redis-cli连自己   redis-cli --raw 解决中文乱码连接
2. 基本 key,value操作
	set key value  // 设值
	get key   	   // 获取key的值
	del key    	   // 删除key的值
3. 序列化并且返回序列化后的值
    dump key
4. 检查是否存在key
    exists key
5. 设置key值有效时间，秒为单位
	EXPIRE key seconds
6. 设置key值有效时间，毫秒为单位
	PEXPIRE key seconds
7. 设置key值有效期，单位为：一个日期，精确到秒
    EXPIREAT key timestamp 
8. 设置key值有效期，单位为：一个日期，精确到毫秒
    PEXPIREAT key timestamp 
9. 查找目前有哪些key
    keys pattern   ?代表1个  *代表多个
10. 批量删除
	redis-cli keys "*" | xargs redis-cli del
11. 切换DB
	select dbindex  
	如： select 0或select 1 ，默认为0
12. 将key剪切到指定库
	move key 1
13. 以秒返回key的有效剩余时间
	TTL key
14. 以毫秒返回key有效剩余时间
	PTTL key
15. 清除有效时间，让key持久保存
	PERSIST key
16. 随机获取一个key
    randomkey
17. 修改key名称 ,如果newkey存在， 则覆盖以前的
	rename oldkey newkey
18. 修改key名称，若有newkey存在， 则修改失败
    renamenx oldkey newkey
19. 返回key的存储类型
    type key

20. 返回包含start和包含end的子串，注意：按字节算，中文算2个字节
	getrange key start end

21. 获取旧值，并且设置新值
	getset key newvalue

22. 获取多个值
	mget key key2 key1 key3
	
23. 将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)。
	SETEX key seconds value
	
24. 只有当key不存在的时候，才设置这个key
	setnx key v

25. 返回key存储的内容的长度
	strlen key
26. 批量设置,会覆盖旧值
    mset key1 value1 key2 value2 ...
27. 批量设置,如果有任何一个存在的,都会失败
    msetnx key1 value1 key2 value2 ...
28. 自增1 , 没有会创建，能转换的整数也可以
     INCR key  	
	 DECR key 自减

29. 自增指定数量，没有会创建，能转换的整数也可以
	incrby key decrement
	DECRBY key decrement 自减
30. 追加内容，没有就创建。
	append key value

## redis常用命令
---------HASH类型操作-----------
 
1. 保存对象的一个属性
hset 对象名称 属性名 属性值

2. 获取对象的一个属性值
hget 对象名称 属性名

3. 保存对象的多个属性
hmset 对象名称 属性1 值1 属性2 值2 ...

4. 获取对象的多个值
hmget 对象名称 属性1 属性2 ...

5. 获取对象所有hash对象和属性 
hgetall 对象名称

6. 删除对象中的一个或多个属性
hdel 对象名称 属性1 属性2 ...

7. 判断对象中是否存在某个属性
hexists 对象名称 属性

8. 让对象中的某个值自增 1
hincrby 对象名称 属性 1

9. 获取对象中所有的key
hkeys 对象名称

10. 获取对象中的属性数量
hlen 对象名称

11. 如果不存在才设置值
hsetnx 对象名称 属性 值

12. 获取对象的所有值
hvals 对象名称


## -------list操作-------
1. 从左边压入多个数据 0的位置为最后进去的
lpush key v1 v2 v3 
2. 从右边压入多个数据 0的位置为最先进去的
rpush key v1 v2 v3
3. 从左边0的位置弹出一个元素，弹出后，集合中会删除。
lpop key  变种： blpop key secondeTimeout 阻塞等待secondeTimeout时间获取数据
4. 从右边最后的位置弹出一个元素，弹出后，集合中会删除。
rpop key  变种： brpop key secondeTimeout 阻塞等待secondeTimeout时间获取数据
5. 从key1中弹出右边的一个元素到key2的左边去
rpoplpush key1 key2 
brpoplpush key1 key2 2000
6. 获取指定位置的元素
lindex key index
7. 获取列表长度
llen key
8. 遍历所有元素
lrange key 0 -1
9. 在指定元素前/后插入元素
linsert key before oldvalue newvalue
10. 往列表中插入一个数据， 前提是要该列表存在
lpushx key value
rpushx key value
11. 删除key中value的值（正数从前往后删除正数的值的数量，
负数从后往前删除负数的值的数量，0代表删除全部匹配）
lrem key -2 value
12. 设置指定位置的值
lset key 1 bigpig
13. 保留1到3的位置的元素，其他两边的数据清除
ltrim key 1 3 

--------Set操作，不能存储重复---------
1. SADD key value1 value2 ...
向集合添加一个或多个成员
2. scard key
获取key的长度
3. sdiff key1 key2
获取key1比key2的差集
4. SDIFFSTORE destination key1 key2
返回给定所有集合的差集并存储在 destination 中
5. sinter key1 key2
获取key1比key2的交集
6. SINTERSTORE destination key1 key2
返回给定所有集合的交集并存储在 destination 中
7. sismember key value1
判断value1是否存在key中
8. smembers key
遍历所有key
9. SMOVE source destination member 
将 member 元素从 source 集合移动到 destination 集合
10. spop key count
随机弹出count个元素， 不写count默认为1个
11. srandmember key count
随机获取count个元素， 不写count默认为1个
12. SREM key member1 [member2] 
移除集合中一个或多个成员
13. 合并多个set的结果，会去重
SUNION key1 key2 ...
14. 所有给定集合的并集存储在 destination 集合中
SUNIONSTORE destination key1 key2


```

