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

# 如何mock static变量
当需要mock一些复杂的static变量, 例如下边Executor需要能抛出异常的场景, 就比较复杂, 可以参照如下样例:  

<script src="https://gist.github.com/DavyJones2010/5097454ac5ce68d2b317cb626850cbf0.js"></script>

# 如何mock 复杂的成员变量
当需要mock一些复杂的成员变量, 例如下边Executor需要能抛出异常的场景, 就比较复杂, 可以参照如下样例:

<script src="https://gist.github.com/DavyJones2010/745d629b457e36fcea0caa585d95dd62.js"></script>

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

# 如何根据不同的输入参数(值), mock不同的输出结果
> 尤其是在输入参数是个List的时候, 需要mock不同的输出

代码片段如下, 核心是使用Delegate, 完整样例参见:[JMockitTest.java](https://github.com/DavyJones2010/test-core/blob/master/src/test/java/edu/xmu/test/javase/jmockit/JMockitTest.java#L27) 
```java
new Expectations() {{
    // 由于这里 userDao 被mock了, 因此不会真正去执行 userDao.insert 方法
    userDao.insert((User) any);
    // 因此使用 Delegate 来根据不同的input来mock userDao.insert的不同output;
    // 如果output为void, 则使用 Delegate<Void>
    result = new Delegate<Void>() {
        // 方法签名需要mock的方法`insert`保持一致
        void insert(User usr) throws UserException {
            // 这里根据不同的input(usr), 对 userDao.insert 的结果进行mock
            if (usr.getName().equalsIgnoreCase("Wang")) {
                System.out.printf("User is Wang!");
                throw new UserException();
            }
        }
    };
}};
```

# 如何Mock @Injectable的Bean的void且修改了参数的方法
> 尤其是@Injectable的 Bean 的方法对输入参数执行了init等操作, 之后的步骤里依赖init之后的值

代码片段如下, 核心是使用Delegate, 完整样例参见:[JMockitTest.java](https://github.com/DavyJones2010/test-core/blob/master/src/test/java/edu/xmu/test/javase/jmockit/JMockitTest.java#L59)
```java
 new Expectations() {{
    // 这里没有真正去执行format, 因此没有把age进行规整
    userDao.format((User) any);
    // 虽然userDao.format无返回结果且被mock了(未执行), 但这里仍然可以使用 result = new Delegate<Void>() {} 对方法执行内容&结果进行Mock
    result = new Delegate<Void>() {
        // 方法签名需要mock的方法`format`保持一致
        public void format(User usr) {
            usr.setAge(25);
        }
    };
}};
```

# 如何mock通过@Resource定义了实际名称的Bean
- 例如要mock的对象如下: 通过`@Resource`定义了使用`userDaoLocalImpl`的实现

```java
@Service
public class UserService {
    @Resource(name = "userDaoLocalImpl")
    UserDao userDao;
//    ...
}
```

- 则在mock时, 需要保证`@Injectable`定义的变量, 变量名称必须与`@Resource`的`name`保持一致! 

```java
@RunWith(JMockit.class)
public class JMockitTest {
    @Tested
    UserService userService;
    
    @Injectable
    UserDao userDaoLocalImpl; // 这里变量名必须为: userDaoLocalImpl
}
```

- 否则会报错: 
```java
java.lang.IllegalStateException: Missing @Injectable for field
```

- 详细参见: [Error "Missing @Injectable for field" with @Resource annotation](https://github.com/jmockit/jmockit1/issues/359)
- 其他需要注意的: 在 1.23 及之前版本的jmockit, 没有对这个进行强限制, 即变量名称不与`@Resourc`的`name`保持一致也能mock成功; 但在 1.28 版本就进行了强限制.  

# Troubleshooting
## java.lang.NoSuchFieldError: $MMB
在本地环境无法复现, 但在集成测试环境, 就会偶现该错误. 
目前看起来没有好的办法, 只能把jmockit版本从 1.23 升级到 1.28, 目前看起来问题已经解决. (果然版本升级大法好? 😂)
[jmockit mockup, getting error java.lang.NoSuchFieldError: $MMB](https://stackoverflow.com/questions/35275899/jmockit-mockup-getting-error-java-lang-nosuchfielderror-mmb)


# 如何进行DAO层测试
数据库测试: [UNITILS库的使用经历](https://www.freesion.com/article/88601080583/)


## 如何防止自动回滚?
使用 `@Transactional(TransactionMode.COMMIT)`其中 `org.unitils.database.annotations.Transactional`

# 其他
- 由于项目历史依赖, 以及自身熟悉程度原因, 使用了 [JMockit - Development history](http://jmockit.github.io/changes.html) 作为Mock测试框架.
- 但该项目在2019年12月之后就停止了更新. 事实上也踩了坑, 不支持Mac M1/M2 ARM架构的JDK, 导致只能使用Hack的方式来绕过. 参见 [Issue #710](https://github.com/jmockit/jmockit1/issues/710)
- 所以针对新的应用, 建议使用 [GitHub - mockito](https://github.com/mockito/mockito), 虽然有一定的学习迁移成本, 但至少至今(2023年08月09日)仍在活跃维护中.
