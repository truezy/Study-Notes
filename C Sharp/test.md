* 类属性  
```c#
public int Age {get;set}
public int Age {get;set} = 42;
public string Name {
    private set {
        _name = value;
    }
    get {
        return _name;
    }
}
public string Name {get; private set;}
```
* 类方法  
```c#
public void xx(param int[] data){
    foreach(var in data){
    }
}
```
* 只读成员  
readonly 修饰，只能在构造中set  
```c#
x {get;} = 123; //C#6
string x => $"{a}{b} //表达式属性
var x = new {a = "A", b = "B"} //匿名类型，结构全等才可赋值
```
* 可空类型  
```c#
int ? x = null;
x.HasValue ? x.Value : -1;
x ?? -1;
```
* 结构  
结构是值类型，总是派生于System.ValueTypes  

* 函数参数  
out, ref  

* 枚举  
```c#
public enum Color : short {
    Red = 1, Green = 2, Blue = 3
}
Color c2 = (Color)2;
short number = (short)c2;
Color red;
if(Enum TryParse<Color>("Red", out red)){
    WriteLine($"sucessfully parsed {red}");
}
foreach(var x in Enum.GetNames(typeof(Color))){
    WriteLine(x);
}
//output:Red\r\nGreen\r\nBlue\r\n
foreach(short x in Enum.GetValues(typeof(Color))){
    WriteLine(x);
}
//output:1\r\n2\r\n3\r\n
```
* 部分类  
partial  

* 扩展方法  
```c#
public static class stringExt {
    //使用this关键字和第一个参数来扩展字符串，这个关键字定义了要扩展的类型
    public static int GetWordCount(this string s) =>
        s.split().Length;
}
//即使扩展方法是静态的，也要使用标准的实例方法语法。注意，这里使用fox变量而没有使用类型名来调用GetWordCount()。
string fox = "the quik brown fox jumped over the  lary dogs down";
int wordCount = fox.GetWordCount();
WriteLine($"{wordCount} words");
//在后台，编译器会把它改为调用静态方法:
int wordCount = stringExt::GetWordCount(fox);
```
* Object类  
>System.Object是所有类的基类  
>对于结构，这个派是间接的：结构总是派生于System.ValueTypes，System.ValueTypes又派生于System.Object  
>成员方法:
>* ToString()方法: 可实现IFormattable接口自定义输出格式
>* GetHasCode()方法
>* Equals() 和 ReferenceEquals()方法
>* Finalize()方法
>* GetType()方法: 返回System.Type派生实例
>* MemberwiseClone()方法
