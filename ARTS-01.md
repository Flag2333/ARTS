# ARTS-01

### **Algorithm**: 

[两数之和](https://leetcode-cn.com/problems/two-sum/ )

```java
public int[] twoSum(int[] nums, int target) {
        // 创建一个<元素，下标>的map
        Map<Integer,Integer> map = new HashMap<>();
        for(int i = 0; i <= nums.length; i++){
            // 计算差值
            int difference = target - nums[i];
            // 如果差值在map中则返回差值的下标和当前元素下标组成的数组
            if (map.containsKey(difference)){
                return new int[] {map.get(difference),i};
            }
            // 把元素放入map中
            map.put(nums[i], i);
        }
        throw new IllegalArgumentException("No two sum solution");
    }
```

### **Review**： 

[The Key To Accelerating Your Coding Skills](http://blog.thefirehoseproject.com/posts/learn-to-code-and-be-self-reliant/ )

打怪升级的初级阶段，要求助于官方文档以及Google，在遇见问题的时候要思考是否以前遇到过相同的问题，能不能用上一次的方法解决，会不会有更好的做法。进阶后要走出舒适区，不然不能升级呀，升级后还会有更舒适的舒适区的哈哈哈哈。再高级一点就收拾收拾准备团战推塔了，找准终极目标，不能迷失在打怪里。

### **Tip:** 

```java
/**
     * 获取缓存
     * 
     * @param key 
     * @param expire  过期时间
     * @param clazz
     * @param cacheLoadable 从数据库读取数据的方法
     * @param <T>
     * @return
     */
public <T> T getCache(String key, int expire, TypeReference<T> clazz, CacheLoadable<T> cacheLoadable) {
        T result = null;
        String value = redisUtils.getString(key);//根据key去缓存中查询
        if (StringUtils.isEmpty(value)) {//如果缓存中没有
            synchronized (this) {//这个key加锁后进行处理
                //重新从缓存获取一次key
                //原因：前一个相同的key可能已经从数据库获取了数据并且更新到了缓存
                value = redisUtils.getString(key);
                if (!StringUtils.isEmpty(value)) {//如果获取到结果 返回
                    return JSON.parseObject(value, clazz);
                }
                result = cacheLoadable.load();//没获取到 从数据库读取结果
                if (result != null) {//从数据库读取的数据不为空
                    String valueJson = JSON.toJSONString(result);
                    redisUtils.setString(key, valueJson, expire);// 更新到缓存
                }
            }
        } else {
            result = JSON.parseObject(value, clazz);
        }
        return result;
    }
```

### **Share**： 

#### Redis学习（一）

#### 1. 什么是Redis

- Redis是一种数据结构服务器，通过命令对可变数据结构进行访问，这些命令使用带有TCP套接字和简单协议的*服务器 - 客户端*模型发送 ，不同进程可以以共享的方式查询和修改相同的数据结构

#### 2. 为什么用Redis

- 支持键值数据类型：字符串，散列，列表，集合，有序集合，并且可以对集合进行运算操作
- Redis所有数据都存储在内存里，读写速度快于硬盘，并且支持将内存中的数据异步写入硬盘中
- 可以用作缓存（设置失效时间），消息队列（list类型实现队列）等
- 支持多数据库（分片）
- 支持事务（multi，exec，watch，incr）

#### 3. 什么时候用Redis

- 缓存

- 单点登录

- 秒杀

  ```java
  	/**
       * 秒杀 气泡
       */
      public boolean secKillBubble(Integer bubbleId, Integer userId) {
          Jedis jedis = getJedis();
          // 第一层保障 从0自增key1
          Long incr = jedis.incr(RedisConstant.bubbleEntryNum + bubbleId);
          // 给key1设置过期时间
          jedis.expire(RedisConstant.bubbleEntryNum+ bubbleId, RedisConstant.bubble_exp);
          try {
              // 监视key2
              jedis.watch(RedisConstant.bubbleForOne + bubbleId);// watchkeys
              // 如果key1没有被自增过则开启事务
              if (incr <=1) {
                  Transaction tx = jedis.multi();// 开启事务
                  // 设置key2的值为用户id(抢到气泡的用户的id)
                  tx.setex(RedisConstant.bubbleForOne + bubbleId,RedisConstant.bubble_exp, userId.toString());
                  // 第二个人过来发现这个气泡已经被设置了用户id 说明已经被别人秒杀掉了 
                  List<Object> list = tx.exec();// 提交事务，如果此时watchkeys被改动了，则返回null
                  if (list != null) {
                      log.info("气泡秒杀成功!" + ",userId" + userId + "，气泡Id：" + bubbleId);
                      return true;
                  }
              }
              log.info("气泡没有秒杀到" + ",userId" + userId + "，气泡Id：" + bubbleId);
              return false;
          } catch (Exception e) {
              log.error("错误：秒杀气泡" + ",userId" + userId + "，气泡Id：" + bubbleId, e);
              e.printStackTrace();
          } finally {
              jedis.close();
          }
          return false;
      }
  ```

- 网站访问排名

- 等

#### 4. 为什么Redis这么快

- Redis所有数据都存储在内存里，内存操作快于硬盘
- 对管道的支持，由于内存操作很快，时间一般都耽误在网络传输上，Redis可以一块执行很多命令后再返回结果
- 采用单线程，如果有多线程竞争锁的时间 ，好几个单线程都执行完毕了

