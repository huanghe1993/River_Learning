# [java设计模式]-工厂模式

## 1.工厂设计模式的定义

​	工厂模式使用的频率非常高，我们在开发中总能见到它们的身影。其定义为：Define an interface for creating an object, but let subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses.即定义一个用于创建对象的接口，让子类决定实例化哪一个类。工厂方法使一个类的实例化延迟到其子类。工厂方法模式的通用类图如下所示：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180406/A1fbg2gHge.png?imageslim)

## 2.工厂设计模式的几种形态

（1）简单工厂（Simple Factory）模式，又称静态工厂方法模式（Static Factory Method Pattern）。

（2）工厂方法（Factory Method）模式，又称多态性工厂（Polymorphic Factory）模式或虚拟构造子（Virtual Constructor）模式；

（3）抽象工厂（Abstract Factory）模式，又称工具箱（Kit 或Toolkit）模式。

##3. 简单工厂模式

​	简单工厂模式(Simple Factory Pattern)：又称为静态工厂方法(Static Factory Method)模式，它属于**类创建型模式**。在简单工厂模式中，可以根据自变量的不同返回不同类的实例。简单工厂模式专门定义一个类来负责创建其他类的实例，被创建的实例通常都具有共同的父类。

### 3.1.简单工厂模式角色

（1）工厂类（Creator）角色：担任这个角色的是工厂方法模式的核心，含有与应用紧密相关的商业逻辑。工厂类在客户端的直接调用下创建产品对象，它往往由一个具体Java 类实现。

（2）抽象产品（Product）角色：担任这个角色的类是工厂方法模式所创建的对象的父类，或它们共同拥有的接口。抽象产品角色可以用一个Java 接口或者Java 抽象类实现。

（3）具体产品（Concrete Product）角色：工厂方法模式所创建的任何对象都是这个角色的实例，具体产品角色由一个具体Java 类实现。

​	如图所示，Product抽象类负责定义产品的共性，实现对事物最抽象的定义，Creator为抽象工厂类，具体如何创建产品类由具体的实现工厂ConcreteCreator来完成。我们来看一下通用的模板代码：

```java
package factory.simple.factory;

/**
 * @Author: River
 * @Date:Created in  12:15 2018/4/6
 * @Description:
 */
public abstract class Product {
    public void method() { //产品类的公共方法，已经实现  
        //实现了公共的逻辑  
    }
    public abstract void method2(); //非公共方法，需要子类具体实现  
}

```

 具体产品类可以有多个，都继承与抽象类Product，如下：

```java
package factory.simple.factory;

/**
 * @Author: River
 * @Date:Created in  12:16 2018/4/6
 * @Description: 具体的产品1
 */
public class ConcreateProduct1 extends Product {

    @Override
    public void method2() {
        //product1的业务逻辑
    }
}
```

```java
package factory.simple.factory;

/**
 * @Author: River
 * @Date:Created in  12:17 2018/4/6
 * @Description: 具体的产品2
 */
public class ConcreateProduct2 extends Product {

    @Override
    public void method2() {
        //product的业务逻辑
    }
}

```

 抽象工厂类负责定义产品对象的产生，代码如下：

```java
public abstract class Creator {
    //创建一个产品对象，其输入参数类型可以自行设置
    public abstract <T extends Product> T createProduct(Class<T> clazz);
}
```

这里用的是泛型，传入的对象必须是Product抽象类的实现类。具体如何产生一个产品的对象，是由具体工厂类实现的，具体工厂类继承这个抽象工厂类：

```java
public class ConcreteCreator extends Creator {  
  
    @Override  
    public <T extends Product> T createProduct(Class<T> clazz) {  
        Product product = null;  
        try {  
            product = (Product) Class.forName(clazz.getName()).newInstance();  
        } catch (Exception e) { //异常处理  
            e.printStackTrace();  
        }  
        return (T) product;  
    }  
}  
```

通过这样的设计，我们就可以在测试类中随意生产产品了，看下面的测试类：

```java
public class FactoryTest {  
  
    public static void main(String[] args) {  
        Creator factory = new ConcreteCreator();  
        Product product1 = factory.createProduct(ConcreteProduct1.class); //通过不同的类创建不同的产品  
        Product product2 = factory.createProduct(ConcreteProduct2.class);  
         /* 
          * 下面继续其他业务处理 
          */  
     }  
}  
```

**我现在只需要一个工厂就可以把人生产出来，我干嘛要具体的工厂对象呢？我只要使用静态方法就好了。这样一想，抽象类去掉了，只保留了ConcreteCreator，就产生了下面的代码：**

```java
package factory.simple.factory;

/**
 * @Author: River
 * @Date:Created in  12:31 2018/4/6
 * @Description: 不用抽象的工厂直接的使用工厂的静态方法
 */
public class StaticFactory {

    public static <T extends Product> T createProduct(Class<T> clazz) {
        Product product = null;
        try {
            product = (Product) Class.forName(clazz.getName()).newInstance();
        } catch (Exception e) { //异常处理
            System.out.println("人种产生错误");
        }
        return (T) product;
    }
}
```

### 3.2反射加配置文件方式创建

新增配置文件product.properties

```
tv=com.zhaofeng.factory.simple.Tv
car=com.zhaofeng.factory.simple.Car
```

新增配置文件读取类，将读出来的内容存储到一个map中

```java
public class PropertyReader {
    public static Map<String, String> map = new HashMap<>();

    public Map<String, String> readPropertyFile(String fileName) {
        Properties pro = new Properties();
        InputStream in = getClass().getResourceAsStream(fileName);
        try {
            pro.load(in);
            Iterator<String> iterator = pro.stringPropertyNames().iterator();
            while (iterator.hasNext()) {
                String key = iterator.next();
                String value = pro.getProperty(key);
                map.put(key, value);
            }
            in.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return map;
    }
}

```

全新的工厂类如下：

```java
public class ProductFactory3 {
    public static Product produce(String key) throws Exception {
        PropertyReader reader = new PropertyReader();
        Map<String, String> map = reader.readPropertyFile("product.properties");
        try {
            Product product = (Product) Class.forName(map.get(key)).newInstance();
            return product;
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        throw new Exception("没有该产品");
    }
}

```

##4.工厂方法模式（Factory Method）

工厂方法模式定义一个用于创建对象的接口，让子类决定实例化哪一个类。Factory Method是一个类的实例化延迟到其子类。

在工厂方法模式中，核心的工厂类不再负责所有的产品的创建，而是将具体创建的工作交给子类去做。这个核心类则摇身一变，成为了一个抽象工厂角色，仅负责给出具体工厂子类必须实现的接口，而不接触哪一个产品类应当被实例化这种细节。其实这就是我前面说的不加静态方法的情形

## 5.抽象工厂模式（多个工厂模式）

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180406/HhcIibAHg7.png?imageslim)

这样的话AbstractHumanFactory抽象类我们就要改写了：

以前的：

```java
public abstract class AbstractHumanFactory{  
    public abstract <T extends Human> T createHuman(Class<T> clazz); //注意这里T必须是Human的实现类才行，因为要造Human嘛  
}  
```

```
public abstract class AbstractHumanFactory {  
    public abstract Human createHuman();  
}  
```

 	注意抽象方法中已经不需要再传递相关类的参数了，因为每个具体的工厂都已经非常明确自己的职责：创建自己负责的产品类对象。所以不同的工厂实现自己的createHuman方法即可：

```java
public class BlackHumanFactory extends AbstractHumanFactory {  
    public Human createHuman() {  
        return new BlackHuman();  
    }  
}  
public class YellowHumanFactory extends AbstractHumanFactory {  
    public Human createHuman() {  
        return new YellowHuman();  
    }  
}  
public class WhiteHumanFactory extends AbstractHumanFactory {  
    public Human createHuman() {  
        return new WhiteHuman();  
    }  
}  
```

这样三个不同的工厂就产生了，每个工厂对应只生产自己对应的人种。所以现在女娲造人就可以不用一个八卦炉了，分工协作，各不影响了！

```java
public class FactoryTest {  
    public static void main(String[] args) {  
        Human blackMan = new BlackHumanFactory().createHuman(); //黑人诞生了  
        Human yellowMan = new YellowHumanFactory().createHuman(); //黄人诞生了  
        Human whiteMan = new WhiteHumanFactory().createHuman(); //白人诞生了  
      }  
}  
```

这种工厂模式的好处是职责清晰，结构简单，但是给扩扩展性和可维护性带来了一定的影响，因为如果要扩展一个产品类，就需要建立一个相应的工厂类，这样就增加了扩展的难度。因为工厂类和产品类的数量是相同的，维护时也需要考虑两个对象之间的关系。但是这种模式还是很常用的。