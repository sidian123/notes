# 介绍

反射能够使你在运行时检查类、接口、字段、泛型、注解、方法和方法参数的信息，不用在编译时知道类、方法的名字。也能通过反射实例化类，调用方法和设置、访问字段值。在jdk8中，甚至可以获得**参数名**。

类的信息由Class类提供，然后可以通过Class对象获得类的所有信息，包括类名、类修饰符、包名、父类、被实现的接口、构造函数、方法、字段和注解。

很多信息都由java.lang.reflect包中对应的类来提供，比如Array对应数组，可以动态创建数组，访问数组；Constructor对应构造函数，可以获得构造函数的信息，创建对象；Field对应字段，这些字段可以来源于类和接口，可以访问和设置字段的值；Method对应方法，可以获得方法的信息，调用方法；Parameter对应参数。这些类都差不多，都可以获得相关的信息，也可以获得之上的注解信息。

# 获得Class对象

使用反射的第一步就是获得Class对象。如果在编译时已经知道了类型信息，可以直接通过类名获得Class对象：

```java
Class myObjectClass = MyObject.class
```

如果编译时不知道，可以在运行时通过完整的类名来获得Class对象：

```java
Class aClass=Class.forName("packageName.ClassName");
//比如Class.forName("com.xxx.MyClass");
```

对于数组的Class对象的获取也可以通过上述两种方法：

```java
	public static void main(String[] arg) throws ClassNotFoundException  {
		Class class1=int[].class;
		Class class2=String[].class;
		Class class3=Class.forName("[I");
		Class class4=Class.forName("[Ljava.lang.String;");
		System.out.println(class1.getName());
		System.out.println(class2.getName());
		System.out.println(class3.getName());
		System.out.println(class4.getName());
	}
```

结果：

```
[I
[Ljava.lang.String;
[I
[Ljava.lang.String;
```

在使用Class.forName时注意写法，有关内容请参考：[java中数组是对象吗？](https://blog.csdn.net/jdbdh/article/details/82117952)[Java Reflection - Arrays](http://tutorials.jenkov.com/java-reflection/arrays.html)

获得Class对象后可以获得很多的信息，也可以实例化对象，但是只会调用无参构造函数。

```
App app=App.class.newInstance();
```

# 反射构造函数

获得Class之后就可以反射构造函数，得到Constructor类，通过这个类就可以实例化其他的构造函数。

```java
public class App 
{
	public static void main(String[] arg) throws ClassNotFoundException, NoSuchMethodException, SecurityException, InstantiationException, IllegalAccessException, IllegalArgumentException, InvocationTargetException  {
		Constructor<?> constructor=C.class.getConstructor(int.class); 
		constructor.newInstance(3);
	}
}
class C{
	public C(){System.out.println("C()");}
	public C(int c){System.out.println("C("+c+")");}
}
```

只有public构造函数才能通过getConstructor来获取。但是可以通过getDeclaredConstructor函数访问其他访问权限的构造函数：

```java
public class App 
{
	public static void main(String[] arg) throws ClassNotFoundException, NoSuchMethodException, SecurityException, InstantiationException, IllegalAccessException, IllegalArgumentException, InvocationTargetException  {
        //通过getDeclared...可以获得非public的构造函数
		Constructor<?> constructor=C.class.getDeclaredConstructor(int.class); 
        //关闭访问权限检查，因此可以通过私有构造函数创建对象
		constructor.setAccessible(true);
        //通过私有构造函数创建对象
		constructor.newInstance(3);
	}
}
class C{
	public C(){System.out.println("C()");}
	private C(int c){System.out.println("C("+c+")");}
}
```

# 反射方法

同样，可Constructor类似，通过Class对象获得Method对象，然后通过Method对象可以获得相关的信息，和调用方法。通过getMethod只能获得public类型的方法，通过setAccessible可以使之调用该私有方法。下面直接给出代码：

```java
public class App 
{
	public static void main(String[] arg) throws NoSuchMethodException, SecurityException, IllegalAccessException, IllegalArgumentException, InvocationTargetException{
		Method method=App.class.getMethod("method", null);
		method.invoke(new App(), null);
		
		Method method2=App.class.getDeclaredMethod("method1", int.class);
		//不进行访问权限检查
		method2.setAccessible(true);
		method2.invoke(new App(), 3333);
	}

    public void method() {
    	System.out.println("call method");
    }
    private void method1(int a) {
    	System.out.println("call private method1("+a+")");
    }
}
```

输出：

```
call method
call private method1(3333)
```

# 反射字段

可以获得字段的信息，即使是私有的也可以获得，在mybatis和spring框架中都可以获得私有字段。

与上述类似，反射就是这么简单：

```java
public class App 
{
	public List<String> afield; 
	A<String,Double> a;
	public static void main(String[] arg) throws Exception{
		//创建一个对象，通过Field可以访问字段
		App app=new App();
		app.afield=new ArrayList<>();
		//获得字段对应的Field
		Field field=App.class.getField("afield");
		//通过field添加数据
		((List<String>)field.get(app)).add("aaaa");
		//打印数据验证
		System.out.println(Arrays.toString(app.afield.toArray()));
		//获得该字段的名字
		System.out.println(field.getName());
		//获得该地段的泛型信息
		ParameterizedType parameterizedType=(ParameterizedType)field.getGenericType();
		//打印类型参数的类型
		System.out.println(parameterizedType.getActualTypeArguments()[0].getTypeName());
	}
｝
```

输出：

```
[aaaa]
afield
java.lang.String
```

也许会问，泛型信息不是在编译后被擦除了吗？怎么还能获取泛型信息？其实运行时确实还是保存着泛型信息，下面介绍。

# 反射泛型

很多文章说，编译时会将泛型信息擦除。其实，这并不完全正确，运行时，在类的字段、方法的参数和返回类型中仍然保留泛型，但是你不能通过被引用的对象来获取泛型信息。下面给出具体代码：

```java
public class App 
{
	A<String,Double> a;
	public static void main(String[] arg) throws Exception{
		//因为非public，因此使用getDeclare...方法获得字段
		Field field=App.class.getDeclaredField("a");
        //获得泛型类型信息
		ParameterizedType parameterizedType=(ParameterizedType)field.getGenericType();
        //获得泛型中每个类型参数
		Type[] types=parameterizedType.getActualTypeArguments();
        //打印所有类型参数的名字
		for(Type t:types) {
			Class c=(Class)t;
			System.out.println(c.getName());
		}
        //打印字段的声明，包含泛型声明
		System.out.println(field.toGenericString());
	}
}
class A<T,Y>{
	
}
```

输出：

```
java.lang.String
java.lang.Double
com.luo.my.test.A<java.lang.String, java.lang.Double> com.luo.my.test.App.a
```

# 反射注解

神奇，通过注解我们也可以获得注解信息，但是这要求注解必须在运行时存在（也就是被@Retention(RetentionPolicy.RUNTIME注解）。同样获取注解有两种方法：getAnnotation和getDeclaredAnnotation。如果你查看java.lang.reflect.AccessibleObject的源码，可以发现，其实他们没啥区别，，，就是说使用哪个都行。下面给出发射注解的代码：

```java
public class App 
{
	public static void main(String[] arg) throws Exception{
		Annotation annotation=A.class.getAnnotation(MyAnnotation.class);
		MyAnnotation myAnnotation=(MyAnnotation)annotation;
		System.out.println(myAnnotation.name());
	}
}
@MyAnnotation(name="A")
class A<T,Y>{
}
@Retention(RetentionPolicy.RUNTIME)
@interface MyAnnotation{
	public String name();
}
```

# 反射参数

Parameter代表参数，可以获得参数类型、参数上的注解等信息。我们知道，代码编译生成的字节码是不包含方法名的信息，因此jdk7之前不能反射方法名，甚至jdk7连Parameter这个类也没有。但是jdk8后，有了Parameter类，它的**getName**方法可以获得方法名，但是获得的是合成的参数名argN，因为字节码中还是没有参数名信息。然而jdk8后编译器也提供了**-parameter**来将参数名编译到字节码中，因此可以获得正确的参数名。

```java
//javac编译时加上参数-parameter才能得到真正的方法名。
Class<String> clz = String.class;
for (Method m : clz.getDeclaredMethods()) {
   System.err.println(m.getName());
   for (Parameter p : m.getParameters()) {
      System.err.println("  " + p.getName());
   }
}
```

在jdk8之前没有Parameter，也就不能通过Parameter来获得方法名，但是可以通过ASM 或BCEL来获得参数名，即使使用默认的编译设置（通过在默认设置编译源码时添加了debug信息来获得参数名的，具体如何实现，不晓滴）。而jdk8后可以通过Parameter来获得参数名，但是需要设置-parameter参数，然而有人提供了[Paranamer](https://github.com/paul-hammant/paranamer)这个库来在默认设置下获取方法名。据说实现是通过注解来实现的，注解可以影响编译行为，在编译时加入一个静态方法来获得参数名。即使我说的不对，然而也可以解释了为什么springMVC可以不通过注解就获得控制器参数名的原因了。关于参数名，可参考：[Java 8 Named Method Parameters](https://www.beyondjava.net/reading-java-8-method-parameter-named-reflection)

# 参考

* http://tutorials.jenkov.com/java-reflection/index.html

* 参数名：https://www.beyondjava.net/reading-java-8-method-parameter-named-reflection

* https://stackoverflow.com/questions/21455403/how-to-get-method-parameter-names-in-java-8-using-reflection