## 第6章 泛型 ##
[上一章][chapter-05] | [目录][readme] | [下一章][chapter-07]

### 泛型概述 ###
泛型是C#语言的一种结构，是CLR定义的，与IL代码紧密地集成。  
C#泛型比C++模板更高级——无需源代码。  
* 性能
在不使用泛型情况下，有时会自动产生装箱拆箱操作，从而影响性能。  
```cs
  var list = new ArrayList();
  list.Add(44); //boxing - convert a value type to a reference type
  int i1 = (int)list[0]; //unboxing - convert a reference type to a value type
  foreach (int i2 in list) {
    WriteLine(i2); //unboxing
  }
```
使用泛型，能避免自动装箱拆箱操作  
```cs
  var list = new List<int>();
  list.Add(44); //no boxing - value types are stored in the List<int>
  int i1 = list[0]; //no unboxing, no cast needed
  forech (int i2 in list) {
    WriteLine(i2);
  }
```
* 类型安全  
在上述未使用泛型的代码中，允许在列表中任意数据如字符串`list.Add("mystring")`，但接下来会在`foreach`块中会出现运行时异常。  
错误应尽早发现，在上述使用了泛型的代码中，就只能添加整数，添加非整数则通不过编译器。  
* 二进制代码的重用  
泛型允许更好地重用二进制，而无需像C++模板那样访问源代码。  
* 代码的扩展  
用特定类型实例化泛型类不会在IL代码中进行复制，但在JIT编码器中，会给每个值类型创建一个新类(各种值类型长度不一)，会让引用类型共享同一本地类的所有相同实现代码(引用类型是内存地址长度固定)。  
* 命名约定  
  * 泛型类型的名称用字母`T`作为前缀  
  * 如无特殊要求，泛型类型允许用任意类替代，且只使用了一个泛型类型，就可以用字母`T`为名称
  * 如泛型类有特定的要求，或使用了两个或多个泛型类型，就应给泛型类型使用描述性的名称  

### 创建泛型类 ###
```cs
  public class LinkedListNode {
    public LinkedListNode(object value) {
      Value = value;
    }
    public object Value { get; private set; }
    public LinkedListNode Next { get; internal set; }
    public LinkedListNode Prev { get; internal set; }
  }
  public class LinkedList: IEnumerable {
    public LinkedListNode First { get; private set; }
    public LinkedListNode Last { get; private set; }
    public LinkedListNode AddLast(object node) {
      var newNode = new LinkedListNode(node);
      if(First == null) {
        First = newNode;
        Last = First;
      }
      else {
        LinkedListNode prev = Last;
        Last.Next = newNode;
        Last = newNode;
        Last.Prev = prev;
      }
      return newNode;
    }
    public IEnumerator GetEnumerator() {
      LinkedListNode current = First;
      while (current != null) {
        yield return current.Value;
        current = current.Next;
      }
    }
  }
```
使用类如下：
```cs
  var list = new LinkedList();
  list.AddLast(2);
  list.AddLast("6");
  foreach (int i in list) {
    WriteLine(i); //"6" throw Exec
  }
```
将上面代码中的类都用泛型实现(类后全加上`<T>`，`object`改为T)后
```cs
  var list = new LinkedList<int>();
  list.AddLast(2);
  list.AddLast("6"); //编译出错
  foreach (int i in list) {
    WriteLine(i);
  }
```

### 泛型类的功能 ###
* 默认值  
编译器不允许把`null`赋予泛型类型: 原因是泛型类型可实例化为值类型，而`null`只能用于引用类型  
为了解决这问题，可以使用`default`关键字`T x = default(T)`，将`null`赋予引用类型，将`0`赋予值类型  
* 约束  
为了限制泛型类型为实现了某接口的类，可以使用`where`关键字进行约束
```cs
  public class DocumentManger<TDocument>
    where TDocument: IDocument {}
```
泛型共支持以下几种约束:
  * `where T: struct` 结构约束，类型`T`必须是值类型
  * `where T: class` 类约束，类型`T`必须是引用类型
  * `where T: IFoo` 接口约束，类型`T`必须实现接口`IFoo`
  * `where T: Foo` 派生约束，类型`T`必须派生自基类`Foo`
  * `where T: new()` 构造函数约束，类型`T`必须有一个默认构造函数
  * `where T1: T2` 派生约束，类型`T1`派生自泛型类型`T2`
只能为默认构造函数定义约束，不能为其它构造函数定义约束  
使用泛型类型还可以合并多个约束: `where T: IFoo, new()`  
`where`子句中，不能定义必须由泛型类型实现的运算符，只能定义基类、接口和默认构造函数。运算符不能在接口中定义。  
* 继承
```cs
  public class LinkedList<T>: IEnumerable<T> {}
  //
  public Base<T> {}
  public class Derived1<T>: Base<T> {}
  public class Derived2<T>: Base<string> {}
  //
  public abstrcat class Calc<T> {
    public abstrcat T Add(T x, T y);
  }
  public class IntCalc: Calc<int> {
    public override int Add(int x, int y) => x + y;
  }
  //
  public class Query<TRequest, TResult> {}
  public StringQuery<TRequest> : Query<TRequest, string> {}
```
* 静态成员
```cs
  public class StaticDemo<T> {
    public static int x;
  }
  StaticDemo<string>.x = 4;
  StaticDemo<int>.x = 5;
  WriteLine(StaticDemo<string>.x); // writes 4
```
### 泛型接口 ###
* 协变和抗变
* 泛型接口的协变
泛型类型用`out`关键字标注`out T`表示`T`是协变的，意味着返回类型只能是`T`。  
```cs
  public interface IIndex<out T> {
    T this[int index] { get; }
    int Count { get; }
  }
  public class RectangleCollection: IIdex<Rectangle> {
    private Rectangle[] data = new Rectangle[2] {
      new Rectangle { Height = 2, Width = 5 },
      new Rectangle { Height = 4.5, Width = 2.9 }
    };
    private static RectangleCollection _coll;
    public static RectangleCollection GetRectangles() =>
      _coll ?? (coll = new RectangleCollection());
    public Rectangle this[int index] {
      get {
        if (index < 0 || index > data.Length)
          throw new ArgumentOutOfRangeException("index");
        return data[index];
      }
    }
    public int Count => data.Length;
  }
  public static void Main() {
    IIndex<Rectangle> rectangles = RectangleCollection.GetRectangles();
    IIndex<Shape> shapes = rectangles;
    for (int i = 0; i < shapes.Count; i++) {
      WriteLine(shapes[i]);
    }
  }
```
* 泛型接口的抗变
泛型类型用`in`关键字标注`in T`表示`T`是抗变的，意味着类型只是用作其方法的输入。  
```cs
  public interface IDisplay<int T> {
    void Show(T item);
  }
  public class shapeDisplay: IDisplay<Shape> {
    public void Show(Shape s) => WriteLine($"{s.GetType().Name} Width: {s.Width}");
  }
  public static void Main() {
    //...
    IDisplay<Shape> shapeDisplay = new shapeDisplay;
    IDisplay<Rectangle> rectangleDisplay = shapeDisplay; // 因IDisplay<T>是抗变
    rectangleDisplay.Show(rectangles[0]);
  }
```
### 泛型结构 ###
泛型结构非常类似于泛型类，只是没有继承特性。.NET Framework定义了泛型结构`Nullable<T>`  
```cs
  public struct Nullable<T> where T: struct {
    public Nullable(T value) { _hasValue = true; _value = value; }
    private bool _hasValue;
    public bool HasValue => _hasValue;
    private T _value;
    public T Value {
      get {
        if(!_hasValue) throw new InvalidOperationException("no value");
        public static explicit operator T(Nullable<T> value) => _value.Value;
        public static implicit operator Nullable<T>(T value) => new Nullable<T>(value);
        public override string ToString() => !_hasValue ? string.Empty : _value.ToString();
      }
    }
  }
  Nullable<int> x;
  x = 4;
  x += 3;
  if(x.HasValue) {
    int y = x.Value;
  }
  x = null;
```
因可空类型使用非常频繁，C#也提供了一种特殊的语法用于可空类型的变量：
```cs
  Nullable<int> x1;
  int? x2;
  int? x3 = GetNullableType();  //占位符，返回一个可空的int
  if(x3 == null) x
  else if(x3 < 0) x
```
可空类型可与`null`和数字比较及算术运算(任一为`null`算术加结果也为`null`)  
合并运算符`??`能将可空类型转移为非可空类型: `x ?? 0`
### 泛型方法 ###
泛型方法可以在非泛型类中定义
```cs
  void Swap<T>(ref T x, ref T y) {
    T temp;
    temp = x;
    x = y;
    y = temp;
  }
  int i = 4, j = 5;
  Swap<int>(ref i, ref j); //因编译器会通过方法获取参数类型，方法调用的泛型类型<int>可省
```
* 带约束的泛型方法
```cs
public static decimal Accumulate<TAccount>(IEnumerable<TAcount> source)
  where TAccount: IAcount {
    decimal sum = 0;
    foreach (TAccount a in source) {
      sum += s.Balance;
    }
    return sum;
  }
```
* 带委托的泛型方法
```cs
  public static T2 Accumulate<T1, T2>(IEnumerable<T1> source,
      Func<T1, T2, T2> action) {
        T2 sum = default(T2);
        foreach(T1 item in source) {
          sum = action(item, sum);
        }
        return sum;
  }
  decimal amount = Accumulate<Account, decimal>(
      accounts, (item, sum) => sum + item.Balance
  );
```
* 泛型方法规范
若实现了同名的多种泛型方法，编译期间会使用最佳匹配。
### 小结 ###
泛型是CLR中的一个非常重要的特性。  
通过泛型类可以创建独立于类型的类，泛型方法是独立于类型的方法。  
接口、结构和委托也可以用泛型的方式创建。  

[上一章][chapter-05] | [目录][readme] | [下一章][chapter-07]

  [readme]: readme.md
  [chapter-05]: chapter-05.md "第5章 托管和非托管的资源"
  [chapter-06]: chapter-06.md "第6章 泛型"
  [chapter-07]: chapter-07.md "第7章 数组和元组"
