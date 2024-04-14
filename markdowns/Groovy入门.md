# groovy是什么

简单地说，Groovy 是下一代的java语言，跟java一样,它也运行在 JVM 中。

作为跑在JVM中的另一种语言，groovy语法与 Java 语言的语法很相似。同时，Groovy 抛弃了java烦琐的文法。同样的语句，使用groovy能在最大限度上减少你的击键次数——这确实是“懒惰程序员们”的福音。

 

# 准备开发环境

- jdk 1.5以上
- eclipse+groovy plugin（支持Groovy 1.5.7）

打开eclipse，通过Software Updates > Find and Install...菜单，使用“Search for new features to install” 下载并安装groovy插件。New一个远程站点，url可使用<http://dist.codehaus.org/groovy/distributions/update/>，插件名：Groovy plug-in。根据需要你可以同时选择groovy和grails（后续会学习到）：

  

# 创建groovy项目

## 1. 新建一个groovy项目

New --> Project à Java Project 创建一个java项目。为了方便管理，建议在source中建两个source文件夹java和groovy，分别用于存储java源文件和groovy源文件：

 

## 2. 添加 Groovy 特性

在项目上右击，Groovy à Add Groovy Nature，这样会在项目中添加 Groovy Libraries。

 

## 3. 添加 Groovy 类

在项目groovy源文件下右键，New à Other àGroovy à Groovy Class

 

自动生成的源代码如下：
```java
public class HelloWorld{
    /**
     * @param args
     */
    public static void main(def args){
       // TODO Auto-generated method stub
    }  
}
```
我们在main方法中加一句打印语句：

`println "Hello World"`

## 4. 编译运行groovy类

在源文件上右键，Compile Groovy File，然后右键，Run As à Groovy ，在控制台中查看运行结果。

实际上 groovy 语法的简练还体现在，就算整个文件中只有println "Hello World"这一句代码（把除这一句以外的语句删除掉吧），程序也照样能够运行。

当然，为了说明groovy 其实就是java，你也可以完全按照java 语法来编写HelloWorld类。

四、Groovy语法简介

1、  没有类型的java

作为动态语言，groovy中所有的变量都是对象(类似于.net framework，所有对象继承自java.lang.Object)，在声明一个变量时，groovy不要求强制类型声明，仅仅要求变量名前使用关键字def（从groovy jsr 1开始，在以前的版本中，甚至连def都不需要）。

修改main 方法中的代码：

**def** var="hello world"

**println** var

**println** var.class

你可以看到程序最后输出了var的实际类型为：java.lang.String

作为例外，方法参数和循环变量的声明不需要def。

2、  不需要的public

你可以把main方法前面的public去掉，实际上，groovy中默认的修饰符就是public，所以public修饰符你根本就不需要写，这点跟java不一样。

3、  不需要的语句结束符

Groovy中没有语句结束符，当然为了与java保持一致性，你也可以使用;号作为语句结束符。在前面的每一句代码后面加上;号结束，程序同样正常运行(为了接受java程序员的顽固习惯)。

4、  字符串连接符

跟java一样，如果你需要把一个字符串写在多行里，可以使用+号连接字符串。代码可以这样写：

​       **def** var="hello "+

​       "world"+

​       ",groovy!"

当然更groovy的写法是：

​       **def** var="""hello

​       world

​       groovy!"""

三个”号之间不在需要+号进行连接（不过字符串中的格式符都会被保留，包括回车和tab）。

5、  一切皆对象

听起来象是“众生平等”的味道，事实上groovy对于对象是什么类型并不关心，一个变量的类型在运行中随时可以改变，一切根据需要而定。如果你赋给它boolean ，那么不管它原来是什么类型，它接受boolean值之后就会自动把类型转变为boolean值。看下面的代码：

**def** var="hello "+

​       "world"+

​       ",groovy!"

​       **println** var;

​       **println** var.**class**;

​       var=1001

​       **println** var.**class**

输出结果：

hello world,groovy!

class java.lang.String

class java.lang.Integer

** **

var这个变量在程序运行中，类型在改变。一开始给它赋值String，它的类型就是String，后面给它赋值Integer，它又转变为Integer。

6、  循环

删除整个源文件内容，用以下代码替代：

​       **def** var="hello "+

​       "world"+

​       ",groovy!"

​       **def** repeat(val){

​            **for**(i = 0; i < 5; i++){

​             **println** val

​            }

​       }

​       repeat(var)

输出：

hello world,groovy!

hello world,groovy!

hello world,groovy!

hello world,groovy!

hello world,groovy!

注意循环变量i前面没有def。当然也没有java中常见的int，但如果你非要加上int也不会有错，因为从Groovy1.1beta2之后开始（不包括1.1beta2），groovy开始支持java经典的for循环写法。

此外，上面的for语句还可以写成：

​            **for**(i **in** 0..5)

这样的结果是一样的。      

7、  String 和 Gstring

除了标准的java.lang.String以外（用’号括住），groovy还支持Gstring字符串类型（用“号括住）。把上面的for循环中的语句改成：

​             **println** "This is ${i}:${val}"

运行一下，你就会明白什么是Gstring。

8、  范围

这个跟pascal中的“子界”是一样的。在前面的for循环介绍中我们已经使用过的**for**(i **in** 0..5)这样的用法，其中的0..5就是一个范围。

范围 是一系列的值。例如 “0..4” 表明包含 整数 0、1、2、3、4。Groovy 还支持排除范围，“0..<4” 表示 0、1、2、3。还可以创建字符范围：“a..e” 相当于 a、b、c、d、e。“a..<e” 包括小于 e 的所有值。

范围主要在for循环中使用。

9、  默认参数值

可以为方法指定默认参数值。我们修改repeat方法的定义：

**def** repeat(val,repeat=3){

​            **for**(i **in** 0..<repeat){

​             **println** "This is ${i}:${val}"

​            }

​       }

可以看到，repeat方法增加了一个参数repeat（并且给了一个默认值3），用于指定循环次数。

当我们不指定第2个参数调用repeat方法时，repeat参数取默认值3。

10、              集合

Groovy支持最常见的两个java集合：java.util.Collection和java.util.Map。前面所说的范围实际也是集合的一种（java.util.List）。

(1) Collection

Groovy 中这样来定义一个Collection：

**def** **collect**=["a","b","c"]

除了声明时往集合中添加元素外，还可以用以下方式向集合中添加元素：

**collect**.add(1);

​       **collect**<<"come on";

​       **collect**[**collect**.**size()**]=100.0

Collection使用类似数组下标的方式进行检索：

​       **println** **collect**[**collect**.**size**()-1]

​       **println** **collect**[5]

groovy支持负索引：

**println** **collect**[-1]      //索引其倒数第1个元素

​       **println** **collect**[-2]      //索引其倒数第2个元素

Collection支持集合运算：

**collect**=**collect**+5        //在集合中添加元素5

​       **println** **collect**[**collect**.**size**()-1]

​       **collect**=**collect**-'a'         //在集合中减去元素a(第1个)

​       **println** **collect**[0]          //现在第1个元素变成b了

同样地，你可以往集合中添加另一个集合或删除一个集合：

​       **collect**=**collect**-**collect**[0..4]   //把集合中的前5个元素去掉

​       **println** **collect**[0]   //现在集合中仅有一个元素，即原来的最后一个元素

​       **println** **collect**[-1]  //也可以用负索引，证明最后一个元素就是第一个元素

(2) Map

Map是“键-值”对的集合，在groovy中，键不一定是String，可以是任何对象(实际上Groovy中的Map就是java.util.`Linke dHashMap`)。

如此可以定义一个Map:

​       **def** map=['name':'john','age':14,'sex':'boy']

添加项：

​       map=map+['weight':25]       //添加john的体重

​       map.put('length',1.27)      //添加john的身高

​       map.father='Keller'         //添加john的父亲

可以用两种方式检索值：

​       **println** map['father']       //通过key作为下标索引

​       **println** map.length          //通过key作为成员名索引

11、              闭包（Closure）

闭包是用{符号括起来的代码块，它可以被单独运行或调用，也可以被命名。类似‘匿名类’或内联函数的概念。

闭包中最常见的应用是对集合进行迭代，下面定义了3个闭包对map进行了迭代：

​       map.**each**({key,value->    //key,value两个参数用于接受每个元素的键/值

​       **println** "$key:$value"})

​       map.**each**{**println** it}     //it是一个关键字，代表map集合的每个元素

​       map.**each**({ **println** it.getKey()+"-->"+it.getValue()})

除了用于迭代之外，闭包也可以单独定义：

**def** say={word->

​           **println** "Hi,$word!"

​       }

调用：

say('groovy')

​       say.**call**('groovy&grails')

输出：

Hi,groovy!

Hi,groovy&grails!

 

看起来，闭包类似于方法，需要定义参数和要执行的语句，它也可以通过名称被调用。然而闭包对象（不要奇怪，闭包也是对象）可以作为参数传递（比如前面的闭包作为参数传递给了map的each方法）。而在java中，要做到这一点并不容易（也许C++中的函数指针可以，但不要忘记java中没有指针）。其次，闭包也可以不命名（当然作为代价，只能在定义闭包时执行一次），而方法不可以。

12、              类

Groovy类和java类一样，你完全可以用标准java bean的语法定义一个groovy 类。但作为另一种语言，我们可以使用更groovy的方式定义和使用类，这样的好处是，你可以少写一半以上的javabean代码：

(1)    不需要public修饰符

如前面所言，groovy的默认访问修饰符就是public，如果你的groovy类成员需要public修饰，则你根本不用写它。

(2)    不需要类型说明

同样前面也说过，groovy也不关心变量和方法参数的具体类型。

(3)    不需要getter/setter方法

不要奇怪，在很多ide（如eclipse）早就可以为序员自动产生getter/setter方法了。在groovy中，则彻底不需要getter/setter方法——所有类成员（如果是默认的public）根本不用通过getter/setter方法引用它们（当然，如果你一定要通过get/set方法访问成员属性，groovy也提供了它们）。

(4)    不需要构造函数

不在需要程序员声明任何构造函数，因为groovy自动提供了足够你使用的构造函数。不用担心构造函数不够多，因为实际上只需要两个构造函数（1个不带参数的默认构造函数，1个只带一个map参数的构造函数—由于是map类型，通过这个参数你可以在构造对象时任意初始化它的成员变量）。

(5)    不需要return

Groovy中，方法不需要return来返回值吗？这个似乎很难理解。看后面的代码吧。

因此，groovy风格的类是这样的：

(6)    不需要()号

Groovy中方法调用可以省略()号（构造函数除外），也就是说下面两句是等同的：

 

```
person1.setName 'kk'
```

```
person1.setName('kk')
```

```
 
```

下面看一个完整类定义的例子：

**class** Person {

 **def** name

 **def** age

 String toString(){//注意方法的类型String，因为我们要覆盖的方法为String类型

​     "$name,$age"

 }

如果你使用javabean风格来做同样的事，起码代码量要增加1倍以上。

我们可以使用默认构造方法实例化Person类：

**def** person1=**new** Person()

person1.name='kk'

person1.age=20

**println** person1

也可以用groovy的风格做同样的事：

**def** person2=**new** Person(['name':'gg','age':22]) //[]号可以省略

**println** person2

 

这样需要注意我们覆盖了Object的toString方法，因为我们想通过println person1这样的方法简单地打印对象的属性值。

然而toString 方法中并没有return 一个String，但不用担心，Groovy 默认返回方法的最后一行的值。

13、              ？运算符

在java中，有时候为了避免出现空指针异常，我们通常需要这样的技巧：

if(rs!=null){

​       rs.next()

​       … …

}

在groovy中，可以使用?操作符达到同样的目的：

rs?.next()

?在这里是一个条件运算符，如果?前面的对象非null，执行后面的方法，否则什么也不做。

14、              可变参数

等同于java 5中的变长参数。首先我们定义一个变长参数的方法sum：

**int** sum(**int**... var) {

**def** total = 0

**for** (i **in** var)

total += i

**return** total

}

我们可以在调用sum时使用任意个数的参数（1个，2个，3个……）：

**println** sum(1)

**println** sum(1,2)

**println** sum(1,2,3)

15、              枚举

定义一个enum：

enum Day {

SUNDAY, MONDAY, TUESDAY, WEDNESDAY,

THURSDAY, FRIDAY, SATURDAY

}

然后我们在switch语句中使用他：

**def** today = Day.SATURDAY

**switch** (today) {

//Saturday or Sunday

**case** [Day.SATURDAY, Day.SUNDAY]:

**println** "Weekends are cool"

**break**

//a day between Monday and Friday

**case** Day.MONDAY..Day.FRIDAY:

**println** "Boring work day"

**break**

**default**:

**println** "Are you sure this is a valid day?"

}

注意，switch和case中可以使用任何对象，尤其是可以在case中使用List和范围，从而使分支满足多个条件（这点跟delphi有点象）。

同java5一样，groovy支持带构造器、属性和方法的enum：

enum Planet {

MERCURY(3.303e+23, 2.4397e6),

VENUS(4.869e+24, 6.0518e6),

EARTH(5.976e+24, 6.37814e6),

MARS(6.421e+23, 3.3972e6),

JUPITER(1.9e+27,7.1492e7),

SATURN(5.688e+26, 6.0268e7),

URANUS(8.686e+25, 2.5559e7),

NEPTUNE(1.024e+26, 2.4746e7)

**double** mass

**double** radius

Planet(**double** mass, **double** radius) {

**this**.mass = mass;

**this**.radius = radius;

}

**void** printMe() {

**println** "${name()} has a mass of ${mass} " +

"and a radius of ${radius}"

}

}

Planet.EARTH.printMe()

16、              Elvis操作符

这是三目运算符“?:”的简单形式，三目运算符通常以这种形式出现：

String displayName = name != null ? name : "Unknown";

在groovy中，也可以简化为（因为null在groovy中可以转化为布尔值false）：

String displayName = name ? name : "Unknown";

基于“不重复”的原则，可以使用elvis操作符再次简化为：

String displayName = name ?: "Unknown"

17、              动态性

Groovy所有的对象都有一个元类metaClass，我们可以通过metaClass属性访问该元类。通过元类，可以为这个对象增加方法（在java中不可想象）！见下面的代码，msg是一个String,通过元类，我们为msg增加了一个String 类中所没有的方法up：

**def** msg = "Hello!"

**println** msg.metaClass

String.metaClass.up = {  delegate.toUpperCase() }

**println** msg.up()

通过元类，我们还可以检索对象所拥有的方法和属性（就象反射）：

msg.metaClass.methods.**each** { **println** it.name }

msg.metaClass.properties.**each** { **println** it.name }

甚至我们可以看到我们刚才添加的up方法。

我们可以通过元类判断有没有一个叫up的方法，然后再调用它：

**if** (msg.metaClass.respondsTo(msg, 'up')) {

​    **println** msg.toUpperCase()

}

当然，也可以推断它有没有一个叫bytes的属性：

**if** (msg.metaClass.hasProperty(msg, 'bytes')) {

​    **println** msg.bytes.encodeBase64()

}

18、              Groovy swing

到现在为止，我们的groovy一直都在控制台窗口下工作。如果你还不满足，当然也可以使用swingbuilder来构建程序：

**import** groovy.swing.SwingBuilder

**import** java.awt.BorderLayout

**import** groovy.swing.SwingBuilder

**import** java.awt.BorderLayout **as** BL

**def** swing = **new** SwingBuilder()

**count** = 0

**def** textlabel

**def** frame = swing.frame(title:'Frame', **size**:[300,300]) {

borderLayout()

textlabel = label(text:"Clicked ${count} time(s).",

constraints: BL.NORTH)

button(text:'Click Me',

actionPerformed: {**count**++; textlabel.text =

"Clicked ${count} time(s)."; **println** "clicked"},

constraints:BorderLayout.SOUTH)

}

frame.pack()

frame.show()

怎么样？是不是跟java 中写swing程序很象？

 

五、单元测试

1、  添加junit

使用 Build PathàAdd Libraries... 把junit添加到项目中。

2、  新建测试

使用 New à Junit Test Case 新建测试例程：PersonTest，在Class under test右边的Browser按钮，选择要进行测试的groovy类Person。

Finish，下面编写测试用例代码（我使用了Junit4）：

import org.junit.*;

public class TestPerson {

​       @Test

​       public void testToString(){

​              Person p=new Person();              //注意因为groovy编译Person时默认所有属性为private

​              p.setName("ddd");                //所以用set方法设置name属性而不用p.name=”ddd”

​              p.setAge(18);

​              Assert.assertEquals("ddd-18", p.toString());

​       }

}

运行Run AsàJunit Test，发现testToString通过测试。

3、使用groovy书写测试用例

除了使用Java来书写测试用例以外，我们也可以使用groovy书写。

New à Other à Groovy à Groovy Class，写一个类GroovyTestPerson：

**import** org.junit.*;

 

**class** GroovyTestPerson {

​    @Test

​     **void** testToString(){

​       Person p=**new** Person("name":"ddd","age":18)

​       Assert.assertEquals("ddd-18", p.toString())

​    }

}

可以看到，这里使用的完全是Groovy风格的书写方式：不需要public，使用map参数构造对象。然而当你Run AsàJunit Test的时候，结果跟用java编写的测试用例没有什么两样。

这也充分说明了，groovy和java，除了语法不一样，本质上没有什么区别（对比.net framework中的C#和VB.net，它们除了语法不同外，本质上它们都使用CLR）。

 