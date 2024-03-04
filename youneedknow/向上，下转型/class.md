```c#
public class Animal
{
	public virtual void Call()
	{
		Console.WriteLine("叫");
	}
}

public class Dog : Animal
{
	public override void Call()
	{
		Console.WriteLine("汪");
	}
	public void New()
	{
		Console.WriteLine("Dog");
	}
}
```

# 向上转型

```c#
Animal a = new Dog();
// a.New() 不能调用Animal中没有的方法
a.Call(); // 调用的 是 重写 后的方法
```

# 向下转型

```C#
Animal a = new Dog();
Animal b = new Animal();
Dog d = (Dog)a; // 向下转型 
// Dog q = (Dog)b;  报错
```

# 虚方法(virtual)

`virtual`关键字用于在基类中修饰方法（或属性、索引器或事件声明），并且允许在派生类中重写这些对象（即`override`可写可不写）。

`virtual`的两种使用情况：
1. 在基类中`virtual`方法在子类没用`override`重写，那么在对子类实例的调用中，该虚方法使用的是基类定义的方法。
2. 在基类中`virtual`方法在派生类中使用`override`重写，那么在对子类实例的调用中，该虚方法使用的是子类重写的方法。

# 抽象方法(abstract)

`abstract`关键字**只能用在抽象类中**修饰方法，并且没有具体的实现（没有方法体），抽象方法必须放在抽象类中（但抽象类中可以没有抽象方法）。
非抽象类的派生类必须全部实现父类的抽象方法和抽象访问器，使用`override`关键字来实现。

# 接口(interface)

接口中的方法不需要修饰符，默认就是公有的（Default / Public）。
接口可以包含方法、属性、索引器、事件。不能有实例对象，例如`int i`等。可以有静态方法。


