---
title: 词法分析的java实现
date: 2020-04-01 19:55:24
tags: [词法分析,编译原理,java]
categories: [技术,]
---

### 简介

使用JAVA实现简单的词法分析,支持识别C语言中关键字,标识符,运算符等.

### 思路

#### DFA

> 由于作图工具限制无法画同心圆,故图中红色圆为终态,每个终态(上方或右方)配有一段关于终态的文字描述

![GG8r8K.png](https://s1.ax1x.com/2020/04/02/GG8r8K.png)

#### 设计数据结构

1. 枚举类的设计
在DFA中,为每个终态顶点设置了一段文字说明.故设计一个枚举类,里面包含所有的终态顶点类型
```java
/**
 * VertexType
 * 顶点类型的枚举类
 */
public enum VertexType {
    ID, // 标识符
    KEYWORD, // 关键字
    NUMBER, // 数字
    SEPARATOR, // 分隔符
    BRACKET, // 括号
    COMMENT, // 评论
    ARI_OPERATOR, // 算术运算符
    lOG_OPERATOR, // 逻辑运算符
    EQUAL_OPERATOR, // 等于号
    COMP_OPERATOR, // 比较运算符
    WHITESPACE, // 空白符
    NEWLINE, // 换行符
    SIMICOLON, // 分号
    ERROR,// 错误符号

}
```
同时,针对图中的每条边的权,也设计了一个枚举类.
这么做的优点是能有效减少顶点之间的路径,缺点则是失去了部分灵活性.
```java
/**
 * CharType
 * 字符类型的枚举
 */
public enum CharType {
    LETTER,         // 字母 a-z A-Z _
    DIGIT,          // 数字 0-9
    DOT,            // 点 .
    SLASH,          // 斜杠 /
    ARI_OPERATOR,   // 算数运算符 +-*%
    lOG_OPERATOR,   // 逻辑运算符 &|!
    EQUAL_OPERATOR, // 等于号 =
    COMP_OPERATOR,  // 比较运算符 ><
    BRACKET,        // 括号 ()[]{}
    SIMECOLON,      // 分号与逗号
    WHITESPACE,     // 空格 正则中\s对应的除\n以外的空白符
    NEWLINE         // 换行 \n
}
```


2. 设计顶点类
顶点类应当至少包含三个信息:状态,是否为终态,类型(或者说对顶点的描述)
``` java
/**
 * Vertex 顶点类
 */
public class Vertex {
    // 顶点状态,与顶点在数组的索引一致
    private int state;
    // 是否为终态
    private boolean isFinal;
    // 顶点类型
    private VertexType type;
}
```

3. 图的设计
从上图可看出,该DFA是一个拥有16个节点的单向带权图.由于图中路径较少,为降低空间复杂度,不采用邻接矩阵结构.同时发现:无须求两个顶点之间的距离,故也不采用邻接链表.
由于只需要通过当前顶点以及当前输入字符的类型来确定状态转移后的顶点
故使用一个HashSet<CharType, Integer>数组来存放所有的边,key为输入字符,value为跳转的顶点,这样可使状态转移的时间复杂度降低为O(1)
```java
    /**
     * 状态转移
     * 
     * @param i        当前顶点索引
     * @param charType 字符类型,即边的权
     * @return 若存在状态转移,则返回转移后的顶点.否则返回null
     */
    public Vertex transfer(int i, CharType charType) {
        Object srcIndex = edges[i].get(charType);
        return srcIndex == null ? null : vertexs[(int) srcIndex];
    }
```

#### 分析算法流程
整个算法的流程图如下
![G8tC6O.png](https://s1.ax1x.com/2020/04/01/G8tC6O.png)
除此之外,算法还有许多细节之处.
例如,关键字(KEYWORD)与表示符(ID)通过词法分析无法区分.对此,当词法分析结束后,会做进一步的判断,判断关键字的方法很简单,建立一个关键字的HashSet,查找是否存在即可,查找的时间复杂度依然是O(1).

### 项目结构介绍
![G8OWzd.png](https://s1.ax1x.com/2020/04/02/G8OWzd.png)

### 使用说明

通过项目结构介绍了解到:默认的输入输出文件位于根目录下.
如果想修改文件路径可以打开Analyze.java启动类,修改如下代码

```java
FileUtil f = new FileUtil();
```

例如,使用带参数的构造函数更改输入输出文件

```java
FileUtil f = new FileUtil("Your input path","Your output path");
```

#### 样例展示

input.txt

``` C
void test(int a,int b,float c,float d){
    //hello analyzer wengyuxian
    int x;
    float y;
    x=a+b;
    y=c/d;
    if(x>y)
        x=10;
    else
        y=12.5;
}
```

output.txt

``` plaintext
KEYWORD       : void      
ID            : test      
BRACKET       : (         
KEYWORD       : int       
ID            : a         
SEPARATOR     : ,         
KEYWORD       : int       
ID            : b         
SEPARATOR     : ,         
KEYWORD       : float     
ID            : c         
SEPARATOR     : ,         
KEYWORD       : float     
ID            : d         
BRACKET       : )         
BRACKET       : {         
COMMENT       : //hello analyzer wengyuxian
KEYWORD       : int       
ID            : x         
SEPARATOR     : ;         
KEYWORD       : float     
ID            : y         
SEPARATOR     : ;         
ID            : x         
EQUAL_OPERATOR: =         
ID            : a         
ARI_OPERATOR  : +         
ID            : b         
SEPARATOR     : ;         
ID            : y         
EQUAL_OPERATOR: =         
ID            : c         
ARI_OPERATOR  : /         
ID            : d         
SEPARATOR     : ;         
KEYWORD       : if        
BRACKET       : (         
ID            : x         
COMP_OPERATOR : >         
ID            : y         
BRACKET       : )         
ID            : x         
EQUAL_OPERATOR: =         
NUMBER        : 10        
SEPARATOR     : ;         
KEYWORD       : else      
ID            : y         
EQUAL_OPERATOR: =         
NUMBER        : 12.5      
SEPARATOR     : ;         
BRACKET       : }         
```
