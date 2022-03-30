---
title: Leaf美团点评分布式ID生成系统
date: 2020-07-12 21:49:54
categories: 杂记

---

# Leaf-美团点评分布式ID生成系统
leaf作为一个分布式id生成系统，代码简洁且高效，理论指导部分为[Leaf——美团点评分布式ID生成系统](https://tech.meituan.com/2017/04/21/mt-leaf.html),工程实践为https://github.com/Meituan-Dianping/Leaf

## 理论
leaf提供了三种方式的分布式id生成方案：
1. 始终为0
2. 号段模式
3. 雪花算法模式 

号段模式通过提前获取号段来优化算法
雪花算法通过等待或超时异常的方式来处理时钟回拨问题


## 代码分析

- SegmentIDGenImpl号段模式实现

1. 对号段进行初始化
```java
if (cache.containsKey(key)) {
    SegmentBuffer buffer = cache.get(key);
      if (!buffer.isInitOk()) {
          //用synchronize锁住即将进行初始化从buffer
          synchronized (buffer) {
              //再次校验资源是否符合
              if (!buffer.isInitOk()) {
                  try {
                      //初始化号段
                      updateSegmentFromDb(key, buffer.getCurrent());
                      logger.info("Init buffer. Update leafkey {} {} from db", key, buffer.getCurrent());
                      buffer.setInitOk(true);
                  } catch (Exception e) {
                      logger.warn("Init buffer {} exception", buffer.getCurrent(), e);
                  }
              }
          }
      }
      //反之从号段中获取
      return getIdFromSegmentBuffer(cache.get(key));
  }
```

2. 获取id

```java
public Result getIdFromSegmentBuffer(final SegmentBuffer buffer) {
        //自旋
        while (true) {
            buffer.rLock().lock();
            try {
                final Segment segment = buffer.getCurrent();
                //判断是否占用下一个号段
                if (!buffer.isNextReady() && (segment.getIdle() < 0.9 * segment.getStep()) && buffer.getThreadRunning().compareAndSet(false, true)) {
                    //异步开启占用下一号段
                    service.execute(new Runnable() {
                        @Override
                        public void run() {
                            Segment next = buffer.getSegments()[buffer.nextPos()];
                            boolean updateOk = false;
                            try {
                                updateSegmentFromDb(buffer.getKey(), next);
                                updateOk = true;
                                logger.info("update segment {} from db {}", buffer.getKey(), next);
                            } catch (Exception e) {
                                logger.warn(buffer.getKey() + " updateSegmentFromDb exception", e);
                            } finally {
                                if (updateOk) {
                                    buffer.wLock().lock();
                                    buffer.setNextReady(true);
                                    buffer.getThreadRunning().set(false);
                                    buffer.wLock().unlock();
                                } else {
                                    buffer.getThreadRunning().set(false);
                                }
                            }
                        }
                    });
                }
                //获取值segment.getValue是线程安全的AtomicLong
                long value = segment.getValue().getAndIncrement();
                if (value < segment.getMax()) {
                    return new Result(value, Status.SUCCESS);
                }
            } finally {
                buffer.rLock().unlock();
            }
            //等待，再次获取
            waitAndSleep(buffer);
            buffer.wLock().lock();
            try {
                final Segment segment = buffer.getCurrent();
                long value = segment.getValue().getAndIncrement();
                if (value < segment.getMax()) {
                    return new Result(value, Status.SUCCESS);
                }
                if (buffer.isNextReady()) {
                    buffer.switchPos();
                    buffer.setNextReady(false);
                } else {
                    logger.error("Both two segments in {} are not ready!", buffer);
                    return new Result(EXCEPTION_ID_TWO_SEGMENTS_ARE_NULL, Status.EXCEPTION);
                }
            } finally {
                buffer.wLock().unlock();
            }
        }
    }
```

3. 异步从数据库中获取号段

- SnowflakeIDGenImpl雪花算法实现

1. 初始化

```java

/**
     * @param zkAddress zk地址
     * @param port      snowflake监听端口
     * @param twepoch   起始的时间戳
     */
    public SnowflakeIDGenImpl(String zkAddress, int port, long twepoch) {
        this.twepoch = twepoch;
        Preconditions.checkArgument(timeGen() > twepoch, "Snowflake not support twepoch gt currentTime");
        //获取第一个网卡的ip地址
        final String ip = Utils.getIp();
        SnowflakeZookeeperHolder holder = new SnowflakeZookeeperHolder(ip, String.valueOf(port), zkAddress);
        LOGGER.info("twepoch:{} ,ip:{} ,zkAddress:{} port:{}", twepoch, ip, zkAddress, port);
        //向zk注册机器和临时节点
        boolean initFlag = holder.init();
        if (initFlag) {
            workerId = holder.getWorkerID();
            LOGGER.info("START SUCCESS USE ZK WORKERID-{}", workerId);
        } else {
            Preconditions.checkArgument(initFlag, "Snowflake Id Gen is not init ok");
        }
        Preconditions.checkArgument(workerId >= 0 && workerId <= maxWorkerId, "workerID must gte 0 and lte 1023");
  }

```

- 获取id

```java
public synchronized Result get(String key) {
        long timestamp = timeGen();//获取当前时间
        if (timestamp < lastTimestamp) {//处理时钟回拨
            long offset = lastTimestamp - timestamp;//时钟回拨超过5就进行等待
            if (offset <= 5) {
                try {
                    wait(offset << 1);
                    timestamp = timeGen();
                    if (timestamp < lastTimestamp) {
                        return new Result(-1, Status.EXCEPTION);
                    }
                } catch (InterruptedException e) {//可响应中断的
                    LOGGER.error("wait interrupted");
                    return new Result(-2, Status.EXCEPTION);
                }
            } else {
                return new Result(-3, Status.EXCEPTION);
            }
        }
        if (lastTimestamp == timestamp) {
            sequence = (sequence + 1) & sequenceMask;
            if (sequence == 0) {
                //seq 为0的时候表示是下一毫秒时间开始对seq做随机
                sequence = RANDOM.nextInt(100);
                //如果时间相等内部就进行等待
                timestamp = tilNextMillis(lastTimestamp);
            }
        } else {
            //如果是新的ms开始
            sequence = RANDOM.nextInt(100);
        }
        lastTimestamp = timestamp;
        //计算id
        long id = ((timestamp - twepoch) << timestampLeftShift) | (workerId << workerIdShift) | sequence;
        return new Result(id, Status.SUCCESS);
    }
```