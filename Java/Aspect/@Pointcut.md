# @Pointcut 切入表达式

标准的AspectJ Aop 的pointcut的表达式类型是很丰富的，但是Spring Aop只支持其中的9种，外加Spring Aop自己扩充的一种：
1. `execution`:一般用于指定方法的执行。即指定方法进行增强。
2. `within`:指定某些类型的全部方法执行，也可以指定一个包。
3. `this`:Spring Aop 是基于动态代理的，生成的bean也是一个代理对象，this就是这个代理对象，当这个对象可以转换为指定的类型时，对应的切入点就是它了，Spring Aop将生效。
4. `target`:当被代理的对象可以转换为指定的类型时，对应的切入点就是它。
5. `args`:当执行的方法的参数是指定类型时生效。
6. `@target`:当代理的目标对象拥有指定的注解时生效。
7. `@args`:当执行的方法参数类型上拥有指定的注解时生效。
8. `@within`:与`@target`类似，`@within`只需要目标对象或者父类上有指定的注解，则`@within`会生效，而`@target`则是必须是目标对象的类上有指定的注解。实际使用两者都是只要目标或父类上有指定的注解即可。
9. `@annotation`:当执行的方法上拥有指定的注解时生效。
10. `reference pointcut`:表示引用其他命名切入点，只有@AspectJ风格支持，Schema风格不支持。
11. `bean`:当调用的方法是指定的bean的方法时生效（Spring Aop自己扩展支持）

Pointcut 定义时，还可以使用 &&,||,!三个运算，进行逻辑运算，可以把各种条件组合起来使用。

AspectJ切入点支持的切入点指示符还有：call、get、set、preinitialization、staticinitialization、initialization、handler、adviceexecution、withincode、cflow、cflowbelow、if、@this、@withincode

# 使用示例

## execution

execution是使用最多的一种Pointcut表达式，表示某个方法的执行，其标准语法如下：
```js
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern?)
```

- 修饰符匹配(modifiers-pattern?)
- 返回值匹配(ret-type-pattern) 可以为*表示任何返回值，全路径的类名等
- 类路径匹配(declaring-type-pattern?)
- 方法名匹配(name-pattern) 可以指定方法名 或者 代表所有，set代表以set开头的所有方法
- 参数匹配(param-pattern) 可以指定具体的参数类型，多个参数间隔用","隔开，各个参数也可以用""来表示匹配任意类型的参数，如(String)表示匹配一个String类型参数的方法，(,String)表示匹配两个参数的方法，第一个参数可以是任意类型，而第二个参数是String类型，可以用(..)表示零个或多个任意参数
- 异常类型匹配(throws-pattern?)
- 其中后面跟着"?"的是可选项

例子：
```js
// 表示匹配所有方法
1) execution(* *(..))
// 表示匹配com.fsx.run.UserService中所有的公共方法
2) execution(public * com.fsx.run.UserService.*(..))
// 表示匹配com.fsx.run包及其子包下的所有方法
3) execution(* com.fsx.run..*.*(..))
```

```js
// 签名：消息发送切面
@Pointcut("execution(* com.fsx.run.MessageSender.*(..))")
private void logSender(){}
// 签名：消息接收切面
@Pointcut("execution(* com.fsx.run.MessageReceiver.*(..))")
private void logReceiver(){}
// 只有满足发送  或者  接收  这个切面都会切进去
@Pointcut("logSender() || logReceiver()")
private void logMessage(){}
```
