# Gradle

[TOC]

##Gradle是什么

自动化的构建工具，一个开源的项目自动化构建工具，建立在apache ant和Apache Maven概念的基础上，并引入基于Groovy的特定领域语言（DSL）,而不再是使用XML的形式管理构建脚本

## 安装Gradle

- 配置环境变量，<font color='red'>GRADLE_HOME</font>
- 添加到path,<font color='red'>%GRADLE_HOME%\bin</font>
- 验证是否安装成功，<font color='red'>gradle -v</font>

## Groovy

Groovy是用于java虚拟机的一种敏捷动态语言，它是一种成熟的面向对象编程语言，既可以用于面向对象进行编程，又可以作为纯粹的脚本语言。使用这种语言不需要编写过多的代码，同时又具有闭包和动态语言中的其他特性

- Groovy是完全的兼容java语法的
- 分号是可选的
- 类和方法默认是public 
- 编译器给属性添加get和set方法
- 属性可以直接的使用点号获取到

高效的Groovy特性：

- assert语句
- 可选类型的定义
- 可选括号
- 字符串
- 集合API
- 闭包

## 构建Gradle项目

使用idea构建Gradle项目

![1538794837524](C:\Users\huanghe\AppData\Roaming\Typora\typora-user-images\1538794837524.png)

### 高级特性

```groovy
//Groovy的高级特性

//1:可选的类型定义（def不是一个明确的类型，类似于javaScript里面的var,变量的类型是自动的推断出来的）
def  version =1

//2:assert在任何地方都是可以执行的
assert version ==1

//3:括号是可以选则的
println(version)

//4:字符串
//可以插入变量
def  s1="huanghe version is ${version}"
def  s2='huanghe '
//可以换行
def  s3='''my name
 is
 huanghe'''

println(s1)
println(s2)
println(s3)

//5:集合API
def bulidTools=['ant','maven']
bulidTools << 'gradle'
assert bulidTools.getClass() == ArrayList
assert bulidTools.size() == 3
//map(LinkedhHashMap)
def buildYears = ['ant': 2000, 'maven': 2004]
buildYears.gradle = 2009

println(buildYears.ant)
println(buildYears['gradle'])\

//6：闭包
//在Gradle的脚本中闭包经常当做方法参数使用
def c1={
    v ->
        println(v)
}

def c2={
    println('hello')
}

def method1(Closure closure) {
    closure('param')
}

def method2(Closure closure){
    closure()
}

method1(c1);
method2(c2);
```

### groovy最常用的功能

```groovy
//gradle的构建脚本

//构建脚本中默认都是有个project实例

//project实例上有一个方法叫apply    plugin的参数的值为java
apply plugin:'java'

//project实例上有一个属性叫做version
version ='0.1'

//repositories是一个方法 , mavenCentral()是一个参数，作为参数进行传递
repositories{
    mavenCentral()
}

dependencies{
    compile 'commons-codec:commons-codec:1.6'
}
```

## gradle java项目

