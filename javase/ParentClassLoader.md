
### 父类加载器

```java
public class Test8 {
    public static void main(String[] args) {
        ClassLoader classLoader = ClassLoader.getSystemClassLoader();
        
        System.out.println(classLoader);
        
        while(null != classLoader) {
            classLoader = classLoader.getParent();    
            System.out.println(classLoader);
        }
    }
}

```