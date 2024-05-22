---
cover: >-
  https://images.unsplash.com/photo-1713990287879-8a6dfa24ef6e?crop=entropy&cs=srgb&fm=jpg&ixid=M3wxOTcwMjR8MHwxfHJhbmRvbXx8fHx8fHx8fDE3MTU5MTc0NjZ8&ixlib=rb-4.0.3&q=85
coverY: 0
layout:
  cover:
    visible: true
    size: hero
  title:
    visible: false
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# 关键字

{% hint style="info" %}
C语言有多少个关键字?sizeof 怎么用?它是函数吗?不知道C语言有多少个关键字？大多数人往往不知道并告诉我 sizeof是函数，因为它后面跟着一对括号。有些关键字可能从来没见过，有的惊讶C语言关键字竞有 32个之多。更有甚者说 sizeof是函数，没想到它居然是关键字!
{% endhint %}

## C语言关键字

<table data-full-width="false"><thead><tr><th width="374" align="center">关键字</th><th align="center">意义</th></tr></thead><tbody><tr><td align="center">Auto</td><td align="center">声明自动变量，缺省时编译器一般默认为Auto</td></tr><tr><td align="center">int</td><td align="center">声明整型变量</td></tr><tr><td align="center">double</td><td align="center">声明双精度变量</td></tr><tr><td align="center">long</td><td align="center">声明长整型变量</td></tr><tr><td align="center">char</td><td align="center">声明字符型变量</td></tr><tr><td align="center">float</td><td align="center">声明浮点型变量</td></tr><tr><td align="center">short</td><td align="center">声明短整型变量</td></tr><tr><td align="center">signed</td><td align="center">声明有符号类型变量</td></tr><tr><td align="center">unsigned</td><td align="center">声明无符号类型变量</td></tr><tr><td align="center">struct</td><td align="center">声明结构体变量</td></tr><tr><td align="center">union</td><td align="center">声明联合数据类型</td></tr><tr><td align="center">enum</td><td align="center">声明枚举类型</td></tr><tr><td align="center">static</td><td align="center">声明静态变量</td></tr><tr><td align="center">switch</td><td align="center">用于开关语句</td></tr><tr><td align="center">case</td><td align="center">开关语句分支</td></tr><tr><td align="center">default</td><td align="center">开关语句中的“其他”分支</td></tr><tr><td align="center">break</td><td align="center">跳出当前循环</td></tr><tr><td align="center">register</td><td align="center">声明寄存器变量</td></tr><tr><td align="center">const</td><td align="center">声明只读变量</td></tr><tr><td align="center">volatile</td><td align="center">说明变量在程序执行中可被隐含地改变</td></tr><tr><td align="center">typedef</td><td align="center">用以给数据类型取别名(当然还有其他作用)</td></tr><tr><td align="center">extern</td><td align="center">声明变量是在其他文件正声明(也可以看做是引用变量)</td></tr><tr><td align="center">return</td><td align="center">子程序返回语句(可以带参数，也可不带参数)</td></tr><tr><td align="center">void</td><td align="center">声明函数无返回值或无参数，声明空类型指针</td></tr><tr><td align="center">continue</td><td align="center">结束当前循环，开始下一轮循环</td></tr><tr><td align="center">do</td><td align="center">循环语句的循环体</td></tr><tr><td align="center">while</td><td align="center">循环语句的循环条件</td></tr><tr><td align="center">if</td><td align="center">条件语句</td></tr><tr><td align="center">else</td><td align="center">条件语句否定分支(与 if 连用)</td></tr><tr><td align="center">for</td><td align="center">一种循环语句(可意会不可言传)</td></tr><tr><td align="center">goto</td><td align="center">无条件跳转语句</td></tr><tr><td align="center">sizeof</td><td align="center">计算对象所占内存空间大小</td></tr></tbody></table>

## 什么是定义？什么是声明？它们有何区别?

举个例子：

&#x20;A)int i；&#x20;

B)extern int i；

&#x20;哪个是定义？哪个是声明？或者都是定义或者都是声明？

### &#x20;什么是定义：

所谓的定义就是(编译器)创建一个对象，为这个对象分配一块内存并给它 取上一个名字，这个名字就是我们经常所说的变量名或对象名。但注意，这个名字一旦和 这块内存匹配起来(可以想象是这个名字嫁给了这块空间，没有要彩礼啊。^\_^)，它们就同 生共死，终生不离不弃。并且这块内存的位置也不能被改变。一个变量或对象在一定的区 域内（比如函数内，全局等）只能被定义一次，如果定义多次，编译器会提示你重复定义 同一个变量或对象。

### 什么是声明：有两重含义，如下：&#x20;

第一重含义：告诉编译器，这个名字已经匹配到一块内存上了(伊人已嫁，吾将何去何 从？何以解忧，唯有稀粥)，下面的代码用到变量或对象是在别的地方定义的。声明可以出 现多次。

&#x20;第二重含义：告诉编译器，我这个名字我先预定了，别的地方再也不能用它来作为变量 名或对象名。比如你在图书馆自习室的某个座位上放了一本书，表明这个座位已经有人预 订，别人再也不允许使用这个座位。其实这个时候你本人并没有坐在这个座位上。这种声 明最典型的例子就是函数参数的声明，例如： void fun(int i, char c); 好，这样一解释，我们可以很清楚的判断:A)是定义；B)是声明。 那他们的区别也很清晰了

{% hint style="info" %}
定义声明最重要的区别：定义创建了对象并为这个对象分配了内存，声明没有分配内存
{% endhint %}

