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