# 第一次过程报告

## 6月27日、28日 上机前的讨论

### 小组第一次开会的商讨内容：

#### **1.对编译器的基本模块进行了划分**

​		编译器分为前端和后端，前端包括 词法分析、语法分析、语义分析以及中间代码生成，后端包括优化和目标代码生成。另外还有符号表和错误处理。

#### **2.商议了设计方案和思路**

​		我们经过商议，每个人都发表出自己的想法，最终确定了一个比较高效的方案：在确定完文法之后，设计出它的翻译文法，利用递归下降子程序的分析方法进行语法分析，将词法分析作为语法分析的一部分，在扫描一遍源程序的情况下将语法、语义分析全部完成，并生成四元式集合。然后在进行优化和中间代码生成。

#### **3.确立了分工**

​	我的暂定分工如下：

​	文法讨论、部分扫描器设计、部分语法分析、符号表设计、中间代码生成、目标代码生成

​	这只是讨论当天暂定的分工，会随着后续的进展进行调整。

#### **4.简单的设计出一套简单的文法**

​		我们小组在讨论当天决定设计基于C语言的文法，并确立了函数定义与声明语句、变量定义与声明语句、条件语句、while循环语句、算术表达式等。

### 讨论后还存在的问题：

1.文法过于简单，需要拓展，并且需要将文法转换为LL（1）文法。

2.符号表的设计未完成，符号表连接着语法和语义分析，文法确定完之后就要抓紧时间确定符号表。

### 我的下一步工作计划：

- [ ] 讨论并且设计符号表


- [ ] 和黄磊同学确定词法分析部分开发工作的分配，开始词法扫描器的开发


## 6月29日 第一次上机

### 讨论了符号表的设计

我们小组一起讨论了符号表的设计，初步确定符号表分为四部分：

1. 符号总表
2. 函数表
3. 数组表
4. 函数参数表

```c++
struct SYNBLNode//符号总表
{
    char content[30];//内容(名字)
    string type;//类型
    char cat[3];//种类
    int addr;//地址
    char active;//活跃信息
    SYNBLNode* next;//下一个节点
};
SYNBLNode* SYNBLHead[50];//首节点

SYNBLNode* SYNBLp2;
struct AINFLNode//数组表
{
    int LOW;//数组下界
    int UP;//数组上界
    string  CTP;//成分
    int CLEN; //成分类型的长度
    AINFLNode* next;//下一个节点
};
AINFLNode * AINFLHead;//首节点
AINFLNode * AINFLPn;

 struct PARAMLNode//函数参数表
{   char content[30];//内容（名字）
    string type;//类型
    char cat[3];//种类
    int addr_num;//活动记录中的地址
    PARAMLNode* next;//下一个节点
};
 PARAMLNode *  PARAMLHead;//首节点

struct PFINFLNode//函数表
{   string content;
    int OFF;//区距–该过函自身数据区起始单元相对该过函值区区头位置
    int FN;//参数个数
    PARAMLNode*  PARAM;//域成分类型指针
    int ENTRY;//该函数目标程序首地址
    PFINFLNode* next;//下一个节点
};
 PFINFLNode *  PFINFLHead;//首节点
PFINFLNode* PFINFLP;
```

​	从上面的数据结构可以看出我们初步设计的符号表为如下结构：

​		符号总表一层一层的记录定义的变量或者函数，其中第零层为全局变量区，包括定义的函数也放在第一层，第一层为定义的第一个函数，然后接下来依次为第二层为定义的第二个函数......一直到main函数层，其中每个函数层里放着函数里定义的变量。这样设计在对变量进行运算时起到了很大的帮助，在对变量运算时，我们先在变量所在函数层寻找它，再去第零层即全局变量区寻找它。

### 我完成的内容：

#### **1.和黄磊同学商议了识别器的数据结构**

​	我们确定采取链表的形式来存储一个个token，其中token的数据结构如下：

```c++
struct Node//token节点
{
    char content[30];//内容
    int type;//类码
    int addr;//入口地址
    Node * next;//下一个节点
};Node * Head;
```

​	之所以设计为链表结构，是因为链表结构可以动态扩展，并且操作方便，并且在语法分析的递归下降子程序中可以直接返回token结构体指针类型，这使递归下降子程序设计更加方便。

#### **2.和黄磊同学就翻译器的设计进行了讨论**

​	我们将标识符定为0，字符定为1，字符串定为2，常数定为3，关键字和界符一词一码，这样做的好处是在扫描完一个单词之后，它的种类也就能唯一确定，并方便后续语法分析的使用。

```
iT标识符：00
cT字符：01
sT字符串：02
CT常数：03
KT关键字：void 04,char 05,short 06,int 07,long long 08,float 09,double 10,bool 11,if 12,else 13,while 14,do 15,for 16,main 17,return 18
PT界符：>= 19,<= 20,== 21,!= 22,> 23,< 24,= 25,&& 26,|| 27,! 28,+ 29,- 30,* 31,/ 32,% 33,<< 34,>> 35,, 36,; 37,( 38,) 39,[ 40,] 41,{ 42,} 43，# 44
```

​	其中最后一个#是为了判断程序是否读取结束

#### **3.和黄磊同学一起完成了扫描器的设计**

​	在对翻译器设计完之后，我和黄磊同学就开始着手设计扫描器我主要设计扫描器的算法流程，黄磊同学负责绘制有限状态自动机，我们暂定整个词法分析包括以下功能函数：

```c++
void initNode();//token结点初始化
int seekKT(char* word);//查询KT表
int seekPT(char* word);//查询PT表
void createNewNode(char* content, int type, int addr);//生成新token结点
void Next(FILE* infile);//扫描器函数
void printtoken();//打印token函数
```

#### **4.将现有文法改造成LL1文法，着手部分语法分析的开发（我和黄磊同学负责）**

​	我们将上机前确定的文法改造为LL（1）文法，并绘制递归下降子程序流程图，确定了一下递归下降子程序函数：

```c++
Node* Program(Node* poi);//程序
Node* TheSubsequentProgram(Node* poi);//后继程序
Node* CompoundStatement(Node* poi); //复合语句
Node* TheStatementList(Node* poi); //语句列表
Node* GeneralVariableDeclaration(Node* poi);//普通变量声明
Node* SubsequentVariableDeclaration(Node* poi);//后继变量声明
Node* LogicStatement(Node* poi);//逻辑语句
Node* LogicItemt(Node* poi);//逻辑项
Node* LogicFactor1(Node* poi);//逻辑因子1
Node* LogicFactor2(Node* poi);//逻辑因子2
```

### 第一次上机出现的问题：

​		今天主要确定了符号表的数据结构，并且设计了词法分析和初步的语法分析，但是我们发现使用递归下降子程序进行语法分析的话，就要赶快确立最终文法，不然等语法分析写完时就很难再修改。另外在进行词法分析设计时，有些地方可能考虑不周，还有待完善。

### 下一步工作计划：

- [ ] 文法拓展
- [ ] 完善词法分析器

- [ ] 对拓展文法进行语法分析的开发


## 6月30日讨论

### **会议内容**：

- 一起详细的探讨了拓展文法，敲定了最终文法，加入了for循环，函数嵌套调用、++和--运算符等


- 汇报了和黄磊同学词法分析方面的进度


- 和黄磊同学商定如何对最终的文法进行语法分析的设计

### 最终确定的文法：

```c++
1.<开始>---><程序> <后继程序>
<Start>---><Program> <The subsequent program>

2.<后继程序>---><程序> <后继程序>| 空
<The subsequent program>---><Program> <The subsequent program>| #(???)

3.<程序> ---> 函数定义 <函数声明> <复合语句> |  变量定义 <变量声明>
<Program> ---> 函数定义 <Function declaration> <Compound statement> |  变量定义 <Variable declarations>

4.<复合语句> ---> {<语句列表>}
<Compound statement> ---> {<The statement list>}

5.<函数声明> ---> <类型1> （<标识符> (<参数列表>) | main ()）
<Function declaration> ---> <Type 1> （<标识符> (<Parameters list>) | main ()）

6.<变量声明> ---> <类型2> （<普通变量声明> <后继变量声明> | <数组变量声明> <后继数组变量声明>）
<Variable declarations> ---> <Type 2> （<General variable declaration> <Subsequent variable declaration> | <Array variable declaration> <Subsequent array variable declaration>）

7.<类型1> ---> int| char | float | double | bool |void
<Type 1> ---> int| char | float | double|bool |void

8.<类型2> ---> int| char | float | double |bool
<Type 2> ---> int| char | float | double |bool

9.<普通变量声明> ---> <标识符> | <标识符> = <算术表达式> | <标识符> = <字符> | <标识符> = <字符串>
<General variable declaration> ---> <标识符> | <标识符> = <Arithmetic expression> | <标识符> = <字符> | <标识符> = <字符串>

10.<后继变量声明> ---> ,<普通变量声明> <后继变量声明> ;| 空
<Subsequent variable declarations> ---> ,<General variable declaration> <Subsequent variable declaration> ;| 空

11.<数组变量声明> ---> <数组名> | <数组赋值语句>
<Array variable declaration> ---> <Array name> | <Array assignment statement>

12.<数组名> ---> <标识符> [<数字>]
<Array name> ---> <标识符> [<数字>]

13.<后继数组变量声明> ---> ，<数组变量声明> <后继数组变量声明>;| 空
<Subsequent array variable declaration> ---> ，<Array variable declaration> <Subsequent array variable declaration>;| 空
14.<数组赋值语句> ---> <数组名> = [<值列表>]
<Array assignment statement> ---> <Array name> = [<Value List >]

15.<值列表> ---> <整数> <后继整数>
<Value List> ---> <整数> <Subsequent integer>

16.<后继整数> ---> ，<整数> <后继整数>| 空
<Subsequent integer> ---> ，<整数> <Subsequent integer>| 空

17.<参数列表> ---> <参数声明> <后继参数声明>| 空
<Parameters list> ---> <Parameter declaration> <Subsequent parameter declaration>| 空

18.<参数声明> ---> <类型2> <普通变量声明>
<Parameter declaration> ---> <Type 2> <General variable declaration>

19.<后继参数声明>--->, <参数声明> <后继参数声明>| 空
<Subsequent parameter declaration>--->, <Parameter declaration> <Subsequent parameter declaration>| 空

20.<语句列表>---><语句><语句列表>|空
<The statement list>---><Statement><The statement list>|空

21.<语句>---> <变量声明> | <赋值语句>; | if <条件语句>|  while <while语句> |return < return 语句> |for<for 语句>|;
<Statement>---> <Variable declarations> | <Assignment statement>; | if <IF Statement >|  while <WHILE Statement> |return < RETURN Statement>; |for <For Statement>|

22.<条件语句>---> (<逻辑语句>)（<语句>|<复合语句> ）|if (<逻辑语句>)（<语句>（else（<语句>|<复合语句> ）|空
）|<复合语句> （else（<语句>|<复合语句> ）|空
）
<IF Statement >---> (<Logic statement>)（<Statement>|<Compound statement> ）|if (<Logic statement>)（<Statement>（else（<Statement>|<Compound statement> ）|空
）|<Compound statement> （else（<Statement>|<Compound statement> ）|空）

 23.<while语句>---> (<逻辑语句>)（<语句>|<复合语句> ）
<WHILE Statement>---> (<Logic statement>)（<Statement>|<Compound statement>  ）

24.<逻辑语句>---> <逻辑项>||<逻辑语句> |<逻辑项>
<Logic statement>---> <Logic item>||<Logic statement> |<Logic item>

25.<逻辑项>---> <逻辑因子>&&<逻辑项>|<逻辑因子>
<Logic item>---> <Logic factor>&&<Logic item>|<Logic factor>

26.<逻辑因子>---><逻辑因子>a1<逻辑因子2>|<逻辑因子2>
<Logic factor>---><Logic factor>a1<Logic factor2>|<Logic factor2>

27.<逻辑因子2>---> <逻辑因子2> a2 <算术表达式>
<Logic factor2>---> <Logic factor2> a2 <Arithmetic expression>

28.<算术表达式> ---><算术表达式> ω0 <项> | <项>     
<Arithmetic expression>  <Arithmetic expression> ω0 <item> | <item>     
  
29.<项> ---> <项> ω1  <因子> | <因子>        
<item> ---> <item> ω1  <factor> | <factor>     
 
30.<因子> ---> <单元> | ω2 <单元>
<factor> ---> <unit> | ω2 <unit>

31.<单元> ---> <算术量> | ( <逻辑语句> )
<unit> ---> <Amount of arithmetic> | ( <Logic statement> )

32.<算术量> ---> <标识符> | <常数> 
<Amount of arithmetic> ---> <标识符> | <常数> 

33.< return 语句> ---> <标识符> |<对应常量>
< RETURN Statement> ---> <标识符> |<对应常量>

34.	<for语句>---> '(' [ [int]＜标识符＞＝＜算术表达式＞];[＜逻辑语句＞];[＜标识符＞(＝ ＜标识符＞ (+|-) ＜无符号整数＞ | ++ | --) ]  ')' （<语句>|<复合语句> ）
<FOR Statement>--->  '(' [ [int]＜标识符＞＝<Arithmetic expression>];[<Logic statement>];[＜标识符＞ (＝ ＜标识符＞ ω0 ＜无符号整数＞ | ω3 )  ]  ')' （<Statement>|<Compound statement>  ）
 其中：  ω0 — +或-
        ω1 — *或/或%
        ω2 — 单目运算符
		ω3 — ++或--
        α1 — ==或!=
        α2 — >=或>或<=或<
```



## 7月1日 第二次上机

### 我的工作部分：

1.参与完成了递归下降子程序流程图的设计

2.组员三人各负责部分语法分析的开发

3.完善词法分析器

​	加入++和--运算符，类码分别为45，46，并在词法分析器中加入了注释功能。

```c++
//处理注释
        else if (ch=='/')
		{
			ch=fgetc(infile);
            if (ch=='*')
			{
				int count=0;
				ch=fgetc(infile);
				while (count!=2)
				{
					count=0;
					while (ch!='*')
					{
						if(ch=='\n') line_count++;
						ch=fgetc(infile);
					}
					count++;
					ch=fgetc(infile);
					if (ch=='/')
					{
						count++;
					}
					else
					{
						ch=fgetc(infile);
					}
				}
				Next(infile);
			}
			else if(ch=='/')
            {
                ch=fgetc(infile);
				while (ch!='\n')
                {
                    ch=fgetc(infile);
                }
                line_count++;
                Next(infile);
            }
            else
            {
                fseek(infile,-2L,SEEK_CUR);//向后回退两位
                ch=fgetc(infile);
                array[i++] = ch;
                word = new char[i+1];
                memcpy(word,array,i);
                word[i] = '\0';
                int seekTemp = seekPT(word);
                createNewNode(word,seekTemp,-1);
            }

		}
```



### 遇到的问题:

​	在进行语法分析时，我们发现了好多漏洞，比如if语句后面只能跟{}复合语句，而不能跟单个语句，还有函数的返回值可以为算数表达式，而不仅仅为一个标识符或常量。

### 第三次上机计划:

- [ ] 1.把文法设计上的Bug找出来

- [ ] 2.把语法分析再完善

- [ ] 3.和组员一起商量符号表如何填写

## 7月2日 第三次上机

### 我的工作：

1.找出了流程图设计上的Bug

​	在<复合语句>的文法上应该实现右递归，一直到识别的单词为 } 时才退出 ，在<语句>的文法上应该加上单个；作为一条语句。

2.解决文法设计上的问题

​	我修改了一下<语句>以及<复合语句>的结构，将if语句改为可跟单个句子的形式。

​	修改后的<复合语句>程序如下：

```c++
Node *CompoundStatement(Node *  poi) //复合语句
{
    while(poi->type!=43)//判断}
        poi=TheStatementList(poi); //进入 语句列表 子程序
    if(poi->type==43)//判断}
    {
        Next(infile);
        poi = poi->next;
        return poi;
    }
    else error();
}
```

3.思考如何填写符号表

​	我们经过商议初步暂定为将扫描器识别为标识符的单词填写符号表，还要具体完善。

### 遇到的问题：

for循环的语法分析设计遇到阻碍，由于我们的设计是一遍语法分析将四元式生成，所以在实现for循环的时候要先配合着李雨芮同学将翻译文法写出来，由于课上没有给过for循环的四元式，我们就面临到了一些问题。

### 进一步计划：

- [ ] 前端工作完成得差不多，需要参与到中间代码生成和符号表填写的开发中去。

- [ ] 配合李雨芮同学将for循环的语法分析以及四元式生成功能实现。

## 7月3日

#### 主要工作：

将for循环设计和实现，下图为我们设计的for循环的四元式：

![https://raw.githubusercontent.com/The-2020-Compiler-Project/NiShuoDeDui/scan_machine/for.jpg?token=APVF7H2AAMUKYZWBMPVQI5K677JCY]()

​	其中比较难实现的是 i++ 的那部分因为词法分析器是从文件里逐词扫描，所以 i++ 这部分就一定在前面先被扫描，那么在执行完复合语句之后怎么回去执行 i++ 呢？面对这个问题我思考了好久，最终想到了一种解决的办法，定义一个类型为token节点指针的vector，在扫描到 i++ 部分时，将指针依次压入向量，再执行完复合语句之后再将指针依次取出。（注：i++ 为代指，也可以为 i=i+1 ）。

​	部分代码：

```c++
vector <Node *> for_node;
 do{
            for_node.push_back(poi);
            Next(infile);
            poi = poi->next;
        }while(poi->type!=39);//判断）
        
```

​	其中，我们还在尝试在for 循环的初始条件定义变量，例 ：

```C++
for(int i=0;i<n;i++)
```

​	但这样的话就又涉及到另一个问题，即局部变量的问题，由于我们符号表的设计只考虑了函数里的局部变量而没有考虑循环里的局部变量问题，所以我们正在努力解决此问题，我的想法是在再一个for的参数表，这样for循环中定义的局部变量就可以和函数的变量区分开，我们正在商议此方法的可行性。

我设计的for参数表如下：

```C++
 struct FORPARAMLNode//for参数表
{   char content[31];//内容（名字）
    string type;//类型
    char cat[3];//种类
    int addr;//地址
    FORPARAMLNode* next;//下一个节点
};
FORPARAMLNode *FORPARAMLHead;
```

​	目前还需检查此方案的可行性。

## 三次上机以来的收获：

​	从开始讨论文法至今，基本上过去了一周的时间，这一周虽然很苦很累，但是却收获颇丰，我们课堂上学的知识在这里得到了实践，这让我对编译的思想有了一个更深的理解和体会，刚开始我们的设计结构看起来很难实现，但是在这几天的辛苦努力下，我们基本上完成了语法分析和中间代码（即四元式）生成，虽然还有许多问题需要完善，但是却给了我们很大的信心去接着向下走，接下来我们将会去挑战一下优化和目标代码生成。在这一周我主要有以下体会：

#### 1.小组合作的重要性

​	由于我们三个成员并不是第一次合作（去年数据结构课设就已经合作），所以默契度还是很可观的，这也很大程度的帮助我们进展如此之快。

#### 2.理论知识的重要性

​	课堂上学习的理论知识看起来毫无用处，只能用来做题，但是当我真正开始实践时，我发现了理论知识是至关重要的，没有自动机的理论，词法分析器凭空想是很难解决的，没有自上而下或自下而上的那套语法分析流程，语法分析根本无从下手，没有符号表的设计就无法判断变量的声明和使用情况等等。

#### 3.文法设计的重要性

​	文法是编译的拦路虎，如果一开始就没把文法设计完善，则后续就会出现各种各样的错误，语法和语义分析实现起来就会特别困难。