[TOC]

# 1. 简单实例

```java
package reflection;
public class Car {
    private String brand;
    private String color;
    private int maxSpeed;
    public Car(){}
    public Car(String brand, String color, int maxSpeed) {
        this.brand = brand;
        this.color = color;
        this.maxSpeed = maxSpeed;
    }
    // 省略set,get,toString,...方法
}
```

```java
public class ReflectTest {
    public static void main(String[] args) throws Exception {
        // 通过类装载器获取对象
        ClassLoader loader = Thread.currentThread().getContextClassLoader();
        Class clazz = loader.loadClass("reflection.Car");
		// 通过类的默认构造对象并通过它实例化Car
        Constructor constructor = clazz.getConstructor((Class[])null);
        Car car = (Car)constructor.newInstance();
        // 通过反射方法设置属性
        Method setBrand = clazz.getMethod("setBrand", String.class);
        setBrand.invoke(car,"红旗CA72");
        clazz.getMethod("setColor", String.class).invoke(car,"红色");
        clazz.getMethod("setMaxSpeed", int.class).invoke(car,300);
        System.out.println(car);
    }
    // Car{brand='红旗CA72', color='红色', maxSpeed=300}
}
```

有几个非常重要的反射类，分别是`ClassLoader`、`Class`、`Constructor`和`Method`。

- **获取当前线程的`ClassLoader`**，然后通过指定的全限制类名`reflection.Car`装载`Car`类对应的反射实例。

```java
ClassLoader loader = Thread.currentThread().getContextClassLoader();
Class clazz = loader.loadClass("reflection.Car");
```

- 通过`Car`的**反射类对象**获取`Car`的构造函数，通过构造函数对象的`newInstrance()`方法实例化`Car`对象 。

```java
Constructor constructor = clazz.getConstructor();
Car car = (Car)constructor.newInstance();
```

- 通过`Car`的反射类对象的`getMethod(String methodName,Class<?>... parameterTypes)`方法获取属性的`Setter`方法对象。在获取对象方法之后，可以使用`invoke(Object obj,Object... args)`方法调用目标类的方法 。

```java
Method setBrand = clazz.getMethod("setBrand",String.class);
setBrand.invoke(car,"红旗CA72");
Method setColor = clazz.getMethod("setColor",String.class);
setColor.invoke(car,"red");
//...
```

# 2. 类装载器`ClassLoader`

## 2.1 类装载器的工作机制

📚 类装载器就是 **寻找类的字节码文件并构造出类在JVM内部表示对象的组件**。在Java中，类装载器把一个类装入JVM中，需要经过以下步骤 ：

- 装载 ：查找和导入`Class`文件

- 链接 ：执行校验 、准备和解析步骤，其中解析步骤是可以选择的

  - 校验 ：检查载入`Class`文件数据的正确性
  - 准备 ：给类的静态变量分配存储空间
  - 解析 ：将符号引用转换为直接引用

- 初始化 ：对类的静态变量 、静态代码块执行初始化操作

类装载工作**由`ClassLoader`及其子类负责** 。`ClassLoader`是运行时系统组件，**负责在运行时查找和装入`Class`字节码文件**。JVM在运行时会产生3个`ClassLoader` ：

- 根装载器

  不是`ClassLoader`的子类，使用C++语言编写，负责 **装载`JRE`核心类库**，`JRE`环境目录下的`rt.jar`、`resource.jar`、`charsets.jar`和`class`等，还可以加载`java -Xbootclasspath/a:xxx`被指定的文件。例如 ：

  ```cmd
  win32   java -Xbootclasspath/a: some.jar;some2.jar; -jar test.jar
  unix    java -Xbootclasspath/a: some.jar:some2.jar: -jar test.jar
  ```

  win32下每个jar用分号隔开，unix下面用冒号

- `ExtClassLoader`——扩展类装载器

  `ClassLoader`的子类，负责装载JRE扩展目录`ext/`中的`jar`包和`class`文件，还可以加载`-D java.ext.dirs`指定目录

- `AppClassLoader` ——应用类装载器

  `ClassLoader`的子类，负责装载`classpath`路径下的类包

🏩 三个装载器之间存在父子层级关系，即根装载器是`ExtClassLoader`的**父装载器**，`ExtClassLoader`是`AppClassLoader`的父装载器。在默认情况下，使用`AppClassLoader`装载应用程序的类 。

```java
ClassLoader loader = Thread.currentThread().getContextClassLoader();
System.out.println(loader);
System.out.println(loader.getParent());
System.out.println(loader.getParent().getParent());
// sun.misc.Launcher$AppClassLoader@18b4aac2
// sun.misc.Launcher$ExtClassLoader@75b84c92
// 根装载器在Java中访问不到，所以返回null
// null
```

🎯 父加载器不是父类 ：

![](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-9-10/85346097.jpg)



以下是`ClassLoader`的部分源码 ：

```java
public class ClassLoader{
    protected ClassLoader() {
        this(checkCreateClassLoader(), getSystemClassLoader());
    }
    protected ClassLoader(ClassLoader parent) {
        this(checkCreateClassLoader(), parent);
    }
    private ClassLoader(Void unused, ClassLoader parent) {
        this.parent = parent;
        //...
    }
    // ...
    public static ClassLoader getSystemClassLoader() {
        initSystemClassLoader();
        // ...
        return scl;
    }
    private static synchronized void initSystemClassLoader() {
        if (!sclSet) {
            if (scl != null)
                throw new IllegalStateException("recursive invocation");
            sun.misc.Launcher l = sun.misc.Launcher.getLauncher();
            if (l != null) {
                Throwable oops = null;
                // 通过getClassLoader()获取parent
                scl = l.getClassLoader();
                // ...
            }
            sclSet = true;
        }
    }
    
}
```

通过源码可以得知，parent赋值有两种情况 ：

- 当外部类创建`ClassLoader`时，指定一个`ClassLoader`为parent
- 由`getSystemClassLoader()`生成，即通过`Launcher.getClassLoader()`获取，也就是`AppClassLoader`

```java
public ClassLoader getClassLoader() {
	return loader;
}
// ...
public Launcher() {
	// ...
    ClassLoader extcl;
    // ...
    extcl = ExtClassLoader.getExtClassLoader();
    // ...
    loader = AppClassLoader.getAppClassLoader(extcl);
	// ...
}
// ...
static class AppClassLoader extends URLClassLoader {
	public static ClassLoader getAppClassLoader(final ClassLoader extcl)
            throws IOException
        {
        	return new AppClassLoader(urls, extcl);
        }
    AppClassLoader(URL[] urls, ClassLoader parent) {
		super(urls, parent, factory);
        ucp = SharedSecrets.getJavaNetAccess().getURLClassPath(this);
        ucp.initLookupCache(this);
	}
}
```

由这部分源码不难看出，`AppClassLoader`的parent是一个`ExtClassLoader`实例，`ExtClassLoader`的parent是Null

JVM装载类时使用”**全盘负责委托机制**“：

- 全盘负责 ：当一个`ClassLoader`装载类时，除非显示的使用另一个`ClassLoader`，该类所依赖及引用的类也由这个`ClassLoader`载入。
- 委托机制 ：先委托父装载器寻找目标类，只有在找不到的情况下才从自己的类路径中查找并装载目标类，这一点是从安全角度考虑的。

## 2.2 类装载器的装载顺序

1. `BootStrap ClassLoader`
2. `Extention ClassLoader`
3. `AppClassLoader`

> sun.misc.Launcher —— Java虚拟机入口应用

```java
public class Launcher{
    // ...
    private static Launcher launcher = new Launcher();
    private static String bootClassPath = System.getProperty("sun.boot.class.path");
    public static Launcher getLauncher(){
        return launcher;
    }
    private ClassLoader loader;
    
    public Launcher() {
        // Create the extension class loader
        ClassLoader extcl;
        try {
            extcl = ExtClassLoader.getExtClassLoader();
        } catch (IOException e) {
            throw new InternalError(
                "Could not create extension class loader", e);
        }

        // Now create the class loader to use to launch the application
        try {
            loader = AppClassLoader.getAppClassLoader(extcl);
        } catch (IOException e) {
            throw new InternalError(
                "Could not create application class loader", e);
        }

        // 设置AppClassLoader为线程上下文类加载器
        Thread.currentThread().setContextClassLoader(loader);
        
        // .....
    }
    
    static class ExtClassLoader extends URLClassLoader {}
    
    static class AppClassLoader extends URLClassLoader {}
}
```

通过代码我们可以得知 ：

1. `Launcher`初始化了`ExtClassLoader`和`AppClassLoader`

2. `Launcher`中并没有看到`BootstrapClassLoader`，但是通过`System.getProperty("sun.boot.class.path")`得到了字符串`bootClassPath`，这个应该就是`BootstrapClassLoader`加载的jar包路径

我们通过代码测试一下`sun.boot.class.path`中的内容 ：

```java
System.out.println(System.getProperty("sun.boot.class.path"));
// D:\Java\jre\lib\resources.jar;
// D:\Java\jre\lib\rt.jar;
// D:\Java\jre\lib\sunrsasign.jar;
// D:\Java\jre\lib\jsse.jar;
// D:\Java\jre\lib\jce.jar;
// D:\Java\jre\lib\charsets.jar;
// D:\Java\jre\lib\jfr.jar;
// D:\Java\jre\classes

```

## 2.3 `ExtClassLoader`源码

```java
static class ExtClassLoader extends URLClassLoader {
    static {
        ClassLoader.registerAsParallelCapable();
    }

    private static volatile ExtClassLoader instance = null;

    private static File[] getExtDirs() {
        String s = System.getProperty("java.ext.dirs");
        File[] dirs;

        if (s != null) {
            StringTokenizer st = new StringTokenizer(s, File.pathSeparator);
            int count = st.countTokens();
            dirs = new File[count];

            for (int i = 0; i < count; i++) {
                dirs[i] = new File(st.nextToken());
            }
        } else {
            dirs = new File[0];
        }

        return dirs;
    }

    // ...
}

```

可以通过`-D java.ext.dirs`参数来添加和改变`ExtClassLoader`加载路径 ：

```java
System.out.println(System.getProperty("java.ext.dirs"));
// D:\Java\jre\lib\ext;C:\WINDOWS\Sun\Java\lib\ext
```

## 2.4 `AppClassLoader`源码

```java
static class AppClassLoader extends URLClassLoader {
    
    public static ClassLoader getAppClassLoader(final ClassLoader extcl)
        throws IOException {
        final String s = System.getProperty("java.class.path");
        final File[] path = (s == null) ? new File[0] : getClassPath(s);

        return AccessController.doPrivileged(new PrivilegedAction<AppClassLoader>() {
                public AppClassLoader run() {
                    URL[] urls = (s == null) ? new URL[0] : pathToURLs(path);

                    return new AppClassLoader(urls, extcl);
                }
            });
    }

    // ......
}

```

可以看到`AppClassLoader`加载的就是`java.class.path`下的路径。

## 2.5 双亲委托

🔜 双亲委托的步骤如下所示 ：

1. 当一个`AppClassLoader`查找资源时候，先看缓存中是否存在，如果存在，直接从缓存中获取，如果不存在，就委托给父加载器
2. 递归，重复第一步操作
3. 如果`ExtClassLoader`也没有加载过，就由`BootstrapClassLoader`出面，首先查找缓存，如果没有找到，就去找自己规定的路径下，也就是`sun.mic.boot.class`下面的路径。找到就返回，没有找到就让子加载器去找
4. 如果`BootstrapClassLoader`没有查找成功，则`ExtClassLoader`自己在`java.ext.dirs`路径中去查找，查找成功就返回，查找不成功再让子加载器找
5. 如果`ExtClassLoader`查找不成功，`AppClassLoader`就自己就在`java.class.path`路径下去寻找，找到就返回，如果没有找到就让子加载器去找，如果没有子加载器的话就抛出各种异常

![](http://zhangzhaolin.oss-cn-beijing.aliyuncs.com/18-9-11/65095495.jpg)

## 2.6 `ClassLoader`重要方法

`ClassLoader`中有以下几种常用方法 ：

```java
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // 首先，检查是否已经加载
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                    	// 父加载器不为空则调用父加载器的loadClass
                        c = parent.loadClass(name, false);
                    } else {
                    	// 父加载器为空则调用Bootstrap loadClass
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    // 父加载没有找到，则调用findClass
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
            	// 调用resolveClass
                resolveClass(c);
            }
            return c;
        }
    }
```

😡 注意 ：如果要編寫一個`ClassLoader`的子類，也就是要自定義一個`ClassLoader`時，**建議覆蓋`findClass()`方法**

|                             方法                             |                             描述                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|      `Class<?> loadClass(String name, boolean resolve)`      | name制定类装载器需要装载类的名称（必须要使用全限定类名），resolve参数告诉装载器是否需要解析该类（在初始化类之前，应该考虑类的解析工作，但并不是所有的类都需要解析） |
| `Class<?> defineClass(String name, byte[] b, int off, int len)` | 将类文件的字节数组转换为JVM内部的`java.lang.Class`对象，字节数组可以从本地文件系统、远程网络获取 |

## 2.7 自定义`ClassLoader`方法

自定义`ClassLoader`的步驟 ：

1. 编写一个类继承自`ClassLoader`抽象类
2. 复写它的`findClass()`方法
3. 在`findClass()`中调用`defineClass()`

```java
public class DiskClassLoader extends ClassLoader {

    private String mLibPath;

    public DiskClassLoader(String mLibPath){
        this.mLibPath = mLibPath;
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        String fileName = getFileName(name);
        File file = new File(mLibPath,fileName);
        try {
            FileInputStream fis = new FileInputStream(file);
            ByteArrayOutputStream output = new ByteArrayOutputStream();
            int len = 0;
            while((len = fis.read()) != -1){
                output.write(len);
            }
            byte[] data = output.toByteArray();
            output.close();
            fis.close();
            return defineClass(name,data,0,data.length);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return super.findClass(name);
    }

    private String getFileName(String name){
        // com.abc.kkk
        int index = name.lastIndexOf(".");
        if (index == -1){
            return name+".class";
        }else{
            return name.substring(index+1) + ".class";
        }
    }
}
```

```java
public class ClassLoaderText {
    public static void main(String[] args) {
        DiskClassLoader loader = new DiskClassLoader("D:/lib");
        try {
            Class clazz = loader.loadClass("reflection.Test");
            Object obj = clazz.newInstance();
            Method method = clazz.getMethod("say",null);
            method.invoke(obj,null);
        } catch (ClassNotFoundException | IllegalAccessException |
                InstantiationException | NoSuchMethodException |
                InvocationTargetException e) {
            e.printStackTrace();
        }
    }
}
```

## 2.8 `Context ClassLoader`线程上下文类加载器

`contextClassLoader`只是一个成员变量，通过`setContextClassLoader`设置，通过`getContextClassLoader`获取 ：

```java
public class Thread implements Runable{
	private ClassLoader contextClassLoader;
    
    public void setContextClassLoader(ClassLoader cl) {
       SecurityManager sm = System.getSecurityManager();
       if (sm != null) {
           sm.checkPermission(new RuntimePermission("setContextClassLoader"));
       }
       contextClassLoader = cl;
   }

   public ClassLoader getContextClassLoader() {
       if (contextClassLoader == null)
           return null;
       SecurityManager sm = System.getSecurityManager();
       if (sm != null) {
           ClassLoader.checkClassLoaderPermission(contextClassLoader,
                                                  Reflection.getCallerClass());
       }
       return contextClassLoader;
   }
}
```

# 3. Java反射机制

`Class`反射对象 ：描述类语义结构，可以从Class对象中获取构造函数、成员变量、方法类等类元素的反射对象。

- `Constructor`：根据`getConstructors()`方法可以获取类的所有构造函数反射对象数组。还可以通过`getConstructor(Class<?>... parameterTypes)`方法获取特定参数的构造函数反射对象。
- `Method`：