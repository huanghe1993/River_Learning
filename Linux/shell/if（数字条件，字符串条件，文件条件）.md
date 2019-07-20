# if

**1.整数比较** 

```shell
-eq 等于,如:if [ "$a" -eq "$b" ] 
-ne 不等于,如:if [ "$a" -ne "$b" ] 
-gt 大于,如:if [ "$a" -gt "$b" ] 
-ge 大于等于,如:if [ "$a" -ge "$b" ] 
-lt 小于,如:if [ "$a" -lt "$b" ] 
-le 小于等于,如:if [ "$a" -le "$b" ] 
<   小于(需要双括号),如:(("$a" < "$b")) 
<=  小于等于(需要双括号),如:(("$a" <= "$b")) 
>   大于(需要双括号),如:(("$a" > "$b")) 
>=  大于等于(需要双括号),如:(("$a" >= "$b"))
```

**2.字符串比较**

```shell
= 等于,如:if [ "$a" = "$b" ] 
== 等于,如:if [ "$a" == "$b" ],与=等价 
```

注意：

> 比较两个字符串是否相等的办法是：
> if [ "$test"x = "test"x ]; then
> 这里的关键有几点：
> 1 使用单个等号
> 2 注意到等号两边各有一个空格：这是unix shell的要求
>
> 3 注意到`"$test"`x最后的x，这是特意安排的，因为当$test为空的时候，上面的表达式就变成了x = testx，显然是不相等的。而如果没有这个x，表达式就会报错：[: =: unary operator expected

注意:==的功能在[[]]和[]中的行为是不同的,如下: 

```shell
[[ $a == z* ]]   # 如果$a以"z"开头(模式匹配)那么将为true 
[[ $a == "z*" ]] # 如果$a等于z*(字符匹配),那么结果为true 
 
[ $a == z* ]     # File globbing 和word splitting将会发生 
[ "$a" == "z*" ] # 如果$a等于z*(字符匹配),那么结果为true

```

**3.shell bash判断文件或文件夹是否存在**

```shell
#shell判断文件夹是否存在

#如果文件夹不存在，创建文件夹
if [ ! -d "/myfolder" ]; then
  mkdir /myfolder
fi

#shell判断文件,目录是否存在或者具有权限


folder="/var/www/"
file="/var/www/log"

# -x 参数判断 $folder 是否存在并且是否具有可执行权限
if [ ! -x "$folder"]; then
  mkdir "$folder"
fi

# -d 参数判断 $folder 是否存在
if [ ! -d "$folder"]; then
  mkdir "$folder"
fi

# -f 参数判断 $file 是否存在
if [ ! -f "$file" ]; then
  touch "$file"
fi

# -n 判断一个变量是否有值
if [ ! -n "$var" ]; then
  echo "$var is empty"
  exit 0
fi

# 判断两个变量是否相等
if [ "$var1" = "$var2" ]; then
  echo '$var1 eq $var2'
else
  echo '$var1 not eq $var2'
fi
```



