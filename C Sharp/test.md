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
        ...
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
