
## 前言

在聊 Dubbo 的 SPI 之前，对 SPI 机制还不是很了解的小伙伴可以先简单了解一下 JDK 的 SPI 机制。

在 Dubbo 中，SPI 是一个非常重要的模块，贯穿整个 Dubbo 框架，以下模块的扩展都是基于 SPI 机制实现的。其实 SPI 用一句话概括就是在程序运行时，动态为接口根据条件生成对应的实现类。

![](https://img2020.cnblogs.com/blog/1326851/202010/1326851-20201026180143277-2109837406.png)

## 文章脉络

本篇文章按照先后顺序包含以下几个模块：

1. Dubbo SPI 使用案例
2. Dubbo SPI 源码分析
3. Dubbo SPI 大致流程图

Dubbo 并未使用 Java SPI，而是重新实现了一套功能更强的 SPI 机制。Dubbo SPI 的相关逻辑被封装在了 ExtensionLoader 类中，通过 ExtensionLoader，我们可以加载指定的实现类。Dubbo SPI 所需的配置文件需放置在 META-INF/dubbo 路径下，配置内容如下。

## Dubbo SPI 示例

![](https://img2020.cnblogs.com/blog/1326851/202010/1326851-20201026180222454-1752089915.png)

> 注意，下面的案例来源于 Dubbo 框架 dubbo-common 模块的改造，读者也可以参看 Dubbo 源代码部分

![](https://img2020.cnblogs.com/blog/1326851/202010/1326851-20201027153602839-2102413431.png)

接口
```Java
@SPI
public interface SimpleExt {
    // @Adaptive example, do not specify a explicit key.

    String echo(URL url, String s);


    String yell(URL url, String s);

    // no @Adaptive
    String bang(URL url, int i);
}
```

实现类一
```Java
public class SimpleExtImpl1 implements SimpleExt {
    public String echo(URL url, String s) {
        return "Ext1Impl1-echo";
    }

    public String yell(URL url, String s) {
        return "Ext1Impl1-yell";
    }

    public String bang(URL url, int i) {
        return "bang1";
    }
}
```

实现类二
```Java
public class SimpleExtImpl2 implements SimpleExt {
    public String echo(URL url, String s) {
        return "Ext1Impl2-echo";
    }

    public String yell(URL url, String s) {
        return "Ext1Impl2-yell";
    }

    public String bang(URL url, int i) {
        return "bang2";
    }

}
```

实现类三
```Java
public class SimpleExtImpl3 implements SimpleExt {
    public String echo(URL url, String s) {
        return "Ext1Impl3-echo";
    }

    public String yell(URL url, String s) {
        return "Ext1Impl3-yell";
    }

    public String bang(URL url, int i) {
        return "bang3";
    }

}
```

测试用例：
```Java
@Test
public void test_getExtension() throws Exception {
    System.out.println(ExtensionLoader.getExtensionLoader(SimpleExt.class).getExtension("impl1").echo(new URL("","",9001),""));
    System.out.println(ExtensionLoader.getExtensionLoader(SimpleExt.class).getExtension("impl2").echo(new URL("","",9001),""));
}
```

测试结果：
```
Ext1Impl1-echo
Ext1Impl2-echo
```

## Dubbo SPI 源码分析

接下来对 Dubbo SPI 机制进行源码分析，在源码分析的过程中会对提到 **Dubbo 相对于 JDK SPI 方式的优点在哪里**，希望能加深小伙伴们对两种 SPI 方式的理解

> 注意，本文源码分析基于 dubbo 2.6.x 版本

**Dubbo SPI 源代码目录：**

![](https://img2020.cnblogs.com/blog/1326851/202010/1326851-20201027153637776-732718319.png)

```Java
public T getExtension(String name) {
    if (name == null || name.length() == 0)
        throw new IllegalArgumentException("Extension name == null");
    if ("true".equals(name)) {
        // 获取默认的拓展实现类
        return getDefaultExtension();
    }
    // Holder，顾名思义，用于持有目标对象
    Holder<Object> holder = cachedInstances.get(name);
    if (holder == null) {
        cachedInstances.putIfAbsent(name, new Holder<Object>());
        holder = cachedInstances.get(name);
    }
    Object instance = holder.get();
    if (instance == null) {
        synchronized (holder) {
            instance = holder.get();
            if (instance == null) {
                // 创建拓展实例
                instance = createExtension(name);
                // 设置实例到 holder 中
                holder.set(instance);
            }
        }
    }
    return (T) instance;
}
```

```Java
private T createExtension(String name) {
    // 从配置文件中加载所有的拓展类缓存到 cachedClasses，并取出 name 对应的 Class
    Class<?> clazz = getExtensionClasses().get(name); // @1
    if (clazz == null) {
        throw findException(name);
    }
    try {
        T instance = (T) EXTENSION_INSTANCES.get(clazz);
        if (instance == null) {
            // 通过反射创建实例
            EXTENSION_INSTANCES.putIfAbsent(clazz, clazz.newInstance());  //@2
            instance = (T) EXTENSION_INSTANCES.get(clazz);
        }
        // 向实例中注入依赖
        injectExtension(instance); // @3
        Set<Class<?>> wrapperClasses = cachedWrapperClasses;
        if (wrapperClasses != null && !wrapperClasses.isEmpty()) {
            // 循环创建 Wrapper 实例
            for (Class<?> wrapperClass : wrapperClasses) {
                // 将当前 instance 作为参数传给 Wrapper 的构造方法，并通过反射创建 Wrapper 实例。
                // 然后向 Wrapper 实例中注入依赖，最后将 Wrapper 实例再次赋值给 instance 变量
                instance = injectExtension(
                    (T) wrapperClass.getConstructor(type).newInstance(instance));
            }
        }
        return instance;
    } catch (Throwable t) {
        throw new IllegalStateException("...");
    }
}
```

```createExtension()``` 方法的逻辑稍复杂一下，包含了如下的步骤：

1. 通过 ```getExtensionClasses()``` 获取所有的拓展类缓存到
```Java
private final Holder<Map<String, Class<?>>> cachedClasses = new Holder<Map<String, Class<?>>>();
```
2. 通过反射 ```clazz.newInstance();``` 创建拓展对象。**注意，这里就能看出来和 JDK 默认的 SPI 的区别，这里是实例化所需要的扩展实例。**
3. ```injectExtension(instance);``` 向拓展对象中注入依赖
4. 将拓展对象包裹在相应的 Wrapper 对象中，返回 Wrapper 对象

以上步骤中，第一个步骤是加载拓展类的关键，第三和第四个步骤是 Dubbo IOC 与 AOP 的具体实现。在接下来的章节中，将会重点分析 getExtensionClasses() 方法的逻辑，以及简单介绍 Dubbo IOC 的具体实现。

### 获取所有的拓展类

我们在通过名称获取拓展类之前，首先需要根据配置文件解析出拓展项名称到拓展类的映射关系表（Map<名称, 拓展类>），之后再根据拓展项名称从映射关系表中取出相应的拓展类即可。相关过程的代码分析如下：

```Java
private Map<String, Class<?>> getExtensionClasses() {
    // 从缓存中获取已加载的拓展类
    Map<String, Class<?>> classes = cachedClasses.get();
    // 双重检查
    if (classes == null) {
        synchronized (cachedClasses) {
            classes = cachedClasses.get();
            if (classes == null) {
                // 加载拓展类
                classes = loadExtensionClasses();
                cachedClasses.set(classes);
            }
        }
    }
    return classes;
}
```

这里也是先检查缓存，若缓存未命中，则通过 ```loadExtensionClasses()``` 加载拓展类。下面进行详细分析

```Java
private Map<String, Class<?>> loadExtensionClasses() {
    // 获取 SPI 注解
    final SPI defaultAnnotation = type.getAnnotation(SPI.class);
    if (defaultAnnotation != null) {
        String value = defaultAnnotation.value();
        if ((value = value.trim()).length() > 0) {
            // 对 SPI 注解内容进行切分
            String[] names = NAME_SEPARATOR.split(value);
            // 检测 SPI 注解内容是否合法，不合法则抛出异常
            if (names.length > 1) {
                throw new IllegalStateException("more than 1 default extension name on extension...");
            }

            // 设置默认名称，参考 getDefaultExtension 方法
            if (names.length == 1) {
                cachedDefaultName = names[0];
            }
        }
    }

    Map<String, Class<?>> extensionClasses = new HashMap<String, Class<?>>();
    // 加载指定文件夹下的配置文件
    loadDirectory(extensionClasses, DUBBO_INTERNAL_DIRECTORY);
    loadDirectory(extensionClasses, DUBBO_DIRECTORY);
    loadDirectory(extensionClasses, SERVICES_DIRECTORY);
    return extensionClasses;
}
```

```loadExtensionClasses()``` 方法总共做了两件事情，一是对 SPI 注解进行解析，二是调用 ```loadDirectory()``` 方法加载 **指定文件夹** 配置文件。

到这里 interface 所有的扩展类 class 都生成完毕，并且以 Map 的形式都缓存到
```Java
private final Holder<Map<String, Class<?>>> cachedClasses = new Holder<Map<String, Class<?>>>();
```
中，其中 Map 的 key 是 配置文件中 key 部分，value 部分则是具体实现类的 Class 对象。

@2 步根据 Class 对象创建了一个对象，并将其缓存到
```Java
private static final ConcurrentMap<Class<?>, Object> EXTENSION_INSTANCES = new ConcurrentHashMap<Class<?>, Object>();
```
里面。

@3 步就是 Dubbo 的自动注入部分，也就是 Dubbo 的 IOC 容器部分，注入上一步生成的 instance 实例所需的属性。

Dubbo IOC 是通过 ```setter()``` 方法注入依赖。Dubbo 首先会通过反射获取到实例的所有方法，然后再遍历方法列表，检测方法名是否具有 setter 方法特征。若有，则通过 ObjectFactory 获取依赖对象，最后通过反射调用 setter 方法将依赖设置到目标对象中。整个过程对应的代码如下：

```Java
private T injectExtension(T instance) {
    try {
        if (objectFactory != null) {
            // 遍历目标类的所有方法
            for (Method method : instance.getClass().getMethods()) {
                // 检测方法是否以 set 开头，且方法仅有一个参数，且方法访问级别为 public
                if (method.getName().startsWith("set")
                    && method.getParameterTypes().length == 1
                    && Modifier.isPublic(method.getModifiers())) {
                    // 获取 setter 方法参数类型
                    Class<?> pt = method.getParameterTypes()[0];
                    try {
                        // 获取属性名，比如 setName 方法对应属性名 name
                        String property = method.getName().length() > 3 ?
                            method.getName().substring(3, 4).toLowerCase() +
                            	method.getName().substring(4) : "";
                        // 从 ObjectFactory 中获取依赖对象
                        Object object = objectFactory.getExtension(pt, property); // @4 根据属性名称和属性对象 Class 类型获取该属性对象
                        if (object != null) {
                            // 通过反射调用 setter 方法设置依赖
                            method.invoke(instance, object);
                        }
                    } catch (Exception e) {
                        logger.error("fail to inject via method...");
                    }
                }
            }
        }
    } catch (Exception e) {
        logger.error(e.getMessage(), e);
    }
    return instance;
}
```

根据对象名称和 Class 类型获取对象
```Java
private ExtensionLoader(Class<?> type) {
    this.type = type;
    objectFactory = (type == ExtensionFactory.class ? null : ExtensionLoader.getExtensionLoader(ExtensionFactory.class).getAdaptiveExtension());
}
```

**ExtensionFactory** 实现类：

![](https://img2020.cnblogs.com/blog/1326851/202010/1326851-20201027105458363-678830056.png)

```Java
@Adaptive
public class AdaptiveExtensionFactory implements ExtensionFactory {

    private final List<ExtensionFactory> factories; // @5

    public AdaptiveExtensionFactory() {
        ExtensionLoader<ExtensionFactory> loader = ExtensionLoader.getExtensionLoader(ExtensionFactory.class);
        List<ExtensionFactory> list = new ArrayList<ExtensionFactory>();
        for (String name : loader.getSupportedExtensions()) {  //@6
            list.add(loader.getExtension(name));
        }
        factories = Collections.unmodifiableList(list);
    }

    @Override
    public <T> T getExtension(Class<T> type, String name) {  // @7
        for (ExtensionFactory factory : factories) {
            T extension = factory.getExtension(type, name);
            if (extension != null) {
                return extension;
            }
        }
        return null;
    }

}
```

```Java
public Set<String> getSupportedExtensions() {
    // 获取所有的 Class
    Map<String, Class<?>> clazzes = getExtensionClasses();
    return Collections.unmodifiableSet(new TreeSet<String>(clazzes.keySet()));
}
```

在 @4 处，```objectFactory``` 变量的类型为 ```AdaptiveExtensionFactory```，AdaptiveExtensionFactory 内部维护了一个 ExtensionFactory 列表，用于 ExtensionFactory 所有的实现。Dubbo 目前提供了两种 ExtensionFactory，分别是 ```SpiExtensionFactory``` 和 ```SpringExtensionFactory```，前者用于创建自适应的拓展，后者是用于从 Spring 的 IOC 容器中获取所需的拓展。而 AdaptiveExtensionFactory 中的 ```factories``` 通过代码 @6 处持有所有的 ExtensionFactory 实现，变成一个列表。所以代码 @4 最终获取注入的属性对象最终就会走 @7 ，然后走 SpiExtensionFactory 和 SpringExtensionFactory 轮流获取，只要获取到就立即返回。

Dubbo IOC 目前仅支持 setter 方式注入，总的来说，逻辑比较简单易懂。

**以上 Dubbo SPI 源代码可以大致总结为下面的流程图**

![](https://img2020.cnblogs.com/blog/1326851/202010/1326851-20201027153711207-1212430025.png)

## 总结

本篇文章简单分别介绍了 Dubbo SPI 用法，并对 Dubbo SPI 的加载拓展类的过程进行了分析。另外，在 Dubbo SPI 中还有一块重要的逻辑这里没有进行分析，即 **Dubbo SPI 的扩展点自适应机制**。该机制的逻辑较为复杂，我们将会在下一篇文章中进行详细的分析。
