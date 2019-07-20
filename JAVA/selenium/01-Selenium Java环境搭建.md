# Selenium Java环境搭建

[TOC]

## 一、介绍

Selenium是一个开源的和便携式的自动化软件测试工具，用于测试web应用程序有能力在各种浏览器和操作系统下运行。 Selenium是一套工具，可以有效的的基于web的应用程序的自动化。

## 二、安装相关依赖

### 通过jar包安装

点击 [Selenium下载](http://docs.seleniumhq.org/download/) 链接 ,下载version [3.141.59](https://bit.ly/2TlkRyu)点击版本号进行下载，下载完成将会得到一个 **selenium-server-standalone-3.141.59.jar** 文件。打开IntelliJ IDEA，导入.jar包。

方式1：jar包放在lib下 ---> 右键 ---->  Add  as Library

![1561541067884](C:\Users\huanghe\AppData\Roaming\Typora\typora-user-images\1561541067884.png)

方式二：

点击菜单栏 `File` --> `Project Structure`（快捷键Ctrl + Alt + Shift + s） ，点击 `Project Structure`界面左侧的`Modules`。在`Dependencies` 标签界面下，点击右边绿色的“+” 号，选择第一个选项`JARs or directories...` ，选择相应的 jar 包，点`OK` ，jar包添加成功。

![mark](http://man.hhaxmm.cn/blog/20190626/DDuoyv1U7Goz.png)

### 通过Maven安装

打开pom.xml 配置Selenium。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.mvn.demo</groupId>
    <artifactId>MyMvnPro</artifactId>
    <version>1.0-SNAPSHOT</version>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <dependencies>

        <!-- selenium-java -->
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
            <version>3.141.59</version>
        </dependency>

    </dependencies>

</project>
```

## 三、配置浏览器驱动

### 下载驱动

当selenium升级到3.0之后，对不同的浏览器驱动进行了规范。如果想使用selenium驱动不同的浏览器，必须单独下载并设置不同的浏览器驱动。

各浏览器下载地址：

Firefox浏览器驱动：[geckodriver](https://github.com/mozilla/geckodriver/releases)

Chrome浏览器驱动：[chromedriver](https://sites.google.com/a/chromium.org/chromedriver/home) [taobao备用地址](https://npm.taobao.org/mirrors/chromedriver)

IE浏览器驱动：[IEDriverServer](http://selenium-release.storage.googleapis.com/index.html)

Edge浏览器驱动：[MicrosoftWebDriver](https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver)

Opera浏览器驱动：[operadriver](https://github.com/operasoftware/operachromiumdriver/releases)

PhantomJS浏览器驱动：[phantomjs](http://phantomjs.org/)

注：部分浏览器驱动地址需要科学上网。

### 设置浏览器驱动

设置浏览器的地址非常简单。 我们可以手动创建一个存放浏览器驱动的目录，如： C:\driver , 将下载的浏览器驱动文件（例如：chromedriver、geckodriver）丢到该目录下。

我的电脑-->属性-->系统设置-->高级-->环境变量-->系统变量-->Path，将“C:\driver”目录添加到Path的值中。

- Path
- C:\driver

测试是否驱动安装好

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.firefox.FirefoxDriver;


public class TestDriver {
    public static void main(String[] args) {
        WebDriver driver1 = new ChromeDriver();    //Chrome浏览器

        WebDriver driver2 = new FirefoxDriver();   //Firefox浏览器

    }

}
```





