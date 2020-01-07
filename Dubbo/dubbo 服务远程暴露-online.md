## 引言

研究源码是一个非常枯燥并且充满疑惑的过程，因为在研究的过程中可能存在诸多疑点，在这里和大家分享一句话：“抓住主要，忽略次要”，一个人的精力是有限的，要想在有限的时间内更高效地学习，相信这句话会对你有所帮助。另外，笔者研究的源码基于 <code><font color="#de2c58">2.6.7</font></code> 版本，不想自己搭 Project 的朋友可以去 Dubbo 官网或者笔者的 [github](https://github.com/tiankaizhi/dubbo/tree/2.6.x) 仓库去下载。

下面我列出了一些 Dubbo 服务提供者在暴露服务过程中的一些重要部分，本篇文章并不会对以下所有的点进行详细的分析，未分析到的会在后面的系列文章中有所介绍。

1. Dubbo 延时暴露机制是如何实现的

2. 何时向注册中心注册服务

3. 服务提供者与注册中心的心跳机制

## 服务暴露总体过程

在详细研究服务暴露细节之前，我们先看一下整体 RPC 的暴露原理，如图 5-4。

从整体上看，Dubbo 框架做服务暴露分为两大部分，第一步将持有的服务实例通过代理转换成 Invoker，第二步把 Invoker 通过具体的协议（比如 DubboProtocol） 转换成 Exporter，框架做了这层抽象大大方便了功能扩展。这里的 Invoker 可以简单的理解成一个真实的服务对象实例，是 Dubbo 框架实体域，所有模型都会向它靠拢，可向他发起 invoke 调用。它可能是一个本地实现，也可能是一个远程实现，还可能是一个集群实现。

![](https://img2018.cnblogs.com/blog/1326851/202001/1326851-20200103152846267-1509479241.png)

Dubbo 和 Spring 整合后，服务暴露触发时机是当 Spring 容器实例化 bean 完成，走到最后一步发布 <code><font color="#de2c58">ContextRefreshEvent</font></code> 事件的时候，<code><font color="#de2c58">ServiceBean </font></code> 会执行 <code><font color="#de2c58">onApplicationEvent</font></code> 方法，该方法调用 <code><font color="#de2c58">ServiceConfig</font></code> 的 <code><font color="#de2c58">export</font></code> 方法。

## ServiceBean

### onApplicationEvent(ContextRefreshedEvent event)

```java
@Override
public void onApplicationEvent(ContextRefreshedEvent event) {
    if (isDelay() && !isExported() && !isUnexported()) {
        if (logger.isInfoEnabled()) {
            logger.info("The service ready on spring started. service: " + getInterface());
        }
        export();
    }
}
```

## ServiceConfig

### export()

```java
public synchronized void export() {
    if (provider != null) {
        if (export == null) {
            export = provider.getExport();
        }
        if (delay == null) {
            delay = provider.getDelay();
        }
    }
    if (export != null && !export) {  // @1
        return;
    }

    if (delay != null && delay > 0) {  // @2
        delayExportExecutor.schedule(new Runnable() {
            @Override
            public void run() {
                doExport();
            }
        }, delay, TimeUnit.MILLISECONDS);
    } else {
        doExport();
    }
}
```

代码 @1，判断是否暴露服务，根据 <code><font color="#de2c58">dubbo:service export="true|false"</font></code> 设置。

代码 @2，如果 <code><font color="#de2c58">delay</font></code> 大于 0，表示延迟多少毫秒后暴露服务，延迟暴露采用的是 JDK 的 <code><font color="#de2c58">ScheduledExecutorService</font></code> 进行调度的。

```java
private static final ScheduledExecutorService delayExportExecutor = Executors.newSingleThreadScheduledExecutor(new NamedThreadFactory("DubboServiceDelayExporter", true));
```

### doExport()

<code><font color="#de2c58">ServiceConfig</font></code> 的 <code><font color="#de2c58">export</font></code> 方法会调用 <code><font color="#de2c58">ServiceConfig</font></code> 的 <code><font color="#de2c58">doExport</font></code> 方法，到 319 行处。接下来调用 <code><font color="#de2c58">ServiceConfig</font></code> 的 <code><font color="#de2c58">doExportUrls</font></code> 正式开始暴露服务。

![](https://img2018.cnblogs.com/blog/1326851/202001/1326851-20200103154204128-608191016.png)

### doExportUrls()

```java
@SuppressWarnings({"unchecked", "rawtypes"})
private void doExportUrls() {

    // @1 加载所有的注册中心 URL 地址
    List<URL> registryURLs = loadRegistries(true);

    // @2 遍历所有协议, 按照协议依次向每个注册中心暴露服务
    for (ProtocolConfig protocolConfig : protocols) {
        doExportUrlsFor1Protocol(protocolConfig, registryURLs);
    }
}
```

代码 @1，调用 ServiceConfig 父类 AbstractInterfaceConfig 的 <code><font color="#de2c58">loadRegistries(boolean provider)</font></code> 方法。参数 **true** 代表服务提供者，**false** 代表服务消费者，如果是服务提供者。

### loadRegistries(true)

```java
    /**
     * 加载注册表并将其转换为{@link URL}，优先级顺序为：系统属性> dubbo注册表配置
     * Load the registry and conversion it to {@link URL}, the priority order is: system property > dubbo registry config
     *
     * 是否是提供方
     * @param provider whether it is the provider side
     * @return
     */
    protected List<URL> loadRegistries(boolean provider) {
        // 校验 RegistryConfig 配置数组。
        checkRegistry();
        List<URL> registryList = new ArrayList<URL>();
        if (registries != null && !registries.isEmpty()) {
            for (RegistryConfig config : registries) {
                // 获得注册中心的地址
                String address = config.getAddress();
                if (address == null || address.length() == 0) {
                    address = Constants.ANYHOST_VALUE;
                }
                String sysaddress = System.getProperty("dubbo.registry.address");  // 系统属性，优先级最高，可以覆盖
                if (sysaddress != null && sysaddress.length() > 0) {
                    address = sysaddress;
                }
                // 地址 address 是有效地址，包括 address != N/A ("N/A" 代表不配置注册中心)
                if (address.length() > 0 && !RegistryConfig.NO_AVAILABLE.equalsIgnoreCase(address)) {
                    Map<String, String> map = new HashMap<String, String>();
                    // 将各种配置对象，添加到 `map` 集合中。
                    appendParameters(map, application);
                    appendParameters(map, config);
                    // 添加 `path` `dubbo` `timestamp` `pid` 到 `map` 集合中。
                    map.put("path", RegistryService.class.getName());
                    map.put("dubbo", Version.getProtocolVersion());
                    map.put(Constants.TIMESTAMP_KEY, String.valueOf(System.currentTimeMillis()));
                    if (ConfigUtils.getPid() > 0) {
                        map.put(Constants.PID_KEY, String.valueOf(ConfigUtils.getPid()));
                    }
                    // 若不存在 `protocol` 参数，默认 "dubbo" 添加到 `map` 集合中。
                    if (!map.containsKey("protocol")) {
                        if (ExtensionLoader.getExtensionLoader(RegistryFactory.class).hasExtension("remote")) { // "remote" 可以忽略。因为，remote 这个拓展实现已经不存在。
                            map.put("protocol", "remote");
                        } else {
                            map.put("protocol", "dubbo");
                        }
                    }
                    // 解析地址，创建 Dubbo URL 数组。（数组大小可以为一）
                    List<URL> urls = UrlUtils.parseURLs(address, map);  // @1
                    // 循环 `url` ，设置 "registry" 和 "protocol" 属性。
                    for (URL url : urls) {
                        // 设置 `registry=${protocol}` 和 `protocol=registry` 到 URL
                        url = url.addParameter(Constants.REGISTRY_KEY, url.getProtocol());
                        // 添加到结果
                        url = url.setProtocol(Constants.REGISTRY_PROTOCOL);
                        if ((provider && url.getParameter(Constants.REGISTER_KEY, true)) // @2 服务提供者 && 注册
                                || (!provider && url.getParameter(Constants.SUBSCRIBE_KEY, true))) { // @3 服务消费者 && 订阅
                            registryList.add(url);
                        }
                    }
                }
            }
        }
        return registryList;
    }
```

代码 @1，这个方法从名字上很难看出具体的含义，看下图的注释

![](https://img2018.cnblogs.com/blog/1326851/202001/1326851-20200105002131763-2003783240.png)

这里有一个问题，如果服务暴露是多注册中心或者是一个注册的中心集群，从配置获取到的 ```List<RegistryConfig>``` 和 loadRegistries 加载之后的注册中心 ```List<URL>``` 分别是什么样子的。

**单注册中心单例情况下:**

```xml
<dubbo:registry id="first-registry" protocol="zookeeper" address="192.168.25.128:2181"/>
```

首先 AbstractInterfaceConfig 的 ```protected List<RegistryConfig> registries```

![](https://img2018.cnblogs.com/blog/1326851/202001/1326851-20200105002252990-1784770658.png)

我们看一下加载完的 ```List<URL> registryURLs``` 值：

![](https://img2018.cnblogs.com/blog/1326851/202001/1326851-20200105002338292-1916124233.png)

**多注册中心，每个注册中心有多个实例的情况下：**

AbstractInterfaceConfig 的 ```protected List<RegistryConfig> registries``` 值：

![](https://img2018.cnblogs.com/blog/1326851/202001/1326851-20200105002400911-1493662432.png)

加载完的 ```List<URL> registryURLs``` 值：

![](https://img2018.cnblogs.com/blog/1326851/202001/1326851-20200105002419066-1508346108.png)

代码 @2，若是服务提供者，判断是否只订阅不注册。如果是，不添加结果到 registryList 中。对应 [《Dubbo 用户指南 —— 只订阅》](http://dubbo.apache.org/zh-cn/docs/user/demos/subscribe-only.html) 文档。
代码 @3，若是服务消费者，判断是否只注册不订阅。如果是，不添加到结果 registryList 。对应 [《Dubbo 用户指南 —— 只注册》](http://dubbo.apache.org/zh-cn/docs/user/demos/registry-only.html) 文档。

### doExportUrlsFor1Protocol(protocolConfig, registryURLs)

加载完所有配置的注册中心 URL 之后，开始按照协议进行暴露。Dubbo 支持相同服务暴露多个协议。

我们先看一下协议和注册中心入参

![](https://img2018.cnblogs.com/blog/1326851/202001/1326851-20200103154904068-2023144680.png)

```java
private void doExportUrlsFor1Protocol(ProtocolConfig protocolConfig, List<URL> registryURLs) {
    String scope = url.getParameter(Constants.SCOPE_KEY);
    // don't export when none is configured
    if (!Constants.SCOPE_NONE.toString().equalsIgnoreCase(scope)) {

        // 本地暴露（这里暂时不分析，后面就本地暴露写一篇文章进行详细地分析）
        // export to local if the config is not remote (export to remote only when config is remote)
        if (!Constants.SCOPE_REMOTE.toString().equalsIgnoreCase(scope)) {
            exportLocal(url);
        }
        // export to remote if the config is not local (export to local only when config is local)
        if (!Constants.SCOPE_LOCAL.toString().equalsIgnoreCase(scope)) {
            if (logger.isInfoEnabled()) {
                logger.info("Export dubbo service " + interfaceClass.getName() + " to url " + url);
            }
            if (registryURLs != null && !registryURLs.isEmpty()) {
                for (URL registryURL : registryURLs) {
                    // @1
                    url = url.addParameterIfAbsent(Constants.DYNAMIC_KEY, registryURL.getParameter(Constants.DYNAMIC_KEY));
                    // 获得监控中心 URL
                    URL monitorUrl = loadMonitor(registryURL);
                    if (monitorUrl != null) {
                        url = url.addParameterAndEncoded(Constants.MONITOR_KEY, monitorUrl.toFullString()); // @2
                    }
                    if (logger.isInfoEnabled()) {
                        logger.info("Register dubbo service " + interfaceClass.getName() + " url " + url + " to registry " + registryURL);
                    }

                    // For providers, this is used to enable custom proxy to generate invoker
                    String proxy = url.getParameter(Constants.PROXY_KEY);
                    if (StringUtils.isNotEmpty(proxy)) {
                        registryURL = registryURL.addParameter(Constants.PROXY_KEY, proxy);
                    }

                    // @3
                    // 根据具体实现类, 实现接口, 以及 registryUrl, 通过 ProxyFactory 将 DemoServiceImpl 封装成一个具体的本地执行的 Invoker
                    // invoker 是 DemoServiceImpl 的代理对象, 具体是怎样动态代理生成的？为什么传递的是注册中心的 URL 呢？后面会详细分析.
                    Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass, registryURL.addParameterAndEncoded(Constants.EXPORT_KEY, url.toFullString()));

                    // @4
                    DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);

                    // 核心代码 @5
                    // 使用 Protocol 将 invoker 封装成 Exporter
                    // 调用 Protocol 生成的适配类的 export 方法
                    Exporter<?> exporter = protocol.export(wrapperInvoker);
                    exporters.add(exporter);
                }
            } else {
                Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass, url);
                DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);

                Exporter<?> exporter = protocol.export(wrapperInvoker);
                exporters.add(exporter);
            }
        }
    }
}
```

代码 @1，<code><font color="#de2c58">dynamic</font></code> 配置项，服务是否动态注册。如果设为 **false** ，注册后将显示后 **disable** 状态，需人工启用，并且服务提供者停止时，也不会自动取消册，需人工禁用。

代码 @2，调用 <code><font color="#de2c58">URL#addParameterAndEncoded(key, value)</font></code> 方法，将监控中心的 URL 作为 <code><font color="#de2c58">monitor</font></code> 参数添加到服务提供者的 URL 中，并且需要编码。通过这样的方式，服务提供者的 URL 中，**包含了监控中心的配置**。

代码 @3，调用 <code><font color="#de2c58">URL#addParameterAndEncoded(key, value)</font></code> 方法，将服务体用这的 URL 作为 <code><font color="#de2c58">export</font></code> 参数添加到注册中心的 URL 中。通过这样的方式，注册中心的 URL 中，**包含了服务提供者的配置**。

代码 @3，调用 <code><font color="#de2c58">ProxyFactory#getInvoker(proxy, type, url)</font></code> 方法，创建 Invoker 对象。该 Invoker 对象，执行 #invoke(invocation) 方法时，内部会调用 Service 对象( ref )对应的调用方法。

代码 @4，创建 <code><font color="#de2c58">com.alibaba.dubbo.config.invoker.DelegateProviderMetaDataInvoker</font></code> 对象。该对象在 Invoker 对象的基础上，增加了当前服务提供者 ServiceConfig 对象

## Protocol

### export(Invoker<T> invoker) throws RpcException

此处 Dubbo SPI 自适应的特性的好处就出来了，可以自动根据 URL 参数，获得对应的拓展实现。例如，invoker 传入后，根据 invoker.url 自动获得对应 Protocol 拓展实现为 DubboProtocol 。

实际上，Protocol 有两个 Wrapper 拓展实现类： ProtocolFilterWrapper、ProtocolListenerWrapper 。所以，#export(...) 方法的调用顺序是：

```java
Protocol$Adaptive => ProtocolFilterWrapper => ProtocolListenerWrapper => RegistryProtocol
=>
Protocol$Adaptive => ProtocolFilterWrapper => ProtocolListenerWrapper => DubboProtocol
```

也就是说，**这一条大的调用链，包含两条小的调用链**。原因是：

首先，传入的是注册中心的 URL ，通过 ```Protocol$Adaptive``` 获取到的是 RegistryProtocol 对象。
其次，RegistryProtocol 会在其 <code><font color="#de2c58">#export(...)</font></code> 方法中，使用服务提供者的 URL ( 即注册中心的 URL 的 <code><font color="#de2c58">export</font></code> 参数值)，再次调用 Protocol$Adaptive 获取到的是 DubboProtocol 对象，进行服务暴露。

为什么是这样的顺序？通过这样的顺序，可以实现类似 AOP 的效果，在本地服务器启动完成后，再向注册中心注册。伪代码如下：

```java
RegistryProtocol#export(...) {

    // 1. 启动本地服务器
    DubboProtocol#export(...);

    // 2. 向注册中心注册。
}
```

这也是为什么上文提到的 “为什么传递的是注册中心的 URL 呢？” 的原因。

## RegistryProtocol

**com.alibaba.dubbo.registry.integration.RegistryProtocol** ，实现 **Protocol** 接口，注册中心协议实现类。

看一下本文涉及到的 **Protocol**

![](https://img2018.cnblogs.com/blog/1326851/202001/1326851-20200103155042831-1920242908.png)

> **友情提示，仅包含本文涉及的属性。**

```java
// ... 省略部分和本文无关的属性。

/**
 * 单例。在 Dubbo SPI 中，被初始化，有且仅有一次。
 */
private static RegistryProtocol INSTANCE;

/**
 * 绑定关系集合。
 *
 * key：服务 Dubbo URL
 */
// To solve the problem of RMI repeated exposure port conflicts, the services that have been exposed are no longer exposed.
// 用于解决 rmi 重复暴露端口冲突的问题，已经暴露过的服务不再重新暴露
// providerurl <--> exporter
private final Map<String, ExporterChangeableWrapper<?>> bounds = new ConcurrentHashMap<String, ExporterChangeableWrapper<?>>();

/**
 * Protocol 自适应拓展实现类，通过 Dubbo SPI 自动注入。
 */
private Protocol protocol;

/**
 * RegistryFactory 自适应拓展实现类，通过 Dubbo SPI 自动注入。
 */
private RegistryFactory registryFactory;

public RegistryProtocol() {
    INSTANCE = this;
}

public static RegistryProtocol getRegistryProtocol() {
    if (INSTANCE == null) {
        ExtensionLoader.getExtensionLoader(Protocol.class).getExtension(Constants.REGISTRY_PROTOCOL); // load
    }
    return INSTANCE;
}
```

+ <code><font color="#de2c58">INSTANCE</font></code> 静态属性，单例。通过 Dubbo SPI 加载创建，有且仅有一次。
  + <code><font color="#de2c58">#getRegistryProtocol()</font></code> 静态方法，获得单例。
+ <code><font color="#de2c58">bounds</font></code> 属性，绑定关系集合。其中，Key 为服务提供者 URL 。
+ <code><font color="#de2c58">protocol</font></code> 属性，Protocol 自适应拓展实现类，通过 Dubbo SPI 自动注入。
+ <code><font color="#de2c58">registryFactory</font></code> 属性，自适应拓展实现类，通过 Dubbo SPI 自动注入。
  + 用于创建注册中心 Registry 对象。


### export(final Invoker<T> originInvoker)

```java
public <T> Exporter<T> export(final Invoker<T> originInvoker) throws RpcException {
    // export invoker
    // 暴露服务
    final ExporterChangeableWrapper<T> exporter = doLocalExport(originInvoker); // @1

    // 获得注册中心 URL
    URL registryUrl = getRegistryUrl(originInvoker);

    // registry provider
    // 根据 invoker 中的 url 获得注册中心对象 Registry 实例
    final Registry registry = getRegistry(originInvoker);

    // 获取注册到注册中心的 URL
    final URL registeredProviderUrl = getRegisteredProviderUrl(originInvoker);

    // to judge to delay publish whether or not
    boolean register = registeredProviderUrl.getParameter("register", true);

    // 向本地注册表，注册服务提供者
    ProviderConsumerRegTable.registerProvider(originInvoker, registryUrl, registeredProviderUrl);

    // 向注册中心注册服务提供者（自己）
    if (register) {
        // 若有消费者订阅此服务, 则推送消息让消费者引用此服务
        // 注册中心缓存了所有提供者注册的服务以供消费者发现
        register(registryUrl, registeredProviderUrl);
        ProviderConsumerRegTable.getProviderWrapper(originInvoker).setReg(true);
    }

    // Subscribe the override data
    // FIXME When the provider subscribes, it will affect the scene : a certain JVM exposes the service and call the same service. Because the subscribed is cached key with the name of the service, it causes the subscription information to cover.
    final URL overrideSubscribeUrl = getSubscribedOverrideUrl(registeredProviderUrl);
    final OverrideListener overrideSubscribeListener = new OverrideListener(overrideSubscribeUrl, originInvoker);
    overrideListeners.put(overrideSubscribeUrl, overrideSubscribeListener);
    registry.subscribe(overrideSubscribeUrl, overrideSubscribeListener);
    //Ensure that a new exporter instance is returned every time export
    return new DestroyableExporter<T>(exporter, originInvoker, overrideSubscribeUrl, registeredProviderUrl);
}
```

代码 @1，调用 <code><font color="#de2c58">#doLocalExport(invoker)</font></code> 方法，暴露服务

### doLocalExport(final Invoker<T> originInvoker)

```java
/**
  * 暴露服务。
  *
  * 此处的 Local 指的是，本地启动服务，但是不包括向注册中心注册服务的意思。
  *
  * @param originInvoker 原始 Invoker
  * @param <T> 泛型
  * @return Exporter 对象
  */
@SuppressWarnings("unchecked")
private <T> ExporterChangeableWrapper<T> doLocalExport(final Invoker<T> originInvoker) {
    // 获得在 `bounds` 中的缓存 Key
    String key = getCacheKey(originInvoker);  // @1
    // 从 `bounds` 获得，是不是已经暴露过服务
    ExporterChangeableWrapper<T> exporter = (ExporterChangeableWrapper<T>) bounds.get(key);  // @2
    if (exporter == null) {
        synchronized (bounds) {
            exporter = (ExporterChangeableWrapper<T>) bounds.get(key);
            // 未暴露过，进行暴露服务
            if (exporter == null) {
                // @3
                // 创建 Invoker Delegate 对象
                final Invoker<?> invokerDelegete = new InvokerDelegete<T>(originInvoker, getProviderUrl(originInvoker));

                // @4
                // 暴露服务，创建 Exporter 对象
                // 使用 创建的 Exporter对象 + originInvoker ，创建 ExporterChangeableWrapper 对象
                exporter = new ExporterChangeableWrapper<T>((Exporter<T>) protocol.export(invokerDelegete),  originInvoker);
                // 添加到 `bounds`
                bounds.put(key, exporter);
            }
        }
    }
    return exporter;
}

```

代码 @2，根据 @1 中的 key 拿到 ExporterChangeableWrapper, 如果为空则未暴露过。

代码 @3，创建 **com.alibaba.dubbo.registry.integration.RegistryProtocol.InvokerDelegete** 对象。
**InvokerDelegete** 实现 **com.alibaba.dubbo.rpc.protocol.InvokerWrapper** 类，主要增加了 **#getInvoker()** 方法，获得真实的，非 InvokerDelegete 的 Invoker 对象。因为，可能会存在 InvokerDelegete.invoker 也是 InvokerDelegete 类型的情况。

代码 @4，调用 <code><font color="#de2c58">DubboProtocol#export(invoker)</font></code> 方法，暴露服务，返回 Exporter 对象。
使用【创建的 Exporter 对象】+【originInvoker】，创建 ExporterChangeableWrapper 对象。这样，originInvoker 就和 Exporter 对象，形成了绑定的关系。

## DubboProtocol

**com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol**，实现 **AbstractProtocol** 抽象类，Dubbo 协议实现类。

**属性相关，代码如下：**

> **友情提示，仅包含本文涉及的属性。**

```java
// ... 省略部分和本文无关的属性。

/**
 * 通信服务器集合
 *
 * key: 服务器地址。格式为：host:port
 */
private final Map<String, ExchangeServer> serverMap = new ConcurrentHashMap<String, ExchangeServer>(); // <host:port,Exchanger>
```

serverMap 属性，通信服务器集合。其中，Key 为服务器地址，格式为 host:port。

### export(Invoker<T> invoker)
```java
@Override
public <T> Exporter<T> export(Invoker<T> invoker) throws RpcException {
    URL url = invoker.getUrl();

    // export service
    // 调用父类 AbstractProtocol 方法获取 key
    String key = serviceKey(url);

    // 创建 DubboExporter 对象，并添加到 exporterMap
    DubboExporter<T> exporter = new DubboExporter<T>(invoker, key, exporterMap);

    // 添加到 exporterMap 中，该属性从父类 (AbstractProtocol) 继承而来
    exporterMap.put(key, exporter);

    //export an stub service for dispatching event
    Boolean isStubSupportEvent = url.getParameter(Constants.STUB_EVENT_KEY, Constants.DEFAULT_STUB_EVENT);
    Boolean isCallbackservice = url.getParameter(Constants.IS_CALLBACK_SERVICE, false);
    if (isStubSupportEvent && !isCallbackservice) {
        String stubServiceMethods = url.getParameter(Constants.STUB_EVENT_METHODS_KEY);
        if (stubServiceMethods == null || stubServiceMethods.length() == 0) {
            if (logger.isWarnEnabled()) {
                logger.warn(new IllegalStateException("consumer [" + url.getParameter(Constants.INTERFACE_KEY) +
                        "], has set stubproxy support event ,but no stub methods founded."));
            }
        } else {
            stubServiceMethodsMap.put(url.getServiceKey(), stubServiceMethods);
        }
    }

    // @1 开启服务器
    openServer(url);
    optimizeSerialization(url);
    return exporter;
}
```

![](https://img2018.cnblogs.com/blog/1326851/202001/1326851-20200103155132130-827625128.png)

### openServer(URL url)
```java
/**
  * 启动通信服务器
  *
  */
private void openServer(URL url) {
    // find server.
    String key = url.getAddress();
    //client can export a service which's only for server to invoke
    boolean isServer = url.getParameter(Constants.IS_SERVER_KEY, true);
    if (isServer) {

        // 从 serverMap 获得对应服务器地址已存在的通信服务器。即，不重复创建
        ExchangeServer server = serverMap.get(key);
        if (server == null) {

            // 通信服务器不存在，调用 #createServer(url) 方法，创建服务器
            serverMap.put(key, createServer(url));
        } else {
            // server supports reset, use together with override
            // 通信服务器已存在，调用 Server#reset(url) 方法，重置服务器的属性
            server.reset(url); // @1
        }
    }
}
```
代码 @1，为什么会存在呢？因为键是 host:port ，那么例如，多个 Service 共用同一个 Protocol ，服务器是同一个对象，不要重复创建。

### createServer(URL url)
```java
/**
 * 创建服务器
 */
private ExchangeServer createServer(URL url) {
    // send readonly event when server closes, it's enabled by default
    // 为服务提供者 url 增加 channel.readonly.sent 属性，默认为 true，表示在发送请求时，是否等待将字节写入 socket 后再返回，默认为 true.
    url = url.addParameterIfAbsent(Constants.CHANNEL_READONLYEVENT_SENT_KEY, Boolean.TRUE.toString());

    // enable heartbeat by default
    // 为服务提供者 url 增加 heartbeat 属性，表示心跳间隔时间，默认为 60*1000，表示 60s.
    url = url.addParameterIfAbsent(Constants.HEARTBEAT_KEY, String.valueOf(Constants.DEFAULT_HEARTBEAT));

    // 为服务提供者 url 增加 server 属性，可选值为 netty，mina 等等，默认为 netty.
    String str = url.getParameter(Constants.SERVER_KEY, Constants.DEFAULT_REMOTING_SERVER);

    // 校验 Server 的 Dubbo SPI 拓展是否存在，若不存在，抛出 RpcException 异常.
    if (str != null && str.length() > 0 && !ExtensionLoader.getExtensionLoader(Transporter.class).hasExtension(str))
        throw new RpcException("Unsupported server type: " + str + ", url: " + url);

    // 设置编解码器为 Dubbo，即 DubboCountCodec.
    url = url.addParameter(Constants.CODEC_KEY, DubboCodec.NAME);
    ExchangeServer server;
    try {
        // 根据服务提供者 URI,服务提供者命令请求处理器 requestHandler 构建 ExchangeServer 实例.
        // requestHandler 的实现具体在以后详细分析 Dubbo 服务调用时再详细分析
        server = Exchangers.bind(url, requestHandler);
    } catch (RemotingException e) {
        throw new RpcException("Fail to start server(url: " + url + ") " + e.getMessage(), e);
    }

    // 校验 Client 的 Dubbo SPI 拓展是否存在
    str = url.getParameter(Constants.CLIENT_KEY);
    if (str != null && str.length() > 0) {
        Set<String> supportedTypes = ExtensionLoader.getExtensionLoader(Transporter.class).getSupportedExtensions();
        if (!supportedTypes.contains(str)) {
            throw new RpcException("Unsupported client type: " + str);
        }
    }
    return server;
}
```

执行 `Exchangers.bind(url,requestHandler)` 方法之前看一下参数列表

![](https://img2018.cnblogs.com/blog/1326851/202001/1326851-20200103155219634-2093649874.png)

## Exchangers

### ExchangeServer bind(URL url, ExchangeHandler handler)

```java
public static ExchangeServer bind(URL url, ExchangeHandler handler) throws RemotingException {
    if (url == null) {
        throw new IllegalArgumentException("url == null");
    }
    if (handler == null) {
        throw new IllegalArgumentException("handler == null");
    }
    url = url.addParameterIfAbsent(Constants.CODEC_KEY, "exchange");
    return getExchanger(url).bind(url, handler); // @1
}
```

代码 @1 的前半部分 ```getExchanger(url)``` 获取的的是默认的 **HeaderExchanger**（这块并不复杂，读者自己去研究），然后调用 bind 方法构建 ExchangeServer。

## HeaderExchanger

### HeaderExchanger.bind(URL url, ExchangeHandler handler)

```java
public class HeaderExchanger implements Exchanger {

    public static final String NAME = "header";

    @Override
    public ExchangeClient connect(URL url, ExchangeHandler handler) throws RemotingException {
        return new HeaderExchangeClient(Transporters.connect(url, new DecodeHandler(new HeaderExchangeHandler(handler))), true);
    }

    @Override
    public ExchangeServer bind(URL url, ExchangeHandler handler) throws RemotingException {
        return new HeaderExchangeServer(Transporters.bind(url, new DecodeHandler(new HeaderExchangeHandler(handler))));
    }

}
```
从这里可以看出，端口的绑定由 Transporters 的 bind 方法实现

## Transporters
### Server bind(URL url, ChannelHandler... handlers)

```java
public static Server bind(URL url, ChannelHandler... handlers) throws RemotingException {
    if (url == null) {
        throw new IllegalArgumentException("url == null");
    }
    if (handlers == null || handlers.length == 0) {
        throw new IllegalArgumentException("handlers == null");
    }
    ChannelHandler handler;
    if (handlers.length == 1) {
        handler = handlers[0];
    } else {
        handler = new ChannelHandlerDispatcher(handlers);
    }
    return getTransporter().bind(url, handler); // @1
}
```

代码 @1 处的 ```getTransporter()``` 根据 SPI 具体实现是 netty4 下的 NettyTransporter（同样这块细节在这里不展开）

## NettyTransporter
### Server bind(URL url, ChannelHandler listener)

```java
public class NettyTransporter implements Transporter {

    public static final String NAME = "netty";

    @Override
    public Server bind(URL url, ChannelHandler listener) throws RemotingException { //@1
        return new NettyServer(url, listener);
    }

    @Override
    public Client connect(URL url, ChannelHandler listener) throws RemotingException {
        return new NettyClient(url, listener);
    }RegistryProtocol#export(invoker)RegistryProtocol#export(invoker)RegistryProtocol#export(invoker)RegistryProtocol#export(invoker)RegistryProtocol#export(invoker)

}
```

注意：这里有一个语义级别的转换，ChannelHandler 转变为 listener 语义，说明 ChannelHandler 的具体实现应该实现了 listener 的语义功能。

![](https://img2018.cnblogs.com/blog/1326851/202001/1326851-20200103155407777-1624431172.png)

![](https://img2018.cnblogs.coRegistryProtocol#export(invoker)m/blog/1326851/202001/1326851-20200103155438677-212238394.png)

## NettyServer

```java
public NettyServer(URL url, ChannelHandler handler) throws RemotingException {
    super(url, ChannelHandlers.wrap(handler, ExecutorUtil.setThreadName(url, SERVER_THREAD_POOL_NAME))); // @1
}
```

@1 中的 <code><font color="#de2c58">ChannelHandlers.wrap(handler, ExecutorUtil.setThreadName(url, SERVER_THREAD_POOL_NAME))</font></code> 这一步涉及到从 netty 服务器接收数据包、业务线程池的派发机制等等很多重要的模块会在后面分析。
<br />  
#####参考:
《深入理解Apache Dubbo与实战》
http://dubbo.apache.org
https://blog.csdn.net/prestigeding/article/details/80637239
http://svip.iocoder.cn/Dubbo/service-export-remote-dubbo/
