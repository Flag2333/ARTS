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

​	(1) 速度快，因为数据存在内存中，类似于HashMap，HashMap的优势就是查找和操作的时间复杂度都是O(1)

​	(2)支持丰富数据类型，支持string，list，set，sorted set，hash

​	(3) 支持事务，操作都是原子性，所谓的原子性就是对数据的更改要么全部执行，要么全部不执行

​	(4) 丰富的特性：可用于缓存，消息，按key设置过期时间，过期后将会自动删除

[面试中关于Redis的问题看这篇就够了](https://blog.csdn.net/qq_34337272/article/details/80012284 )

#### 3. 为什么Redis这么快

​	1、完全基于内存，绝大部分请求是纯粹的内存操作，非常快速。数据存在内存中，类似于HashMap，HashMap的优势就是查找和操作的时间复杂度都是O(1)；

​	2、数据结构简单，对数据操作也简单，Redis中的数据结构是专门进行设计的；

​	3、采用单线程，避免了不必要的上下文切换和竞争条件，也不存在多进程或者多线程导致的切换而消耗 CPU，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗；

​	4、使用多路I/O复用模型，非阻塞IO；

​	5、使用底层模型不同，它们之间底层实现方式以及与客户端之间通信的应用协议不一样，Redis直接自己构建了VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求；

以上几点都比较好理解，下边我们针对多路 I/O 复用模型进行简单的探讨：

（1）多路 I/O 复用模型

多路I/O复用模型是利用 select、poll、epoll 可以同时监察多个流的 I/O 事件的能力，在空闲的时候，会把当前线程阻塞掉，当有一个或多个流有 I/O 事件时，就从阻塞态中唤醒，于是程序就会轮询一遍所有的流（epoll 是只轮询那些真正发出了事件的流），并且只依次顺序的处理就绪的流，这种做法就避免了大量的无用操作。

这里“多路”指的是多个网络连接，“复用”指的是复用同一个线程。采用多路 I/O 复用技术可以让单个线程高效的处理多个连接请求（尽量减少网络 IO 的时间消耗），且 Redis 在内存中操作数据的速度非常快，也就是说内存内的操作不会成为影响Redis性能的瓶颈，主要由以上几点造就了 Redis 具有很高的吞吐量。

