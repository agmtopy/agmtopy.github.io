---
title: redis如何实现持久化的
date: 2022-01-20 23:07:22
categories: 杂记
tags:
  - redis
---

# redis如何实现持久化的

redis目前实现持久化主要有两种方式,一种是通过<B>RDB文件</B>,另外一种是通过<B>AOF文件</B>.
redis对于持久化支持4种部署方式
- 无持久性
- RDB
- AOF
- RDB + AOF

让就让我们来详细比较和分析以下redis底层是如何实现的RDB和AOF机制的

## RDB文件

- RDB定义
> RDB 持久性指的是在指定时间间隔后执行记录当前数据集的全部数据快照

RDB文件就类似于MySql进行全量备份,记录某一个时间点的全部数据

- RDB优点
1. 文件即数据,不浪费空间
2. 异步生成文件,性能损耗最小
3. 不需要重新执行指令,数据恢复更快
4. 支持主从切换后的数据同步

- RDB缺点
1. 没有采用LSM日志处理,会丢失数据
2. 对于需要持久化大量数据时,性能不如AOF增量模式

### 使用实例
RDB有两种触发方式:
1. 配置文件触发
2. 命令触发

- 配置文件触发

配置文件触发为在<B>redis.conf</B>中设置

```config

  #文件路径
  dir

  #RDB文件名称
  dbfilename 

  #是否启用数据校验
  rdbchecksum yes

  #是否启用RDB文件压缩格式
  rdbcompression yes

  #bgsave异常时是否停止写入缓存
  stop-writes-on-bgsave-error yes

  #60秒内有5个值发生变化时触发生成RDB文件
  save 60 5

```

- 命令触发

  - save 
    同步阻塞命令
  - bgsave
    fork子进程异步生成RDB文件


### 源码解析

- bgsave的执行调用链


![bgsave的执行链](https://hexo-1254947285.cos.ap-chengdu.myqcloud.com/hexo/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202022-01-22%20230847.jpg)

>  <B>server.main()</B> -> <B>server.processCommand()</B> -> <B>rdb.rdbSaveBackground()</B> -> <B>rdb.redisFork()</B>

下面自底向上依次分析下这个四个方法

 - <B>rdbSave()</B>

  ```c
    /* 将数据保存在磁盘上 */
    int rdbSave(char *filename, rdbSaveInfo *rsi) {
        char tmpfile[256];
        char cwd[MAXPATHLEN]; /* Current working dir path for error messages. */
        FILE *fp = NULL;
        rio rdb;
        int error = 0;

        snprintf(tmpfile,256,"temp-%d.rdb", (int) getpid());
        fp = fopen(tmpfile,"w");
        if (!fp) {
            char *cwdp = getcwd(cwd,MAXPATHLEN);
            serverLog(LL_WARNING,
                "Failed opening the RDB file %s (in server root dir %s) "
                "for saving: %s",
                filename,
                cwdp ? cwdp : "unknown",
                strerror(errno));
            return C_ERR;
        }

        rioInitWithFile(&rdb,fp);
        startSaving(RDBFLAGS_NONE);

        if (server.rdb_save_incremental_fsync)
            rioSetAutoSync(&rdb,REDIS_AUTOSYNC_BYTES);

        if (rdbSaveRio(&rdb,&error,RDBFLAGS_NONE,rsi) == C_ERR) {
            errno = error;
            goto werr;
        }

        /* Make sure data will not remain on the OS's output buffers */
        if (fflush(fp)) goto werr;
        if (fsync(fileno(fp))) goto werr;
        if (fclose(fp)) { fp = NULL; goto werr; }
        fp = NULL;
        
        //重命名文件来实现写入文件的原子性
        if (rename(tmpfile,filename) == -1) {
            char *cwdp = getcwd(cwd,MAXPATHLEN);
            serverLog(LL_WARNING,
                "Error moving temp DB file %s on the final "
                "destination %s (in server root dir %s): %s",
                tmpfile,
                filename,
                cwdp ? cwdp : "unknown",
                strerror(errno));
            unlink(tmpfile);
            stopSaving(0);
            return C_ERR;
        }

        serverLog(LL_NOTICE,"RDB落盘成功");
        server.dirty = 0;
        server.lastsave = time(NULL);
        server.lastbgsave_status = C_OK;
        stopSaving(1);
        return C_OK;

    werr:
        serverLog(LL_WARNING,"Write error saving DB on disk: %s", strerror(errno));
        if (fp) fclose(fp);
        unlink(tmpfile);
        stopSaving(0);
        return C_ERR;
    }

  ```


<B>rdbSave()</B>完成对RDB文件的写入操作:
1. 创建一个临时文件,将内存中的数据进行写入,然后刷盘
2. 对临时文件进行rename(👍写入文件不一定是原子,但是rename file一定是原子性的),来保证文件写入的原子性

- <B>rdbSaveBackground</B>

```c
int rdbSaveBackground(char *filename, rdbSaveInfo *rsi) {
    pid_t childpid;

    if (hasActiveChildProcess()) return C_ERR;

    server.dirty_before_bgsave = server.dirty;
    server.lastbgsave_try = time(NULL);

    //fork子进程进行处理RDB文件,这里的fork()返回值-1表示没有创建新进程成功,0表示创建新进程成功
    if ((childpid = redisFork(CHILD_TYPE_RDB)) == 0) {
        int retval;

        /* Child */
        redisSetProcTitle("redis-rdb-bgsave");
        redisSetCpuAffinity(server.bgsave_cpulist);
        serverLog(LL_WARNING,"RDB文件为: %s",filename);
        retval = rdbSave(filename,rsi);//生成rdb文件
        if (retval == C_OK) {
            sendChildCowInfo(CHILD_INFO_TYPE_RDB_COW_SIZE, "RDB");
        }
        exitFromChild((retval == C_OK) ? 0 : 1);
    } else {
        /* 父进程处理 */
        if (childpid == -1) {
            server.lastbgsave_status = C_ERR;
            serverLog(LL_WARNING,"Can't save in background: fork: %s",
                strerror(errno));
            return C_ERR;
        }
        serverLog(LL_NOTICE,"Background saving started by pid %ld",(long) childpid);
        server.rdb_save_time_start = time(NULL);
        server.rdb_child_type = RDB_CHILD_TYPE_DISK;
        return C_OK;
    }
    return C_OK; /* unreached */
}
```

在这个方法内部通过<B>redisFork()</B>创建了一个子进程来完成<B>rdbSave()</B>的调用

父进程不会阻塞而是直接打印RDB文件开始处理的消息

需要注意一点的是fork()进程的方法返回值是0-成功/-1-失败

在这一步的调用中省略<B>rdb.bgsaveCommand()</B>的调用逻辑,直接分析<B>server.c</B>中的逻辑


- <B>server.c()</B>

```c
struct redisCommand redisCommandTable[] = {
    {"module",moduleCommand,-2,
     "admin no-script",
     0,NULL,0,0,0,0,0,0},

    {"get",getCommand,2,
     "read-only fast @string",
     0,NULL,1,1,1,0,0,0},
    {"bgsave",bgsaveCommand,-1,
     "admin no-script",
     0,NULL,0,0,0,0,0,0},
    //省略其他命令
}


```

这里使用命令模式将所有客户端命令以及命令的处理方式放到<B>redisCommandTable</B>中进行处理,然后在和网络时间绑定就形成最开始说的调用链结构


- cron方式启动
cron方式启动的逻辑位于<B>server.serverCron()</B>中




### 小结

RDB文件是通过fork()子进程来处理文件,采用的是rename temp file的方式保证原子性. 父进程并不会阻塞,而是启动子进程后立即返回




## AOF文件
AOF文件就类似于MySql中的binlog使用的Statement格式记录数据变化,每次只记录指令,并且超过设定的大小后会进行覆盖

### 使用实例

- 配置文件

```config
  #开启redis aof
  appendonly yes
  
  #aof刷新机制 always:每一条都写入磁盘,everysec:每秒写入磁盘一次,no:文件系统刷新
  appendfsync everysec

  #aof文件扩容的阀值比例
  auto-aof-rewrite-percentage 100

  #aof文件重写后的初始大小
  auto-aof-rewrite-min-size 64mb

```

### 源码解析




### 小结


## 总结

## 参考文章

[Redis Persistence](https://redis.io/topics/persistence)
[Redis persistence demystified](http://oldblog.antirez.com/post/redis-persistence-demystified.html)
[Redis 延迟问题排查](https://redis.io/topics/latency)
[TxFS：利用文件系统崩溃一致性来提供 ACID 事务](https://dl.acm.org/doi/fullHtml/10.1145/3318159)