# 第五章 - 泛型

Java 1.5中添加了泛型特性。在没有泛型之前，从集合类中读取的对象必须进行显式转换，如果对集合插入不同类型对象，则在显式转换时可能带来运行时异常，如：

```
List strings = new ArrayList();  // a List of String without generics
strings.add("hello");
strings.add(123);

for (int i = 0; i < strings.size(); ++i) {
	System.out.println("element " + i + ": " + (String) strings.get(i));  // throw ClassCastException in runtime
}
```

以上代码因尝试将`Integer`直接转换为`String`，在运行时会抛出`java.lang.ClassCastException`。泛型出现后，在创建集合类时可以指定其元素类型，这样既能避免运行时异常，又能免去显式类型转换。本章具体来看如何在降低复杂度的前提下最大化泛型带来的好处。

## [23]. 不要使用原始类型

先来看一个概念，类／接口在声明时使用一个或多个*类型参数*进行修饰，称这种类／接口为泛型类／泛型接口，如引入泛型后的`java.util.List`：

```
public interface List<E> extends Collection<E> { ... }
```

尽量使用泛型类／接口，而不要使用原始类型，以保证类型安全。如：

```
class Stamp { ... }
class Coin {...}

List<Stamp> stamps = new ArrayList<>();
stamps.add(new Stamp());  // correct
stamps.add(new Coin());  // compile error
```

向元素类型声明为`Stamp`的`List`中添加非`Stamp`类型的元素会直接导致编译异常，可以避免异常发生在运行时。同时，使用`for-each`循环时也可以保证类型安全，且不需要显式类型转换：

```
for (Stamp stamp : stamps) { ... }  // typesafe & no need to cast
```

由于需要向后兼容泛型特性添加前的代码，泛型并不强制使用，但作为使用者应该强制自己使用。运行时类型安全、无需显式转换等都是泛型带来都好处。由于Java中的泛型使用类型擦出实现，因此两种情况存在例外，需要使用原始类型：

1. 类型属性上需要使用原始类型

`List.class`，`String[].class`，`int.class`都正确的使用了原始类型，不能使用泛型方式，即`List<String>.class`，`List<?>.class`是错误的。

2. 使用`instanceof`关键字进行类型判断

由于运行时类型擦出，在使用`instanceof`进行集合类型判断时使用泛型并没有意义，此时可直接使用原始类型：
```
if (o instanceof Set) { ... }
```

## [24]. 消除未检查的警告(unchecked warnings)

使用泛型时，经常会看到类似`unchecked warnings`这种提示信息。每个提示信息都表示可能在运行时抛出`java.lang.ClassCastException`异常，因此应该对每条提示信息进行检查。

如果可以通过使用泛型消除提示信息，则引入泛型，如：

```
List<String> strings = new ArrayList();  // unchecked warnings
// use generic, eliminate unchecked warnings
List<String> strings = new ArrayList<>();  // warnings eliminated
```

如果无法通过使用泛型进行消除，则在人工检查后，使用`@SuppressWarnings("checked")`注解消除警告信息。需要注意的是，`@SuppressWarnings`注解可在局部变量、方法、类等任何粒度使用，在使用时需要保证最小粒度，以避免覆盖未检查过的警告信息。如下代码含有未检查的警告：

```
public <T> T[] toArray(T[] a) {
	if (a.length < size)
		return (T[]) Arrays.copyOf(elements, size, a.getClass());  // unchecked warnings
	System.arraycopy(elements, 0, a, 0, size);
	if (a.length > size)
		a[size] = null;
	return a;
}
```

经检查该转换类型安全，使用局部变量粒度`@SuppressWarning`消除警告，并添加注释信息，解释为什么类型安全：

```
public <T> T[] toArray(T[] a) {
	if (a.length < size)
		// This cast is correct because the array we're creating
		// is of the same type as the one passed in, which is T[].
		@SuppressWarnings("unchecked")
		T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());  // checked class cast
		return result;
	System.arraycopy(elements, 0, a, 0, size);
	if (a.length > size)
		a[size] = null;
	return a;
}
```

## [25]. List优于Array



## [26]. 使用泛型类型


## [27]. 使用泛型方法


## [28]. 使用有界通配符增强API灵活性


## [29]. 使用类型安全异构容器（？）


```
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);


// wrong version
public static void swap(List<?> list, int i, int j) {
	list.set(i, list.set(j, list.get(i)));
}

// correct version
public static void swap(List<?> list, int i, int j) {
	swapHelper(list, i, j);
}

private static <E> void swapHelper(List<E> list, int i, int j) {
	list.set(i, list.set(j, list.get(i)));
}
```




`typesafe heterogeneous container`

```
public class Favorities {
	private Map<Class<?>, Object> favorities = new HashMap<>();

	public <T> void putFavorite(Class<T> type, T instance) {
		if (type == null)
			throw new NullPointerException("null type");
		favorities.put(type, instance);
	}

	public <T> T getFavorite(Class<T> type) {
		return type.cast(favorities.get(type));
	}

}
```