### 自定义classloader.md
```java

public class LoaderTest16 extends ClassLoader {
    private String classLoaderName;

    private final String fileExtension = ".class";

    public LoaderTest16 (String classLoaderName){
        super();
        this.classLoaderName = classLoaderName;
    }
    public LoaderTest16(ClassLoader parent, String classLoaderName){
        super(parent);
        this.classLoaderName = classLoaderName;
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] data = this.loadClassData(name);
        return this.defineClass(name, data, 0, data.length);
    }

    private byte[] loadClassData(String name){
        InputStream is = null;
        byte[] data = null;
        ByteArrayOutputStream baos = null;
        try {
            this.classLoaderName = this.classLoaderName.replace(".", "/");
            is = new FileInputStream(new File(name + this.fileExtension));
            baos = new ByteArrayOutputStream();

            int ch = 0;
            while (-1 != (ch = is.read())){
                baos.write(ch);
            }
            data = baos.toByteArray();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            try {
                is.close();
                baos.close();
            }catch (Exception e){
                e.printStackTrace();
            }
        }
        return data;
    }

    public static void main(String[] args) throws Exception {
        LoaderTest16 l16 = new LoaderTest16("l16");

        Class<?> clazz = l16.loadClass("com.gs.jvm.loadering.LoaderTest1");
        Object obj = clazz.newInstance();

        System.out.println(obj);
    }
}
```
    findClass是被loadClass所调用，可通过java doc了解。
    defineClass 将一个字节数据转换成class实例。在使用之前需要解析。

    loadClass以下顺序执行
    1、查找是否已加载
    2、调用父类加载器，如果它是null,就会调用启动类加载器加载。
    3、查找类（即我们重写的那个类）

1. 上面的示例虽然能成功加载 ，但返回的class的classLoader并不是我们定义的，而是系统加载器
2. 修改一下
    is = new FileInputStream(new File(name + this.fileExtension));
    修改为：
    is = new FileInputStream(new File(this.path + name + this.fileExtension));
    path = "/Users/ilin0/Desktop";
    如果从桌面加载（前提是把classpath下的class删除），那么加载它的类就是自定义的类加载器，
    如果有二classpath与desktop下同时存在，那么只会加载 classpath下的，desktop并不会加载。因为它会委托父类加载器加载。

3. 命名空间
    1、每个类加载器都有自己的命名空间，命名空间由该加载器及所有父加载器所加载的类组成。
    2、在同一个命名空间中，不会出现类的完整名字相同的两个类。
    3、在不同的命名空间中，有可能会出现类的完整名字相同的两个类。
4. 意思就是说，如果我们生成二个自定义的实例，去加载同一个类，那么就能加载二次（它们不是一个命名空间）没有父子关系。
5. 也就是说如果是父子关系就只会被加载一次，如果不是父子关系就会被加载多次。

#### 验证卸载  修改代码如下
```java
LoaderTest16 l16 = new LoaderTest16("l16");

        l16.setPath("/Users/ilin0/Desktop/");

        Class<?> clazz = l16.loadClass("com.gs.jvm.loadering.LoaderTest1");
        System.out.println("class: " + clazz.hashCode());

        Object obj = clazz.newInstance();

        System.out.println(obj);

        System.out.println(clazz.getClassLoader());

        

        l16 = new LoaderTest16("l16");

        l16.setPath("/Users/ilin0/Desktop/");

        clazz = l16.loadClass("com.gs.jvm.loadering.LoaderTest1");
        System.out.println("class: " + clazz.hashCode());

        obj = clazz.newInstance();

        System.out.println(obj);

        System.out.println(clazz.getClassLoader());

```

##### 输出如下
    class: 1510467688
    com.gs.jvm.loadering.LoaderTest1@76ed5528
    com.gs.jvm.loadering.LoaderTest16@5451c3a8
    class: 1523554304
    com.gs.jvm.loadering.LoaderTest1@4617c264
    com.gs.jvm.loadering.LoaderTest16@2c7b84de
    它们的hashCode是不一样的。加上参数可以更好的看到卸载信息
    -XX:+TraceClassUnloading

        l16 = null;
        obj = null;
        clazz=null;
        System.gc(); 调用这步就能看到输出如下：
    class: 1510467688
    com.gs.jvm.loadering.LoaderTest1@76ed5528
    com.gs.jvm.loadering.LoaderTest16@5451c3a8
    [Unloading class com.gs.jvm.loadering.LoaderTest1 0x00000007c0061028]
    class: 1523554304
    com.gs.jvm.loadering.LoaderTest1@4617c264
    com.gs.jvm.loadering.LoaderTest16@2c7b84de

    可以通过jvisualvm工具查看监视情况。