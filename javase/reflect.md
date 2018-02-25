### 获取某个类或某个对象所对应的 Class 对象的常用的 3 种方式:
    1. 使用 Class 类的静态方法 forName:
        Class.forName(“java.lang.String”);
    2. 使用类的.class 语法:String.class;
    3. 使用对象的 getClass()方法:
        String s = “aa”; 
        Class<?> clazz = s.getClass();

### 示例
```
    public static Object buildObject(Class<?> tClass) throws Exception {
        try {

            Object entity = tClass.newInstance();
            Field[] fields = tClass.getDeclaredFields();

            for (Field field : fields) {
                String fieldName = field.getName();

                if (fieldName.equals("serialVersionUID")) {
                    continue;
                }

                String fieldType = field.getType().getName();

                // 设置私有方法可访问
                field.setAccessible(true);
                
                if (fieldType.endsWith("BigDecimal")) {
                    field.set(entity, new BigDecimal("1.38"));
                } else if (fieldType.endsWith("Date")) {
                    field.set(entity, DateUtil.getCurrDate());
                } else if (fieldType.endsWith("Long")) {
                    field.set(entity, 2l);
                } else if (fieldType.endsWith("Integer")) {
                    field.set(entity, 9);
                } else {
                    field.set(entity, "123");
                }
            }
            return entity;
        } catch (Exception e) {
            throw e;
        }
    }

```