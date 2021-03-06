## java中的异常

### 继承关系
Exception和Error都继承自Throwable

### throw和catch
throw出Throwable类型的对象（包括子类）

catch到Throwable类型的对象（包括子类）

```
try {
	throw new Error();
} catch (Exception e) {
	e.printStackTrace();
}catch ( Error  e) {
	System.out.println( "q" );
	e.printStackTrace();
}
```

### finally

不管 try 语句块正常结束还是异常结束，finally语句块都是要被执行的

在try语句块或catch语句块中执行到System.exit(0)直接退出程序

finally块中的return语句会覆盖try块中的return返回

finally 语句块在 catch语句块中的return语句之前执行

### 多catch捕获顺序

从上到下以此catch，类似于switch，遇到匹配的类型 执行，下面的catch不会去判断和执行。


### Java中有的方法必须抛出异常，有的不用

Java异常有Runtime（运行时异常）和Checked（编译时异常），其中，所有RuntimeException类及其子类的实例被称为Runtime异常，不是RuntimeException类及其子类的异常实例都被称为Checked异常

Java中除了RuntimeException及其任何子类，其他异常类都被Java异常强制处理机制强制异常处理。这些被强制异常处理的必须进行异常处理。否则就会提示 Unhandled exception type Exception 错误警告。而提示这个错误信息的方法，一定是这个方法所在的类用throws抛出了异常，所以在使用这个方法时候必须处理这个异常。

```
public void a(){
	// NullPointException 继承自 RuntimeException
	throw new NullPointException();
}
//下面这段代码编译器报错,可以使用throws处理活着try...catch处理
public void b(){
	throw new Exception();
}
```