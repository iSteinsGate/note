# redis分布式锁

> **简介：**分布式锁的实现一般有三种方式：
>
> 1. 数据库乐观锁
> 2. 基于Redis的分布式锁
> 3. 基于Zookeeper的分布式锁

本文主要介绍Redis的分布式锁实现方式

## 要求

- 互斥性：一个锁只能被一个线程持有
- 高可用：不会发生死锁，即使客户端崩溃业可超时释放锁
- 加锁可解锁必须是同一个客户端，客户端不能把别人加的锁给释放了

## 方案

Redis具有极高的性能, 且其命令对分布式锁支持友好, 借助`SET`命令即可实现加锁处理.

> SET
>
> - `EX` *seconds* -- Set the specified expire time, in seconds.
> - `PX` *milliseconds* -- Set the specified expire time, in milliseconds.
> - `NX` -- Only set the key if it does not already exist.
> - `XX` -- Only set the key if it already exist.

```shell
set Lock_lock_key xxx EX 10 NX
```

NX保证互斥   EX保证高可用 

**加锁实现**

```java
boolean locked = redisTemplate.opsForValue().setIfAbsent(lockKey, lockValue, expireTime, TimeUnit.SECONDS);
```

**解锁实现：**使用lua脚本进行对比锁的值和删除操作，保证原子性。防止删了别人的锁

```java
private static final String COMPARE_AND_DELETE =
            "if redis.call('get',KEYS[1]) == ARGV[1] then return redis.call('del',KEYS[1]) else return 0 end";
Long result = redisTemplate.execute(new DefaultRedisScript<>(COMPARE_AND_DELETE, Long.class),Collections.singletonList(lockKey), lockValue);
boolean successFlag = Objects.equals(result, 1L);
```

## 代码实现

**RedisLock类实现**

```java
@Slf4j
public class RedisLock {

    /** 默认锁的有效时间(s) */
    private static final long DEFAULT_EXPIRE_TIME = 60L;
    /** 默认请求锁的超时时间(ms 毫秒) */
    private static final long DEFAULT_TIME_OUT = 3000L;
    /** 重试间隔(ms)*/
    private static final long WAIT_INTERVAL_IN_MS  = 10L;
    /** 锁前缀 */
    private static final String LOCK_KEY_PREFIX = "LOCK:";
    /** 锁后缀*/
    private static final String LOCK_KEY_SUFFIX = "_lock";
    /** 释放锁成功返回值 */
    private static final Long RELEASE_LOCK_SUCCESS_RESULT = 1L;
    /** redis操作*/
    private StringRedisTemplate redisTemplate;
    /** 锁对应的key */
    private String lockKey;
    /** 锁对应的值 */
    private String lockValue;
    /** 锁的有效时间(s) */
    private long expireTime = DEFAULT_EXPIRE_TIME;
    /** 请求锁的超时时间(ms) */
    private long timeOut = DEFAULT_TIME_OUT;
    /** 锁标记 */
    private volatile boolean locked = false;
    /**
     * 使用lua脚本进行对比锁的值和删除操作，保证原子性。防止删了别人的锁
     */
    private static final String COMPARE_AND_DELETE =
            "if redis.call('get',KEYS[1]) == ARGV[1] then return redis.call('del',KEYS[1]) else return 0 end";

    /**
     * 使用默认的过期时间和超时时间
     * @param redisTemplate
     * @param lockKey 锁的key
     */
    public RedisLock(StringRedisTemplate redisTemplate,String lockKey) {
        this.redisTemplate = redisTemplate;
        this.lockKey = LOCK_KEY_PREFIX+lockKey+LOCK_KEY_SUFFIX;
        this.lockValue = UUID.randomUUID().toString();
    }

    /**
     * 设置key过期时间
     * @param expireTime 锁key过期时间(s)
     * @return
     */
    public RedisLock expireTime(long expireTime) {
        this.expireTime = expireTime;
        return this;
    }

    /**
     * 设置获取锁超时
     * @param timeOut 加锁超时时间(ms)
     * @return
     */
    public RedisLock timeOut(long timeOut) {
        this.timeOut = timeOut;
        return this;
    }

    /**
     * 加锁,失败直接返回
     * @return
     */
    public boolean lock() {
        locked = redisTemplate.opsForValue().setIfAbsent(lockKey, lockValue, expireTime, TimeUnit.SECONDS);
        if (locked) {
            log.debug("lock success. key : " + lockKey + ",lockValue :"+ lockValue+", expire seconds : " + expireTime);
        } else {
            log.debug("lock fail. key : " + lockKey + ",lockValue :"+lockValue+", expire seconds : " + expireTime);
        }
        return locked;
    }
    /**
     * 尝试加锁,失败会重试直到超时
     * @return
     */
    public boolean tryLock(){
        // 超时时间：纳秒
        long timeout = timeOut * 1000000;
        //当前时间：纳秒
        long currentTime = System.nanoTime();
        while ((System.nanoTime() - currentTime) < timeout) {
            if (locked = redisTemplate.opsForValue().setIfAbsent(lockKey, lockValue, expireTime, TimeUnit.SECONDS)) {
                log.debug("lock success. key : " + lockKey + ",lockValue :"+ lockValue+", expire seconds : " + expireTime);
                return locked;
            }
            //等待一段时间继续获取
            sleep();
        }
        if (!locked) {
            log.debug("lock fail. key : " + lockKey + ",lockValue :"+lockValue+", expire seconds : " + expireTime);
        }
        return locked;
    }
    /**
     * 释放锁
     * @return
     */
    public boolean releaseLock() {
        if (locked) {
            Long result = redisTemplate.execute(new DefaultRedisScript<>(COMPARE_AND_DELETE, Long.class), Collections.singletonList(lockKey), lockValue);
            boolean successFlag = Objects.equals(result, RELEASE_LOCK_SUCCESS_RESULT);
            if (successFlag) {
                log.debug("release lock success. key : " + lockKey + ",lockValue :" + lockValue + ", expire seconds : " + expireTime);
            } else {
                log.debug("release lock fail. key : " + lockKey + ",lockValue :" + lockValue + ", expire seconds : " + expireTime);
            }
            return successFlag;
        } else {
            log.debug("lock未上锁, 不用释放. key: " + lockKey + ",lockValue :" + lockValue + ", expire seconds : " + expireTime);
        }
        return true;
    }

    /**
     * 睡眠
     */
    private void sleep() {
        try {
            TimeUnit.MILLISECONDS.sleep(WAIT_INTERVAL_IN_MS);
        } catch (InterruptedException e) {
            log.info("获取分布式锁休眠被中断：", e);
        }
    }
}
```

**LockService调用**

```java
@Service
public class LockService {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    /**
     * 加锁，如果未加锁成功，直接返回
     * @param lockKey 锁key
     * @param handler 执行的业务逻辑
     */
    public void lock(String lockKey, LockHandler handler) {
        RedisLock redisLock = new RedisLock(stringRedisTemplate, lockKey);
        try {
            if (redisLock.lock()) {
                handler.process();
            } else {
                ServiceException.normal(NormalErrorEnum.FAILURE,"未获取到锁，无法处理业务");
            }
        } finally {
            redisLock.releaseLock();
        }
    }
    /**
     * 尝试加锁，如果未加锁成功，会重试，直到超时
     * @param lockKey 锁key
     * @param handler 执行的业务逻辑
     */
    public void tryLock(String lockKey, LockHandler handler) {
        RedisLock redisLock = new RedisLock(stringRedisTemplate, lockKey);
        try {
            if (redisLock.tryLock()) {
                handler.process();
            } else {
                ServiceException.normal(NormalErrorEnum.FAILURE,"未获取到锁，无法处理业务");
            }
        } finally {
            redisLock.releaseLock();
        }
    }
    /**
     * 尝试加锁，如果未加锁成功，会重试，直到超时
     * @param lockKey 锁key
     * @param handler 执行的业务逻辑
     * @param timeout 加锁超时时间(ms)
     */
    public void tryLock(String lockKey, LockHandler handler,long timeout) {
        RedisLock redisLock = new RedisLock(stringRedisTemplate, lockKey).timeOut(timeout);
        try {
            if (redisLock.tryLock()) {
                handler.process();
            } else {
                ServiceException.normal(NormalErrorEnum.FAILURE,"未获取到锁，无法处理业务");
            }
        } finally {
            redisLock.releaseLock();
        }
    }
}
```

## 存在问题

- **单点问题。**上面的实现只要一个master节点就能搞定，这里的单点指的是单master，就算是个集群，如果加锁成功后，锁从master复制到slave的时候挂了，也是会出现同一资源被多个client加锁的。
- **执行时间超过了锁的过期时间。**上面写到为了不出现一直上锁的情况，加了一个兜底的过期时间，时间到了锁自动释放，但是，如果在这期间任务并没有做完怎么办？由于GC或者网络延迟导致的任务时间变长，很难保证任务一定能在锁的过期时间内完成。