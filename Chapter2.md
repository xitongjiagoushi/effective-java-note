# 第二章 - 创建／销毁对象

## 1. 静态工厂方法创建对象

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

---
**旁白：**现在来看并不需要这种方法，从Java 7开始，范型实例初始化可直接省略实例中的范型声明，即：

```
Map<String, List<String>> m = new HashMap<>();
```
---


同时有以下**缺点**：

1. 静态工厂方法的使用一般都会将构造方法设为private，导致该类无法被继承
2. 静态工厂方法和该类的其他静态方法容易混淆，不好区分


```
class SubCollections extends Collections {  // There is no default constructor available in 'java.util.Collections'

}
```

常用静态工厂方法名称：`valueOf`, `of`, `getInstance`, `newInstance`, `getType`, `newType`



## 2. 