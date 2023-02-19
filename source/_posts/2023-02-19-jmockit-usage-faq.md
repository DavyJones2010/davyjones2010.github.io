---
title: jmockit测试框架常用技巧
date: 2023-02-19 23:38:39
tags: [java, jmockit, unit-test, ut, test-framework]
---

# @Injectable  与 @Mocked 有啥区别?
[https://www.cnblogs.com/shoren/p/jmokit-summary.html](https://www.cnblogs.com/shoren/p/jmokit-summary.html)

1. Injectable只会mock当前实例
2. Mocked会mock该类下所有实例

针对 @Tested 的类, 所有field必须使用 Injectable 来注入, 使用 Mocked 来标识的properties, 不会被注入.

# 如何测试private方法?

`mockit.Deencapsulation#invoke(java.lang.Object, java.lang.String, java.lang.Object...)`

---

## 如何测试private方法, 并且传入null作为参数?

使用`Deencapsulation.invoke` 测试private方法时, 如果需要传入null作为参数, 如果直接传入null,  会报错, 样例如下:

```java
// 第1, 第3个参数为null
Deencapsulation.invoke(testResourceService, "testMethod",
            new Object[] {null, param1, null, param2, param3, param4, param5});
```

```java
java.lang.IllegalArgumentException: Invalid null value passed as argument 0

	at com.xxx.ServiceImplTest.testMethod(xxxTest.java:1141)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:69)
	at com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:33)
	at com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:235)
	at com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:54)

```

解决方案:

```java
// 第1, 第3个参数, 使用 类名.class 来传入
Deencapsulation.invoke(testResourceService, "testMethod",
            new Object[] {Integer.class, param1, String.class, param2, param3, param4, param5});
```

参见:  [https://stackoverflow.com/questions/23096026/how-to-pass-null-string-to-a-private-method-using-jmockit-while-unit-testing-it](https://stackoverflow.com/questions/23096026/how-to-pass-null-string-to-a-private-method-using-jmockit-while-unit-testing-it)

# 如何mock static方法

例如需要mock EnvUtil.get() 这个static方法, 如下即可:

```java
new MockUp<EnvUtil>() {
    @Mock
    public <T> T get(String key) { // 注意, 这里不要加 static 标识
        if ("inventoryResourceProviderUseLocal".equalsIgnoreCase(key)) {
            return (T)"true";
        }
        return (T)"false";
    }
};

```

参见: [https://www.baeldung.com/jmockit-static-method](https://www.baeldung.com/jmockit-static-method)

# 如何mock Tested对象的方法

由于 Tested 的对象, 通常都是当前UT需要测试的对象本身. 在测试的目标方法(如下例子中的calcUserScore), 需要本身依赖到当前类的其他方法, 且逻辑非常复杂(例如下例中的getUserById), 则可以将测试对象本身的部分方法也进行mock. 如下:

```java
@Slf4j
@RunWith(JMockit.class)
public class UserServiceMockTest {
    @Tested
    UserServiceImpl userService;

    
    @Test
    public void calcUserScoreTest() {

        new MockUp<UserServiceImpl>() {
            @Mock
            public User getUserById(Long uid) {
                return new User(xxx);
            }
        };
        userService.calcUserScore(1);
    }

}
```

# 如何mock Injectable对象的方法

Injectable对象通常是当前要测试对象依赖的其他对象. 方法如下:

```java
@Slf4j
@RunWith(JMockit.class)
public class UserServiceMockTest {
    @Tested
    UserServiceImpl userService;

    @Injectable
    AddrService addrSvc;

    
    @Test
    public void getUsrAddrTest() {
        Addr testAddr = new Addr("cn", "hangzhou");

        new Expectations() {{
            addrSvc.getAddrByAddrId(anyLong);
            result = testAddr;
        }};
        
        userService.getUsrAddr(1);
    }

}
```

# 如何进行DAO层测试
数据库测试: [UNITILS库的使用经历](https://www.freesion.com/article/88601080583/)


## 如何防止自动回滚?
使用 `@Transactional(TransactionMode.COMMIT)`其中 `org.unitils.database.annotations.Transactional`
