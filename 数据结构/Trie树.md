## **前缀树的结构**

Trie树，又叫字典树、前缀树（Prefix Tree）、单词查找树或键树，是一种多叉树结构。如下图：

![1533563736520](../../../../AppData/Local/Temp/1533563736520.png)

上图是一棵Trie树，表示了关键字集合{“a”, “to”, “tea”, “ted”, “ten”, “i”, “in”, “inn”} 。从上图可以归纳出Trie树的基本性质：

 **①根节点不包含字符，除根节点外的每一个子节点都包含一个字符。** 

**②从根节点到某一个节点，路径上经过的字符连接起来，为该节点对应的字符串。**

 **③每个节点的所有子节点包含的字符互不相同。**

  **④从第一字符开始有连续重复的字符只占用一个节点，比如上面的to，和ten，中重复的单词t只占用了一个节点。** 

## **前缀树的应用**

**1、前缀匹配** 
**2、字符串检索** 
**3、词频统计** 
**4、字符串排序**

下面看看怎样使用前缀树来实现前缀匹配的。

## **前缀匹配**

了解了前缀树的结构后，就可以利用前缀树的性质来解决现实中的问题。比如说查找一个字符串数组中是否含有前缀单词，什么是前缀单词：上面的 in，就是 inn 的前缀单词。如果有十几万条单词，并且每个单词的长度都是5-10以内，这样必定存在大量重复的字符，因此利用前缀树来求解不仅速度快而且空间复杂度也比较好。 
**①定义前缀树结构**

```java
class Tries{
    Boolean isTrie ;
 HashMap<Character, Tries> children=new HashMap<Character, Tries>(); 
}
```

上面的 isTrie 用来标记单词是否遍历完。children表示该节点的子节点。如上面的t节点的子节点有o和e两个。 

**②建立前缀树**

```java
    public static boolean insertNode(String str,Tries head)
    {
        if(str==null||str.length()==0)
            return false;
            //如果插入的单词为null 或者单词长度为0直接返回false，false代表该单词不是前缀树中某个单词的前缀，
         //或者前缀树中某个单词是该单词的前缀。
        char chs[]=str.toCharArray();
        int i=0;
        Tries cur=head;
        //将字符串的每个字符插入到前缀树中
        while(i<chs.length)
        {           
            if(!cur.children.containsKey(chs[i]))
            {

                cur.children.put(chs[i], new Tries());
                //如果当前节点中的子树节点中不包含当前字符，新建一个子节点。
            }
            //否则复用该节点
            cur=cur.children.get(chs[i]);
            if(cur.count==true)
            {
                System.out.println(" trie tree");
                return true;
                //判断前缀树中是否有字符串为当前字符串的前缀。
            }
            i++;
        }
        cur.count=true;
        if(cur.children.size()>0)
        {
            System.out.println(" trie tree");
            return true;
            //判断当前字符串是否是前缀树中某个字符的前缀。
        }
        return false;
    }
```

前缀树的建立过程就是插入字符串的过程，同时在插入节点的时候可以判断插入的字符串是否是前缀树里面某个单词的前缀，或者前缀树中的某个单词是否是该单词的前缀。

**①先将字符串转换为字符数组，然后对每个字符进行处理，如果当前节点的子节点中包含有要处理的字符字节复用。否则新建一个子节点。** 
**②判断是否是前缀单词的时候，有两个步骤，首先要看该字符串是否是其他字符串的前缀，还有看其他字符串是否是当前字符串的前缀。**

**判断前缀单词的完整代码：**

```
public class isTrie {

    public static void main(String[] args) {
        Tries tries=new Tries();
        String strs[]={"abc","abd","b","abdc"};

        for(int i=0;i<strs.length;i++)  
            insertNode(strs[i], tries);                 
    }
public static boolean insertNode(String str,Tries head)
    {
        if(str==null||str.length()==0)
            return false;
        char chs[]=str.toCharArray();
        int i=0;
        Tries cur=head;
        while(i<chs.length)
        {           
            if(!cur.children.containsKey(chs[i]))
            {

                cur.children.put(chs[i], new Tries());
            }
            cur=cur.children.get(chs[i]);
            if(cur.count==true)
            {
                System.out.println(" trie tree");
                return true;
            }
            i++;
        }
        cur.count=true;
        if(cur.children.size()>0)
        {
            System.out.println(" trie tree");
            return true;
        }
        return false;
    }
}
class Tries{
    boolean isTrie;
    HashMap<Character, Tries> children=new HashMap<Character, Tries>(); 
}
```