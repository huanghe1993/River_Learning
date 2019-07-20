# springMVC @initBinder 使用

个主要是讲解一下自己遇到的@InitBinder的问题，这个@InitBinder实际上就是在springmvc中，当表单向后台传递日期类型的数据时，由于springmvc不能自动在后台转化Date类型的参数（int可以自动转化成String等等），所以用到了@InitBinder。实际上客户端向服务器传递参数都是通过request.getparameter来传递的，传递到后台实际上是字符串，这时我们就需要一个转化工具WebDataBinder，将字符串转化成我们所需要的类型。 

在SpringMVC中，bean中定义了Date，double等类型，如果没有做任何处理的话，日期以及double都无法绑定。

解决的办法就是使用spring mvc提供的@InitBinder标签

在我的项目中是在BaseController中增加方法initBinder，并使用注解@InitBinder标注，那么spring m[vc](http://www.2cto.com/kf/ware/vc/)在绑定表单之前，都会先注册这些编辑器，当然你如果不嫌麻烦，你也可以单独的写在你的每一个controller中。剩下的控制器都继承该类。spring自己提供了大量的实现类，诸如CustomDateEditor  ，CustomBooleanEditor，CustomNumberEditor等许多，基本上够用。

```java
import org.springframework.beans.propertyeditors.PropertiesEditor; 
public class DoubleEditor extends PropertiesEditor {      
    @Override      
    public void setAsText(String text) throws IllegalArgumentException {                if (text == null || text.equals("")) 
       {      text = "0";          
    }         
                                                                        setValue(Double.parseDouble(text));      
                                                                       }        @Override      
    public String getAsText() {
        return getValue().toString();      
    }  
} 
```

