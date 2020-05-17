---
title: LL(1)文法的语法分析器
date: 2020-05-17 14:16:47
tags: [语法分析,编译原理,Java]
categories: [技术,]
---

## 实验目的

&emsp; &emsp;...

## 实验内容

&emsp; &emsp;...

## 实验程序清单

根目录/src

&emsp; &emsp;algorithm&emsp; &emsp;FIRST集,FOLLOW集,预测分析表,驱动器算法的实现.

&emsp; &emsp;cfg&emsp; &emsp;CFG包,包括CFG的读取,消除左递归,消除公共左因子等算法.

&emsp; &emsp;data&emsp; &emsp;数据包

&emsp; &emsp;&emsp; &emsp;cfg&emsp; &emsp;存放CFG文本的包.

&emsp; &emsp;&emsp; &emsp;test&emsp; &emsp;存放测试用例的包

&emsp; &emsp;&emsp; &emsp;log&emsp; &emsp;存放输出日志的包

&emsp; &emsp;util&emsp; &emsp;工具包,实现读/写文件的操作.

&emsp; &emsp;Parsing.java&emsp; &emsp;主调函数

## 程序的主要部分及其功能说明

### 功能说明

#### 文法规范

1. 每个产生式独占一行.

2. 推导符号为->,中间不允许有空格.

3. 所有符号以空格分隔.

4. 以大写字母开头的符号为非终结符,其余均为终结符(ε除外).

5. #为输入结束符,不要使用#定义文法.

6. 行首行尾无空格且文本中无空行.

&emsp; &emsp;示例:

```textplain
L -> E ; L
    | ε
E -> E + T
    | E - T
    | T
T -> T * F
    | T / F
    | T mod F
    | F
F -> ( E )
    | id
    | num
```

#### 测试用例规范

1. 每个测试用例独占一行.

2. #代表终结符,测试用例中不能出现#.

3. 测试用例中不能出现ε.

4. 测试用例中所有符号以空格分隔.

5. 行首行尾无空格且文本中无空行.

&emsp; &emsp;示例:

```textplain
id + id * id ;
id + id * num - num / id ;
id + num ; num - id ; id * ( num ) ; num / ( id ) ;
```

#### 文件路径

&emsp; &emsp;src/Parsing.java数据结构中指定了CFG,测试,输出文本的路径:

```java
public class Parsing {
    // 默认CFG路径
    static String baseCfg = "src/data/cfg/";
    // 默认测试文件路径
    static String baseTest = "src/data/test/";
    // 默认输出路径
    static String baseOut = "src/log/";
}
```

&emsp; &emsp;主调函数parse结构如下:

```java
    /**
     * 语法分析主调函数
     * 
     * @param cfgPaths    CFG文件
     * @param testPaths   测试文件
     * @param outputPaths 输出文件
     * @param num         分析数目
     */
    public static void parse(String[] cfgPaths, String[] testPaths, String outputPaths[], int num) {
    }
```

&emsp; &emsp;cfgPaths\[i]指定第i个文法,testPaths\[i]指定第i个文法的测试用例,outputPaths\[i]指定输出文件.num指定i的最大值.

&emsp; &emsp;在main函数中有如下示例:

```java
    public static void main(String[] args) {

        String[] cfgPaths = { "cfg1.txt", "cfg2.txt" };
        String[] testPaths = { "test1.txt", "test2.txt" };
        String outputPaths[] = { "log1.txt", "log2.txt" };

        parse(cfgPaths, testPaths, outputPaths, 2);
    }
```

#### 后缀

&emsp; &emsp;在src/cfg/CFG.java中定义了后缀:

```java
public class CFG implements Cloneable {
    public static final char appendChar = '`';
}
```

&emsp; &emsp;消除左递归及公共左因子过程中产生的新产生式左部会不断添加该后缀直至无重复名.

#### 系统环境要求

&emsp; &emsp;JDK8及以上(支持lambda).

### 主要功能

#### CFG构造

&emsp; &emsp;我们规定产生式结构描述如下:

![Y2wjbQ.png](https://s1.ax1x.com/2020/05/17/Y2wjbQ.png)

##### 数据结构

```java
public class CFG implements Cloneable {

    // 非终结符
    public Set<String> nonTerminals;
    // 终结符
    public Set<String> terminals;
    // 开始符号
    public String startSymbol;
    // 产生式组的映射
    public Map<String, ProductionGroup> productionGroupMap;
}
```

```java
/**
 * 一个产生式头对应的产生式组
 */
public class ProductionGroup implements Cloneable {

    // 该产生式组的产生式头部
    public String productionHead;
    // 该组是否有epsilon产生式
    public boolean hasEpsilon;
    // 为了输出效果，用LinkedHashSet
    public Set<Production> productions = new LinkedHashSet<>();
}
```

```java
/**
 * 一条产生式, 由子项序列组成 每个产生式都有标记
 */
public class Production implements Cloneable {
    public static int globalIdCount = -1;

    public int id;
    public LinkedList<SubItem> subItems = new LinkedList<>();
}
```

```java
public class SubItem implements Cloneable {

    public String value;
    public SubItemType type;
}
```

```java
/**
 * 子项属性类型
 */
public enum SubItemType {
	nonTerminal,
    terminal,
    epsilon
}
```

&emsp; &emsp;可以看到,数据结构层数比较深.且为了降低时间复杂度,需要查找的集合都使用了Hash(后续算法也是).

&emsp; &emsp;因此程序中大量的使用了lambda表达式,例如:

```java

cfg.productionGroupMap.forEach((key, val) -> {
            // ...
            val.productions.forEach(production -> {
                // ...
            });
});
```

&emsp; &emsp;因此,系统环境要求JDK8及以上.

#### 消除左递归

&emsp; &emsp;算法与书上相同,不再赘述.

##### 示例

```textplain
S ->  a A B e
A -> b
    | A b c
B -> d
```

&emsp; &emsp;消除左递归后的文法:

```textplain
S     -> a A B e 

A     -> b A` 

B     -> d 

A`    -> b c A` 
A`    -> ε 
```

&emsp; &emsp;由于顺序并未严格排序,故后续求FIRST集,FOLLOW集都采用递归求解.

#### 消除公共左因子

&emsp; &emsp;算法与书上相同,不再赘述.

##### 示例

```textplain
S -> A
A -> a b
    | a c d
    | a c e
```

&emsp; &emsp;消除公共左因子后的文法:

```textplain
S     -> A

A     -> a A`

A`    -> b
A`    -> c A``

A``   -> d
A``   -> e
```


#### FIRST集

&emsp; &emsp;由于产生式非严格有序,采用递归求解FIRST集.

##### 数据结构

```java
public class FirstSet {

    public CFG cfg;
    public Map<String, Set<String>> firstSet;
    public boolean isUpdated = false;
}
```

#### FOLLOW集

&emsp; &emsp;由于产生式非严格有序,采用递归求解FOLLOW集.

##### 数据结构

```java
public class FollowSet {

    public static final String InputRightEndSym = "#";

    public CFG cfg;
    public Map<String, Set<String>> firstSet;
    public Map<String, Set<String>> followSet;
    public boolean isUpdated = false;
}
```

##### 示例

```textplain
CFG:
开始符号 : S
S     -> a A B e 

A     -> b A` 

B     -> d 

A`    -> b c A` 
A`    -> ε 

FIRST集:
S    :[a]
A    :[b]
B    :[d]
A`   :[b, ε]

FOLLOW集:
S    :[#]
A    :[d]
B    :[e]
A`   :[d]
```

#### 构造预测分析表

##### 数据结构

```java
public class AnalyzeTable {
    public CFG cfg;
    public Map<String, Set<String>> firstSet;
    public Map<String, Set<String>> followSet;
    // 预测分析表
    public Map<String, Map<String, List<String>>> table;
    public boolean isUpdated;
}
```

##### 示例

```textplain
预测分析表:
          a         e         b         c         d         #         
S         aABe                                                        
A                             bA`                                     
B                                                 d                   
A`                            bcA`                ε                   
```

#### 驱动器算法

&emsp; &emsp;能给出每一步的信息以及错误信息.

##### 示例

```textplain
测试用例:abcde#
step      stack                    input                                   option                        info
1         #S                       abcde#                                  POP(S),PUSH(aABe)             S->aABe
2         #eBAa                    abcde#                                  POP(a),NEXT(ip)               匹配
3         #eBA                     bcde#                                   POP(A),PUSH(bA`)              A->bA`
4         #eBA`b                   bcde#                                   POP(b),NEXT(ip)               匹配
error:分析表[A`,c]结果为error,ip位置:3
error:未抵达输入流结尾,当前ip:c,ip位置:3
```

## 实验收获体会

&emsp; &emsp;...

## 改进意见

&emsp; &emsp;...