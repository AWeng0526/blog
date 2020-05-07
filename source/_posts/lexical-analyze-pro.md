---
title: 从正规式到词法分析
date: 2020-04-12 13:36:46
tags: [词法分析,编译原理,Java]
categories: [技术,]
---

## 实验目的

1.	掌握生成词法分析器的方法，加深对词法分析原理的理解。
2.	掌握设计、编制并调试词法分析程序的思想和方法。
3.	本实验是高级语言程序设计、数据结构和编译原理中词法分析原理等知识的综合。

## 实验内容

1. 	选择一种熟悉的高级语言（如C语言，C++，VB或VC等），设计、编写一个词法分析子程序。
2.	待分析的源程序为一个简单的C语言程序，如下所示：

```c
    void test(int a, int b, float c, float d){ 
        int x;
        float y;
        x = a + b;
        y = c / d;
        if(x>y)
            x = 10;
        else
            y = 12.5;
}
```

&emsp; &emsp;将该源程序的源文件经词法分析后输出以二元组形式表示的单词符号序列。


3.	编写的程序具有一定的查错能力。提交的实验报告中要有实验名称、实验目的、实验内容、实验程序清单、调试过程和运行结果，程序的主要部分作出功能说明，并有实验收获体会或改进意见等内容。


## 实验程序清单

根目录

&emsp; &emsp;log &emsp; &emsp;日志包,存有DFA,NFA,MinDFA等结构的信息

&emsp; &emsp;reg.txt &emsp; &emsp;默认正规式文本

&emsp; &emsp;input.txt &emsp; &emsp;默认源程序文本

&emsp; &emsp;output.txt &emsp; &emsp;默认输出结果文本

&emsp; &emsp;work/wengyuxian

&emsp; &emsp;&emsp; &emsp;dfa &emsp; &emsp;dfa相应的数据结构包

&emsp; &emsp;&emsp; &emsp;thompson &emsp; &emsp;包括NFA的数据结构以及汤普森算法的实现

&emsp; &emsp;&emsp; &emsp;util &emsp; &emsp;工具类,包括文件的读取,写入.字母表的定义等

## 调试过程和运行结果

&emsp; &emsp;在input.txt和reg.txt中输入源程序和正规式后,调用work/wengyuxian/util/AnalyzeUtil.java中main函数即可.

&emsp; &emsp;output.txt会记录记号流,log文件夹中会保存DFA,NFA,MinDFA(最小化DFA),FinalMinDFA(合并后的DFA)的结构信息.

## 程序的主要部分及其功能说明

### 功能说明

#### 正规式规范

&emsp; &emsp;类似LEX,本程序对正规式有以下的规范要求:

- 正规式文本中不要包含空行.
- 每一行代表一个正规式.
- 行首不要留空格,正规式定义中也不要留有空格.
- 正规式分两部分:名称+内容,两者以空格分隔.即"名称 内容".
- 不支持递归定义正规式,但内置了digit(0-9)和letter(a-zA-Z_)两个正规式.可以用{digit}/{letter}调用.
- 更多规范见下文.

&emsp; &emsp;示例:

```textplain
NUMBER          {digit}+(.{digit}+|#)(E(\+|-|#){digit}+|#)
ID              {letter}({letter}|{digit})*
BRACKET         \(|\)|[|]|{|}
SEPARATOR       ,|;
OPERATOR        \+|-|\*|/|&|\||!|=|>=|<=|>|<|==|!=
WHITESPACE      (\s|\t|\r)+
NEWLINE         (\n)+
```

#### 字母表

&emsp; &emsp;字母表的定义位于work/wengyuxian/util/Alphabets.java中,可根据需求添加/删减元素

```java
    // 字母表
    public static HashSet<Character> alphabets = new HashSet<Character>(Arrays.asList('0', '1',     '2', '3', '4', '5', '6',
            '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'd', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q',
            'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L',
            'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z', '_', '.', '+', '-', '*', '/', '&',
            '!', '\\', '(', ')', '[', ']', '{', '}', '>', '<', '=', ';', ',', ' ', '\n', '#'));
```

#### 支持操作

&emsp; &emsp;支持连接,并,星闭包,正闭包,括号操作.

&emsp; &emsp;除连接符(.)以外,若想匹配操作符,请使用\转义.由于连接符是默认行为,无需在正规式中写出,字符'.'并不会被识别为操作符.即a.b正规式接受字符串a.b而不接受字符串ab.

#### 转义

&emsp; &emsp;\为转义操作符,除上述操作符外,还支持以下字符转义:

1. "\s"-->'\s'

2. "\r"-->'\r'

3. "\t"-->'\t'

4. "\n"-->'\n'

5. "\\\\"-->'\\'

&emsp; &emsp;字母表中的其他元素使用\转义将得到与原字符一样的效果.

#### 特殊字符

&emsp; &emsp;使用#代表空串,除非明确的知道需要使用空串(例如实现?操作符),请不要使用#定义正规式.

#### 关键字

&emsp;&emsp;在work/wengyuxian/Alphabets.java中有这么一段代码:

```java
    // 关键字
    public static HashSet<String> keyWords = new HashSet<>(Arrays.asList("void", "char", "int", "float", "double",
            "boolean", "short", "long", "signed", "unsigned", "struct", "union", "enum", "typedef", "sizeof", "auto",
            "static", "register", "extern", "const", "volatile", "return", "continue", "break", "goto", "if", "else",
            "switch", "case", "default", "for", "do", "while"));
```

&emsp;&emsp;由于关键字与ID从词法上无法区分,所以定义一个关键字列表,在驱动器算法中利用这个集合来筛选关键字。

#### 正规式有关限制

- 请保证你的正规式无误,正规式错误是不被容许的.程序无法识别正规式时会立即结束.

- 请确保你的正规式最终生成的最小化DFA中,终态不包含初态.这点会在后面MinDFA给出说明.

#### 更改文件路径

&emsp; &emsp;在work/wengyuxian/util/AnalyzeUtil.java中,找到如下代码:

```java
        // 默认路径为 ./reg.txt ./input.txt ./output.txt
        FileUtil fileUtil = new FileUtil();

        // other code
        // 写入
        fileUtil.writeFile("./log/NFA.txt", NFAStr.toString());
        fileUtil.writeFile("./log/DFA.txt", DFAStr.toString());
        fileUtil.writeFile("./log/MinDFA.txt", MinDFAStr.toString());
        fileUtil.writeFile("./log/FinalMinDFA.txt", finalDfa.toString());

```

&emsp; &emsp;若想更改NFA等数据结构信息存储文件直接修改对应代码即可.

&emsp; &emsp;若想改变正规式/源程序/输出结果文件路径,程序提供了FileUtil的重载函数如下:

```java
    public FileUtil(String input, String output, String reg) {
        inputFilePath = input;
        outputFilePath = output;
        regularExpressionPath = reg;
    }
```

#### 屏蔽项

&emsp; &emsp;在work/wengyuxian/util/Alphabets.java中有这么一段代码

```java
public static HashSet<String> shield = new HashSet<>(Arrays.asList("WHITESPACE", "NEWLINE"));
```

&emsp; &emsp;其作用为:当识别出WHITESPACE(空格)或NEWLINE(换行)的TOKEN时,不记录至output.txt.可根据需求添加/删除.

&emsp;&emsp;假设有两个字符串"NFA.states"和"NFA . states",如果忽略空格我们会得到两组一样的记号流:ID DOT ID.但显然后者语法上有错误,这时候我们希望交付给语法分析器这样一个记号流:ID WHITESPACE DOT WHITESPACE ID,以便识别出错误的串.但我们仅仅考虑词法分析的话希望屏蔽这些信息,这就是这个字段的由来.

#### 错误信息

&emsp;&emsp;当源程序中有错误时,控制台会打印出错误信息,包括出错行号,出错字符位置,出错单词.

[![YmUcxe.png](https://s1.ax1x.com/2020/05/07/YmUcxe.png)](https://imgchr.com/i/YmUcxe)

### 主要部分

### 汤普森算法

&emsp; &emsp; 对于一个正规式,我们可以将其看作一个表达式.表达式中的操作数为字母表中的字符,表达式中的操作符包括.(连接),|(或运算),*(星闭包),+(正闭包)以及用于表达优先级的().

&emsp; &emsp;每个操作数可以看做由汤普森算法得到的NFA,例如a|b可以表达为:

&emsp; &emsp;操作数:

![GLKB1f.png](https://s1.ax1x.com/2020/04/12/GLKB1f.png)

&emsp; &emsp;操作符:

![GLMk4I.png](https://s1.ax1x.com/2020/04/12/GLMk4I.png)

&emsp; &emsp; 所以,正规式到NFA只是一个简单的表达式求值过程.有点特殊的是,.(连接符)是缺省的,故我们需手动补齐.例如abc我们将其补全为a.b.c

#### 核心算法

&emsp; &emsp;整个算法的主要流程图如下,图中省略了部分细节

![GLlh9I.png](https://s1.ax1x.com/2020/04/12/GLlh9I.png)

&emsp; &emsp;字符读取完成后,还需要计算操作数栈,操作符栈中表达式,其过程类似上图中操作符分支,故此处不再赘叙.

#### 数据结构

&emsp; &emsp;NFA的数据结构如下:

``` java
public class NFA {
    public ArrayList<Vertex> states;
    public ArrayList<Trans> transitions;
    // 对于一个正规式而言,只会有一个终态
    public int finalState;
}
```

&emsp; &emsp;我们默认状态0为初态,故数据结构中无需记录初态.下面给出Vertex与Trans的数据结构:

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

&emsp;&emsp;可以将中缀表达式转换为后缀表达式,例如a+b\*转换为ab\*+,这样求值的过程会方便许多.但是两种算法本质是一样的,毕竟中缀表达式转后缀表达式也需要用到双栈.后缀表达式最大的好处在于模块化.

&emsp; &emsp;如果阅读Thompson.java的源码,会发现其中实现交,并,闭包等的操作函数形式如下:

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

&emsp; &emsp;考虑到强制转换类型可能失败,我们还需要添加一段try-catch:

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

### NFA转DFA

&emsp; &emsp;NFA转DFA的算法课本上已经讲的十分详尽了,故此处不再赘述.这里只简单介绍数据结构以及部分注意事项.

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

&emsp; &emsp;transitions的作用是:当你需要获得顶点n的所有状态转移时,你可以调用transitions.get(n)而非去迭代遍历Dtrans并添加if判断.在/log/DFA.txt文件中可以看到,ID生成的DFA有400多个节点,循环遍历十分耗时.

&emsp; &emsp;transMaps的作用是:当你需要知道顶点n经字符c转移后得到的状态时,你可以调用transMaps.get(n).get(c)而非去transitions中查找.哈希查找的时间复杂度为O(1).

### DFA最小化

&emsp; &emsp;DFA最小化是一个回溯+剪枝的问题.

&emsp; &emsp;假设有ABCDE五个DFA结点需划分,我们现在按如下步骤对其进行划分:

1. A分别与BCDE比较,发现无同组元素,A独立一组.
2. B分别与CDE比较,发现无同组元素,B独立一组.
3. C分别与DE比较,发现无同组元素,C独立一组.
4. D与E比较,二者不同组,D独立一组.

&emsp; &emsp;可以发现,在ABCDE各分一组这种糟糕(但并非最糟糕)的情况下,DFA最小化已经退化成一个求组合的问题.

&emsp; &emsp;再来看一种比较好的情况:

1. A分别与BCDE比较,发现皆同组,ABCDE合并至一组.
2. 无需再比较,划分结束.

&emsp; &emsp;可以看到,剪枝可以有效降低时间复杂度.但剪枝的条件比较苛刻,我们再来看一种情况:

1. A分别与BCDE比较,发现无同组元素,A独立一组.
2. B分别与CDE比较,发现与D同组,B,D合并至一组.

&emsp; &emsp;自然而然的我们会想到:与D同组的元素也与B同组,可以对D进行剪枝.

&emsp; &emsp;但此时不能对D剪枝,考虑这样两条状态转移:smove(B,a)=B,smove(C,a)=D.在执行步骤2时,由于BD还未合并至一组,这两条状态转移可以作为BC不同组的依据.但步骤2之后这两条状态转移不能作为BC不同组的依据.

&emsp; &emsp;接下来考虑最差的情况:

1. ABC经过比较后各分一组.
2. DE合并至一组.
3. 重新比较,CDE合并至一组.
4. 重新比较,BCDE合并至一组.
5. 重新比较,ABCDE合并至一组.

&emsp; &emsp;可以看到,在最差的情况下,回溯的次数增加了一倍.

&emsp; &emsp;这里采用一种比较特殊的手段,可以不借助递归也不借助栈即可实现回溯.

#### 核心算法

&emsp; &emsp;初次划分时,我们将终态置于组0,其他置于组1.这与课本上的设定相反(一般地,我们认为编号0是初态而终态为编号最大值),这是因为我们只需要对非终态进行划分,这样做使得只需从组1开始扫描.当然,你也可以保持终态为1,然后每次反向扫描.

&emsp; &emsp;接下来我们开始检查组1,我们默认每个组的第一个元素为该组代表,所有与代表不同组的元素我们添加到一个新的组中.而且一旦发现新的组别,我们需重置迭代索引来实现回溯.

&emsp; &emsp;最终,我们直接以组号代表该小组.

&emsp; &emsp;主要流程图如下:

[![YmsPts.png](https://s1.ax1x.com/2020/05/07/YmsPts.png)](https://imgchr.com/i/YmsPts)

#### 实例分析

&emsp; &emsp;这里再给出一个实例,假设有如下DFA结点0,1,2,3,4.其中4为终态.最终划分结果为每个结点各一组.

1. 初次划分g0={4},g1={0,1,2,3}.

2. 遍历g1,发现1,2,3皆与0不同组,将其加入g2.目前g1={0},g2={1,2,3}.

3. 重新遍历,g1通过.遍历g2,发现2与1不同组,将其加入g3,得到g2={1,3},g3={2}.

4. 重新遍历,g1通过.由于2被划分到g3,3与1此时不同组,将其加入g4.得到g2={1},g3={2},g4={3}.

5. 重新遍历,均通过,以组号作为该组代表编号,结束划分.

#### 数据结构

&emsp; &emsp;由于在最小化DFA过程中打乱了顺序,故我们需要记录初态

```java
public class MinDFA {

    public int inital;// 初态
    public ArrayList<DVertex> Dstates = new ArrayList<>();
    public ArrayList<HashMap<Character, Integer>> transMaps = new ArrayList<>();
}
```

### MinDFA合并

&emsp; &emsp;至此,我们已经为每个正规式得到了其最小化的DFA,即MinDFA.接下来我们需要将这些DFA合并成一个大的DFA.

&emsp; &emsp;DFA的合并与汤普森算法中的|操作非常相似,只不过DFA的合并是将所有初态合并为一个新初态,并且不添加新的终态.

#### 核心算法

&emsp; &emsp;算法主要流程图如下:

[![YmTilD.png](https://s1.ax1x.com/2020/05/07/YmTilD.png)](https://imgchr.com/i/YmTilD)

#### 补充说明

&emsp; &emsp;在计算DFA结点新编号时,需要注意结点编号小于初态与结点编号大于初态的偏移量是不同的.这是因为所有初态被合并至一个新初态.例如:当我们合并第一个DFA时,偏移量=1(因为已经有一个公共初态结点0).设该DFA中Dstates为{0,1,2,3},其中1为初态.

&emsp; &emsp;合并DFA时各个结点的新结点分别为:

1. 0=0+1=1

2. 1=0(被合并至公共初态)

3. 2=2+1-1=2(初态结点被合并)

4. 3=3+1-1=3(初态结点被合并)

&emsp; &emsp;现在来解释前文提到的

> 请确保你的正规式最终生成的最小化DFA中,终态不包含初态.

&emsp; &emsp;来看一个例子,正规式a*得到的MinDFA如下:

![GvyyVO.png](https://s1.ax1x.com/2020/04/13/GvyyVO.png)

&emsp; &emsp;可以发现0既是初态又是终态,而每个正规式的初态都会被合并到一个公共初态.发现问题了吗?合并后的MinDFA的初态是唯一的,而这个唯一的初态也是终态.在分组过程中我们似乎很难对其划分.这也是这条规定的由来.

&emsp; &emsp;回头看一看我们的分组算法,对于上述例子,我们会直接得到一个终态分组{0}然后直接结束分组.得到的MinDFA没有初态,也就是说得到了一个死的DFA.最直接的影响就是这条正规式失效,但还有其他隐患.

&emsp; &emsp;如果你定义了这样的正规式,可能会缺少终态但保留状态转移,可能会使程序崩溃.这取决于你的正规式内容,你的正规式位置以及其他的正规式.总而言之,请避免这样的行为.

&emsp; &emsp;还需注意的是,多个MinDFA合并后得到的并非是MinDFA,严格来讲,可能合并结果连DFA都不是.举个例子,有如下两个DFA:

[![GLbyff.png](https://s1.ax1x.com/2020/04/12/GLbyff.png)](https://imgchr.com/i/GLbyff)

&emsp; &emsp;合并后会得到:
![GLbot0.png](https://s1.ax1x.com/2020/04/12/GLbot0.png)

&emsp; &emsp;可以看出,结点0接受字符/时有两条状态转移,这不符合DFA的定义.在编码中,由于使用HashMap,第二个DFA会覆盖第一个DFA,也就是说最终得到的DFA不接受字符/.好在目前的正规式不会有这样的冲突.当然,从拓展性上来讲想要实现移位操作符(>>,<<)会变得十分困难,因为它和操作符>,<冲突了.

&emsp; &emsp;那是否有什么办法能够补救?似乎可以将合并结果转为NFA,然后再执行一次NFA->DFA->MinDFA.但很遗憾,这很难做到.对比MinDFA和DFA以及NFA的数据结构,我们发现MinDFA舍弃很多辅助信息.因为的确不需要这些信息,所以我们现在想通过MinDFA反推得到NFA会变成一件很麻烦的事.

&emsp; &emsp;更好的解决办法是:在得到各个正规式的NFA时候对其进行合并.这也很符合我们的逻辑,C语言从词法上无非就是各个正规式并操作之后取闭包,即(ID|NUMBER|......)*.但这么做还要考虑下面两个问题:

1. 对终态的描述需要做出改变,合并后的NFA不止有一个终态.而汤普森算法得到的NFA只有一个终态,这意味着我们的数据结构需要做出改变.

2. 程序难以调试,仅ID一个正则式的NFA顶点数目已经上百,多个正规式合并后可能由上千个顶点和更多的状态转移.

&emsp; &emsp;综上,考虑词法分析的实现度和编码的工作量,采取对MinDFA合并而非对NFA合并.

### 驱动器算法

&emsp; &emsp;将源代码识别为记号流.

#### 核心算法

&emsp;&emsp;主要流程图如下:

[![YA5crV.png](https://s1.ax1x.com/2020/05/06/YA5crV.png)](https://imgchr.com/i/YA5crV)

&emsp;&emsp;识别结束后还需根据前文提及的关键字列表来区分ID与关键字.

#### 补充说明

&emsp; &emsp;来看一个例子:123abc.显然,这既不符合ID的正规式也不符合NUMBER的正规式,但是程序会将其识别为两个Token:NUMBER 123与ID abc.原因在于:当程序扫描到字符a时,发现无状态转移且123已经位于终态NUMBER,故将其识别成两部分.

&emsp; &emsp;这么做是否有必要?为什么我们不在无状态转移时直接报告错误呢?来看一个例子:if(true).理想情况下我们应该得到四个Token:if,(,true,).我们并不希望扫描到(时发现无法状态转移而直接认定这是一个错误的字符,而我们不能也不该强制要求各个单词之间必须以空格分隔,所以采取了这种折中的办法.

&emsp; &emsp;另外一点好处是:可以满足最长匹配.例如>和>=都属于运算符,当程序中出现>=时我们希望得到一个Token而非两个,上述算法能满足这个需求.换而言之,这种行为是贪婪的.

&emsp; &emsp;有没有更好的解决方法呢?有的.我们规定一组分隔符,例如{'(',')',';',....}.当我们发现无状态转移时只要检查字符是否属于分隔符,如果不是分隔符则应当判定为错误.诚然,这么做似乎能解决上述两个问题,但同时也有两个问题是需要我们思考的:

1. 有哪些元素属于分隔符?仔细想想可能(,[,{,+,-等都能成为分隔符.这意味着我们模糊了每个字符的定位,每个字符扮演的角色摸棱两可.对于程序而言,这种做法增加了耦合.

2. 这种做法是否保留了状态转移的思想?似乎没有,这种做法似乎越来越向硬编码靠近了.

3. 词法分析器的行为是错误的吗?

&emsp; &emsp;严格来讲,词法分析器这么做是对的.这个错误会在语法分析器中被报告出来.我们可以看看IDEA中的两个例子.

[![Y9jd78.png](https://s1.ax1x.com/2020/05/04/Y9jd78.png)](https://imgchr.com/i/Y9jd78)

[![Y9jFkF.png](https://s1.ax1x.com/2020/05/04/Y9jFkF.png)](https://imgchr.com/i/Y9jFkF)

&emsp; &emsp;对于"123 abc"和"123abc",IDEA给出了一样的错误信息,这说明语法分析器在后续过程中能识别出这个错误.

## 时间复杂度分析

### 汤普森算法

&emsp; &emsp;设有n个正规式,每个正规式平均有k个字符分别用m1,m2...mk表示.字符间的操作考虑复杂度高的|操作.即正规式为m1|m2|m3...

&emsp; &emsp;最终生成的NFA大概如下:(图中省略了部分信息)

[![Yes3R0.png](https://s1.ax1x.com/2020/05/07/Yes3R0.png)](https://imgchr.com/i/Yes3R0)

&emsp; &emsp;生成一条状态转移的时间复杂度为O(1),下面来分析过程:

1. m1,m2状态转移数目为:1,1.m1|m2得到1+1+4=6条状态转移.

2. m1|m2,m3状态转移数目为:6,1.m1|m2|m3得到6+1+4=11条状态转移.

3. m1|m2|m3,m4状态转移数目为:10,1.m1|m2|m3|m4得到11+1+4=16条状态转移.

&emsp; &emsp;不难看出,这是一个等差数列.所以该部分时间复杂度为O(k^2).

&emsp; &emsp;综上,汤普森算法的时间复杂度为O(n*k^2).

### NFA->DFA

&emsp; &emsp;设字母表有n个元素,NFA中有m个结点.由于使用Hash,状态转移的数目不影响状态转移的速率,时间复杂度为O(1).

&emsp; &emsp;表达式中有大量的并操作会导致时间复杂度的急剧上升,如果查看log中文件,可以发现正规式ID的NFA结点数目为460,而DFA中每个结点的状态集数目为130-170,约为NFA结点的1/3.也就是说,可以假设每次求空闭包的时间复杂度高达O(m).对每个结点,我们要求n次(遍历字母表):

&emsp; &emsp;空闭包(smove(m1,字母表中元素))

&emsp; &emsp;生成一个结点后需要查找该结点是否存在,该部分时间复杂度为1+2+3+..k,k为DFA中节点数目,观察log中文件,我们不妨假设k=m/3.即时间复杂度为O(m^2).

&emsp;&emsp;故最差时间复杂度为O(nm^2)+O(m^2),即O(nm^2)

&emsp;&emsp;最优时间复杂度为只有连接操作,求空闭包时间复杂度为O(1).

&emsp;&emsp;综上,NFA->DFA部分时间复杂度为O(nm^2)

### DFA->MinDFA

&emsp; &emsp;设有n个非终态结点,字母表中有m个元素.最差情况为n个结点属于n个分组.这里给出一个例子,设n=5.


| 扫描组号 | 扫描组内容 | 检查同组次数 | 分组情况  |
|:--------:|:----------:|:------------:|:---------:|
|    1     |   12345    |      4       |  1,2345   |
|    1     |     1      |      0       |  1,2345   |
|    2     |    2345    |      3       |  1,2,345  |
|    1     |     1      |      0       |  1,2,345  |
|    2     |     2      |      0       |  1,2,345  |
|    3     |    345     |      2       | 1,2,3,45  |
|    1     |     1      |      0       | 1,2,3,45  |
|    2     |     2      |      0       | 1,2,3,45  |
|    3     |     3      |      0       | 1,2,3,45  |
|    4     |     45     |      1       | 1,2,3,4,5 |
|    1     |     1      |      0       | 1,2,3,4,5 |
|    2     |     2      |      0       | 1,2,3,4,5 |
|    3     |     3      |      0       | 1,2,3,4,5 |
|    4     |     4      |      0       | 1,2,3,4,5 |
|    5     |     5      |      0       | 1,2,3,4,5 |

&emsp; &emsp;检查两个DFA是否同组需要遍历字母表,比较所有的状态转移结果是否同组.程序中使用Hash,查找时间复杂度为O(1),故检查两个DFA的时间复杂度为O(m).

&emsp; &emsp;先考虑扫描组元素数目大于1的情况,可以看到组1需要检查4个元素是否与1同组,组2需要检查3个元素是否与2同组......显然,这是一个等差数列,时间复杂度为O(n^2)O(m).

&emsp; &emsp;再来看扫描组元素数目等于1的组,可以看到组1扫描了4次,组2扫描了3次......同样的,这也是一个等差数列,由于不需要检查两个DFA是否同组,每次扫描的时间复杂度为O(1),故时间复杂度同为O(n^2).

&emsp; &emsp;故最差时间复杂度为O(n^2)O(m)+O(n^2),即O(mn^2).

&emsp; &emsp;当只有一个分组时时间复杂度最优,为O(n).
综上,DFA->MinDFA时间复杂度为O(mn^2).

### 合并DFA

&emsp; &emsp;该部分时间复杂度计算比较简单,设最终共有n个顶点,m条状态转移.时间复杂度为O(n)+O(m)

### 驱动器算法

&emsp; &emsp;设源程序字符数目为n,则时间复杂度为O(n)

### 小结

&emsp; &emsp;以上时间复杂度计算有部分遗漏,例如DFA中transMaps的构建,但这不是最复杂的工作,对最终结果影响不大.

## 实验收获体会

1. 掌握生成词法分析器的方法，加深对词法分析原理的理解。
2. 对自动机有了更深的认识,可以利用自动机实现更多应用(例如正则表达式)

## 改进意见

1. 可以优化流程为:汤普森算法->NFA合并->NFA转DFA->DFA最小化->驱动器算法.
2. 可以添加更多的操作如?操作.
3. 汤普森算法可拆分为中缀转后缀和后缀表达式求值两部分,降低程序耦合性.