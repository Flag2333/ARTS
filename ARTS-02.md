## ARTS-02

### **Algorithm**:

[26. 删除排序数组中的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)

```java
public int removeDuplicates(int[] nums) {
        if (nums.length == 0) {
            return 0;
        }
        int i = 0;// 慢
        for (int j = 1; j < nums.length; j++) {// 快
            // 如果不相同就慢+1 并且把快复制给慢 让两个相等
            if (nums[j] != nums[i]) {
                i++;
                nums[i] = nums[j];
            }
        }
        // 慢加上自身最开始所在位置 数组长度就是 慢移动次数+1
        return i+1;
    }

```

### **Review**：

### **Tip:**

@Transactional

aop代理，在运行时生成代理对象，这个代理对象决定@Transactional方法是否由拦截器来拦截，如果是拦截器则在目标方法开始执行之前创建并加入事务，根据执行情况来决定是否提交or回滚事务

- 只能应用在public方法上

  因为在拦截之前会先调用一个方法检查是否是public，如果不是则不会获取@Transactional的属性信息，所以不会拦截目标方法

- private等方法调用有@Transactional修饰的public方法也会被事务忽略出现异常事务不会回滚

造成上边两个的问题的主要原因就是aop代理的问题，解决方式用aspectJ取代Spring aop代理

### **Share**：

#### Redis学习（二）

####  redis的数据类型，以及每种数据类型的使用场景

- ##### String ：

  - 允许存储的数据的最大容量是512M

  - 可以存储

    1. 简单字符串
    2. 复杂字符串（json，xml）
    3. 数字（整数，浮点）
    4. 二进制（图片，音频，视频）

  - 批量设置值 mset key value [key value...]

    批量获取值 mget key [key...]

  - 计数 incr key

    结果有三种情况 

    1. 值不是整数，返回错误
    2. 值是整数，返回自增后的结果
    3. 键不存在，按照值为0自增，返回结果1

  - 内部编码 ：Redis根据字符串长度，自动选择内部编码

    1. int：8个字节长整型
    2. embstr：小于等于39个字节的字符串
    3. raw：大于39个字节的字符串

  - 使用场景

    1. 缓存功能：先从Redis中取数据，如果Redis中没有则从数据库获取，并且写回到Redis中并且设置过期时间
    2. 计数
    3. 共享session：把用户登录信息存到Redis中并设置过期时间7天等
    4. 限速：Redis开启计数模式，规定在某一时间长度内可以访问多少次，如果超过限制则不予通过

- ##### hash

  - 批量设置或获取field-value

    设置：hmset key field [key field...]

    获取：hmget key field [field...]

  - 计算field个数 hlen key

  - - 获取所有field hkeys key

    ​	1) "name"

    ​	2) "age"

    ​	3) "city"

    - 获取所有value hvals key

    ​	1) "mike"

    ​	2) "12"

    ​	3) "天津"

    - 获取所有的field-value hgetall key

    ​	1) "name"

    ​	2) "mike"

    ​	3) "age"

    ​	4) "12"

  - 内部编码

    1. ziplist （压缩列表）使用更加紧凑的结构实现多个元素的连续存储，在节省内存方面比hashtable更加优秀，条件：
       - 当哈希类型的元素个数小于 hash-max-ziplist-entries配置（默认512个）
       - 并且所有值都小于hash-max-ziplist-value配置（默认64个）
    2. hashtable （哈希表）不能用ziplist的时候用，读写时间复杂度O(1)

  - 使用场景

    hmset user:1 name tom age 23 city beijing

    hmset user:2 name tim age 24 habit football birth 19980425

    - 与关系型数据库的区别
      1. hash类型是稀疏的，关系型是结构化的；例如hash每个键可以有不同的field，而关系型添加一列，每一行都要设置值（即使为null）
      2. 关系型数据库可以进行关系查询，Redis不能

