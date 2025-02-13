# 分布式锁

在单机多线程环境中，我们经常遇到多个线程访问同一个共享资源，了维护数据的一致性，我们需要某种机制来保证只有满足某个条件的线程才能访问资源，不满足条件的线程只能等待，在下一轮竞争中重新满足条件时才能访问资源。这个机制指的是，为了实现分布式互斥，在某个地方做个标记，这个标记每个线程都能看到，到标记不存在时可以设置该标记，当标记被设置后，其他线程只能等待拥有该标记的线程执行完成，并释放该标记后，才能去设置该标记和访问共享资源。这里的标记，就是我们常说的锁。也就是说，锁是实现多线程同时访问同一共享资源，保证同一时刻只有一个线程可访问共享 资源所做的一种标记。

分布式锁是控制分布式系统之间同步访问共享资源的一种方式。在分布式系统中，常常需要协调他们的动作。如果不同的系统或是同一个系统的不同主机之间共享了一个或一组资源，那么访问这些资源的时候，往往需要互斥来防止彼此干扰来保证一致性，在这种情况下，便需要使用到分布式锁。为了保证多个进程能看到锁，锁被存在公共存储(比如 Redis、 Memcache、数据库等三方存储中)，以实现多个进程并发访问同一个临界资源，同一时 刻只有一个进程可访问共享资源，确保数据的一致性。

# 分布式锁的实现

## 分布式锁设计

衡量分布式锁实现的正确性有两个基本原则：

- 互斥性（Mutual Exclution）：即在分布式系统环境下，分布式锁应该能保证一个资源或一个方法在同一时间只能被一个机器的一个线程或进程操作。
- 存活性（Liveness）：无论是锁持有者发生故障，还是锁服务提供方发生局部故障，锁能在预期时间窗口内被其他存活者抢占并持有。

## 主流实现方案

实现分布式锁的 3 种主流方法，即:

- 基于数据库实现分布式锁，这里的数据库指的是关系型数据库;
- 基于缓存实现分布式锁;
- 基于 ZooKeeper 实现分布式锁。

| 特性         | 排名                      |
| ------------ | ------------------------- |
| 难易程度     | 数据库 > 缓存 > ZooKeeper |
| 实现的复杂性 | ZooKeeper > 缓存 > 数据库 |
| 性能         | 缓存 > ZooKeeper > 数据库 |
| 可靠性       | ZooKeeper > 缓存 > 数据库 |

总结来说，ZooKeeper 分布式锁的可靠性最高，有封装好的框架，很容易实现分布式锁的 功能，并且几乎解决了数据库锁和缓存式锁的不足，因此是实现分布式锁的首选方法。

## 生存时间

为避免特殊原因导致锁无法释放, 在加锁成功后, 锁会被赋予一个生存时间(通过 lock 方法的参数设置或者使用默认值), 超出生存时间锁将被自动释放。锁的生存时间默认比较短(秒级, 具体见 lock 方法), 因此若需要长时间加锁, 可以通过 expire 方法延长锁的生存时间为适当的时间. 比如在循环内调用 expire。系统级的锁当进程无论因为任何原因出现 crash，操作系统会自己回收锁，所以不会出现资源丢失；但分布式锁不同。若一次性设置很长的时间，一旦由于各种原因进程 crash 或其他异常导致 unlock 未被调用，则该锁在剩下的时间就变成了垃圾锁，导致其他进程或进程重启后无法进入加锁区域。

# Links

- https://mp.weixin.qq.com/s/35aCS_5GqLyzZS3VobL6fg 万字长文说透分布式锁  

- https://cubox.pro/c/mjHTaU https://mp.weixin.qq.com/s/tdkwSK1jd7tAznYTXVYOhA 浅谈分布式锁 提取其中内容到独立分布式锁对比

- https://mp.weixin.qq.com/s?__biz=MjM5ODI5Njc2MA==&mid=2655821234&idx=1&sn=da774b745fc11bf07e76ce10faa811ba&chksm=bd74d0658a035973b790c0b9cd702a90a15987cd1f895323240e987439a3a6a94c17ae332f31&scene=21%23wechat_redirect

- https://zhuanlan.zhihu.com/p/80311843

- https://zhuanlan.zhihu.com/p/88607398

- https://mp.weixin.qq.com/s/hOPT41HIAGE8iZ5jLlREmg 看似忠良的分布式锁
