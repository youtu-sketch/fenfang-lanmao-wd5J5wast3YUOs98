
## 说明


本文中的 JVM 参数和代码在 JDK 8 版本生效。


## 哪里有用户类？


用户类是由开发者和第三方定义的类，它是由应用程序类加载器加载的。


Java 程序可以通过`CLASSPATH` 环境变量，JVM 启动参数 `-cp` 或者 `-classpath` 指定用户需要加载的类的路径。这两个配置的优先级从低到高，后面的配置会覆盖前面的配置，默认值是「.」，即当前路径。


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400785-209640951.png)](https://github.com)


接下来对默认值和优先级进行验证：


### 验证默认值是当前路径


现在有一个 Temp.java 类，它不在任何包路径下：



```
public class Temp {  
    public static void main(String[] args) {  
        System.out.println("Executed!");  
    }  
}

```

同时这个时候系统没有配置 `CLASSPATH` 这个环境变量，如下图所示：


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400808-401891996.png)](https://github.com)


然后拷贝这个编译后的 `Temp.class` 文件放到 E 盘的下，然后执行命令 `java Temp` 命令，是能够正常运行这个 Class 文件的。这个时候并没有配置 `CLASSPATH` 环境变量，同时也没有在执行命令时指定任何参数，说明类加载器是根据 class path 的默认值去找到这个 Class 文件的，这个默认值就是当前路径。如下图所示：


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400850-1538959924.png)](https://github.com)


根据[官方文档](https://github.com)所说 Java 程序启动的时候会把 class path 的值放到 `java.class.path` 这个系统属性中，如下图所示：


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400882-1043734228.png)](https://github.com)


修改上面的代码，在程序运行的时候把实际的 class path 打印出来，代码如下：



```
public class Temp {  
    public static void main(String[] args) {  
        System.out.println("Executed!");  
        System.out.println("The actual class path is :" + System.getProperty("java.class.path"));  
    }  
}

```

代码执行结果如下图所示：
[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400875-106702631.png)](https://github.com)


可以看到代码打印的结果是「.」，即当前路径。


### 验证 CLASSPATH 环境变量的作用


增加 Windows 系统环境变量，因为上面是把 Temp.class 文件放到了 E 盘下面，所以这里设置的 CLASSPATH 环境变量也是 E 盘，如下图所示：


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400848-359443434.png)](https://github.com)


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400899-1671943105.png)](https://github.com)


再次运行程序，执行结果如下图所示：
[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400804-75311691.png)](https://github.com)


程序能够正常执行说明通过配置的 E: 这个路径，类加载器能够找到 `Temp.class` 文件。同时打印的 class path 也是 E: ，符合设置。


### 验证 `-cp` 或者 `-classpath` 参数的作用


把上面设置的 `CLASSPATH` 环境变量删除，然后通过执行 java 命令的时候指定 `-cp` 参数来设置 class path 的路径。如下图所示：


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400808-401891996.png)](https://github.com)


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400857-2002880540.png)](https://github.com)


程序执行的效果和通过 `CLASSPATH` 环境变量设置的相同。


### 验证 `-cp`或 `-classpath` 参数的优先级高于 `CLASSPATH` 环境变量


设置 `CLASSPATH` 环境变量为 D: ，如下图所示：


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400806-230912413.png)](https://github.com)


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400839-122539851.png)](https://github.com)


如果不带 `-cp` 参数执行执行会提示找不到类，因为 D: 路径下没有 `Temp.class` 这个文件。如下图所示：
[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400834-1503699015.png)](https://github.com)


带上 `-cp` 参数后就能够正常执行，这个时候两个配置都有，但是 `-cp` 参数的配置生效了，说明 `-cp` 参数的优先级高于 `CLASSPAHTH` 环境变量，如下图所示：
[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400807-1491236696.png)](https://github.com)


## 哪里有引导类？


### `sun.boot.class.path` 系统属性的值


引导类指的是构成 Java 平台的类，包括 rt.jar 中的类以及其他几个重要的 jar 文件中的类，它们是由引导类加载器（Bootstrap ClassLoader）加载的。


在前面可以看到如果直接在 `Temp.class` 文件所在的路径下执行 `java Temp`命令就能够正常执行。那这个 `Temp` 类的父类是 `Object` 类，这个类是在 `jre/lib` 目录下的 `rt.jar` 包中，但是没有任何地方指定了这个路径，那引导类加载器（BootstrapClassLoader） 是如何找 Object 类并加载的呢？


根据[官方文档](https://github.com)说的引导类加载器加载的 class path 可以通过 `sun.boot.class.path` 这个系统属性获取到，如下图所示：
[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400831-601370254.png)](https://github.com)


把上面的代码修改为如下：



```
public class Temp {  
    public static void main(String[] args) {  
        System.out.println("Executed!");  
        System.out.println("The actual class path is :" + System.getProperty("sun.boot.class.path"));  
    }  
}

```

通过 `java Temp` 命令执行后(不配置 `CLASSPATH` 环境变量，让它使用默认值)，结果如下：


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400870-1538029927.png)](https://github.com)


可以看到输出结果里面有 rt.jar 包的绝对路径。实际上并没有任何地方指定了这个路径，那么这个路径怎么获取到并设置到 `sun.boot.class.path` 这个系统属性中的呢？


### `sun.boot.class.path` 系统属性赋值源码分析


这里以 Windows 平台为例分析一下 HotSpot 虚拟机的源码实现。这里主要涉及到三个文件的内容，分别是：
`hotspot\src\share\vm\runtime\arguments.cpp`
`hotspot\src\share\vm\runtime\os.cpp`
`hotspot\src\os\windows\vm\os_windows.cpp`


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400893-1031315151.png)](https://github.com)


源代码的调用链路如下：


[![arguments.jpg](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400854-797629742.jpg)](https://github.com)


`arguments.cpp` 负责处理 JVM 启动的参数，在这个文件中会初始化 `_java_home` 和`_sun_boot_class_path` 系统属性，如下图所示：


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400885-1827808804.png)](https://github.com):[西部世界梯子官网](https://lalami.org)


然后调用 `os_windows.cpp` 的 `init_system_properties_values()` 方法，在该方法中又会调用 `os_windows.cpp` 中的 `jvm_path()` 方法，该方法中会尝试去获取 `jvm.dll` 的绝对路径，如下图所示：


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400890-1634386459.png)](https://github.com)


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400879-1414173540.png)](https://github.com)
[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400877-2099225419.png)](https://github.com)


然后返回到`os_windows.cpp` 的 `init_system_properties_values()` 方法，去除掉路径中的 `jvm.dll`，`server/client`，`bin` 然后放入到前面创建的 `_java_home` 系统属性中，如下图所示：


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400912-2072552421.png)](https://github.com)


然后再继续调用 `os.cpp` 中的 `set_boot_path()` 方法，在这个方法中获取 `_java_home` 系统属性中的值，用来格式化引导类 jar 包的路径，然后放入到 `_sun_boot_class_path` 中。 如下图所示：


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400884-606655374.png)](https://github.com)


这就是 `sun.boot.class.path` 系统属性值在 Java 程序启动时的设置过程。


[深入理解Java虚拟机](https://github.com)中介绍到「引导类加载器负责加载存放在/lib 目录下或者被 `-Xbootclasspath` 参数所指定的路径中存放的，且是 Java 虚拟机能够识别的（按照文件名识别，例如 rt.jar，名字不符合的类库即使是放在 lib 目录中也不会被加载）」。这里所描述的「按照文件名识别」指的应该就是上面 `os.cpp` 的 `set_boot_path()` 方法中定义的路径常量，只有这些路径常量才会被格式化最终放到 `sun.boot.class.path` 系统属性中。


目前这个系统属性在 JDK 9 中已经被移除了，如下图所示：


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400888-2142499927.png)](https://github.com)


引导类的路径可以通过 `sun.boot.class.path` 系统属性或者 `-Xbootclasspath` JVM 参数设置。


### 神奇的 `-Xbootclasspath/p` 参数


除此之外 JVM 还提供了两个参数 `-Xbootclasspath/p` 和 `-Xbootclasspath/a`，分别用于在默认的引导类路径前面和后面增加所配置的路径。如下图所示：


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400935-1313464796.png)](https://github.com)


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400917-651089151.png)](https://github.com)


`-Xbootclasspath/p` 这个参数有点意思，它可以用来修复引导类的 Bug 或者扩展类的功能。


比如现在把 `java.util.Collections` 类拷贝出来，给它增加一个方法 `extendMethod()`，然后打包成 jar 包，如下图所示：


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400801-1964854146.png)](https://github.com)
[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400947-708567613.png)](https://github.com)


在代码中通过反射的方式调用 `extendMethod()` 方法，代码如下所示：



```
public class Temp {  
    public static void main(String[] args) throws Exception {  
        Method method = Collections.class.getDeclaredMethod("extendMethod");  
        method.invoke(null);  
    }  
}

```

在执行 java 命令时通过 `-Xbootclasspath/p` 配置上这个 jar 包。可以看到新增的方法被成功调用了，说明 `extend.jar` 包中的 `Collections` 类覆盖了默认的 `java.util.Collections` 类，因为它在所有的路径前面，所以先被类加载器加载。如下图所示：


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400895-1694870917.png)](https://github.com)


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400950-1549125271.png)](https://github.com)


这个参数在 JDK 9 中也被移除了，取而代之的是 `--patch-module` 参数，如下图所示：


[![image.png](https://img2024.cnblogs.com/blog/1878162/202412/1878162-20241225200400887-1940428450.png)](https://github.com)


## 参考


[findingclasses](https://github.com)
[PATH and CLASSPATH](https://github.com)
[JDK 9 Release Notes](https://github.com)
[JEP 261](https://github.com)
[How can we overwrite java.base/java.lang.Integer from OpenJDK 11 using \-\-patch\-module?](https://github.com)
[深入理解Java虚拟机](https://github.com)


