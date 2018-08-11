## 第8章 运算符和类型强制转换 ##
[上一章][chapter-07] | [目录][readme] | [下一章][chapter-09]

### 运算符和类型转换 ###

### 运算符 ###
- 索引运算符(用于数组和索引器): `[]`  
- 委托连接和删除运算符*: `+` `-`  
- 类型信息运行符: `sizeof` `is` `typeof` `as`  
- 溢出异常控制运算符: `checked{}` `unchecked{}`  
- 间接寻址运算符*: `[]`  
- 名称空间别名限定符: `::`  
- 空合并运算符: `??`  
- 空值传播运算符: `?.` `?[]`  
- 标识符的名称运算符: `nameof()`(C#6)  
> 有4个运算符(`sizeof`、`*`、`->`、`&`)只能用于不安全的代码(无C#的类型安全性检查)

* `typeof`运算符返回一个表示特定类型的`System.Type`对象，例如`typeof(string)`返回表示System.String类型的`Type`对象，能方便的用于反射技术动态查找对象信息。
* 可空类型和运算符  
  ```cs
  int? n = null; // Nullable<int> n = null;
  int? i = n + 5; // i is null
  n > 0 ? true : false; // false
  n <= 0 ? true : false; // false
  ```
  不能因为一个条件是`false`,就认为其对立面就是`true`  
  不能随意合并表达式中的可空类型和非可空类型  
* 空合并运算符  
  ```cs
  null ?? 10; // 10
  3 ?? 10; // 3
  ```
* 空值传播运算符(C#6的一个杰出新功能)  
  ```cs
  p?.FirstName; // p==null ? p : p.FirstName
  p?.Age ?? 0; // p==null || p.Age == null ? 0 : p.Age
  p?.Address?.City; // p!=null && p.Address!=null ? p.Address.City : null
  arr?[0] ?? 0; // arr==null || arr[0]==null ? 0 : arr[0]
  ```
* 运算符的优化级  
 1. 基本运算符: `()` `.` `[]` `x++` `x--` `new` `typeof` `sizeof` `checked` `unchecked`
 2. 一元运算符: `+` `-` `!` `~` `++x` `--x` 数据类型强制转换
 3. 乘除运算符: `*` `/` `%`
 4. 加减运算符: `+` `-`
 5. 移位运算符: `<<` `>>`
 6. 关系运算符: `<` `>` `<=` `>=` `is` `as`
 7. 比较运算符: `==` `!=`
 8. 按位ADD运算符: `&`
 9. 按位XOR运算符: `^`
 10. 按位OR运算符: `|`
 11. 条件AND运算符: `&&`
 12. 条件OR运算符: `||`
 13. 空合并运算符: `??`
 14. 条件运算符: `?:`
 15. 赋值运算符和lambda: `=` `+=` `-=` `*=` `/=` `%=` `&=` `|=` `^=` `<<=` `>>=` `>>>=` `=>`
* 运算符的关联性  
除了赋值运算符和条件运算符是右关联外，其它所有二元运算符都是左关联。  
  ```cs
  //左关联
  x + y + z // (x + y) + z
  //右关联
  x = y = z // x = ( y = z)
  a ? b : c ? d : e // a ? b : ( c ? d : e)
  ```
### 类型的安全性 ###
* 隐式转换
	```cs
	byte byte1 = 10;
	byte byte2 = 23;
	byte byte3 = byte1 + byte2; // Cannot implicitly convert type 'int' to 'byte'
	byte byte4 = byte(byte1 + byte2); //显式转换
	int int1 = byte1 + byte2; //无转换  byte1+byte2为int
	int int2 = byte1; //隐式转换(从小宽度到大宽度)
	float float1 = int2; //隐式转换(从int到float是等宽转换但精度变低)
	int int3 = float1; //隐式转换(从float到int是等宽转换同等但精度变低)
	uint uint1 = 35;
	int int4 = uint1; //将无符号变量转为有符号时，只要在范围之内即可
	float? nafloat1 = 180802.2259;
	int? naint1 = nafloat1; //可空类型同样可隐式转换为等宽或更宽其它可空类型
	int? naint2 = int1; //非可空类型同样可隐式转换为等宽或更宽其它可空类型
  int int5 = naint1; //Error: 可空类型不能隐式地转换为非可空类型
	```
* 显式转换
会丢失数据的转换，或可空转非空，只能进行显式转换
  ```cs
  long val = 3000000000; //0xB2D05E00 3000000000
  int i1 = (int)val; //0xB2D05E00 -1294967296
  int i2 = checked((int)val); //checked发现溢出会抛出溢出异常
  WriteLine((char)43); //"+"
  string s = i1.ToString(); //"-1294967296"
  int i3 = int.Parse(s);  //-1294967296
  int.Parse('hello'); //抛出异常
  ```
* 装箱和拆箱
  ```cs
  long l = 333333423; //0x 13DE 43AF
  object o = (object)l;
  (int)o; //8字节数据箱拆放到4字节上,空间不足,会抛出InvalidCastException异常
  ```

### 比较对象的相等性 ###
对象相等判断，取决于比较的是引用类型(类实例)还是值类型(基本数据类型、结构或枚举实例)
* 比较引用类型的相等性
  `System.Object`定义了3个方法及`==`用于比较相等性。
  1. `ReferenceEquals(x, y)`方法
  ```cs
  SomeCalss x = new SomeCalss(), y = new SomeCalss();
  bool B1 = ReferenceEquals(null, null); //return true;
  bool B2 = ReferenceEquals(null, x); //return false;
  bool B3 = ReferenceEquals(x, y); //return false because x and y point to different objects
  ```
  2. `x.Equals(y)`虚方法
  可在自己类中重写从而按值来比较对象。
  3. 静态`Equals(x, y)`方法
  如果参数存在`null`则抛出异常  
  重载版本首先要检查参数, 如参数都是`null`则返回`true`，一个`null`则返回`false`，无`null`则调用`x.Equals(y)`
  4. 比较运算符`==`
  最好将`==`看作严格的值比较和严格的引用比较之间的中间选项。  
  如果类可以看作值则最好重写比较运算符以执行值的比较(例如`System.String`的`==`是重写的)。  
* 比较值类型的相等性
  与引用类型比较规则相同，主要区别是值类型需要先装箱转为引用。  
  1. `ReferenceEquals(x, y)`方法对于值类型永远返回false, 即使以`(x, x)`参数调用也如此(各自装箱会得到不同引用)，所以无意义。
  2. `System.ValueType`类中已经重载了实例方法`x.Equals(y)`，如值类型包含引用类型数据就需要重写。
  3. `==`: 一般不能对自己的结构重载此运算符，`x==y`会导致编译错误，除非提供了此运算符的重载版本。

### 运算符重载 ###
* 运算符的工作方式
  编译器会根据参与运算的数据查找最匹配的运算符重载方法。  
  ```cs
    struct Vector {
      public Vector(double x, double y, double z) { X = x; Y = y; Z = z;}
      public Vector(Vector v) { X = v.X; Y = v.Y; Z = v.Z;}
      public double X { get; }
      public double Y { get; }
      public double Z { get; }
      public override string ToString() => $"( {X}, {Y}, {Z})";
      public static Vector operator +(Vector left, Vector right) =>
        new Vector(left.X + right.X, left.Y + right.Y, left.Z + right.Z);
      public static Vector operator *(double left, Vector right) =>
        new Vector(left * right.X, left * right.Y, left * right.Z);
      public static Vector operator *(Vector left, double right) =>
        right * left;
      public static double operator *(Vector left, Vector right) =>
        left.X * right.X + left.Y * right.Y + left.Z * right.Z);
    }
  ```
  * C#要求所有的运算符重载都声明为`public`和`static`
  * `+=`实际对应的操作为相加和赋值两步，C#不允许重载`=`运算符
  * 如果重载了`+`运算符，编译器会自动使用`+`来执行`+=`，其它`-=`、`*=`等等类似
* 比较运算符的重载
  C#有6个比较运算符，分为3对: `==`和`!=`，`>`和`<`，`>=`和`<=`。  
  他们必须成对重载，且要同时重写`Equals`、`GetHashCode`，否则编译器会报错。  
  他们必须返回布尔值。  
  ```cs
    public static bool operator ==(Vector left, Vector right) {
      if (object.ReferenceEquals(left, right)) return true;
      return left.X == right.X && left.Y == right.Y && left.Z == right.Z;
    }
    public static bool operator !=(Vector left, Vector right) => !(left == right);
    public override bool Equals(object ogj) {
      if(obj == null) return false;
      return this == (Vector)obj;
    }
    public override int GetHashCode() =>
      X.GetHashCode() + (Y.GetHashCode() << 4) + (Z.GetHashCode() << 8);
  ```
  对于值类型，也应该实现接口`IEquatable<T>`————`Equals`方法的一个强类型化版本，由基类`Object`定义
  ```cs
    public bool Equals(Vector other) => this == other;
  ```
* 可以重载的运算符  
  1. 算术二元运算符: `+` `*` `/` `-` `%`
  2. 算术一元运算符: `+` `-` `++` `--`
  3. 按位二元运算符: `&` `|` `^` `<<` `>>`
  4. 按位一元运算符: `!` `~` `true` `false`  
    `true`和`false`运算符必须成对重载
  5. 比较运算符: `==` `!=` `>=` `<` `<=` `>`  
    必须成对重载
  6. 赋值运算符: `+=` `-=` `*=` `/=` `>>=` `<<=` `%=` `&=` `|=` `^=`  
    单个运算符重载后会被整个隐式重写
  7. 索引运算符: `[]`  
    不能直接重载，第2章介绍的索引器成员类型允许在类和结构上支持索引运算符
  8. 类型强制转换运算符: `()`  
    不能直接重载。允许自定义转换行为

### 实现自定义的索引运算符 ###
  ```cs
  public class PersonCollection {
    private Person[] _people;
    public PersonCollection(params Person[] people) {
      _people = people.ToArray();
    }
    public Person this[int index] {
      get { return _people[index]; }
      set { _people[index] = value; }
    }
    public IEnumerable<Persion> this[DateTime birthDay] {
      get { return _people.Where(p => p.BirthDay == birthDay); }
    }
    //C#6:
    public IEnumerable<Persion> this[DateTime birthDay] =>
      _people.Where(p => p.BirthDay == birthDay);
  }
  static void Main() {
    var coll = new PersonCollection(new Person("Ayrton", "Senna", new DateTime(1960, 3, 2)), ...);
    WriteLine(coll[2]);
    foreach(var r in coll[new DateTime(1960, 3, 2)]) {
      WriteLine(r);
    }
  }
  ```

### 实现用户定义的类型强制转换 ###
  ```cs
  int i = 3;
  long l = i; //implicit 安全无损的转换——隐式强制转换
  short s = (short)i; //explict 可能失败或损失的转换——显式强制转换
  ```
  定义类型强制转换的语法同于重载运算符，必须同时声明为`public`和`static`(与C++实例成员用法不同)
  ```cs
  public static implicit operator float (Currency value) { ... }
  ```
* 实现用户定义的类型强制转换
  ```cs
  public struct Currency {
    public uint Dollars { get; }
    public ushort Cents { get; }
    public Currency(uint dollars, ushort cents) {
      Dollars = dollars; Cents = cents;
    }
    public override string ToString() => $"${Dollars}.{Cents,-2:00}";
  }
  ```
  ```cs
  public static implicit operator float (Currency value) =>
    value.Dollars + (value.Cents/100.0f);
  float f = new Currency(10, 50); // 10.50
  ```
  ```cs
  public static explict operator Currency (float value) {
    uint dollars = (uint)value;
    ushort cents = (ushort)((value-dollars)*100);
    return new Currency(dollars, cents);
  }
  var c1 = (Currency)50.35; //50.34 转换精度问题导致变样
  var c2 = (Currency)(-50.50);
  //c2.dollars: -50=>0xFFFFFFCE=>4294967246
  //c2.cents: ((-50-4294967246)*100)&0xffff => (-429496729600)&0xffff => 0xFFFFFF9C00000000&0xffff => 0x0000 => 0
  //超范围未抛异常
  ```
  以上两个问题纠正如下
  ```cs
  public static explict operator Currency (float value) {
    cheched { //修正"超范围无异常"问题
      uint dollars = (uint)value;
      ushort cents = Convert.ToUInt16((value-dollars)*100); //修正"转换精度"及"超范围无异常"问题
      return new Currency(dollars, cents);
    }
  }
  1. 类之间的类型强制转换
    * 基类子类间不能定义类型强制转换(编译器已经提供)
    * 类型强制转换必须在源数据类型或目标数据类型的内部定义
    对于C->B->A, D->B->A，唯一合法的自定义类型强制转换是兄弟C、D之间的转换，且只能在C或D的内部定义(为防止第三方把类型强制转换引入类中)
  2. 基类和派生类之间的类型强制转换
    编译器已经提供了基类和派生类之间的强制转换(是基于引用的转换)。
    ```cs
    MyBase derived = new MyDerived();
    MyBase base = new MyBase();
    MyDerived derived1 = (MyDerived) derived; //OK
    MyDerived derived2 = (MyDerived) base; //Throws exception
    ```
    要将基类强制转换为派生类，可定义一个派生类的构造函数，以基类实例为参数
    ```cs
    class DerivedClass: BaseClass {
      public DevivedClass(BaseClass base) { ... }
    }
    ```
    3. 装箱和拆箱类型强制转换
      装箱是一种隐式的强制转换，拆箱是一种显式的强制转换。  
      装箱和拆箱都是把数据复制到新装箱或拆箱的对象上，不会影响原始值类型的内容。  
* 多重类型强制转换
  若在数据类型转换时没有可用的直接转换方式，C#编译器就会寻找一种间接组合的转换。

[上一章][chapter-07] | [目录][readme] | [下一章][chapter-09]

  [readme]: readme.md
  [chapter-07]: chapter-07.md "第7章 数组和元组"
  [chapter-08]: chapter-08.md "第8章 运算符和类型强制转换"
  [chapter-09]: chapter-09.md "第9章 委托、lambda表达式和事件"
