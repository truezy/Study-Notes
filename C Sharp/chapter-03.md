## 第3章 对象和类型 ##
[上一章][chapter-02] | [目录][readme] | [下一章][chapter-04]

### 类属性 ###
```cs
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
### 类方法 ###
```cs
public void xx(param int[] data){
    foreach(var in data){
    }
}
```
readonly 修饰，只能在构造中set  
### 只读成员 ###
```cs
x {get;} = 123; //C#6
string x => $"{a}{b}"; //表达式属性
var x = new {a = "A", b = "B"}; //匿名类型，结构全等才可赋值
```
### 可空类型 ###
```cs
int ? x = null;
x.HasValue ? x.Value : -1;
x ?? -1;
```
### 结构 ###
结构是值类型，总是派生于System.ValueTypes  

### 函数参数 ###
out, ref  

### 枚举 ###
```cs
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
### 部分类 ###
partial  

### 扩展方法 ###
```cs
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
### Object类 ###
`System.Object` 是所有类的基类  
对于结构，这个派生是间接的：结构总是派生于`System.ValueTypes`，`System.ValueTypes`又派生于`System.Object`  
#### 成员方法:  
* `ToString()`方法: 可实现`IFormattable`接口自定义输出格式  
* `GetHasCode()`方法  
* `Equals()` 和 `ReferenceEquals()`方法  
* `Finalize()`方法  
* `MemberwiseClone()`方法  
* `GetType()`方法: 返回`System.Type`派生实例  

[上一章][chapter-02] | [目录][readme] | [下一章][chapter-04]

  [readme]: readme.md
  [chapter-02]: chapter-02.md "第2章 核心C#"
  [chapter-03]: chapter-03.md "第3章 对象和类型"
  [chapter-04]: chapter-04.md "第4章 继承"
