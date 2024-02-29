# ref_out_in

在方法进行参数传递时，我们可以使用ref、out、in关键字对参数进行修饰。当参数使用ref、out、in修饰后，参数则会按引用传递，而非按值传递。

## in(默认)

in 用来修饰方法的参数，表示这个参数不能被改变。

例如:
```c#
public static void test(in int x) {
    x = 1;
}
```

此时就会报错 因方法参数中的 x 是只读变量。

## out

out 用来修饰方法的参数，表示这个参数不可读，但是方法中可以写。有时候我们可能需要一个方法返回多个值，虽然我们可以使用ref来完成，在方法中修改即可，但是提供了一个属性类型，关键字为out。保证每一个被传出的变量都被赋值。

例如:
```c#
public static void test(out int x) {
    int m = x;
}
```

此时会报错，错误“因使用了未赋值的参数x”。因为out 让 x 并不需要在声明时赋值，但是可以赋值，因此 参数x 有没有被赋值是不清楚的，因此在方法中是不可读的。

## ref

就相当于引用传递，方法中可以读写。

例如:
```c#
public static void test(ref int x) {
    x = 100;
}

public static void Main(String[] args) {
    int x = 1;
    test(x);
    Console.Write($"{x}");
}
```

最终输出`100`。

