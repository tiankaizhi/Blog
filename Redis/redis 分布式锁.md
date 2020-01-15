#### redis 分布式锁实现方式
https://wudashan.cn/2017/10/23/Redis-Distributed-Lock-Implement/#releaseLock-wrongDem2
https://www.cnblogs.com/linjiqin/p/8003838.html

##### 加锁
```
@Override
  public String set(final String key, final String value, final String nxxx, final String expx, final long time) {
    return new JedisClusterCommand<String>(connectionHandler, maxAttempts) {
      @Override
      public String execute(Jedis connection) {
        return connection.set(key, value, nxxx, expx, time);
      }
    }.run(key);
  }
```
##### 解锁
```
@Override
  public Object eval(final String script, final List<String> keys, final List<String> args) {
    return new JedisClusterCommand<Object>(connectionHandler, maxAttempts) {
      @Override
      public Object execute(Jedis connection) {
        return connection.eval(script, keys, args);
      }
    }.run(keys.size(), keys.toArray(new String[keys.size()]));
  }
```
