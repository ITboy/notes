# Java泛型

Java泛型其实就是模板类，是定义类的模板，使得类在被使用时可以传递参数，得到一个新的类，可以减少代码中很多强制转换，并且使得类型信息出现在代码中而不是程序员的脑子或注释里。

## 定义模板类 ##

在定义模板类时，可以使用参数来临时指定一个类型，就像函数中的形参一样，实际上是为模板类定义一些类型约束。

具体用法：
1. 为模板类定义类型，使得返回的类型和传入的参数类型一致
   ``` java
    public class C <T> {
        private List<T> list = new ArrayList<>();
        
        T get(int i) {
            return list.get(i);
        }

        void set(T t) {
            list.put(t);
        }
    }
   ```
2. 为传入的类型指定约束
    ``` java
        public class C <T extends User> {
            ...
        }

        C <Long> c = new C <> (); // error: Long is not the subtype of User.
    ```

模板类参数传递时机：
1. 对象实例化时 
    ``` java
        List<String> list = new ArrayList<>();
    ```
2. 方法调用时
    ``` java
       public class A {
            public T get (T t) {
                return t;
            }
       }

       String s = new A().get('abc');
    ```
3. 类继承或接口实现时
    ``` java
        public interface<T> IMoo {
            T moo () {
            }
        }

        public bird<T> implements interface<T> IMoo {
            ...
        }
    ```

## 泛型通配符 ##

泛型通配符是在使用模板类（不是定义模板类）的时候使用的，因为泛型的类型不像数组是协变的。

``` java
Integer[] intergers = [1, 2, 3];
Number[] numbers = intergers; 
```
`Integer`继承`Number`，所以`Integer[]`也继承`Number[]`，称之为`协变`。

而在泛型中却这种方式行不通。

``` java
List<Integer> intergers = new ArrayList<>();
List<Number> numbers = intergers; 
```

为了解决这个问题，就出现了`List<?> list = new ArrayList<String>()`.
在上述情况下，可以使用`Object`来接收`get`的值，`Object o = list.get(0)`。

更高级的功能：

``` java 
List<? extends User> list = new ArrayList<User>()`;
User user = list.get(0);
```