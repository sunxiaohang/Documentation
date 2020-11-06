#### 基本概念
- 要实现一门语言，我们必须`构建一个能读取句子以及对发现的短语和输入符号作出适当反应的应用`。
- 能识别特定语言的所有有效的句子、短语和子短语 a = 5 区别于 a+b
- 识别语言的程序被称为`语法分析器`
- 语法指代控制语言成员的规则，`每条规则都表示一个短语的结构`
- 为了更容易地实现识别语言的程序，通常我们会把识别语言的语法分析拆解成两个相似但不同的任务或阶段。
- 把字符组成单词或符号（记号）的过程被称为词法分析或简单标记化。我们把标记输入的程序称为词法分析器。
- 词法分析器能把相关的记号组成记号类型，例如INT（整数）、ID（标志符）、FLOAT（浮点数）等。
- 当语法分析器只关心类型的时候，词法分析器会把词汇符号组成类型，而不是单独的符号。记号至少包含两块信息：记号类型（确定词法结构）和匹配记号的文本。
- 第二阶段是真正的语法分析器，它使用这些记号去识别句子结构，在本例中是赋值语句。默认情况下，ANTLR生成的语法分析器会构建一个称为语法分析树或语法树的数据结构，它记录语法分析器如何识别输入句子的结构和它的组件短语
- 语法分析树的内部节点是分组和确认它们子节点的短语名字。
- 通过生成语法分析树，语法分析器给应用的其余部分提供了方便的数据结构，它们含有关于语法分析器如何把符号组成短语的完整信息。树是非常容易处理的，并且也能被程序员很好的理解。更好的是，语法分析器能自动地生成语法分析树。
- 通过操作语法分析树，需要识别相同语言的多个应用能重用同一个语法分析器
- assign : ID '=' expr ;    // 匹配赋值语句像"a=5"

#### 实现语法分析器
ANTLR工具根据语法规则，例如我们刚才看到的assign，生成递归下降语法分析器。递归下降语法分析器只是递归方法的一个集合，每个规则一个方法。下降这个术语指的是分析从语法分析树的根开始向着叶子进行（记号）。我们首先调用的规则，即prog符号，成为语法分析树的根。那也就意味着对前面部分的语法分析树来说需要调用方法prog()。这类分析更通用的术语是自顶向下分析：递归下降语法分析器仅仅是自顶向下语法分析器实现的一种。
要了解递归下降语法分析器是什么样子，这里是ANTLR为规则assign生成的方法（稍微整理）：
```
// assign : ID '=' expr ;
void assign() {    // 根据规则assign生成的方法
    match(ID);     // 比较ID和当前输入符号然后消费
    match('=');
    expr();        // 通过调用expr()匹配表达式
}
```
递归下降语法分析器最酷的部分是通过调用方法prog()、assign()和expr()跟踪出的调用关系图反映了内部的语法分析树节点。match()的调用对应语法分析树叶子。为了在一个手工构建的语法分析器中手动构建一颗语法分析树，我们需要在每个规则方法的开始处插入“添加新子树根”操作，以及给match()一个“添加新叶子节点”操作。

方法assign()只是检查确保所有必要的记号存在且以正确的顺序。当语法分析器进入assign()时，它不必在多个选项之间进行选择。选项是规则定义右边的选择之一。例如，调用assign的prog规则可能有其它类型的语句。

```
/** 匹配起始于当前输入位置的任何语句 */
prog
    : assign    // 第一个选项（'|'是选项分隔符）
    | ifstat    // 第二个选项
    | whilestat
    ...
    ;
```
prog的分析规则看起来像一条switch语句：
```
void prog() {
    switch ( «current input token» ) {
        CASE ID : assign(); break;
        CASE IF : ifstat(); break;    // IF是关键字'if'的记号类型
        CASE WHILE : whilestat(); break;
        ...
        default : «raise no viable alternative exception»
    }
}
```
方法prog()必须通过检查下一个输入记号作出分析决定或预测。分析决定预判哪一个选项将会成功。在本例中，当看到WHILE关键字时会预判是规则prog的第三个选项。规则方法prog()然后就会调用whilestat()。你以前可能听说过术语预读记号，那只是下一个输入记号。预读记号可以是语法分析器在匹配和消费它之前嗅探的任何记号。

有时候，语法分析器需要一些预读记号去预判哪个选项会成功。它甚至必须考虑从当前位置直到文件结尾的所有的记号！ANTLR默默地为你处理所有的这些事情，但是对决策过程有个基本的了解是有帮助的，可以让调试生成的语法分析器更容易。

为更好地理解分析决定，想象有个单一入口和单一出口的迷宫，有单词写在地板上。每个沿着从入口到出口路径的单词序列表示一个句子。迷宫的结构与定义一门语言的语法规则类似。为测试一个句子在一门语言中的成员身份，我们在穿越迷宫时把句子的单词和沿着地板的单词作比较。如果通过句子的单词我们能到达出口，那么句子是有效的。

为了通过迷宫，我们必须在每个岔口选择一条有效路径，正如我们必须在语法分析器中选择选项。我们必须决定该走哪条路，通过把我们句子中下一个单词（们）和沿着来自每个岔口的每条路径上可见的单词比较。我们能从岔口看到的单词与预读记号类似。当每条路径以唯一的单词开始时决定是相当容易的。在规则prog中，每个选项从唯一的记号开始，因此prog()可以通过查看第一个预读记号识别选项。

当单词从一个岔口重叠部分开始每条路径时，语法分析器需要继续往前看，扫描可以识别选项的单词。ANTLR根据需要为每个决定自动上下调节预读数量。如果预读的结果是多条同样的到出口的路径，即当前的输入短语有多种解释。解决这样的二义性将是我们的下一个主题。

##### 二义性
一个模棱两可的短语或句子通常是指它有不止一种解释。换句话说，短语或句子能适配不止一种语法结构。要解释或转换一个短语，程序必须要能唯一地确认它的含义，这意味着我们必须提供无歧义的语法，以便生成的语法分析器能用明确的一个方法匹配每个输入短语。

在这里，让我们展示一些有歧义的语法以便让二义性的概念更具体。如果你以后在构建语法时陷入二义性，你可以参考本节的内容。

一些明显有歧义的语法：

assign
    : ID '=' expr    // 匹配一个赋值语句，例如f()
    | ID '=' expr    // 前面选项的精确复制
    ;

expr
    : INT ;
大多数时候二义性是不明显的，如同以下的语法，它能通过规则stat的两个选项匹配函数调用：

stat
    : expr          // 表达式语句
    | ID '(' ')'    // 函数调用语句
    ;

expr
    : ID '(' ')'
    | INT
    ;
这里是两个输入f()的解释，从规则stat开始：



左边的语法分析树显示f()匹配规则expr。右边的语法分析树显示f()匹配规则stat的第二个选项。

因为大部分语言它们的语法都被设计成无歧义的，有歧义的语法类似于编程缺陷。我们需要识别语法以便为每个输入短语提交单一选择给语法分析器。如果语法分析器发现一个有歧义的短语，它必须选一个可行的选项。ANTLR通过选择涉及决定的第一个选项解决二义性。在本例中，语法分析器将选择与左边的语法分析树有关的f()的解释。
#### 匹配arraylist
```
init :'{' value ( ',' value )* '}' ;
value : init
    |INT ;
INT : [0..9]+ ;
WS : [ \t\r\n]+ -> skip;
```
#### 在语法中嵌入任意的操作
如果我们不想付出构建语法分析树的开销，或者想要在分析期间动态地计算值或把东西打印出来，那么可以通过在语法中嵌入任意代码实现。它的比较困难的，因为我们必须明白在语法分析器上的操作的影响，以及在哪里放置这些操作。

为了解释嵌入在语法中的操作，让我们先来看下文件rows.txt中的数据：

parrt   Terence Parr    101
tombu   Tom Burns       020
bke     Kevin Edgar     008
这些列是由TAB分隔的，每一行用一个换行结束。匹配这种类型的输入在语法上还是相当简单的。下面是此语法文件Rows.g的内容：

file : (row NL)+ ;    // NL is newline token: '\r'? '\n'
row  : STUFF+ ;
我们需要创建一个构造器以便我们能传递我们想要的列号（从1开始计数），所以我们需要在规则中添加一些操作来做这些事情：
```
grammar Rows;

@parser::members {    // add members to generated RowsParser
    int col;
    public RowsParser(TokenStream input, int col) {    // custom constructor
        this(input);
        this.col = col;
    }
}

file: (row NL)+ ;

row
locals [int i=0]//locals子句定义的局部变量
    : ( STUFF
        {
        $i++;
        if ( $i == col ) System.out.println($STUFF.text);
        }
      )+
    ;

TAB  :  '\t' -> skip ;    // match but don't pass to the parser tab
NL   :  '\r'? '\n' ;      // match and pass to the parser 换行
STUFF:  ~[\t\r\n]+ ;      // match any chars except tab, newline tab和换行之外的字符
```
- 在上述语法中，操作是被花括号括起来的代码片段；members操作的代码将会被注入到生成的语法分析器类中的成员区
- 在规则row中的操作访问的$i是由locals子句定义的局部变量，该操作也用$STUFF.text获取最近匹配的STUFF记号的文本内容
- STUFF词法规则匹配任何非TAB或换行的字符，这意味着在列中可以有空格字符。

现在，是时候去思考如何使用定制的构造器传递一个列号给语法分析器，并且告诉语法分析器不要构建语法分析树了：

public class Rows {

    public static void main(String[] args) throws Exception {
        ANTLRInputStream input = new ANTLRInputStream(System.in);
        RowsLexer lexer = new RowsLexer(input);
        CommonTokenStream tokens = new CommonTokenStream(lexer);
        int col = Integer.valueOf(args[0]);
        RowsParser parser = new RowsParser(tokens, col);    // pass column number!
        parser.setBuildParseTree(false);    // don't waste time bulding a tree
        parser.file();
    }
}
现在，让我们核实下我们的语法分析器能否正确匹配一些示例输入：

antlr -no-listener Rows.g  # don't need the listener
compile *.java
run Rows 1 < rows.txt
这时你会看到rows.txt文件的第1列内容被输出：

parrt
tombu
bke
如果将上面命令中的1换成2，你会看到rows.txt文件的第2列内容被输出；如果换成3，那么rows.txt文件的第3列内容将会被输出。

##### 使用语义谓词实现语法分析
```
grammar IData;

file : group+ ;

group: INT sequence[$INT.int] ;

sequence[int n]
locals [int i = 1;]
     : ( {$i<=$n}? INT {$i++;} )*    // match n integers
     ;

INT  : [0-9]+ ;  // match integers
WS   : [ \t\n\r]+ -> skip ;    // toss out all whitespace
```
##### XML解析
```
lexer grammar XMLLexer;

// Default "mode": Everything OUTSIDE of a tag
OPEN        :   '<'                 -> pushMode(INSIDE) ;//起始标记
COMMENT     :   '<!--' .*? '-->'    -> skip ;//跳过注释
EntityRef   :   '&' [a-z]+ ';' ;//字母 & ;等特殊符号
TEXT        :   ~('<'|'&')+ ;    // match any 16 bit char minus < and &//正文文本 剔除&和<

// ----------------- Everything INSIDE of a tag ---------------------
mode INSIDE;

CLOSE       :   '>'                 -> popMode ;    // back to default mode//开始结束符
SLASH_CLOSE :   '/>'                -> popMode ;//结尾结束符
EQUALS      :   '=' ;
STRING      :   '"' .*? '"' ;//匹配字符串
SlashName   :   '/' Name ;//匹配元素名
Name        :   ALPHA (ALPHA|DIGIT)* ;//匹配字母或者数字
S           :   [ \t\r\n]           -> skip ;//跳过tab和换行

fragment
ALPHA       :   [a-zA-Z] ;

fragment
DIGIT       :   [0-9] ;
```

##### 运算表达式
```
grammar Express

prog
		: stat+
		;

stat 
		: INT
		| expr
		| ID '=' expr
		;

expr 
		: ID
		| INT
		| expr ('+'|'-') expr
		| expr ('*'|'/') expr
		| '(' expr ')'
		;
ID : [a-zA-Z]+ ;
INT : [0-9]+ ;
WS : [ \t\r\n]+ -> skip ;
```