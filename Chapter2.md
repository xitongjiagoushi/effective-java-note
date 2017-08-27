# 第二章 - 创建／销毁对象

## [1]. 静态工厂方法创建实例

除了使用构造函数，一个类可以通过提供静态工厂方法，返回自身实例，如：

```
public static Boolean valueOf(boolean b) {
	return b ? Boolean.TRUE : Boolean.FALSE;
}
```

使用静态工厂方法而非构造函数创建对象，主要有以下**优点**：

1. 静态工厂方法有自己的方法名
2. 静态工厂方法不必在每次被调用时都创建新对象
3. 静态工厂方法可以返回本类的任意子类型
4. 静态工厂方法可以减少创建范型实例时的复杂性


```
Map<String, List<String>> m = new HashMap<String, List<String>>();
```

可以用如下方式代替(HashMap.java中无此方法，只为举例)：

```
public static <K, V> HashMap<K, V> newInstance() {
	return new HashMap<K, V>();
}
```

***注*** 现在来看并不需要这种方法，从Java 7开始，范型实例初始化可直接省略实例中的范型声明，即：

```
Map<String, List<String>> m = new HashMap<>();
```

同时有以下**缺点**：

1. 静态工厂方法的使用一般都会将构造方法设为private，导致该类无法被继承
2. 静态工厂方法和该类的其他静态方法容易混淆，不好区分


```
// There is no default constructor available in 'java.util.Collections'
class SubCollections extends Collections {

}
```

常用静态工厂方法名称：`valueOf`, `of`, `getInstance`, `newInstance`, `getType`, `newType`

## [2]. Builder创建实例

一个类如果有多个参数，如何构造？两种方式：

1. 使用“望远镜”型构造函数(构造函数中赋值成员变量的个数依次增加)
2. 使用JavaBeans依次调用成员变量的set方法

“望远镜”型构造函数的缺点是可读性太差；JavaBeans#set方法的缺点是在多线程情况下可能存在线程安全问题。此时非常适合通过Builder创建实例：

```
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    private NutritionFacts(Builder builder) {
        this.servingSize = builder.servingSize;
        this.servings = builder.servings;
        this.calories = builder.calories;
        this.fat = builder.fat;
        this.sodium = builder.sodium;
        this.carbohydrate = builder.carbohydrate;
    }

    public static class Builder {
        private final int servingSize;
        private final int servings;
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;


        private Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int calories) {
            this.calories = calories;
            return this;
        }

        public Builder fat(int fat) {
            this.fat = fat;
            return this;
        }

        public Builder sodium(int sodium) {
            this.sodium = sodium;
            return this;
        }

        public Builder carbohydrate(int carbohydrate) {
            this.carbohydrate = carbohydrate;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    public static Builder builder(int servingSize, int servings) {
        return new Builder(servingSize, servings);
    }
}
```

***注*** [lombok](https://projectlombok.org/)可以通过注解方式自动生成类对应的builder：

```
@lombok.Builder
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
}
```

Builder模式有一个缺点是在创建类实例时，需要先创建一个Builder实例，在某些对性能要求极高的场景下可能会存在问题。

补充：抽象工厂(泛型)

```
public interface Builder<T> {
	T build();
}
```

## [3]. 使用私有构造方法或枚举类型强制单例属性？？？

```
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	private Elvis() {}
	public static Elvis getInstance() {
		return INSTANCE;
	}
	public void leaveTheBuilding() {}
}
```

```
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	private Elvis() {}
	public static Elvis getInstance() {
		return INSTANCE;
	}
}
```


## [4]. 使用私有构造函数强制非实例化



## [5]. 禁止创建不必要的对象


## [6]. 消除过期的对象引用

貌似是不需要的现在，JIT会做调优

## [7]. 禁止finalize调用

