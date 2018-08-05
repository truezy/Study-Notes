## 第5章 托管和非托管的资源 ##
[上一章][chapter-04] | [目录][readme] | [下一章][chapter-06]

### 资源 ###
垃圾收集器(GC)只释放托管资源(托管堆中的对象)，不释放非托管资源(本地堆中的对象)。

### 后台内存管理 ###
32位处理器上的每个进程都可以可以使用4GB的内存(虚拟地址空间),其中有一个区域称为栈。
* 值数据类型
栈存储非对象成员的值数据类型及调用方法时的参数副本。
栈存储是由高内存地址向低内存地址填充，当数据入栈(出栈)，栈指针随之递增(递减)。
栈中的变量先入后出，生存期是嵌套的。
* 引用数据类型
引用数据存放在堆中，其内存是由低向高分配的。
可有多个变量(栈)引用堆中同一对象，当变量超过作用域时会从栈中移除，但对象直到程序终止或不被任何变量引用时被GC删除。
* 垃圾回收
托管堆会在GC删除对象后对剩余对象进行下移压缩。
一般情况GC由.NET运行库决定运行，但也可以调用`System.GC.Collect()`方法强迫GC运行。
堆的第一部分称第0代始终存放最新对象，后部分为第1代、第2代
 * 为避免大对象压缩影响性能，大于85000字节的对象，会放在一个特殊的堆上，称做大对象堆。
 * 垃圾回收的平衡，专用于服务器。
    一个堆用尽了内存会触发垃圾回收，所有其它堆也会得益于此；如果线程所用内存差异过大会导致垃圾回收触发时部分线程无需处理，不太高效。
平衡这些堆————小对象堆和大对象堆，可以减少不必要的回收。
 * 为了利用包含大量内在的硬件，垃圾回收过程添加了`GCSettings.LatencyMode`属性，可以控制垃圾回收方式，其GCLatencyMode枚举如下：
 `Batch`,`Interactive`,`LowLatency`,`SustainedLowLatency`,`NoGCregion`
 `LowLatency`或`NoGCregion`设置使用的时间应为最小值，分配的内存量应尽可能小，否则可能出现溢出内存错误。

### 强引用和弱引用 ###
```cs
  var myCache = new MyCache();
  var myClassVariable = new MyClass();
  myCache.Add(myClassVariable);
  myClassVariable = null;
```
此时`myClassVariable`引用内存仍在`myCache`中引用，并不能被GC回收，此类引用很容易忘记。
使用由`WeakReference`类创建的弱引用，可以避免上述情况。
```cs
  var myWeakReference = new WeakReference(new DataObject());
  if(myWeakReference.IsAlive){
    DataObject = strongReference = myWeakReference.Target as DataObject;
    if(strongReference != null){
      //use the strongReference
    }
  }
  else {
    // reference not available
  }
```
### 处理非托管的资源 ###
垃圾回收器不知如何释放非托管资源：例如文件句柄、网络连接、数据库连接。
托管类在封装对非托管资源的直接或间接引用时，需要制定专门的规则，以确保正常释放。
* 析构函数（或终结器）
C#会将析构函数隐式地编译为等价于重写`Finalize()`方法的代码，以确保执行父类的`Finalize()`方法。
C#中析构函数较少使用，原因有：
 * GC的工作方式导致无法确定它何时执行
 * 其执行要多占用一次GC机会导致延迟回收——一次析构一次删除。
 另外，运行库使用一个线程来执行所有对象的`Finalize()`方法，如果频繁使用析构或长时执行清理任务，会显著影响性能。
* `IDisposable`接口
在C#中，推荐使用此接口替代析构，它为释放非托管资源提供了确定的机制，并能避免析构与GC相关的问题。
它声明了一个`void Dispose()`方法
```cs
  class MyClass: IDisposable {
    public void Dispose() {
      //implementation
    }
  }
  var theInstance = new MyClass();
  //do your processing
  theInstance.Dispose();
```
* `using`语句
使用`try`/`finally`可以保证`theInstance`的处理出现异常也能调用`Dispose()`。
另外`using`语法可以实现`IDisposable`接口对象引用超出作用域时自动调用`Dispose()`:
```cs
  using (var theInstance = new ResourceGobbler()) {
    //do your processing
  }
```
* 实现`IDisposable`接口和析构函数
以防没有调用`Dispose()`,下面是一个利用析构的双重实现例子:
```cs
  using System;
  public class ResourceHolder: IDisposable {
    private bool _isDisposed = false;
    public void Dispose() {
      Dispose(true);
      GC.SuppressFinalize(this); //告诉GC此类不需调用析构了
    }
    protected virtual void Dispose(bool disposing) {
      if(_isDisposed == false){
        if(disposing){
          // Cleanup managed objects by calling their Dispose() methods.
        }
        // Cleanup unmanaged objects
      }
      _isDisposed = true;
    }
    ~ResourceHolder() {
      Dispose(false);
    }
    public void SomeMethod() {
      // Ensure object not already disposed before execution of any method
      if(_isDisposed) {
        throw new ObjectDisposeException("ResourceHolder");
      }
      // method implementation...
    }
  }
```
* `IDisposable`和终结器的规则
 * 若类定义了实现`IDisposable`的成员，则该类也应该实现`IDisposable`
 * 实现`IDisposable`不一定要实现终结器。终结器会带来额外的开销，要释放本地资源，才需要终结器
 * 若实现了终结器，也应该实现`IDisposable`接口，以保证本机资源尽早释放。
 * 终结器的实现中，不能访问已终结的对象，其执行顺序无保证。
 * 若一个对象实现了`IDisposable`接口，就在不需要对象时调用`Dispose()`；若在方法中使用此对象，推荐使用更方便的`using`语句；若对象是类一个成员，就让类也实现`IDisposable`

### 不安全的代码 ###
 C#因使用了GC和引用，故隐藏了大部分基本内存管理，但有时也需要直接访问内存
* 用指针直接访问内存
  1. 用`unsafe`关键字编写不安全的代码
  在类和结构定义前、方法前、成员前、代码块前添加`unsafe`标记
  在编译命令中带上/unsafe选项，或在项目设置中把Build配置指定为Allow Unsafe Code
  满足以上两点，才可在内部使用指针
  2. 指针的语法(基本同C++)
  3. 将指针强制转换为整数类型
  `checked`关键字不能用于涉及指针的转换。
  4. 指针类型之间的强制转换
  5. `void`指针
  6. 指针算术的运算
  7. `sizeof`运算符
  8. 结构指针: 指针成员访问运算符
  9. 类成员指针
  对象存储在堆上，GC过程中会移动对象，因此编译器不允许把托管类型的成员地址分配给指针
  但可使用`fixed`关键字告诉GC有成员被指针引用暂不可移动对象
  ```cs
    class MyClass {
      public long L;
      public float F1;
      public float F2;
    }
    var myObject = new MyClass();
    long* pL = &(myObject.X); //wrong -- compilation error
    fixed (long* pL = &(myObject.L))
    fixed (float* pF1 = &(myObject.F1), pF2 = &(myObject.F2))
    //前面可放置一条或多条fixed语句, 也可一条fixed语句中初始多个类型相同的变量
    {
      // do something
    }
  ```
* 指针示例: PointerPlayground
  栈向下给变量分配内存，且栈中的内存块总是按照4个字节的倍数进行分配。
* 使用指针优化性能
  1. 创建基于栈的数组
  ```cs
    double* pDouble = stackalloc double[10];
    int size = 20;
    decimal* pDecimals = stackalloc decimal[size];
    pDecimals[100] = 32; //ok
    decimal myDecimalArray = new decimal[size];
    myDecimalArray[100] = 32; //throw exception
  ```
  2. `QuickArray`示例
  ```cs
    using static System.Console;
    namespace QuickArray {
      public class Program {
        Write("How big an array to you want?\n>");
        string userInput = ReadLine();
        uint size = uint.Parse(userInput);
        long* pArray = stackalloc long[(int)size];
        for (int i = 0; i < size; i++) {
          pArray[i] = i * i;
          WriteLine($"Element {i} = {*(pArray + i)}");
        }
      }
    }
    /*
    How big an array to you want?
    > 3
    Element 0 = 0
    Element 1 = 1
    Element 2 = 4
    */
  ```

### 平台调用 ###
  并非Windows API调用的所有特性都适用于.Net。
  要重用一个非托管库，不含COM只含导出的功能，就可使用平台调用(P/Invoke)。
  ```cpp
  BOOL CreateHardLink(LPCTSTR lpFileName, LPCTSTR lpExistingFileName, LPSECURITY_ATTRIBUTES lpSecurityAttributes);
  ```
  ```cs
  using System;
  using System.CompoentMode;
  using System.IO;
  using System.Runtime.InteropServices;
  using System.Security;
  using System.Security.Permissions;
  namespace Wrox.ProCSharp.Interop {
    [SecurityCritical]
    internal static class NativeMethods {
      [DllImport("kernel32.dll", SetLastError="true",
        EntryPoint="CreateHardLink", CharSet=CharSet.Unicode)]
      [return: MarshalAs(UnmanagedType.Bool)]
      private static extern bool CreateHardLink (
        [In, MarshalAs(UnmanagedType.LPWStr)] string newFileName,
        [In, MarshalAs(UnmanagedType.LPWStr)] string existingFileName,
        IntPtr securityAttributes);

      internal static void CreateHardLink(string oldFileName, string newFileName) {
        if(!CreateHardLink(newFileName, oldFileName, IntPtr.Zero)){
          var ex = new Win32Exception(Marshal.GetLastWin32Error());
          throw new IOexception(ex.Message, ex);
        }
      }
    }
    public static class FileUtility {
      [FileIOPermission(SecurityAction.LinkDemand, Unrestricted = true)]
      public static void CreateHardLink(string oldFileName, string newFileName) {
        NativeMethods.CreateHardLink(oldFile, newFileName);
      }
    }
  }
  ```
  ```cs
  using PInvokeSampleLib;
  using System.IO;
  using static System.Console;
  namespace PInvokeSample {
    public class Program {
      public static void Main(string[] args) {
        if(args.Length != 2) {
          WriteLine("usage: PInvokeSample existingFileName newFileName");
          return;
        }
        try {
          FileUtility.CreateHardLink(args[0], args[1]);
        }
        catch (IOexception ex) {
          WriteLine(ex.Message);
        }
      }
    }
  }
  ```
  ### 小结 ###

[上一章][chapter-04] | [目录][readme] | [下一章][chapter-06]

  [readme]: readme.md
  [chapter-04]: chapter-04.md "第4章 继承"
  [chapter-05]: chapter-05.md "第5章 托管和非托管的资源"
  [chapter-06]: chapter-06.md "第6章 泛型"
