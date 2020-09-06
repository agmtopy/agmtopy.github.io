---
title: 基于testcontainers的集成测试方案
date: 2020-07-31 22:42:41
categories: 杂记
tags: 测试
---

# 基于testcontainers的集成测试方案
> Testcontainers是一个Java库，支持JUnit测试，它提供常见数据库，Selenium Web浏览器或可以在Docker容器中运行的任何其他东西的轻量级，一次性的实例。

<B>testcontainers</B>官网为https://www.testcontainers.org/


## 基础工程
1. docker环境
2. springBoot工程

### docker环境
> https://download.docker.com/win/stable/Docker%20Desktop%20Installer.exe
下载Docker Desktop Installer.exe进行安装Docker
- 查看Docker版本
![a3z3Md.png](https://s1.ax1x.com/2020/08/01/a3z3Md.png)
- Docker 服务启动/关闭命令

```bash
    # 启动docker服务
    Net start com.docker.service

    # 停止docker服务
    Net stop com.docker.service
```



### springBoot应用

> https://github.com/agmtopy/testcontainers-simple

## redis
- RedisTestContainersTest.java

```java
@Testcontainers
public class RedisTestContainersTest {

    private RedisCommands<String,String> redisCommands;

    /**
     * 初始化redis 容器
     */
    @Container
    public GenericContainer redis = new GenericContainer("redis:5.0.3-alpine")
            .withExposedPorts(6379);

    @BeforeEach
    public void setUp() {
        String address = redis.getHost();
        Integer port = redis.getFirstMappedPort();
        //初始化redis客户端
        redisCommands = RedisClient.create(String.format("redis://%s:%d/0", address, port)).connect().sync();
    }

    @Test
    public void testSimplePutAndGet() {
        redisCommands.set("test", "example");
        String retrieved = redisCommands.get("test");
        System.out.println("retrieved : " + retrieved);
        assertEquals("example", retrieved);
    }
}

```

## mysql

- MysqlTestContainersTest

```java
/**
 * mysql 容器测试
 */
@SpringBootTest(
        classes = MysqlTestConfiguration.class,
        properties = {
                "spring.profiles.active=test",
                "embedded.mysql.install.enabled=true",
                "embedded.mysql.init-script-path=initScript.sql"
        })
public class MysqlTestContainersTest {

    @Autowired
    ConfigurableEnvironment environment;

    /**
     * jdbcTemplate
     */
    @Autowired
    JdbcTemplate jdbcTemplate;

    @Test
    public void propertiesAreAvailable() {
        assertThat(environment.getProperty("embedded.mysql.port")).isNotEmpty();
        assertThat(environment.getProperty("embedded.mysql.host")).isNotEmpty();
        assertThat(environment.getProperty("embedded.mysql.schema")).isNotEmpty();
        assertThat(environment.getProperty("embedded.mysql.user")).isNotEmpty();
        assertThat(environment.getProperty("embedded.mysql.password")).isNotEmpty();
        assertThat(environment.getProperty("embedded.mysql.init-script-path")).isNotEmpty();
    }

    @Test
    public void shouldInitDBForMySQL() throws Exception {
        assertThat(jdbcTemplate.queryForObject("select count(first_name) from users where first_name = 'Sam' ", Integer.class)).isEqualTo(1);
    }
}
```


## kafka
- 



## 参考资料
[testcontainers官网](https://www.testcontainers.org/)
[testcontainers项目地址](https://github.com/testcontainers/testcontainers-java/)
[testcontainers-spring-boot](https://github.com/testcontainers/testcontainers-spring-boot)

