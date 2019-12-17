## 引言

研究源码是一个充满疑惑并且非常枯燥的过程，因为在研究的过程中可能很多你都看不懂，所以我们最好先带着问题去看源码，这样即使研究完后还是存在很多疑问，但是我们至少抓住了重点。

## 带着这些问题

1. 延时暴露和立即暴露。

2. 什么时候建立与注册中心的连接。

3. 服务提供者什么时候向注册中心注册服务。

4. 服务提供者与注册中心的心跳机制。

## 暴露总体过程

在详细研究服务暴露细节之前，我们先看一下整体 RPC 的暴露原理，如图 5-4。

从整体上看，Dubbo 框架做服务暴露分为两大部分，第一步将持有的服务实例通过代理转换成 Invoker，第二步把 Invoker 通过具体的协议（比如 Dubbo） 转换成 Exporter，框架做了这层抽象大大方便了功能扩展。这里的 Invoker 可以简单的理解成一个真实的服务对象实例，是 Dubbo 框架实体域，所有模型都会向它靠拢，可向他发起 invoke 调用。它可能是一个本地实现，也可能是一个远程实现，还可能是一个集群实现。

![](assets/markdown-img-paste-20191216231628258.png)

在正式研究源码之前有必要先说一下，笔者研究的源码基于 <code><font color="#f52814">2.7.3</font></code> 版本的 <code><font color="#f52814">dubbo-samples-api</font></code> 工程，不想自己搭 Demo 的朋友可以去 Dubbo 官网或者笔者的 [github](https://github.com/tiankaizhi/dubboSourceCodeAnalysis) 仓库去下载。

## 暴露细节

1、 创建 ServiceConfig 对象用来承载配置信息

2、 暴露服务入口 <code><font color="#f52814">service.export()</font></code>

![](assets/markdown-img-paste-20191217124949246.png)

2.1、

代码 ① 处判断是否暴露服务，根据<code><font color="#f52814"><dubbo:service export="true|false"></font></code> 设置

代码 ② 处如果 <code><font color="#f52814">delay</font></code> 大于 0，表示延迟多少毫秒后暴露服务

![](assets/markdown-img-paste-20191217180649526.png)

延迟暴露采用的是 JDK 的 <code><font color="#f52814">ScheduledExecutorService</font></code> 进行调度的。

![](assets/markdown-img-paste-20191217130003135.png)

延时调度机制触发时机是当 Spring 容器实例化 bean 完成，走到最后一步发布 <code><font color="#f52814">ContextRefreshEvent</font></code> 事件的时候，<code><font color="#f52814">ServiceBean </font></code> 会执行 <code><font color="#f52814">onApplicationEvent</font></code> 方法，该方法调用 <code><font color="#f52814">ServiceConfig</font></code> 的 <code><font color="#f52814">export</font></code> 方法。

![](assets/markdown-img-paste-20191217130504955.png)

第 412 行判断 <code><font color="#f52814">path</font></code> 为空后，将 <code><font color="#f52814">interfaceName</font></code> 赋给 <code><font color="#f52814">path</font></code>

![](assets/markdown-img-paste-20191217144530385.png)

<code><font color="#f52814">doExportUrls</font></code>  正式开始暴露服务

```java
    private void doExportUrls() {

        // 获取当前服务对应的注册中心实例
        List<URL> registryURLs = loadRegistries(true);   // @1 获取当前服务所有的注册中心地址

        for (ProtocolConfig protocolConfig : protocols) { // @2 如果服务指暴露多个协议，则依次暴露

            String pathKey = URL.buildKey(getContextPath(protocolConfig).map(p -> p + "/" + path).orElse(path), group, version);
            ProviderModel providerModel = new ProviderModel(pathKey, ref, interfaceClass);
            ApplicationModel.initProviderModel(pathKey, providerModel);

            doExportUrlsFor1Protocol(protocolConfig, registryURLs); // @3
        }
    }
```

代码 @1：首先遍历 <code><font color="#f52814">ServiceBean</font></code> 的 <code><font color="#f52814">List< RegistryConfig> registries</font></code>（所有注册中心的配置信息），然后将地址封装成 <code><font color="#f52814">URL 对象</font></code> ，关于注册中心的所有配置属性，最终转换成 <code><font color="#f52814">url</font></code> 的属性(?属性名=属性值)。<code><font color="#f52814">loadRegistries(true)</font></code>，参数 true 代表服务提供者，false 代表服务消费者，如果是服务提供者，则检测注册中心的配置，如果配置了 <code><font color="#f52814">register="false"</font></code>，则忽略该地址，如果是服务消费者，并配置了 <code><font color="#f52814">subscribe="false"</font></code> 则表示不从该注册中心订阅服务，故也不返回。如果没有显示指定的服务注册中心，则默认会使用全局配置的注册中心。

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
        //必要时检查&&覆盖
        // check && override if necessary
        List<URL> registryList = new ArrayList<URL>();
        if (CollectionUtils.isNotEmpty(registries)) {
            for (RegistryConfig config : registries) {
                String address = config.getAddress();
                if (StringUtils.isEmpty(address)) {
                    address = ANYHOST_VALUE;
                }
                if (!RegistryConfig.NO_AVAILABLE.equalsIgnoreCase(address)) {
                    Map<String, String> map = new HashMap<String, String>();
                    appendParameters(map, application);
                    appendParameters(map, config);
                    map.put(PATH_KEY, RegistryService.class.getName());
                    appendRuntimeParameters(map);
                    if (!map.containsKey(PROTOCOL_KEY)) {
                        map.put(PROTOCOL_KEY, DUBBO_PROTOCOL);
                    }
                    List<URL> urls = UrlUtils.parseURLs(address, map);

                    for (URL url : urls) {
                        url = URLBuilder.from(url)
                                .addParameter(REGISTRY_KEY, url.getProtocol())
                                .setProtocol(REGISTRY_PROTOCOL)
                                .build();
                        if ((provider && url.getParameter(REGISTER_KEY, true))
                                || (!provider && url.getParameter(SUBSCRIBE_KEY, true))) {
                            registryList.add(url);
                        }
                    }
                }
            }
        }
        return registryList;
    }
```

我们看一下加载完的 <code><font color="#f52814">registryList</font></code> 值：

![](assets/markdown-img-paste-20191217190322787.png)

代码 @2，加载完所有配置的注册中心 URL 之后，开始按照协议进行暴露。Dubbo 支持相同服务暴露多个协议。

![](assets/markdown-img-paste-20191217203557972.png)

我们看一下 <code><font color="#f52814">doExportUrlsFor1Protocol</font></code> 方法具体暴露逻辑

```java
private void doExportUrlsFor1Protocol(ProtocolConfig protocolConfig, List<URL> registryURLs) {
        String name = protocolConfig.getName();
        if (StringUtils.isEmpty(name)) {
            name = DUBBO;
        }

        Map<String, String> map = new HashMap<String, String>();
        map.put(SIDE_KEY, PROVIDER_SIDE);

        appendRuntimeParameters(map);
        appendParameters(map, metrics);
        appendParameters(map, application);
        appendParameters(map, module);
        // remove 'default.' prefix for configs from ProviderConfig
        // appendParameters(map, provider, Constants.DEFAULT_KEY);
        appendParameters(map, provider);
        appendParameters(map, protocolConfig);
        appendParameters(map, this);
        if (CollectionUtils.isNotEmpty(methods)) {
            for (MethodConfig method : methods) {
                appendParameters(map, method, method.getName());
                String retryKey = method.getName() + ".retry";
                if (map.containsKey(retryKey)) {
                    String retryValue = map.remove(retryKey);
                    if ("false".equals(retryValue)) {
                        map.put(method.getName() + ".retries", "0");
                    }
                }
                List<ArgumentConfig> arguments = method.getArguments();
                if (CollectionUtils.isNotEmpty(arguments)) {
                    for (ArgumentConfig argument : arguments) {
                        // convert argument type
                        if (argument.getType() != null && argument.getType().length() > 0) {
                            Method[] methods = interfaceClass.getMethods();
                            // visit all methods
                            if (methods != null && methods.length > 0) {
                                for (int i = 0; i < methods.length; i++) {
                                    String methodName = methods[i].getName();
                                    // target the method, and get its signature
                                    if (methodName.equals(method.getName())) {
                                        Class<?>[] argtypes = methods[i].getParameterTypes();
                                        // one callback in the method
                                        if (argument.getIndex() != -1) {
                                            if (argtypes[argument.getIndex()].getName().equals(argument.getType())) {
                                                appendParameters(map, argument, method.getName() + "." + argument.getIndex());
                                            } else {
                                                throw new IllegalArgumentException("Argument config error : the index attribute and type attribute not match :index :" + argument.getIndex() + ", type:" + argument.getType());
                                            }
                                        } else {
                                            // multiple callbacks in the method
                                            for (int j = 0; j < argtypes.length; j++) {
                                                Class<?> argclazz = argtypes[j];
                                                if (argclazz.getName().equals(argument.getType())) {
                                                    appendParameters(map, argument, method.getName() + "." + j);
                                                    if (argument.getIndex() != -1 && argument.getIndex() != j) {
                                                        throw new IllegalArgumentException("Argument config error : the index attribute and type attribute not match :index :" + argument.getIndex() + ", type:" + argument.getType());
                                                    }
                                                }
                                            }
                                        }
                                    }
                                }
                            }
                        } else if (argument.getIndex() != -1) {
                            appendParameters(map, argument, method.getName() + "." + argument.getIndex());
                        } else {
                            throw new IllegalArgumentException("Argument config must set index or type attribute.eg: <dubbo:argument index='0' .../> or <dubbo:argument type=xxx .../>");
                        }

                    }
                }
            } // end of methods for
        }

        if (ProtocolUtils.isGeneric(generic)) {
            map.put(GENERIC_KEY, generic);
            map.put(METHODS_KEY, ANY_VALUE);
        } else {
            String revision = Version.getVersion(interfaceClass, version);
            if (revision != null && revision.length() > 0) {
                map.put(REVISION_KEY, revision);
            }

            String[] methods = Wrapper.getWrapper(interfaceClass).getMethodNames();
            if (methods.length == 0) {
                logger.warn("No method found in service interface " + interfaceClass.getName());
                map.put(METHODS_KEY, ANY_VALUE);
            } else {
                map.put(METHODS_KEY, StringUtils.join(new HashSet<String>(Arrays.asList(methods)), ","));
            }
        }
        if (!ConfigUtils.isEmpty(token)) {
            if (ConfigUtils.isDefault(token)) {
                map.put(TOKEN_KEY, UUID.randomUUID().toString());
            } else {
                map.put(TOKEN_KEY, token);
            }
        }
        // export service
        String host = this.findConfigedHosts(protocolConfig, registryURLs, map);
        Integer port = this.findConfigedPorts(protocolConfig, name, map);
        URL url = new URL(name, host, port, getContextPath(protocolConfig).map(p -> p + "/" + path).orElse(path), map);

        if (ExtensionLoader.getExtensionLoader(ConfiguratorFactory.class)
                .hasExtension(url.getProtocol())) {
            url = ExtensionLoader.getExtensionLoader(ConfiguratorFactory.class)
                    .getExtension(url.getProtocol()).getConfigurator(url).configure(url);
        }

        String scope = url.getParameter(SCOPE_KEY);
        // don't export when none is configured
        if (!SCOPE_NONE.equalsIgnoreCase(scope)) {

            // export to local if the config is not remote (export to remote only when config is remote)
            if (!SCOPE_REMOTE.equalsIgnoreCase(scope)) {
                exportLocal(url);
            }
            // export to remote if the config is not local (export to local only when config is local)
            if (!SCOPE_LOCAL.equalsIgnoreCase(scope)) {
                if (!isOnlyInJvm() && logger.isInfoEnabled()) {
                    logger.info("Export dubbo service " + interfaceClass.getName() + " to url " + url);
                }
                if (CollectionUtils.isNotEmpty(registryURLs)) {
                    for (URL registryURL : registryURLs) {
                        //if protocol is only injvm ,not register
                        if (LOCAL_PROTOCOL.equalsIgnoreCase(url.getProtocol())) {
                            continue;
                        }
                        url = url.addParameterIfAbsent(DYNAMIC_KEY, registryURL.getParameter(DYNAMIC_KEY));
                        URL monitorUrl = loadMonitor(registryURL);
                        if (monitorUrl != null) {
                            url = url.addParameterAndEncoded(MONITOR_KEY, monitorUrl.toFullString());
                        }
                        if (logger.isInfoEnabled()) {
                            logger.info("Register dubbo service " + interfaceClass.getName() + " url " + url + " to registry " + registryURL);
                        }

                        // For providers, this is used to enable custom proxy to generate invoker
                        String proxy = url.getParameter(PROXY_KEY);
                        if (StringUtils.isNotEmpty(proxy)) {
                            registryURL = registryURL.addParameter(PROXY_KEY, proxy);
                        }

                        Invoker<?> invoker = PROXY_FACTORY.getInvoker(ref, (Class) interfaceClass, registryURL.addParameterAndEncoded(EXPORT_KEY, url.toFullString()));
                        DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);

                        Exporter<?> exporter = protocol.export(wrapperInvoker);
                        exporters.add(exporter);
                    }
                } else {
                    Invoker<?> invoker = PROXY_FACTORY.getInvoker(ref, (Class) interfaceClass, url);
                    DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);

                    Exporter<?> exporter = protocol.export(wrapperInvoker);
                    exporters.add(exporter);
                }
                /**
                 * @since 2.7.0
                 * ServiceData Store
                 */
                MetadataReportService metadataReportService = null;
                if ((metadataReportService = getMetadataReportService()) != null) {
                    metadataReportService.publishProvider(url);
                }
            }
        }
        this.urls.add(url);
    }
```




Kafka 和其他消息中间件（RabbitMQ，RocketMQ......）都具备 **系统解耦、异步通信、流量削峰、** 缓冲、冗余存储、扩展性、可恢复性等功能。与此同时，Kafka 还提供了大多数消息系统难以实现的 **<font color="#e50e0e">消息顺序性保障</font>** 及 **<font color="#e50e0e">回溯消费的功能</font>** 。

### 存储系统

Kafka 把消息持久化到磁盘，相比于其他基于内存存储的系统而言，有效地降低了数据丢失的风险。也正是得益于 Kafka 的 **<font color="#e50e0e">消息持久化功能 </font>** 和 **<font color="#e50e0e">多副本机制 </font>** ，我们可以把 Kafka 作为长期的数据存储系统来使用，只需要把对应的数据保留策略设置为“永久”或启用主题的日志压缩功能即可。

### 流式处理平台

Kafka 不仅为每个流行的流式处理框架提供了可靠的数据来源，还提供了一个完整的流式处理类库，比如窗口、连接、变换和聚合等各类操作。

## kafka 基本组成

一个典型的 Kafka 体系架构包括 **若干 Producer、若干 Broker、若干 Consumer，以及一个 ZooKeeper 集群**，如下图所示。

![](assets/markdown-img-paste-20190826182221334.png)

其中各个组成部分的作用分别是：

### Zookeeper

ZooKeeper 是 Kafka 用来负责 **<font color="#e50e0e">集群元数据的管理、控制器的选举</font>** 等操作的。

### Producer

Producer 负责创建消息，然后将消息发送到 Broker。

### Broker

Broker 负责将收到的消息存储到磁盘中。对于 Kafka 而言，Broker 可以简单地看作一个独立的 Kafka 服务节点或者是一个 kafka 服务。一个或多个 Broker 组成了一个 Kafka 集群。

### Consumer

Consumer 负责从 Broker 订阅并消费消息。


整个 Kafka 体系结构中除了上面描述的几个组成部分之外，还有两个特别重要的概念— **主题（Topic）** 与 **分区（Partition）** 。Kafka 中的 **消息以主题为单位进行归类**，生产者负责将消息发送到特定的主题（发送到 Kafka 集群中的每一条消息都要指定一个主题），而消费者负责订阅主题并进行消费。

### Topic & Partition

1、 主题是一个逻辑上的概念，它还可以细分为多个分区，**<font color="#e50e0e">一个分区只属于一个主题</font>** 。

2、 **<font color="#e50e0e">同一主题下的不同分区包含的消息是不同的</font>**，分区在存储层面可以看作一个 **可追加的日志（Log）文件**，消息在被追加到分区日志文件的时候都会分配一个特定的偏移量（offset）。

3、 offset 是消息在分区中的唯一标识，Kafka 通过它来保证消息在分区内的顺序性，不过 offset 并不跨越分区，也就是说， **<font color="#e50e0e">Kafka 保证的是分区有序</font>** 而不是主题有序。

![](assets/markdown-img-paste-20190826182416753.png)

如上图所示，主题中有 4 个分区，消息被顺序追加到每个分区日志文件的尾部。**Kafka 中的分区可以分布在不同的服务器（broker）** 上，也就是说，**<font color="#e50e0e">一个主题可以横跨多个 broker</font>** ，以此来提供比单个 broker 更强大的性能。

每一条消息被发送到 broker 之前，会根据分区规则选择存储到哪个具体的分区。如果分区规则设定得合理，所有的消息都可以均匀地分配到不同的分区中。**<font color="#e50e0e">如果一个主题只对应一个文件，那么这个文件所在的机器 I/O 将会成为这个主题的性能瓶颈</font>**，而分区解决了这个问题。在创建主题的时候可以通过指定的参数来设置分区的个数，当然也可以在主题创建完成之后去修改分区的数量，通过增加分区的数量可以实现水平扩展。

#### Partition 的多副本机制

Kafka 为分区引入了多副本（Replica）机制，通过增加副本数量可以提升容灾能力。

**<font color="#e50e0e">同一分区的不同副本中保存的是相同的消息</font>** （在同一时刻，副本之间并非完全一样），副本之间是 “一主多从” 的关系，**其中 leader 副本负责处理读写请求，follower 副本只负责与 leader 副本的消息同步**。副本处于不同的 broker 中，当 leader 副本出现故障时，从 follower 副本中重新选举新的 leader 副本对外提供服务。**Kafka 通过多副本机制实现了故障的自动转移**，当 Kafka 集群中某个 broker 失效时仍然能保证服务可用。

![](assets/markdown-img-paste-20190826182435650.png)

如上图所示，Kafka 集群中有 4 个 broker，某个主题中有 3 个分区，且副本因子（即副本个数）也为 3，如此每个分区便有 1 个 leader 副本和 2 个 follower 副本。生产者和消费者只与 leader 副本进行交互，而 follower 副本只负责消息的同步，很多时候 follower 副本中的消息相对 leader 副本而言会有一定的滞后。

#### 分区的副本种类

分区中的所有副本统称为 <code><font color="#f52814">AR（Assigned Replicas）</font></code>。所有与 leader 副本保持一定程度同步的副本（包括 leader 副本在内）组成 <code><font color="#f52814">ISR（In-Sync Replicas）</font></code>，<code><font color="#f52814">ISR</font></code> 集合是 <code><font color="#f52814">AR</font></code> 集合中的一个子集。消息会先发送到 leader 副本，然后 follower 副本才能从 leader 副本中拉取消息进行同步，同步期间内 follower 副本相对于 leader 副本而言会有一定程度的滞后。

前面所说的 “一定程度的同步” 是指可忍受的滞后范围，这个范围可以通过参数进行配置。与 leader 副本同步滞后过多的副本（不包括 leader 副本）组成 <code><font color="#f52814">OSR（Out-of-Sync Replicas）</font></code>，由此可见，<code><font color="#f52814">AR = ISR + OSR</font></code>。在正常情况下，所有的 follower 副本都应该与 leader 副本保持一定程度的同步，即 <code><font color="#f52814">AR = ISR</font></code>，<code><font color="#f52814">OSR</font></code> 集合为空。

leader 副本负责维护和跟踪 <code><font color="#f52814">ISR</font></code> 集合中所有 follower 副本的滞后状态，当 follower 副本落后太多或失效时，leader 副本会把它从 <code><font color="#f52814">ISR</font></code> 集合中剔除。如果 <code><font color="#f52814">OSR</font></code> 集合中有 follower 副本“追上”了 leader 副本，那么 leader 副本会把它从 <code><font color="#f52814">OSR</font></code> 集合转移至 <code><font color="#f52814">ISR</font></code> 集合。默认情况下，当 leader 副本发生故障时，只有在 <code><font color="#f52814">ISR</font></code> 集合中的副本才有资格被选举为新的 leader，而在 <code><font color="#f52814">OSR</font></code> 集合中的副本则没有任何机会（不过这个原则也可以通过修改相应的参数配置来改变）。

#### HW & LEO

<code><font color="#f52814">ISR</font></code> 与 <code><font color="#f52814">HW</font></code> 和 <code><font color="#f52814">LEO</font></code> <code><font color="#f52814">HW</font></code> 是 <code><font color="#f52814">High Watermark</font></code> 的缩写，俗称高水位，它标识了一个特定的消息偏移量 <code><font color="#f52814">offset</font></code>，消费者只能拉取到这个 <code><font color="#f52814">offset</font></code> 之前的消息。

![](assets/markdown-img-paste-2019082618245529.png)

如上图所示，它代表一个日志文件，这个日志文件中有 9 条消息，第一条消息的 <code><font color="#f52814">offset</font></code>（<code><font color="#f52814">LogStartOffset</font></code>）为 0，最后一条消息的 <code><font color="#f52814">offset</font></code> 为 <code><font color="#f52814">offset</font></code> 为 9 的消息用虚线框表示，代表下一条待写入的消息。日志文件的 <code><font color="#f52814">HW</font></code> 为 6，表示消费者只能拉取到 <code><font color="#f52814">offset</font></code> 在 0 至 5 之间的消息，而 <code><font color="#f52814">offset</font></code> 为 6 的消息对消费者而言是不可见的。

<code><font color="#f52814">LEO</font></code> 是 <code><font color="#f52814">Log End Offset</font></code> 的缩写，它标识当前日志文件中下一条待写入消息的 <code><font color="#f52814">Offset</font></code>，上图中 <code><font color="#f52814">Offset</font></code> 为 9 的位置即为当前日志文件的 <code><font color="#f52814">LEO</font></code>，<code><font color="#f52814">LEO</font></code> 的大小相当于当前日志分区中最后一条消息的 <code><font color="#f52814">Offset</font></code> 值加 1。**<font color="#e50e0e">分区 ISR 集合中的每个副本都会维护自身的 LEO </font>**，而 <code><font color="#f52814">ISR</font></code> 集合中最小的 <code><font color="#f52814">LEO</font></code> 即为分区的 <code><font color="#f52814">HW</font></code>，对消费者而言只能消费 <code><font color="#f52814">HW</font></code> 之前的消息。

>**<font color="#e50e0e">小贴士：</font>** 很多资料中误将上图中的 <code><font color="#f52814">Offset</font></code> 为 5 的位置看作 <code><font color="#f52814">HW</font></code>，而把 <code><font color="#f52814">offset</font></code> 为 8 的位置看作 <code><font color="#f52814">LEO</font></code>，这显然是不对的。

![](assets/markdown-img-paste-20190826182519947.png)

为了更好地理解 <code><font color="#f52814">ISR</font></code> 集合，以及 <code><font color="#f52814">HW</font></code> 和 <code><font color="#f52814">LEO</font></code> 之间的关系，下面通过一个简单的示例来进行相关的说明。如上图所示，假设某个分区的 <code><font color="#f52814">ISR</font></code> 集合中有 3 个副本，即 1 个 leader 副本和 2 个 follower 副本，此时分区的 <code><font color="#f52814">LEO</font></code> 和 <code><font color="#f52814">HW</font></code> 都为 3。消息 3 和消息 4 从生产者发出之后会被先存入 leader 副本，如下图所示。

![](assets/markdown-img-paste-20190826182533531.png)

在消息写入 leader 副本之后，follower 副本会发送拉取请求来拉取消息 3 和消息 4 以进行消息同步（这里有一个疑问： follower 副本是定时去 leader 副本中拉数据进行同步，还是 leader 副本有新消息进入后通知 follower 副本来拉消息？）。

![](assets/markdown-img-paste-20190826182548482.png)

在同步过程中，不同的 follower 副本的同步效率也不尽相同。如上图所示，在某一时刻 follower1 完全跟上了 leader 副本而 follower 2 只同步了消息 3，如此 leader 副本的 <code><font color="#f52814">LEO</font></code> 为 5，follower 1 的 <code><font color="#f52814">LEO</font></code> 为 5，follower 2 的 <code><font color="#f52814">LEO</font></code> 为 4，那么当前分区的 <code><font color="#f52814">HW</font></code> 取最小值 4，此时消费者可以消费到 <code><font color="#f52814">offset</font></code> 为 0 至 3 之间的消息。

> **小贴士：<font color="#e50e0e">消费者能够消费到的消息一定是分区中 ISR 副本已经同步完成的消息</font>**，这种方式就很好了规避了有一些同步滞后的副本同步速度慢的问题，从而影响性能。

写入消息（情形 4）如下图所示，所有的副本都成功写入了消息 3 和消息 4，整个分区的 <code><font color="#f52814">HW</font></code> 和 <code><font color="#f52814">LEO</font></code> 都变为 5，因此消费者可以消费到 offset 为 4 的消息了。

![](assets/markdown-img-paste-20190826182603172.png)

由此可见，**<font color="#e50e0e">Kafka 的复制机制既不是完全的同步复制，也不是单纯的异步复制</font>** 。事实上，同步复制要求所有能工作的 follower 副本都复制完，这条消息才会被确认为已成功提交，这种复制方式极大地影响了性能。而在异步复制方式下，follower 副本异步地从 leader 副本中复制数据，数据只要被 leader 副本写入就被认为已经成功提交。在这种情况下，如果 follower 副本都还没有复制完而落后于 leader 副本，突然 leader 副本宕机，则会造成数据丢失。Kafka 使用的这种 **<font color="#e50e0e"> ISR 的 HW 和 LEO 方式</font> 则有效地权衡了<font color="#e50e0e"> 数据可靠性和性能</font>之间的关系** 。

> **<font color="#e50e0e">小贴士：</font>** kafka 在只有一个 leader 一个 follower 的情况下，如果 follower 副本都还没有复制完而落后于 leader 副本，突然 leader 副本宕机，则会造成数据丢失。这种情况丢失的数据最大可能为 <code><font color="#f52814">LEO - HW</font></code> 部分。

## kafka 消费端的容灾能力

Kafka 消费端也具备一定的容灾能力。Consumer 使用拉（Pull）模式从服务端拉取消息，并且保存消费的具体位置，当消费者宕机后恢复上线时可以根据之前保存的消费位置重新拉取需要的消息进行消费，这样就不会造成消息丢失。
