# greenDao

关于greendao，相信做android的大部分开发者都使用过了，速度快，体积小，使用简单，内存开销小等一些列优点，且api非常简单方便，使其在开发中会经常使用到。

下面我简单介绍greendao的使用方法，介绍中黏贴部分代码，供参考，如有不对的地方，欢迎大家指正，谢谢。

1. 引入greendao。

   在build中先引入greendao:
    //在root build.gradle引入相关插件
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.3'
        classpath 'org.greenrobot:greendao-gradle-plugin:3.2.2'
    }
    
    //In your app projects build.gradle file:
    apply plugin: 'com.android.application'
    apply plugin: 'org.greenrobot.greendao' 
    android {  
         ...  
         greendao{
            schemaVersion 1 //表示数据库版本号
            daoPackage 'com.tsy.greendaouse.db'//自动生成代码所在包名，默认在build/generated/source/greendao
            targetGenDir 'src/main/java'  //生成的DAOMaster和DaoSession的位置
        }
    } 
    dependencies {
          compile 'org.greenrobot:greendao:3.2.2'//引入greenDAO的类库
          //使用数据库升级辅助GreenDaoUpgradeHelper
          //compile 'com.github.yuweiguocn:GreenDaoUpgradeHelper:v1.2.0'
    }

2. 代码中定义实体类，并且添加注释代码：
    @Id：对象的Id，使用Long类型作为EntityId，否则会报错。(autoincrement = true)表示主键会自增，如果false就会使用旧值 
    @Entity：告诉GreenDao该对象为实体，只有被@Entity注释的类才能被dao类操作
    @Property：表示该属性将作为表的一个字段
    @Transient：表示该属性不会被存入数据库的字段中
    @Unique：表示该属性值在数据库中是唯一值
    @NotNull：表示该属性不能为空 
    @Generated：编译后自动生成的构造函数、方法等的注释，提示构造函数、方法等不能被修改
    
3. 实体类中包含对象和列表类型，不单单是基本类型：
  a. 首先写一个转换类，继承PropertyConverter<T,String>,其中T表示泛型，可以为任何对象，代码如下
    /**
    * db类型转换类
    * @param <T>
    */
    public class BaseDaoConvert<T> implements PropertyConverter<T,String> {
        @Override
        public T convertToEntityProperty(String databaseValue) {
            if (databaseValue == null) {
                return null;
            }
            return base64Str2Object(databaseValue);
        }

        @Override
        public String convertToDatabaseValue(T entityProperty) {
            if (entityProperty == null) {
                return null;
            }
            return object2Base64Str(entityProperty);
        }
    }
    
    
    // String to Object
    public static <T> T base64Str2Object(String productBase64) {
        T device = null;
        if (productBase64 == null) {
            return null;
        }
        // 读取字节
        byte[] base64 = Base64.decode(productBase64.getBytes(), Base64.DEFAULT);

        // 封装到字节流
        ByteArrayInputStream bais = new ByteArrayInputStream(base64);
        try {
            // 再次封装
            ObjectInputStream bis = new ObjectInputStream(bais);
            // 读取对象
            device = (T) bis.readObject();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return device;
    }

    // Object to String
    public static <T> String object2Base64Str(T object) {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        try {   //Device为自定义类
            // 创建对象输出流，并封装字节流
            ObjectOutputStream oos = new ObjectOutputStream(baos);
            // 将对象写入字节流
            oos.writeObject(object);
            // 将字节流编码成base64的字符串
            return new String(Base64.encode(baos
                    .toByteArray(), Base64.DEFAULT));
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
  
  b. 在注释的实体类中在创建一个静态类，实现对象的转换，例如：
    public static class BlackBoxBeanConvert extends BaseDaoConvert<List<List<DsBean>>> {}
  
  c. 在实体类中的注释如下：
    @Entity
    public class BlackBoxEntity {
        @Id
        private Long id;

        @Convert(converter = BlackBoxEntityConvert.class, columnType = String.class)
        private List<List<DsBean>> dsLists; // 解析后的数据

        public static class BlackBoxEntityConvert extends BaseDaoConvert<List<List<DsBean>>> {}
    }

 4. 简单的使用方法：
    xx.getDaoSession().getBlackBoxEntityDao().insert(entity);
    xx.getDaoSession().getBlackBoxEntityDao().queryBuilder().list();
    

总结:配置什么的其实都很简单，主要是在第3步的类型转换。有问题，欢迎随时沟通，谢谢




