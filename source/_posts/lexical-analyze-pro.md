---
title: 从正规式到词法分析
date: 2020-04-12 13:36:46
tags: [词法分析,编译原理,java]
categories: [技术,]
---

## 简介

使用JAVA实现简单的词法分析,支持识别C语言中关键字,标识符,运算符等.

## 思路

### 正规式->NFA

&emsp; &emsp; 对于一个正规式,我们可以将其看作一个表达式.表达式中的操作数为字母表中的字符,表达式中的操作符包括.(连接),|(或运算),*(星闭包),+(正闭包)以及用于表达优先级的().
每个操作数可以看做单字符的汤普森表示,例如a|b可以表达为:

操作数:

![GLKB1f.png](https://s1.ax1x.com/2020/04/12/GLKB1f.png)

操作符:

![GLMk4I.png](https://s1.ax1x.com/2020/04/12/GLMk4I.png)

&emsp; &emsp; 所以,正规式到NFA只是一个简单的表达式求值过程.有点特殊的是,.(连接符)是缺省的,故我们需手动补齐.例如abc我们将其补全为a.b.c

#### 核心算法

整个算法的主要流程图如下,图中省略了部分细节

![GLlh9I.png](https://s1.ax1x.com/2020/04/12/GLlh9I.png)

字符读取完成后,还需要计算操作数栈,操作符栈中表达式,其过程类似上图中操作符分支,故此处不再赘叙.

#### 数据结构

NFA的数据结构如下:

``` java
public class NFA {
    public ArrayList<Vertex> states;
    public ArrayList<Trans> transitions;
    // 对于一个正规式而言,只会有一个终态
    public int finalState;
}
```

我们默认状态0为初态,故数据结构中无需记录初态.下面给出Vertex与Trans的数据结构:

```java
public class Vertex {

    // 顶点状态,应该与顶点索引保持一致
    public int state;
    // 是否为终态
    public boolean isFinal;
    // 类型
    public String type;
}
```

```java
public class Trans {
    public int stateFrom, stateTo;
    public char transSymbol;
```

#### 补充说明

如果阅读Thompson.java的源码,会发现其中实现交,并,闭包等的操作函数形式如下:

```java
    /**
     * *闭包
     * @param n 操作数
     * @return 返回*闭包的结果
     */
    public static NFA kleene(NFA n) {
        // todo...
    }
```

&emsp; &emsp; 熟悉java的人可能会认为由于类的引用机制,完全可以对n进行修改而没必要返回一个新的NFA.的确,这样做似乎可以节省一些空间.但个人认为这么做带来的坏处更大.
&emsp; &emsp; 首先,NFA中的states是ArrayList类型,*闭包需要添加一个新的初态,这意味着需要执行一次插入操作.对一个随机访问型的列表执行插入操作是十分耗时的(最后生成的NFA可能有几百个结点,这意味一次插入操作需要移动几百个结点).那如果改用LinkedList呢?答案是否定的,因为我们需要随机访问.那这么做会浪费很多空间吗?不见得,java的gc机制会自动帮我们回收空间.
&emsp; &emsp; 最后再说一点,过多的依赖引用可能会造成高耦合低内聚.这意味着可能你在某个地方修改了某个变量的值,导致整个程序崩溃,这也是我宁愿多花费一点空间的原因.
&emsp; &emsp; 接下来思考另一个问题.熟悉java多态的人可能会认为在声明属性时可以使用List声明,等到需要实例化时再给予具体的类型.确实,利用多态可以为我们省下许多功夫,但是他也有着他的限制性.
&emsp; &emsp; 如果所有属性都采用接口声明,很有可能你需要在某些时刻进行一些强制类型转换(例如你需要调用LinkedList中的方法,但你使用List接口声明),他们的代码看起来可能是这样的:

```java
    if( someValues instanceof someClass){
        // do class cast
    }else{
        // do other thing
    }
```

考虑到强制转换类型可能失败,我们还需要添加一段try-catch:

```java

    try{
        if( someValues instanceof someClass){
        // do class cast
    }else{
        // do other thing
    }
    }catch (Exception e){
        // do some thing
    }
```

&emsp; &emsp;现在问题暴露出来了,使用接口的确能使我们的代码更抽象,但也给我们带来了很多麻烦.而且过多的依赖接口也说明你对算法的流程掌握不到位,你无法正确的选择数据结构.

**以上的两个问题在整个词法分析过程中皆有出现.这两个问题困扰了我很久,可能我做出的选择对程序不是最优解,但对写代码来的人来讲也不是最差解.**

### NFA转DFA

NFA转DFA的算法课本上已经将的十分详尽了,故此处不再赘述.这里只简单介绍数据结构以及部分注意事项.

#### 数据结构

DFA:

```java
public class DFA {
    public ArrayList<DVertex> Dstates = new ArrayList<>();
    public ArrayList<Trans> Dtrans = new ArrayList<>();
    // 每个顶点的状态转移集的集合,即transitions.get(i)为顶点i的状态转移
    public ArrayList<ArrayList<Trans>> transitions;
    // 类似transitions,transMaps为每个顶点的状态转移映射的集合
    public ArrayList<HashMap<Character, Integer>> transMaps;
}
```

DVertex:

```java
/**
 * DVertex DFA中的顶点
 */
public class DVertex {
    public int id;
    public HashSet<Integer> states;
    public boolean isFinal = false;
    public String type = null;
}
```

#### 补充说明

&emsp; &emsp;可以看到,创建了一个二维数组transitions以及一个HashMap的集合transMaps.二者都是状态转移的辅助描述.
&emsp; &emsp;transitions的作用是:当你需要获得顶点n的所有状态转移时,你可以调用transitions.get(n)而非去迭代遍历Dtrans并添加if判断.学习过计算机组成原理的人知道,CPU是以超标量流水线方式工作的,也就是说CPU希望执行的指令是连续的,if判断会产生一条jump指令,这会使程序性能下降.
&emsp; &emsp;transMaps的作用是:当你需要知道顶点n经字符c转移后得到的状态时,你可以调用transMaps.get(n).get(c)而非去transitions中查找.哈希查找的时间复杂度为O(1).

### DFA最小化

DFA最小化需要不断回溯,这里采用一种比较特殊的手段,可以不借助递归也不借助栈即可实现回溯.

#### 核心算法

&emsp; &emsp;初次划分时,我们将终态置于组0,其他置于组1.这与之间的设定完全相反(一般地,我们认为编号0是初态而终态为编号最大值),这是因为我们只需要对非终态进行划分,这样做使得只需从组1开始扫描.当然,你也可以保持终态为1,然后每次反向扫描.
&emsp; &emsp;接下来我们开始检查组1,我们默认每个组的第一个元素为改组代表,所有与代表不同组的元素我们添加到一个新的组中.而且一旦发现新的组别,我们需重置迭代索引来实现回溯.
&emsp; &emsp;最终,我们直接以组号代表该小组.

主要流程图如下:
[![GL4TMR.png](https://s1.ax1x.com/2020/04/12/GL4TMR.png)](https://imgchr.com/i/GL4TMR)

#### 实例分析

这里再给出一个实例,假设有如下DFA结点0,1,2,3,4.其中4为终态.最终划分结果为每个结点各一组.

1. 初次划分g0={4},g1={0,1,2,3}.
2. 遍历g1,发现1,2,3皆与0不同组,将其加入g2.目前g1={0},g2={1,2,3}.
3. 重新遍历,g1通过.遍历g2,发现2与1不同组,将其加入g3,得到g2={1,2},g3={3}.
4. 重新遍历,g1通过.由于2被划分到g3,3与1此时不同组,将其加入g4.得到g2={1},g3={3},g4={2}.
5. 重新遍历,均通过,以组号作为该组代表编号,结束划分.

#### 数据结构
由于在最小化DFA过程中打乱了顺序,故我们需要记录初态

```java
public class MinDFA {

    public int inital;// 初态
    public ArrayList<DVertex> Dstates = new ArrayList<>();
    public ArrayList<HashMap<Character, Integer>> transMaps = new ArrayList<>();
}
```

### MinDFA合并

&emsp; &emsp;至此,我们已经为每个正规式得到了其最小化的DFA,即MinDFA.接下来我们需要将这些DFA合并成一个大的DFA.
DFA的合并与汤普森算法中的|操作非常相似,只不过DFA的合并是将所有初态合并为一个新初态,并且不添加新的终态.

#### 核心算法

算法主要流程图如下:

[![GLTIk6.png](https://s1.ax1x.com/2020/04/12/GLTIk6.png)](https://imgchr.com/i/GLTIk6)

#### 补充说明

&emsp; &emsp;在计算DFA结点新编号时,需要注意结点编号小于初态与结点编号大于初态的偏移量是不同的.这是因为所有初态被合并至一个新初态.例如:当我们合并第一个DFA时,偏移量=1(因为已经有一个公共初态结点0).设该DFA中Dstates为{0,1,2,3},其中1为初态.

&emsp; &emsp;合并DFA时各个结点的新结点分别为:
1. 0=0+1=1
2. 1=0(被合并至公共初态)
3. 2=2+1-1=2(初态结点被合并)
4. 3=3+1-1=3(初态结点被合并)

&emsp; &emsp;还需注意的是,多个MinDFA合并后得到的并非是MinDFA,严格来讲,可能合并结果连DFA都不是.举个例子,有如下两个DFA:

[![GLbyff.png](https://s1.ax1x.com/2020/04/12/GLbyff.png)](https://imgchr.com/i/GLbyff)

合并后会得到:
![GLbot0.png](https://s1.ax1x.com/2020/04/12/GLbot0.png)

可以看出,结点0接受字符/时有两条状态转移,这不符合DFA的定义.在编码中,由于使用HashSet,第二个DFA会覆盖第一个DFA,也就是说最终得到的DFA不接受字符/.好在目前的正规式不会有这样的冲突.

### 后续过程

后续过程可参考我的另一篇博客:[后续过程](https://wengyuxian.work/lexical-analyze/)

## 项目说明

### 项目结构

根目录
&emsp; &emsp;work/wengyuxian
&emsp; &emsp;&emsp; &emsp;dfa &emsp; &emsp;dfa相应的数据结构包
&emsp; &emsp;&emsp; &emsp;thompson &emsp; &emsp;包括NFA的数据结构以及汤普森算法的实现
&emsp; &emsp;&emsp; &emsp;util &emsp; &emsp;工具类,包括文件的读取,写入.字母表的定义等
&emsp; &emsp;reg.txt &emsp; &emsp;默认正规式文本
&emsp; &emsp;input.txt &emsp; &emsp;默认源程序文本
&emsp; &emsp;output.txt &emsp; &emsp;默认输出结果文本
