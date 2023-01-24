# 复制值类型的变量和类

## 值类型变量的复制

C#的大多数基元数据类型，包括 `int`, `float`, `double` 和 `char` 等，不包括 `string`，都是**值类型**的。当你声明值类型的变量时，编译器会生成代码来分配足以容纳这种类型数据的内存块。例如，声明 `int` 类型的变量会导致编译器分配4字节的内存块，而向这种变量赋值会将值复制到内存块中。

听起来很抽象，我们可以用下面的代码来理解这种机制：

```C#
int a = 17;
int a_copy = a;
a++;
Console.WriteLine(a_copy);
```

输出的结果是多少呢？直觉上看，输出应该是 `17`。事实也确实如此，因为执行第二行赋值语句时，`a` 存储的值被复制到 `a_copy` 中，新变量 `a_copy` 与原变量仅是值相等，`a++` 操作并不影响 `a_copy` 的值，事实上，执行第二行代码时，`a_copy` 被分得另一块内存块，与原来的变量 `a` 分得的内存块并不相同。`a++` 作用于原来的内存块，不影响新分配的内存块。

既然有符合直觉的情况，那还有不符合直觉的情况吗？

有，那就是引用类型。

## 引用类型变量的复制

引用类型的变量，在复制时会有不同的行为，我们用下面的 `Circle` 类来说明这一点。

```C#
class Circle
{
    public const double Pi = 3.14; //这里圆周率的精度取得比较低
    public double radius;
    public Circle(double r) => radius = r; // 构造器也是可以用表达式主体方法的qwq
    public double Area() => Pi * radius * radius;
}
```

这个 `Circle` 类有一个 `radius` 字段表示圆的半径，有一个 `Area()` 方法来获得圆的面积。我们为它定义一个构造器，这个构造器接受一个参数 `r` 来初始化 `radius`。你应该知道，`Circle` 属于引用类型。接下来我们用这个类型来声明一个变量：

```C#
Circle circle = new Circle(0.5);
Circle circle2 = circle;
circle.radius = 2.5; //猜猜会发生什么？
Console.WriteLine(circle2.radius); //试着猜猜输出的值？
```

直觉上看，输出的值应该是 `0.5`，但如果你实际运行一下这段代码，你会发现它输出的值竟然是 `2.5`！

很神奇吧？让我们先猜测一下，我们把 `circle` 赋值给 `circle2`，但改变 `circle` 中字段的值，`circle2` 中的字段也同步发生了改变，根据这样的现象，我们可以猜测，`circle` 和 `circle2` 其实是同一个东西，只是有不同的“叫法”，~~就好比小裙子可以叫裙裙~~，那么对它们的操作自然也是对同一个对象的操作。事实上，**声明**一个 `Circle` 变量时，编译器并不生成代码来分配足以容纳一个 `Circle` 对象的内存块。相反，它把一小块内存分配出来，使得它可以容纳一个**地址**，该地址称为对内存块的**引用**。而在使用 `new` 关键字实际创建对象时，对象实际占用的内存才被分配，然后，赋值时，`Circle` 实际占用内存块的地址被填充到变量中。类是**引用类型**的一个例子。

我们可以用下面这个示意图表示值类型和引用类型的区别：

<img src="https://fs49.org/wp-content/uploads/2023/01/image-20230124152152857.png" alt="image-20230124152152857" style="zoom:33%;" />

引用类型和值类型的区别十分重要。方法参数的行为也取决于参数是值类型和引用类型

## 方法参数的行为

将上面的 `Circle` 类搬到你的 `Program` 类所在的命名空间中，然后让我们来体验一下在方法传参时两种类型的差异。

写下面的两个方法，然后在 `Main()` 方法中调用，代码如下：

```C#
static void  OperationOnCircle(Circle cir)
{
    cir.radius = 114514; // 对参数cir的radius赋值
}
static void OperationOnInt(int val)
{
    val = 114514; //对参数val赋值
}
static void Main(string[] args)
{
    Circle circle = new Circle(114);
    int number = 0;
    
    OperationOnInt(number);
    OperationOnCircle(circle);
    Console.WriteLine(number);
    Console.WriteLine(circle.radius);
    //根据上面学到的知识，尝试推测一下运行结果
}
```

如果你善用VS的自动提示，你会发现VS已经指出了其中的猫腻：

<img src="https://fs49.org/wp-content/uploads/2023/01/image-20230124153301470.png" alt="image-20230124153301470" style="zoom:67%;" />

为什么会发生这种事情呢，别急，让我们先运行一下代码，你会发现输出结果是下面这样：

```C#
0
114514
```

这表明当 `OperationOnCircle` 方法被调用时，方法内的语句对参数的修改作用到了原来的变量上，而 `int` 类型却没有。

下面的示意图表示了 `Circle` 类型的变量被传入方法时发生的事。在传入参数时，变量 `circle` 中存储的地址被传给参数 `cir` ，然后在方法内部对 `cir` 进行的任何操作，自然也会作用于同一对象，因此最终 `circle.radius` 的值为114514。

![image-20230124154221660](https://fs49.org/wp-content/uploads/2023/01/image-20230124154221660.png)

而作为值类型的 `int`，在作为参数时，其存储的值被复制一份，该值传进方法，因此方法中的操作并不影响原来的变量。

这一节的内容有些难，希望你能够反复品味，加深理解。

## 复制引用类型与Clone

我们有时不希望引用类型的变量以上面的模式，在复制时传递引用，因此我们需要以一定的方法手动复制一个对象。例如，如果我们要复制一个 `Circle`，我们可以像下面这样写：

```C#
Circle circle1 = new Circle(114);
Circle copy_circle = new Circle(0);
copy_circle.radius = circle1.radius;
// 手动复制字段，如果有更多的字段，也这样手动赋值
```

诚然，上面的写法实现了对象的复制，但如果 `radius` 是 `private` 的怎么办？这时你无法从外部访问该字段，在不知道它的值的情况下，你该怎么复制它呢？

我们为 `Circle` 类引入一个 `Clone()` 方法，该方法返回自己的新实例，并填充相同的数据。`Clone` 方法能访问对象的私有数据，并把数据复制到同一个类的另一个实例中。示例代码如下：

```C#
class Circle
{
    private int radius;
    // 构造器什么的自己写罢
    // ...
    public Circle Clone()
    {
        //创建一个新的Circle对象
        Circle clone = new Circle();
        
        //复制数据
        clone.radius = this.radius; 
        
        //返回新对象
        return clone;
    }
}
```

如果所有私有数据都是值类型，这种实现方式没有任何问题。但是，如果类中还包含引用类型的字段，这些引用类型也需要提供类似的 `Clone()` 方法，否则 `Circle` 类的 `Clone()` 就只能复制对这些字段的引用。只复制引用称为**浅拷贝**，而如果这些字段的类型也提供了对应的 `Clone()` 方法，能够复制引用的对象，那么 `Circle` 的 `Clone()` 能完全复制对象，就称为**深拷贝**。

此外，上面的代码中有一个有趣的问题：`radius` 被标记为私有，为什么在一个对象的 `Clone()` 方法中能够访问另一个对象的 `radius` 字段呢？我们之前说过，`private` 关键字创建了不能从类外访问的字段或方法。但是，这并不是说它只能由此对象访问。同一个类的几个对象，它们分别能互相访问彼此的私有数据。这听起来很怪，但像 `Clone()` 这样的方法正是依赖此特性。

因此，类中的“私有”是指“在类的级别上私有”，而非“在对象的级别上私有”。另外，静态和私有也是两码事，字段声明为私有，类的每个实例都有一份自己的数据；字段声明为静态时，所有实例共享同一份数据。

# 理解null值和可空类型

## 为什么要有null

我们知道，变量应当尽量在声明时初始化。对于值类型，我们经常用下面的写法：

```C#
int i = 0;
double d = 0.0;
// 声明时初始化是好文明！（赞赏）
```

为了初始化引用类型的变量，可以创建类的实例，然后将对象的引用赋给变量：

```C#
Circle circle = new Circle(42);
```

到现在为止，一切都很美好。但是，如果我们不想创建新对象怎么办？例如，我们只是想用变量来存储对一个现有对象的引用。在下面的例子中，`Circle` 类型的变量 `copy` 先被初始化，但稍后又将对另一个 `Circle` 对象的引用赋值给它。

```C#
Circle c = new Circle(42);
Circle copy = new Circle(99);
//...
copy = c;
```

将 `c` 赋给 `copy` 后，原来的 `new Circle(99)` 实例会发生什么事情？那个实例已经“落单”了，现在不存在任何对它的引用。在这种情况下，运行时通过垃圾回收机制来回收内存，这是会在后面的章节中介绍的。就目前而言，你只需要垃圾回收是一个可能会比较耗时的操作，你不应该创建从来不用的对象，这会浪费时间和资源。

我们有时候会碰到下面这种情况：

```C#
Circle c = new Circle(42);
Circle copy; 
//...
if (copy == ...) // 只有copy还没初始化时对它赋值，但这里应该写什么？
{
    copy = c;
    //...
}
```

因此，我们引入一个特殊值 `null`，该值表示“空”。如果一个引用类型的变量值为 `null`，就表示它不引用内存中的任何对象。所以上述代码的正确形式是：

```C#
Circle c = new Circle(42);
Circle copy = null; //声明时初始化是好文明！（赞赏） 
//...
if (copy == null) // 如果copy为空(null)
    copy = c;
    //...
}
```

## 空条件操作符

在C# 6.0中添加了一种新的操作符来测试一个变量是否为 `null`，它的使用例如下：

```C#
Circle c = null;
Console.WriteLine($"The area of circle c is {c.Area()}");
```

此方法会抛出一个 `NullReferenceException` 异常。这很合理，因为你无法计算 `null` 的面积。因此有下面的写法：

```c#
if (c != null)
{
	Console.WriteLine($"The area of circle c is {c.Area()}");
}
```

引入空条件操作符后我们可以让它更简洁：

```C#
Console.WriteLine($"The area of circle c is {c?.Area()}");
```

这时的输出会是：

```C#
The area of circle c is 
```

这两种方法都很有效，可以满足你在不同情况下的需要。空条件操作符有助于保持代码简洁。在处理复杂的嵌套、引用类型时，如果它们可能全都为空值，空条件操作符会显得非常有用，就像下面这样：

```C#
Circle circle = null;
if (circle?.radius > 50)
{
    Console.WriteLine("Radius is greater than 50");
}
```

控制台不会输出任何东西。

你可以参考C#官方文档中的[相关部分](https://learn.microsoft.com/zh-cn/dotnet/csharp/language-reference/operators/member-access-operators#null-conditional-operators--and-)。

## 可空类型（未完成）

.NET 6中的可空类型我还不熟，先看看[官方文档](https://learn.microsoft.com/zh-cn/dotnet/csharp/tutorials/nullable-reference-types)吧。

# 使用ref和out参数

向方法传递实参时，该形参会以实参的拷贝初始化——无论是值类型还是引用类型。即使在传入引用时，你可以通过该引用对原对象进行操作，但你无法让原来的变量引用不同的对象。换句话说，你不可能在这些情况中修改实参本身。大多时候，这个机制能减少程序的bug，但有时我们希望方法能实际地修改一个实参，因此提供了 `ref` 和 `out` 关键字。

## 使用ref参数

如果我们在声明方法时，为参数列表中的参数添加 `ref` 前缀，C#编译器将生成代码传递对实参的引用而非拷贝。使用 `ref` 参数，作用于参数的所有操作都会应用于原来的实参，因为它们引用同一个对象。作为 `ref` 参数传递的实参也必须附加 `ref` 前缀。这个语法明确告诉你，传入的参数可能发生改变。下面是一个例子：

```C#
static int doIncrement(ref int num)
{
    num++;
}
static void Main(string[] args)
{
    int i = 42;
    doIncrement(ref i); // 传参也要加上ref
    Console.WriteLine(i); // 输出是43
}
```

“变量使用前必须赋值”的规则也适用于方法实参。不能将未初始化的值作为实参传入方法，即使是 `ref` 实参。例如，下面的代码：

```C#
static int doIncrement(ref int num)
{
    num++;
}
static void Main(string[] args)
{
    int i;
    doIncrement(ref i); // 编译错误
}
```

对于引用类型，你可以通过下面的代码理解 `ref` 引起的差别。

```C#
class Circle
{
    public const double Pi = 3.14;
    public double radius;
    public Circle(double r) => radius = r;
    public double Area() => Pi * radius * radius;
}
internal class Program
{
    static void OperateObject(Circle c)
    {
        c.radius = 114514; // 操作对象
    }
    static void ChangeParam(Circle c)
    {
        c = new Circle(100); // 尝试改变参数
    }
    static void RefParam(ref Circle c)
    {
        c = new Circle(100); // 加上ref后尝试改变参数
    }

    static void Main(string[] args)
    {
	   Circle circle = new Circle(1919);
       OperateObject(circle); 
       Console.WriteLine(circle.radius); // 输出114514，因为对象被操作
       ChangeParam(circle);
       Console.WriteLine(circle.radius); // 仍输出114514
       RefParam(ref circle);
       Console.WriteLine(circle.radius); // 输出100
    }
}
```

如果你善用VS你还是能看到这条提示消息：

![image-20230124174547869](https://fs49.org/wp-content/uploads/2023/01/image-20230124174547869.png)

第二行的输出是114514而非100，因为这个赋值操作没有作用于原来的对象。

## 使用out参数

编译器会在调用方法前验证它的 `ref` 参数已被赋值。但我们有时候希望由方法自身初始化参数，所以希望向它传入未初始化的实参。因此我们引入了 `out` 关键字。和 `ref` 的语法相似，它可在参数列表中作为形参的前缀；在传入实参时，也必须附加前缀，且在方法中应用于参数的任何操作也会应用于原来的实参。

向方法传递 `out` 参数后，**必须**在方法内部对其进行赋值。例如：

```C#
static void DoInitialize(out int num)
{
    num = 42; // 删去对num的初始化将导致编译错误
}
```

由于 `out` 参数必须在方法中赋值，所以调用方法时不需要对实参进行初始化，例如以下代码：

```C#
static void doInitialize(out int num)
{
    num = 114514;
}
static void Main(string[] args)
{
    int arg; // 不初始化
    doInitialize(out arg);
    Console.WriteLine(arg);
    doInitialize(out int arg2); // 一种更简单的写法
    Console.WriteLine(arg2);
}
```

~~下面是裙子的教程中用到的比方（也可以是裙子语录）~~

<img src="https://fs49.org/wp-content/uploads/2023/01/image-20230124175547062-1.png" alt="image-20230124175547062" style="zoom:67%;" />

# *计算机内存的组织方式（理论）

计算机使用内存来容纳要执行的程序以及这些程序使用的数据。为了理解值类型和引用类型的区别，我们有必要理解数据在内存中是如何组织的。操作系统和运行时将用于容纳数据的内存划分为两个独立的区域，**栈**(Stack)和**堆**(Heap)。

它们的设计目标完全不同，也以不同的方式管理。

- 调用方法时，它的参数和局部变量所需的内存总是从栈中获取。方法结束时要么正常返回，要么抛出异常，所以为参数和局部变量分配的内存将自动归还到栈中，并可在另一个方法调用时重新使用。栈上方法参数和局部变量具有良好定义的生存期。方法开始，生存期开始；方法结束，生存期结束。
- 使用 `new` 关键字创建对象（类的实例）时，构造对象所需的内存总是从堆中获取。前面讲过，使用引用变量可以从多个地方引用同一个对象，对象的最后一个引用消失之后，对象占用的内存就可以重用（虽然不一定立刻被回收）。堆上创建的对象具有较不确定的生存期，使用 `new` 关键字将创建对象，但只有在删除了最后一个对象引用之后的某个不确定时刻，它才会消失。

> 所有值类型都在栈上创建，所有引用类型的实例（对象）都在堆上创建。可空类型实际上是引用类型，所以在堆上创建。

栈和堆这两个词来源于运行时的内存管理方式。

- 栈内存就像一系列堆得越来越高的箱子。调用方法时它的每一个参数都被放入一个箱子，并将这个箱子放到站的最顶部，每个局部变量也同样分配到一个箱子，并同样放到栈顶。方法结束后，它的所有箱子都从栈中移出。
- 堆内存则像散布在房间里的一大堆箱子，不像栈那样，每个箱子都严格地堆在另一个箱子上，每个箱子都有一个标签，标记了这个箱子是否正在使用。创建新对象时，运行时查找空箱子，把它分配给对象，对对象的引用则存储在栈上的一个局部变量中。运行时跟踪每个箱子的引用数量（记住，两个变量可能引用同一个对象），一旦最后一个引用消失，运行时就会将箱子标记为未使用，将来某个时候会清除箱子里的东西，使之能被重用。

请思考，在你调用以下方法时会发生什么：

```C#
void Method(int param)
{
    Circle param;
    c = new Circle(param);
    //...
}
```

假定参数 `i` 为42。调用方法时，栈中将分配一个 `int` 大小的内存并用值42初始化。在方法内部，还要从栈中分配出另一小块内存，它刚够存储一个引用（一个内存地址），只是暂时不进行初始化（它是为变量 `c` 准备的）。接着要从堆中分配一个足够大的内存区域来容纳一个 `Circle` 对象，这正是 `new` 关键字所执行的操作，它运行 `Circle` 构造器，将这个原始的堆内存转换成 `Circle`对象，对这个 `Circle` 对象的引用将存储到变量 `c` 中。下图对此进行了演示：

<img src="https://fs49.org/wp-content/uploads/2023/01/image-20230124191339844-1.png" alt="image-20230124191339844" style="zoom: 33%;" />

这时应注意以下两点：

1. 虽然对象本身存储在堆中，但对象引用（即变量 `c`）存储在栈中。
2. 堆内存是有限的资源。如果资源耗尽，`new` 操作符将抛出 `OutOfMemoryException`，对象创建失败。

> 构造器本身抛出异常时，分配的堆内存会全部回收，构造器返回 `null`。

方法结束后，参数和局部变量将离开作用域。为 `c` 和 `param` 分配的内存会被自动回收到栈。运行时发现已不存在对 `Circle` 对象的引用，所以会在将来的某个时候安排垃圾回收器回收它的内存。

# System.Object类

## 引入object类

`System.Object` 类是C#中最重要的引用类型之一。要想理解它的重要性，你需要先理解继承（后面的内容）。就目前而言，你需要暂时接受这样的说法：所有类都派生自 `System.Object` 。此类型的变量能引用任何对象。由于它相当重要，C#提供了 `object` 关键字作为它的别名。实际写代码时应该尽量使用 `object`。

下面的例子中变量 `c` 和 `o` 引用同一个 `Circle` 对象。它们从不同角度引用内存中的同一个东西。

```C#
Circle c = new Circle(42);
object o = c; //OK
```

<img src="https://fs49.org/wp-content/uploads/2023/01/image-20230124192809505-1.png" alt="image-20230124192809505" style="zoom: 33%;" />

## 装箱

如前所述，`object` 变量能引用任何类型的任何对象，它自然也能引用值类型的实例。例如，下面的例子：

```C#
int i = 42;
object o = i; //OK
```

执行第2个语句时，所发生的事情需要仔细思考一下。`i` 是值类型，所以它在栈中。如果直接引用 `i`，那么引用的将是栈。但这种行为是不允许的，所有的引用必须引用堆上的对象。实际发生的事情是运行时在堆中分配一小块内存，将 `i` 的值复制到这块内存中，然后让变量 `o` 引用这份拷贝。这种自动将数据项从栈复制到堆的行为称为**装箱**。

<img src="https://fs49.org/wp-content/uploads/2023/01/image-20230124193320673-1.png" alt="image-20230124193320673" style="zoom:33%;" />

> 修改 `i` 的原始值，`o` 所引用的堆上的值不会改变。对 `o` 的操作也不影响该原始值。

## 拆箱

由于 `object` 类型的变量可以引用值的装箱拷贝，所以通过该变量也应该能获取装箱的值。

我希望你不要写出这种~~RL都写不出来~~的代码：

```C#
int i = o;
```

很显然，稍微动动脑子就能知道上面的语法是不正确的，因为 `o` 可能引用任何东西，而不仅仅是一个 `int`。为了访问已装箱的值，我们必须进行**强制类型转换**，简称**转型**。这个操作会先检查是否能将一种类型安全地转换成另一种类型，然后才执行转换。

为了进行转型，我们需要在 `object` 变量前添加一对圆括号，并输入类型名称，如下例：

```C#
int i = 42;
object o = i; // 装箱
i = (int)o; // 可通过编译
```

转型的过程需要稍微解释一下。编译器发现指定了类型 `int`，所以会在运行时生成代码检查 `o` 到底是个什么玩意。如果不是 `int`，就抛出异常；如果真的是 `int`，转型就会成功执行，编译器生成的代码就会从装箱的 `int` 中提取出值。这个过程称为**拆箱**。

<img src="https://fs49.org/wp-content/uploads/2023/01/image-20230124194227199.png" alt="image-20230124194227199" style="zoom:33%;" />

如果 `o` 引用的东西和转型目标不同，就会出现类型不匹配的情况，造成转型失败，抛出 `InvalidCastException`。

下面是拆箱失败的例子：

```C#
Circle c = new Circle(42);
object o = c; // 无需装箱，因为Circle是引用类型
int i = (int)o; // 可编译，但在程序运行时抛出异常
```

<img src="https://fs49.org/wp-content/uploads/2023/01/image-20230124194507418.png" alt="image-20230124194507418" style="zoom:33%;" />

注意，装箱和拆箱都会产生较大的开销，因为它们涉及不少的检查工作，而且需要分配额外的堆内存。滥用装箱拆箱将严重影响性能。关于一项与装箱异曲同工的技术——泛型，将在第17章中详细介绍。

# 数据的安全转型

强制类型转换是一厢情愿地指定一个对象引用的数据具有某种类型，而且可以用那种类型“安全地”引用对象。这里的关键词是“一厢情愿”。C#变压器中生成应用程序时只能相信你的判断。但运行时对此持怀疑态度，并通过检查来加以确认。为了尽量避免复杂的异常处理工作，C#提供了两个有用的操作符来让你~~死得更体面~~更方便地进行安全的类型转换——`is` 和 `as`。

## is操作符

用 `is` 操作符可以验证对象的类型是不是自己所期望的，如下所示：

```C#
object o = new Circle(42); // 这是合法的哦，创建的Circle对象直接被o引用
//...
if (o is Circle)
{
    Circle temp = (Circle)o;
    //...
}
```

`is` 操作符取两个操作数，左边是对象引用，右边是类型名称。如果左边的对象**是**右边的类型，则该表达式求值为 `true`，反之为 `false`。因此上面的代码能确定转型一定成功。

在新版本的C#中引入了许多关于 `is` 的语法糖，比如下面这种爽到飞起的写法：

```C#
if (o is Circle temp) // 转换，声明，引用，一气呵成！
{
    Console.WriteLine(temp.Area());
    //...
}
```

 以及很多奇妙的模式匹配，你可以自行查询[相关文档](https://learn.microsoft.com/zh-cn/dotnet/csharp/fundamentals/tutorials/safely-cast-using-pattern-matching-is-and-as-operators)了解。

## as操作符

`as` 充当了和 `is` 类似的角色，只是~~更逊~~功能被弱化了。当转型不成功时，返回 `null`。如下例：

```C#
object o = new Circle(42);
//...
Circle c = o as Circle;
if (c != null)
{
    //...
}
```

后面的章节中我们还会进一步讨论这两种操作符。



本章的内容较多且具有一定理解难度，请务必反复品味哦~
