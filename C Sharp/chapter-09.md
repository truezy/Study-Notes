## 第9章 委托、lambda表达式和事件 ##
[上一章][chapter-08] | [目录][readme] | [下一章][chapter-10]

### 引用方法 ###
* 在C++中函数指针只是一个指向内存的指针，非类型安全，无法判断实际指向，像参数和返回类型等项就更无从知晓了。  
* .NET版本的寻址方法——委托，是类型安全的类，它定义了返回类型和参数的类型，可以包含对一个或多个方法的引用。
* lambda表达式与委托直接相关，当参数是委托类型时，就可以使用lambda表达式实现委托引用的方法。

### 委托 ###
在.NET要传递方法，必须把方法的细节封装在一种新的对象类型中，即委托。  
委托是一种特殊类型的对象，只能包含一个或多个方法的地址。  
* 声明委托
  ```cs
  delegate void IntMethodInvoker(int x);
  delegate double TwoLongsOp(long first, long second);
  ```
  定义一个委托就是定义一个新类。可以在定义类的任何相同地方定义委托。根据定义的可见性和作用域，可以冠上`public`、`private`、`protected`等。  
* 使用委托
  ```cs
  delegate string GetAString();
  int x = 40;
  GetAString firstMethod = new GetAString(x.ToString);
  firstMethod(); //=>firstMethod.Invoke()=>x.ToString()=>"40"
  ```
* 简单的委托示例
  ```cs
  class MathOp {
    public static double x2(double value) => value * 2;
    public static double xs(double value) => value * value;
  }
  delegate double DoubleOp(double x);
  DoubleOp[] operations = { MathOp.x2, MathOp.xx };
  operations[0](18.0811);
  operations[1](22.41);
  ```
* `Action<T>`和`Func<T>`委托  
  * 泛型`Action<T>`委托表示引用一个`void`返回类型的方法，可以传递至多16种不同的参数类型。  
    无参数即`Action`  
    1个参数`Action<in T>`  
    8个参数`Action<in T1, in T2, in T3, in T4, in T5, in T6, in T7, in T8>`。    
  * 泛型`Func<T>`比`Action<T>`多一个返回类型定义。  
    无参数即`Func<out TResult>`  
    4个参数`Func<in T1, in T2, in T3, in T4, outTResult>`。  
  ```cs
  //用`Func<double, double>`替代`delegate double DoubleOp(double x)`:
  Func<double, double>[] operations = { MathOp.x2, MathOp.xx };
  ```
* `BubbleSorter`示例  



[上一章][chapter-08] | [目录][readme] | [下一章][chapter-10]

  [readme]: readme.md
  [chapter-08]: chapter-08.md "第8章 运算符和类型强制转换"
  [chapter-09]: chapter-09.md "第9章 委托、lambda表达式和事件"
  [chapter-10]: chapter-10.md "第10章 字符串和正则表达式"
