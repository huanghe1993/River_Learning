# JUnit - 使用断言

## 断言

所有的断言都包含在 Assert 类中

```java
public class Assert extends java.lang.Object
```

这个类提供了很多有用的断言方法来编写测试用例。只有失败的断言才会被记录。**Assert** 类中的一些有用的方法列式如下：

| 序号 | 方法和描述                                                   |
| ---- | ------------------------------------------------------------ |
| 1    | **void assertEquals(boolean expected, boolean actual)** 检查两个变量或者等式是否平衡 |
| 2    | **void assertTrue(boolean expected, boolean actual)** 检查条件为真 |
| 3    | **void assertFalse(boolean condition)** 检查条件为假         |
| 4    | **void assertNotNull(Object object)** 检查对象不为空         |
| 5    | **void assertNull(Object object)** 检查对象为空              |
| 6    | **void assertSame(boolean condition)** assertSame() 方法检查两个相关对象是否指向同一个对象 |
| 7    | **void assertNotSame(boolean condition)** assertNotSame() 方法检查两个相关对象是否不指向同一个对象 |
| 8    | **void assertArrayEquals(expectedArray, resultArray)** assertArrayEquals() 方法检查两个数组是否相等 |

下面我们在例子中试验一下上面提到的各种方法。在 **C:\ > JUNIT_WORKSPACE** 路径下创建一个文件名为 TestAssertions.java 的类

```
import org.junit.Test;
import static org.junit.Assert.*;

public class TestAssertions {

   @Test
   public void testAssertions() {
      //test data
      String str1 = new String ("abc");
      String str2 = new String ("abc");
      String str3 = null;
      String str4 = "abc";
      String str5 = "abc";
      int val1 = 5;
      int val2 = 6;
      String[] expectedArray = {"one", "two", "three"};
      String[] resultArray =  {"one", "two", "three"};

      //Check that two objects are equal
      assertEquals(str1, str2);

      //Check that a condition is true
      assertTrue (val1 < val2);

      //Check that a condition is false
      assertFalse(val1 > val2);

      //Check that an object isn't null
      assertNotNull(str1);

      //Check that an object is null
      assertNull(str3);

      //Check if two object references point to the same object
      assertSame(str4,str5);

      //Check if two object references not point to the same object
      assertNotSame(str1,str3);

      //Check whether two arrays are equal to each other.
      assertArrayEquals(expectedArray, resultArray);
   }
}
```

接下来，我们在 **C:\ > JUNIT_WORKSPACE** 路径下创建一个文件名为 **TestRunner.java** 的类来执行测试用例

```
import org.junit.runner.JUnitCore;
import org.junit.runner.Result;
import org.junit.runner.notification.Failure;

public class TestRunner2 {
   public static void main(String[] args) {
      Result result = JUnitCore.runClasses(TestAssertions.class);
      for (Failure failure : result.getFailures()) {
         System.out.println(failure.toString());
      }
      System.out.println(result.wasSuccessful());
   }
} 
```

用 javac 编译 Test case 和 Test Runner 类

```
C:\JUNIT_WORKSPACE>javac TestAssertions.java TestRunner.java
```

现在运行将会运行 Test Case 类中定义和提供的测试案例的 Test Runner

```
C:\JUNIT_WORKSPACE>java TestRunner
```

检查运行结果

```
true
```

## 注释

注释就好像你可以在你的代码中添加并且在方法或者类中应用的元标签。JUnit 中的这些注释为我们提供了测试方法的相关信息，哪些方法将会在测试方法前后应用，哪些方法将会在所有方法前后应用，哪些方法将会在执行中被忽略。
JUnit 中的注释的列表以及他们的含义：

| 序号 | 注释和描述                                                   |
| ---- | ------------------------------------------------------------ |
| 1    | **@Test** 这个注释说明依附在 JUnit 的 public void 方法可以作为一个测试案例。 |
| 2    | **@Before** 有些测试在运行前需要创造几个相似的对象。在 public void 方法加该注释是因为该方法需要在 test 方法前运行。 |
| 3    | **@After** 如果你将外部资源在 Before 方法中分配，那么你需要在测试运行后释放他们。在 public void 方法加该注释是因为该方法需要在 test 方法后运行。 |
| 4    | **@BeforeClass** 在 public void 方法加该注释是因为该方法需要在类中所有方法前运行。 |
| 5    | **@AfterClass** 它将会使方法在所有测试结束后执行。这个可以用来进行清理活动。 |
| 6    | **@Ignore** 这个注释是用来忽略有关不需要执行的测试的。       |

在 **C:\ > JUNIT_WORKSPACE** 路径下创建一个文件名为 JunitAnnotation.java 的类来测试注释

```
import org.junit.After;
import org.junit.AfterClass;
import org.junit.Before;
import org.junit.BeforeClass;
import org.junit.Ignore;
import org.junit.Test;

public class JunitAnnotation {

   //execute before class
   @BeforeClass
   public static void beforeClass() {
      System.out.println("in before class");
   }

   //execute after class
   @AfterClass
   public static void  afterClass() {
      System.out.println("in after class");
   }

   //execute before test
   @Before
   public void before() {
      System.out.println("in before");
   }

   //execute after test
   @After
   public void after() {
      System.out.println("in after");
   }

   //test case
   @Test
   public void test() {
      System.out.println("in test");
   }

   //test case ignore and will not execute
   @Ignore
   public void ignoreTest() {
      System.out.println("in ignore test");
   }
}
```

接下来，我们在 **C:\ > JUNIT_WORKSPACE** 路径下创建一个文件名为 **TestRunner.java** 的类来执行注释

```
import org.junit.runner.JUnitCore;
import org.junit.runner.Result;
import org.junit.runner.notification.Failure;

public class TestRunner {
   public static void main(String[] args) {
      Result result = JUnitCore.runClasses(JunitAnnotation.class);
      for (Failure failure : result.getFailures()) {
         System.out.println(failure.toString());
      }
      System.out.println(result.wasSuccessful());
   }
} 
```

用 javac 编译 Test case 和 Test Runner 类

```
C:\JUNIT_WORKSPACE>javac TestAssertions.java TestRunner.java
```

现在运行将会运行 Test Case 类中定义和提供的测试案例的 Test Runner

```
C:\JUNIT_WORKSPACE>java TestRunner
```

检查运行结果

```
in before class
in before
in test
in after
in after class
true
```