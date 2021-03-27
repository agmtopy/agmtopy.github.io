---
title: mockito使用小结
date: 2020-10-13 22:02:22
categories: mockito
tags: mockito
---
# mockito使用小结
mock框架是测试中必不可少的，它的主要作用是模拟一些在应用中不容易构造或者比较复杂的对象，从而把测试与测试边界以外的对象隔离开。目前市面上流行的mock框架主要有Mockito、JMock、EasyMock、JMock。这几种框架的对比如下

## Mockito使用示例

- 模拟对象

```java
    @Test
    void mockMethod() {
        //模拟一个MockTestInterface的实例
        MockTestInterface mock = Mockito.mock(MockTestInterface.class);
        //未指定方法返回值因此打印null
        System.out.println(mock.method1());
    }
```

- 模拟方法的返回值

```java
    @Test
    void mockMethod2() {
        //模拟一个MockTestInterface的实例
        MockTestInterface mock = Mockito.mock(MockTestInterface.class);

        //1.mock方式不会执行指定方法
        Mockito.doReturn("mock method").when(mock).method2();
        Assert.hasText(mock.method2(),"mock method");

        //2.stub方式会执行指定方法
        MockTestInterface mock1 = Mockito.spy(MockTestInterface.class);
        Mockito.when(mock1.method2()).thenReturn("mock method");
        Assert.hasText(mock1.method2(),"mock method");
    }
```

- 模拟方法抛出异常

```java
    void mockMethod3() {
        //模拟一个MockTestInterface的实例
        MockTestInterface mock = Mockito.mock(MockTestInterface.class);
        Mockito.doThrow(new NullPointerException("mock NPE")) .when(mock).method2();
        assertThrows(NullPointerException.class,()-> mock.method2(),"mock NPE");
    }
```










ps:Mock与Stub的区别
Mock主要是对不可信赖的代码进行"模拟行为"，Stub主要是将不可信赖的代码片段用"简化"的代码片段进行替代


https://segmentfault.com/a/1190000010385247