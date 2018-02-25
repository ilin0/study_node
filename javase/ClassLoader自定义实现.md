
```
public class MyClassLoader extends ClassLoader {
    private String name; //加载器的名字
    
    private String path = "d:\\"; // 加载 的位置
    
    private final String fileType = ".class"; // class文件扩展名
    
    public MyClassLoader(String name) {
        super(); 
        this.name = name;
    }
    
    public MyClassLoader(ClassLoader parent, String name) {
        super(parent); // 
        
        this.name = name;
    }
    
    @Override
    public String toString() {
        return this.name;
    }

    public String getPath() {
        return path;
    }

    public void setPath(String path) {
        this.path = path;
    }
    
    @Override
    public Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] data = this.loadClassData(name);
        
        
        // 将一个字节数组转换成一个 class 对象
        return this.defineClass(name, data, 0, data.length);
    }
    
    // 将文件读取到 为字节数组
    private byte[] loadClassData(String name){
        InputStream is = null;
        byte[] data = null;
        ByteArrayOutputStream baos = null;
        
        try{
            this.name = this.name.replace(".", "\\");
            
            is = new FileInputStream(new File(path + name + fileType));
            
            baos = new ByteArrayOutputStream();
            
            int ch = 0;
            
            while(-1 != (ch = is.read())){
                baos.write(ch);
            }
            
            data = baos.toByteArray();
        }
        catch(Exception ex){
            ex.printStackTrace();
        }
        finally{
            try{
                is.close();
                baos.close();
            }catch(Exception ex){
                ex.printStackTrace();
            }
        }
        return data;
    }
    
    public static void main(String[] args) throws Exception{
        MyClassLoader loader1 = new MyClassLoader("loader1");
        
        loader1.setPath("d:\\myapp\\serverlib");
        
        // loader2 是 loader1 的父加载 器
        MyClassLoader loader2 = new MyClassLoader(loader1, "loader2");
        
        loader2.setPath("d:\\myapp\\clientlib");
        
        // 第一个参数 为null,就表示它是一个根加载器
        MyClassLoader loader3 = new MyClassLoader(null, "loader3");
        
        loader3.setPath("d:\\myapp\\otherlib");
        
        test(loader2);
        test(loader3);
    }
    
    public static void test(ClassLoader loader) throws Exception{
        Class clazz = loader.loadClass("Sample");
        
        Object object = clazz.newInstance();
    }
}
```