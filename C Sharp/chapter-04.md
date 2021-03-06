## 第4章 继承 ##
[上一章][chapter-03] | [目录][readme] | [下一章][chapter-05]

### 继承 ###
面向对象的三个最重要的概念是继承、封装和多态性。

### 继承的类型 ###
* 多重继承: C#不支持类的多重继承，但支持接口的多重继承
* 结构和类: 结构是值类型，类是引用类型，结构不支持继承，但结构可以实现接口
  * 结构总是直接派生自`Sytem.ValueType`，或同时派生自任意多个接口
  * 类总是直接或间接派生自`System.Object`或其它类，或同时派生自任意多个接口

### 实现继承 ###  
如果类和接口都用于派生，则类总是必须放在接口的前面。  
如果在类定义中未指定基类，则默认`System.Object`是基类。  
* 虚方法: `virtual`修饰，就可在任何派生类中通过`override`修饰重写(避免C++中签名类似时重载的不确定性)
  ```cs
  public virtual void Draw(){}
  ```
* 虚属性
  ```cs
  public virtual Size Size{get;set;}
  ```
* 多态性
* 隐藏方法
  在派生类中直接覆盖基类非虚同名函数，编译会警告，在方法声明前冠上`new`关键字可避免警告
  此时在派生类中要调用基类方法可用`base.`做限定语
* 抽象类和抽象方法: `abstract`修饰。
  抽象类不能实例化，抽象方法不能直接实现，必须在非抽象派生类中重写。
  抽象方法本身是虚拟的，不可再加`virtual`关键字。
* 密封类和密封方法: `sealed`修饰。
  密封类不可派生，密封方法不可重写。
* 派生类的构造函数
  当基类无默认构造有非默认构造时，派生类需用构造函数初始化器初始化调用基类构造。
  ```cs
    public abstract class Shape {
      public Sharp(int width, int height, int x, int y) {
        Size = new Size { Width = width, Height = height };
        Postion = new Postion { X = x, Y = y };
      }
      public Postion Postion { get; }
      public Size Size { get; }
    }
    public class Rectangle : Shape {
      public Rectangle() : base(width:0, height:0, x:0, y:0){
          //
      }
    }
  ```  

### 修饰符 ###
* 访问修饰符: `public`, `protected`, `internal`, `private`, `protected internal`
* 其它修饰符: `new`, `static`, `virtual`, `abstract`, `override`, `sealed`, `extern`

### 接口 ###
* 接口不能有任何实现代码，一般只能包含方法、属性、索引器和事件的声明。
* 接口永远不能实例化，其成员总是抽象的，不需`abstract`关键字
* 接口成员不允许带修饰符，其成员总是隐式为`public`，也不能声明为`virtual`。
* 接口可派生
  ```cs
    public interface IBankAccount {
      void PayIn(decimal amount);
      bool Withdraw(decimal amount);
      decimal Balance {get;}
    }
    class SaveAccount: IBankAccount {
      private decimal _balance;
      public void PayIn(decimal amount) => _balance += amount;
      public bool Withdraw(decimal amount){
        if(_balance >= amount){
          _balance -= amount;
          return true;
        }
        WriteLine("Withdraw attempt failed.");
        return false;
      }
      public decimal Balance => _balance;
      public override string ToString() =>
        $"Venus Bank Saver: Balance = {_balance,6:C}";
    }
    public interface ITransferBankAccount: IBankAccount {
      bool TransferTo(IBankAccount dest, decimal amount);
    }
    class CurrentAccount: ITransferBankAccount {
      ...
      public override string ToString() =>
        $"Jupiter Bank Current Account: Balance = {_balance,6:C}";
      public bool TransferTo(IBankAccount dest, decimal amount) {
        bool result = Withdraw(amount);
        if(result) { dest.PayIn(amount); }
        return result;
      }
    }
    class Program {
      static void Main() {
        IBankAccount venusAccount = new SaveAccount();
        ITransferBankAccount jupiterAccount = new CurrentAccount();
        venusAccount.PayIn(200);
        jupiterAccount.PayIn(500);
        jupiterAccount.TransferTo(venusAccount, 100);
        WriteLine(venusAccount.ToString());
        //output: Venus Bank Saver: Balance = $100.00
        WriteLine(jupiterAccount.ToString());
        //output: Jupiter Bank Current Account: Balance = $400.00
      }
    }
  ```

### `is` 和 `as` 运算符 ###
  ```cs
    public void XXX(object o) {
      //
      try {
        IBankAccount account1 = (IBankAccount)o;
      }
      catch(InvalidCastException){ }
      //as
      IBankAccount account2 = o as IBankAccount;
      if(account2 != null){ }
      //is
      if(o is IBankAccount) {
        IBankAccount account3 = (IBankAccount)o;
      }
    }
  ```

[上一章][chapter-03] | [目录][readme] | [下一章][chapter-05]

  [readme]: readme.md
  [chapter-03]: chapter-03.md "第3章 对象和类型"
  [chapter-04]: chapter-04.md "第4章 继承"
  [chapter-05]: chapter-05.md "第5章 托管和非托管的资源"
