## 第7章 数组和元组 ##
[上一章][chapter-06] | [目录][readme] | [下一章][chapter-08]

### 同一类型和不同类型的多个对象 ###
C#使用特殊记号声明、初始化和使用数组  
Array类在后台发挥作用，提供了方法对元素进行排序、过滤  

### 简单数组 ###
  ```cs
    //声明
    int[] myArray;
    //初始化
    myArray = new int[4];
    //一同声明初始化
    int[] myArray = new int[4];
    int[] myArray = new int[] { 4, 7, 11, 2};
    int[] myArray = {4, 7, 11, 2};
  ```
   在指定了数组的大小后，如不复制元素，就不能重新设置大小。如事先不能确定大小，就可以使用集合。
* 访问数组元素
  ```cs
    int v1 = myArray[0];
    myArray[3] = 44;
    for (int i = 0; i < myArray.Length; i++) WriteLine(myArray[i]);
    foreach (var val in myArray) WriteLine(val); //此语句利用了IEnumerable和IEnumerator接口, 从第一个索引遍历数组直到最后一个索引
  ```
* 使用引用类型
  ```cs
  Person[] myPersons = new Person[2];
  myPersons[0] = new Person { FirstName = "Ayrton", LastName = "Senna"};
  ...
  //简化语法
  Person[] myPersons = {
    new Person { FirstName = "Ayrton", LastName = "Senna"},
    ...
  }
  ```

### 多维数组 ###
  ```cs
  //数组声明后不能修改阶数
  int[,] twodim = new int[3,3];
  twodim[0,0] = 1;
  ...
  //使用数组初始化器，必须初始化所有元素，不能有任何遗漏
  int[,] twodim = { {1,2,3}, {4,5,6}, {7,8,9} };
  int[,,] threedim = { {{1,2},{3,4}}, {{5,6},{7,8}}, {{9,10},{11,12}} };
  ```

### 锯齿数组 ###
  ```cs
  int[][] jagged = new int[3][];
  jagged[0] = new int[2] {1, 2};
  ...
  jagged[2] = new int[3] = {9, 10, 11};
  for (int row = 0; row < jagged.Length; row++) {
    for (int elem = 0; elem < jagged[row].Length; elem++) {
      WriteLine($"row:{row}, elem: {elem}, value:{jagged[row][elem]}");
    }
  }
  ```

### `Array`类 ###
用方括号声明数组是C#中使用`Array`类的表示法，都会创建一个派生自抽象基类的`Array`的新类。  
`Array`类除了`Length`属性外，还有表示海量数组长度的`LongLength`属性，表示维数的`Rank`属性。  
* 创建数组
除了C#方括号语法外，还可以使用静态方法`CreateInstance()`创建数组，在未知元素类型时非常有用。
```cs
  Array intArray1 = Array.CreateInstance(typeof(int), 5);
  for (int i = 0; i < 5; i++) { intArray1.SetValue(33, i); }
  for (int i = 0; i < 5; i++) { WriteLine(intArray1.GetValue(i)); }
  //还可以做以下强制转换
  int[] intArray2 = (int[])intArray1;
  //还可以创建多维和不基于0的数组
  int[] lengths = {2, 3}; //准备2x3的二维数组
  int[] lowerBounds = {1, 10}; //第一维基于1， 第二维基于10
  Array racers = Array.CreateInstance(typeof(Person), lengths, lowerBounds);
  racers.SetValue(new Person { FirstName = "Alain", LastName = "Prost"}, 1, 10);
  ...
  racers.SetValue(new Person { FirstName = "Jenson", LastName = "Button"}, 2, 12);
  //只要不超出边界，也可以用C#表示法
  Person[,] racers2 = (Person[,])racers;
  Pereson frist = racer2[1,10];
  Pereson last = racer2[2,12];
```
* 复制数组
数组是引用类型，要真正复制数组，有两个方法:
 * 利用`Clone()`方法(源于`ICloneable`接口)创建并返回数组的浅表副本。  
 * 利用`Array.Copy()`静态方法从一个数组复制到另个数组，两者需同阶且有足够元素。  
如果要做数组的深度复制，必须迭代数组并创建新对象  
* 排序
`Array.Sort()`方法(使用Quicksort算法)可对元素进行排序，但需要元素实现`IComparable`接口，或另带一个基于`IComparer`接口的对象的参数，也或另带一个委托参数
```cs
  //简单类型`System.String`和`System.Int32`已实现`IComparable`接口
  string[] names = {"abc","123", "ABC"};
  Array.Sort(names);
  //自定义类型，实现IComparable接口后，也能使用`Array.Sort()`方法对该类型的数组排序
  public class Person: IComparable<Person> {
    public int CompareTo(Person other) {
      if(other == null) return 1;
      int result = string.Compare(this.LastName, other.LastName);
      return result ? result : string.Compare(this.FirstName, other.FirstName);
    }
    //...
  }
  Person[] persons = {...};
  Array.Sort(persons);
  //另建实现`IComparer`接口的类来辅助排序
  public class PersonFirstNameComparer: IComparer<Person> {
    public int Compare(Person x, Person y) {
      if(x==null && y==null)return 0;
      if(x==null)return 1;
      if(y==null)return -1;
      return string.Compare(x.FirstName, y.FirstName);
    }
  }
  Array.Soft(persons, new PersonFirstNameComparer());
```

### 数组作为参数 ###
方法要返回一个数组，则要把数组声明为返回类型:  
```cs
  static Person[] GetPersons() { return new Person[] { ... }; }
```
方法要带数组参数，应把数组声明为参数:  
```cs
  static void DisplayPersons(Person[] persons) { ... }
```
* 数组协变
```cs
  object[] data = new Person[3];
```
* `ArraySegment<T>`
数组的部分段引用
```cs
  int[] ar = {1, 4, 5, 11, 13, 18};
  var segment = new ArraySegment<int>(ar, 1, 3);// 4, 5, 11
  int sum = 0;
  foreach(int = segment.Offset; i < segment.Offset + segment.Count; i++){
    sum += segment.Array[i];
  }
```

### 枚举 ###
在`foreach`语句中使用了一个枚举器(`IEnumerator`接口)迭代集合中的元素，且无须知道元素个数。  
Client --`IEnumerator.GetEnumerator()`-- Enumberator --`IEnumberable`-- Collection  
* `IEnumerator`接口
  * `Current`属性返回光标所在元素
  * `bool MoveNext()`方法后移光标，返回是否存在
  * `Reset()`方法与COM交互(许多.NET枚举器以抛出`NotSupportedException`类型异常实现)
* `foreach`语句
C#编译器会把`foreach`语句转换为`IEnumerator`接口的方法属性:
```cs
foreach (var p in persons) {
  ...
}
```
```cs
IEnumerator<Person> enumerator = person.GetEnumerator();
while (enumerator.MoveNext()) {
  Person p = enumerator.Current;
  ...
}
```
* `yield`语句
C# 2.0添加了`yield`语句创建枚举器，`yeild return`返回集合的一个元素并跳到下个元素，`yield break`停止迭代  
包括`yield`语句的方法或属性称为迭代块，它必须声明为返回`IEnumerator`或`IEnumberable`接口或其泛型版本，它可包含多条`yeild return`或`yield break`语句，但不能包含`return`语句  
```cs
  public class  HelloCollection {
    public IEnumerator<string> GetEnumerator() {
      yield return "Hello";
      yield return "World";
    }
  }
  //再可以用foreach语句迭代集体了:
  var helloCollection = new HelloCollection();
  foreach(var s in helloCollection) { WriteLine(s); }
```
使用迭代块，编译器会生成一个`yield`类型，其中包含一个状态机，并实现`IEnumerator`和`IDisposable`接口
```cs
  public class HelloCollection {
    public IEnumerator GetEnumerator() => new Enumberator(0);
    public class Enumberator: IEnumerator<string>, IEnumerator, IDisposable {
      private int _state;
      private string _current;
      public Enumberator(int state) { _state = state; };
      bool System.Collections.IEnumerator.MoveNext() {
        switch(state) {
          case 0: _current = "Hello"; _state = 1; return true;
          case 1: _current = "World"; _state = 2; return true;
          case 2: break;
        }
        return false;
      }
      void System.Collections.IEnumerator.Reset() {
        throw new NotSupportedException();
      }
      string System.Collections.Generic.IEnumerator<string>.Current => current;
      object System.Collections.IEnumerator.Current => current;
      void IDisposable.Dispose() { }
    }
  }
```
  1. 迭代集合的不同方式
  ```cs
    public class MusicTitles {
      string[] names = {"a", "b", "c"};
      public IEnumerator<string> GetEnumerator() {
        for (int i = 0; i < 4) yield return names[i];
      }
      public IEnumerator<string> Reverse() {
        for (int i = 3; i >=0; i--) yield return names[i];
      }
      public IEnumerator<string> Subset(int index, int length) {
        for (int i = index; i < index + length; i++) yield return names[i];
      }
    }
    var title = new MusicTitles();
    foreach (var title in titles) { ... } //省去了默认方法名GetEnumerator
    foreach (var title in titles.Reverse()) { ... }
    foreach (var title in titles.Subset(2,2)) { ... }
  ```
  2. 用`yield return`返回枚举器
  ```cs
    public class GameMoves {
      private IEnumerator _cross;
      private IEnumerator _circle;
      public GameMoves() { _cross = Cross(); _circle = Circle(); }
      private int _move = 0;
      public IEnumerator Cross() {
        while (true) {
          WriteLine($"Cross move {_move}");
          if(++_move >= 4) {
            yield break;  
          }
          yield return _circle;
        }
      }
      public IEnumerator Cirle() { ... } //与Cross类似, 返回_cross
    }
    var game = new GameMoves();
    IEnumerator enumerator = game.Cross();
    while (enumerator.MoveNext()) {
      enumerator = enumerator.Current as IEnumerator;
    }
    ```
    程序的输出会交替显示信息
    > Cross, move 0  
    > Circle, move 1    
    > Cross, move 2  
    > Circle, move 3  

### 元组 ###
数组合并了相同类型对象，而元组合并了不同类型对象。  
.NET Framework定义了8个泛型`Tuple`类和一个静态`Tuple`类，用作元组的工厂。  
```cs
  public static Tuple<int, int> Divide(int dividend, int divisor) {
    return Tuple.Create(dividend/divisor, dividend%divisor);
  }
  var result = Divide(5, 2);
  WriteLine($"result of division: {result.Item1}, remainder: {result.Item2}");
```
如果元组包含的项超过8个，就必须使用嵌套实现，将第8个模板参数声明为元组`Tuple`类，这样就可以创建带任意个参数的元组了。
```cs
  //public class Tuple<T1, T2, T3, T4, T5, T6, T7, TRest>
  var tuple = Tuple.Create<string, string, string, int, int, int, double,
      Tuple<int, int>>("a", "b", "c", 18, 7, 29, 17.4,
        Tuple.Create<int, int>(2018, 729));
```

### 结构比较 ###
数组和元组都实现接口`IStructuralEquatable`和`IStructuralComparable`用于比较引用和内容，前者用于比较是否有相同内容，后者用于排序。  
```cs
  public class Person: IEquatable<Person> {
    ...
    public override int GetHashCode() => Id.GetHashCode();
    public override bool Equals(object obj) {
      return object == null ? base.Equals(obj) : Equals(obj as Person);
    }
    public bool Equals(Person other) {
      return object == null ? base.Equals(other) : FirstName == other.FirstName && ...;
    }
  }
  Person[] persons1 = { ... };
  Person[] persons2 = { ... };
  //即使内容相同，但实质引用了不同数组，且`Array`类未重写带一个参数的`Equals()`方法，故下式成立
  if(persons1 != persons2){ WriteLine("not the smae reference"); } //ok
```
`IStructuralEquatable`接口定义的`Equals()`方法有两个参数：`object`类型，`IEqualityComparer`类型。通过`EqualityComparer<T>`类完成`IEqualityComparer`的一个默认实现：检查该类型是否实现`IEquatable.Equals()`，如实现则调用，否则就调用`object`基类中的`Equals()`方法:
```cs
  if((persons1 as IStructuralEquatable).Equals(persons2, EqualityComparer<Person>.Default)){ WriteLine("the same content"); }
```
元组例子
```cs
  var t1 = Tuple.Create(1, "abc");
  var t2 = Tuple.Create(1, "abc");
  if( t1 != t2) { WriteLine("not the same reference to the tuple"); } //ok
  if(t1.Equals(t2)){ WriteLine("the same cotent"); } //ok
```
还可以用类`TupleComparer`创建一个自定义`IEqualityComparer`
```cs
  calss TupleComparer: IEqualityComparer {
    //实现Equals()方法需要new修饰符或隐式实现的接口，因为基类object也定义了两参数的静态Equals()方法
    public new bool Equals(object x, object y) => x.Equals(y);
    public int GetHashCode(object obj) => obj.GetHashCode();
  }
  //Tuple类的Equals()为要比较的每一项调用TupleComparer.Equals()，所以对于Tuple<T1,T2>要调用两次TupleComparer以检查所有项是否相等
  if(t1.Equals(t2, new TupleComparer())) {
    WriteLine("equals using TupleComparer");
  }
```
### 小结 ###
创建和使用简单数组、多维数组、锯齿数组的C#表示法。  
使用`IComparable`、`IComparer`接口给数组中的元素排序。  
使用和创建枚举器、`IEnumberable`、`IEnumerator`接口，以及`yield`语句。  
数组、元组  

[上一章][chapter-06] | [目录][readme] | [下一章][chapter-08]

  [readme]: readme.md
  [chapter-06]: chapter-06.md "第6章 泛型"
  [chapter-07]: chapter-07.md "第7章 数组和元组"
  [chapter-08]: chapter-08.md "第8章 运算符和类型强制转换"
