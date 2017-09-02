# 第四章 - 类和接口

类和接口是Java的核心，是抽象的基本单元。本章的目的是让类和接口更通用、健壮和灵活。

## [13]. 赋予类和成员变量最窄的可见范围

区分模块设计是否足够好的一个最重要的因素是模块对外能否隐藏其内部数据和实现细节。设计良好的模块隐藏其内部所有实现细节，将其与API明确的区分开，模块间通过API进行交互，完全不关心对方的实现方式。简言之，设计良好的模块其封装性(encapsulation)较好。

对于非内部类和接口，只有两种可见性：`package private`和`public`，在可能的前提下，优先将可见性设置为`package private`。如果`package private`的顶级类(top-level class)或接口只被一个类使用，应考虑将其修改为使用方的内部类，这样就可以将其可见性从`package private`变为只有其外部类可以使用。

常量字段可见性绝不设置为`public`，否则非`final`修饰的字段或`final`修饰的对可变对象的引用就可能被修改。例外的情况是被`public static final`修饰的字段，但应注意这种字段如果为对象引用，则指向的对象必须为不可变对象。长度非零的数据是可变的，不应直接返回数组，可以通过以下两种方式包装后返回，具体选用哪种方式看具体使用方希望使用的类型：

```
private static final Thing[] PRIVATE_VALUES = {...};  // an Thing array
// 方式一
public static final List<Thing> VALUES () {
	return Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
}
// 方式二
public static final Thing[] values () {
	return PRIVATE_VALUES.clone();
}
```

**总结**：

1. 尽可能降低可见性，无论是class/interface/member/member method
2. 设计对外API后，将与对外无关的类或接口设置为package private
3. 除了`public static final`修饰的字段，`public`修饰的类不应有其余`public`字段
4. 被`public static final`修饰的引用必须指向不可变对象


## [14]. 在public类中使用get方法访问成员变量

**总结**：

1. `public`类中将成员变量可见性设置为`private`，通过`get`等类似方法访问成员变量
2. `public`类中不能暴露可变成员变量
3. `package private`类或`private`内部类通常可以暴露字段，无论字段是否可变，使得字段访问更便利


## [15]. 尽可能降低可变性

不可变类指的是其实例不能被修改的类，实例包含的所有信息在生成实例时确定，且在实例生命周期内不可修改。不可变类有许多优点：更容易设计、实现，不易导致错误，更加安全。

构造不可变类遵循以下5点：

1. 不提供setter(mutators)方法
2. 类不可被继承(final / no default constructor)
3. 所有成员变量设为final(在使用缓存如缓存hashCode计算值时可以存在非final修饰成员变量的情况)
4. 所有成员变量可见性设为private
5. 不返回所有对可变对象的引用(defensive copy,[39]?)

不可变类的一个例子：

```
public final class Complex {
	private final double re;
	private final double im;

	public Complex(double re, double im) {
		this.re = re;
		this.im = im;
	}

	// getters, no setters
	public double realPart() {
		return re;
	}

	public double imaginaryPart() {
		return im;
	}

	public Complex add(Complex c) {
		return new Complex(re + c.re, im + c.im);
	}

	public Complex substract(Complex c) {
		return new Complex(re - c.re, im - c.im);
	}

	@Override
	public boolean equals(Object o) {
		if (o == this)
			return true;
		if (!(o instanceof Complex))
			return false;
		Complex c = (Complex) o;
		return Double.compare(re, c.re) == 0 && Double.compare(im, c.im) == 0;
	}

	private volitile int hashCode;  // cache & reuse, volitile, non final(exception for immutable)
	@Override
	public int hashCode() {
		int result = hashCode;
		if (result == 0) {
			result = 17 + hashDouble(re);
			result = 31 * result + hashDouble(im);
			hashCode = result;
		}
		return result;
	}
	private int hashDouble(double val) {
		long longBits = Double.doubleToLongBits(re);
		return (int) (longBits ^ (longBits >>> 32));
	}
	// omit toString method
```

使类不可继承的另一种方式使将构造函数可见性设为`private`：

```
public class Complex {
	private final double re;
	private final double im;

	private Complex(double re, double im) {
		this.re = re;
		this.im = im;
	}

	public static Complex valueOf(double re, double im) {  // static factory method
		return new Complex(re, im);
	}
}
```

更推荐这种方式使类不可继承，可以通过静态工厂方法做实例初始化时的缓存，优化性能。

不可变类天生线程安全，不需要同步，可以自由共享。基于此可以对不可变类常用实例进行复用，如：

```
// 方式一
public static final Complex ZERO = new Complex(0, 0);
public static final Complex ONE = new Complex(1, 0);

// 方式二
private static final Complex ZERO = new Complex(0, 0);
private static final Complex ONE = new Complex(1, 0);

public static Complex valueOf(double re, double im) {
	if (Double.compare(re, 0) == 0 && Double.compare(im, 0) == 0)
		return ZERO;
	if (Double.compare(re, 1) == 0 && Double.compare(im, 0) == 0)
		return ONE;
	return new Complex(re, im);
}
```
在JDK中如java.lang.Integer中有专门用于缓存小整数的内部类IntegerCache，可以部分弥补性能上的缺陷。另一种方式可以使用内部或外部可变companion class解决性能问题，如java.lang.StringBuilder就是java.lang.String的外部可变companion class。


## [16]. 组合优于继承

这里的继承指的是类继承另一个类，而非类实现接口或接口继承接口。

继承：

```
public class InstrumentedHashSet<E> extends HashSet<E> {

    private int addCount;

    public InstrumentedHashSet() {
        super();
    }

    public InstrumentedHashSet(int initSize, float loadFactor) {
        super(initSize, loadFactor);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```

组合：

```
public class ForwardingSet<E> implements Set<E> {

    private Set<E> set;

    public ForwardingSet(Set<E> set) {
        this.set = set;
    }

    @Override
    public int size() {
        return set.size();
    }

    @Override
    public boolean isEmpty() {
        return set.isEmpty();
    }

    @Override
    public boolean contains(Object o) {
        return set.contains(o);
    }

    @Override
    public Iterator<E> iterator() {
        return set.iterator();
    }

    @Override
    public Object[] toArray() {
        return set.toArray();
    }

    @Override
    public <T> T[] toArray(T[] a) {
        return set.toArray(a);
    }

    @Override
    public boolean add(E e) {
        return set.add(e);
    }

    @Override
    public boolean remove(Object o) {
        return set.remove(o);
    }

    @Override
    public boolean containsAll(Collection<?> c) {
        return set.containsAll(c);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        return set.addAll(c);
    }

    @Override
    public boolean retainAll(Collection<?> c) {
        return false;
    }

    @Override
    public boolean removeAll(Collection<?> c) {
        return set.removeAll(c);
    }

    @Override
    public void clear() {
        set.clear();
    }
}

public class InstrumentedSet<E> extends ForwardingSet<E> {

    private int addCount;

    public InstrumentedSet(Set<E> set) {
        super(set);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```

组合(composition)是装饰器模式的实现原理。有时组合也被称为委派(delegation)，但严格来说委派要求包装类将其自身传入被包装类中(参考java.util.concurrent.Executors#DelegatedExecutorService)。


## [17]. 精心设计并为继承书写Java doc，否则不要使用继承

在使用类间继承时，必须经过精心设计并书写`Java doc`。`Java doc`一般以`This implementation`开头。这会影响设计良好的API的宗旨，即接口不必关注实现方式，只需声明自己是做什么的(???)。

设计类间继承时，为了方便的进行子类继承，父类一般需要提供钩子(hook)，谨慎的选择可见性为`protected`的方法，某些情况下也需要提供可见性为`protected`的字段。如何决定提供哪些`protected`方法和字段没有银弹，最好的方式是仔细考虑，并通过书写子类进行测试，在灵活性和稳定性间寻找平衡。测试时可以参考的经验是书写三个子类，其中一个或多个由非父类的书写者完成。

在设计类间继承时，还要遵循以下两点：

1. 构造函数不能调用可重写的方法
2. 如果父类实现了`Cloneable`或`Serializable`接口，由于`clone`和`readObject`方法与构造方法十分相似，所以`clone`和`readObject`方法也不能调用可重写的方法

另外，对于未设计为可继承类的普通类来说，经常会发生被“偷偷”继承的情况，导致在普通类进行修改时给使用方带来问题。解决该问题的最好方式是禁止对普通类进行继承，禁止的两种方式在之前也提到过，这里再加深下印象：

1. `final`修饰`class`
2. `private`或`package private`修饰构造函数，通过提供静态工厂方法进行实例化(更推荐该方式)

这种禁止继承普通类的方式有时是有争议的，因为使用方在很多情况下希望对普通类进行功能扩展。两种情况，如果普通类实现了某个描述其功能的接口，那么完全可以禁止对普通类进行继承，通过`[16]`中介绍的装饰器模式进行功能扩展；如果普通类没有实现这种接口，担心禁止继承会引起使用方不变，则必须确保普通类不会调用任何可重写的方法，这样可以使普通类可以安全的被继承，重写方法不会对其他方法造成影响。


## [18]. 接口优于抽象类

Java提供了接口和抽象类两种机制来进行类型声明，并允许多种不同实现。接口和抽象类间最明显的不同是抽象类允许包含方法实现，而接口中不允许(***注*** Java 8中允许接口中实现方法，通过`default`关键字修饰，如`default void sayHi() {  // implementation }`)。另一个重要的不同点是，


补充：
skeletal implementation (abstract)
simple implementation (concrete)



相对于使用接口指定类型来说，使用抽象类有一个较大的优点是抽象类更新十分容易。在更新抽象类时，只需在抽象类中将新方法实现即可，且所有子类都可以使用该新方法。接口无法做到这一点(***注*** Java 8中接口已可以包含一个或多个默认实现)，接口一旦发布并被广泛使用后，就几乎不可能再被修改，因此一定要精心设计，且在版本确定前让尽可能多的使用者使用接口，以便在还可以修改时发现问题。






## [21]. 使用方法对象表示策略



举的一个`java.lang.String`中的例子，用


```
public static final Comparator<String> CASE_INSENSITIVE_ORDER
                                     = new CaseInsensitiveComparator();
private static class CaseInsensitiveComparator
        implements Comparator<String>, java.io.Serializable {
    // use serialVersionUID from JDK 1.2.2 for interoperability
    private static final long serialVersionUID = 8575799808933029326L;

    public int compare(String s1, String s2) {
        int n1 = s1.length();
        int n2 = s2.length();
        int min = Math.min(n1, n2);
        for (int i = 0; i < min; i++) {
            char c1 = s1.charAt(i);
            char c2 = s2.charAt(i);
            if (c1 != c2) {
                c1 = Character.toUpperCase(c1);
                c2 = Character.toUpperCase(c2);
                if (c1 != c2) {
                    c1 = Character.toLowerCase(c1);
                    c2 = Character.toLowerCase(c2);
                    if (c1 != c2) {
                        // No overflow because of numeric promotion
                        return c1 - c2;
                    }
                }
            }
        }
        return n1 - n2;
    }
    /** Replaces the de-serialized object. */
    private Object readResolve() { return CASE_INSENSITIVE_ORDER; }
}
```

## [22]. 静态内部类优于非静态内部类


static member class
non-static member class
anonymous class
local class

---
a local class can only access local variables that are declared final
(in java 8, there is 'effectively final')