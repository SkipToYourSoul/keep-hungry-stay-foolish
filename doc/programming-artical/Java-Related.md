# Java浅拷贝和深拷贝

浅拷贝：按位拷贝对象，会创建一个新对象。如果属性是基本类型，拷贝的就是基本类型的值；如果属性是内存地址（引用类型），拷贝的就是内存地址 ，因此如果其中一个对象改变了这个地址，就会影响到另一个对象。

深拷贝：在拷贝引用类型成员变量时，为引用类型的数据成员另辟了一个独立的内存空间，实现真正内容上的拷贝。

对象拷贝：简单的拷贝了内存地址，其中一个对象修改，另一个也会对应修改。

https://www.jianshu.com/p/94dbef2de298

# 泛型

泛型

# 反射

> 能够分析类能力的程序被称为反射（reflective），通过反射可以在运行时（而不是在编译时）动态地分析软件组件并描述组件的功能。

通过反射，我们可以

- 分析类的能力
- 获得类的对象，即使你不知道这个类到底是个啥玩意儿

用一个例子说明。

```java
public class ReflectTest {
    public static <T> T create(Class clazz) {
        // 通过反射获得一个类的对象
        Object object = null;
        try {
            object = Class.forName(clazz.getName()).newInstance();
        } catch (InstantiationException | IllegalAccessException | ClassNotFoundException e) {
            e.printStackTrace();
        }

        // 通过反射进行类分析
        System.out.println("====Construct====");
        for (Constructor constructor : clazz.getConstructors()) {
            System.out.println(constructor);
        }
        System.out.println("====Fields====");
        for (Field field : clazz.getFields()) {
            System.out.println(field);
        }
        System.out.println("====DeclaredFields====");
        for (Field field : clazz.getDeclaredFields()) {
            System.out.println(field);
        }
        System.out.println("====Methods====");
        for (Method method : clazz.getMethods()) {
            System.out.println(method);
        }
        System.out.println("====DeclaredMethods====");
        for (Method method : clazz.getDeclaredMethods()) {
            System.out.println(method);
        }

        return (T) object;
    }

    public static void main(String[] args) {
        Cat cat = ReflectTest.create(Cat.class);
        System.out.println(cat.tail);
        System.out.println(cat.getVoice());
    }
}

class Cat {
    private String voice;
    public String tail = "tail";

    public Cat() {
    }

    public Cat(String voice) {
        this.voice = voice;
    }

    private void printVoice() {
        System.out.println(this.voice + this.tail);
    }

    public String getVoice() {
        return voice;
    }
}
```



参考文献：https://juejin.im/post/5e578437518825493775fbdb?utm_source=gold_browser_extension

# 设计模式

单例、Builder

# 类加载

https://mp.weixin.qq.com/s/eHqFONXXNc-LD4ugaKM6UA

类加载器、双亲委派模型也是非常重要的知识点。