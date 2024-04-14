# 解决Another unnamed CacheManager already exists in the same VM



今天尝试把同一个tomcat下的几个项目中重复的jar文件抽出来，放到tomcat下面的share/lib中，这个文件夹可能不存在，我新建的。

结果启动的时候报错。

```
 Another CacheManager with same name 'xxxxxxx' already exists in the same VM.


Caused by: org.hibernate.cache.CacheException: net.sf.ehcache.CacheException: Another unnamed CacheManager already exists in the same VM. Please provide unique names for each CacheManager in the config or do one of following:
1. Use one of the CacheManager.create() static factory methods to reuse same CacheManager with same name or create one if necessary
2. Shutdown the earlier cacheManager before creating new one with same name.
The source of the existing CacheManager is: DefaultConfigurationSource [ ehcache.xml or ehcache-failsafe.xml ]
        at org.hibernate.cache.ehcache.EhCacheRegionFactory.start(EhCacheRegionFactory.java:107)
        at org.hibernate.internal.CacheImpl.<init>(CacheImpl.java:70)
        at org.hibernate.engine.spi.CacheInitiator.initiateService(CacheInitiator.java:40)
        at org.hibernate.engine.spi.CacheInitiator.initiateService(CacheInitiator.java:35)
        at org.hibernate.service.internal.SessionFactoryServiceRegistryImpl.initiateService(SessionFactoryServiceRegistryImpl.java:91)
        at org.hibernate.service.internal.AbstractServiceRegistryImpl.createService(AbstractServiceRegistryImpl.java:251)
        ... 118 more
Caused by: net.sf.ehcache.CacheException: Another unnamed CacheManager already exists in the same VM. Please provide unique names for each CacheManager in the config or do one of following:
1. Use one of the CacheManager.create() static factory methods to reuse same CacheManager with same name or create one if necessary
2. Shutdown the earlier cacheManager before creating new one with same name.
The source of the existing CacheManager is: DefaultConfigurationSource [ ehcache.xml or ehcache-failsafe.xml ]
        at net.sf.ehcache.CacheManager.assertNoCacheManagerExistsWithSameName(CacheManager.java:460)
        at net.sf.ehcache.CacheManager.init(CacheManager.java:349)
        at net.sf.ehcache.CacheManager.<init>(CacheManager.java:237)
        at org.hibernate.cache.ehcache.EhCacheRegionFactory.start(EhCacheRegionFactory.java:86)
        ... 123 more
```

字面意思是说不能有相同名字的cacheManager。

猜测可能是shiro还有hibernate中用的ehcache的问题。

所以将shiro和hibernate中用到ehcache的地方全部干掉。

包括关掉hibernate的二级缓存，还有把shiro的ehcache改成自带的MapCache。

```xml
<!-- 二级缓存EhCache配置 -->
<!-- 注掉 
<prop key="hibernate.cache.region.factory_class">org.hibernate.cache.ehcache.EhCacheRegionFactory
				</prop>
				<prop key="hibernate.cache.use_query_cache">true</prop>
				<prop key=" hibernate.cache.use_second_level_cache">true</prop>
-->
```



```xml
 <!-- 配置shiro自带的缓存管理器 -->
<bean id = "shiroSpringCacheManager" class="org.apache.shiro.cache.MemoryConstrainedCacheManager"/>
```



发现启动正常。



那就应该不是有重复jar包的问题。



所以继续排查 ，发现shiro用了ehcache

```xml
    <bean id="shiroCacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager">
        <property name="cacheManager" ref="cacheManager"/>
    </bean>

    <bean id="cacheManager" class="org.springframework.cache.ehcache.EhCacheManagerFactoryBean">
        <property name="configLocation" value="classpath:ehcache-local.xml"/>
    </bean>
```

再看看ehcache-local.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache updateCheck="false" name="txswx-ehcache">
    <diskStore path="java.io.tmpdir"/>
    <!-- 默认缓存配置. -->
    <defaultCache maxEntriesLocalHeap="100" eternal="false" timeToIdleSeconds="300" timeToLiveSeconds="600"
                  overflowToDisk="true" maxEntriesLocalDisk="100000" />

    <!-- 系统缓存 -->
    <cache name="sysCache" maxEntriesLocalHeap="100" eternal="true" overflowToDisk="true"/>

</ehcache>
```



嗯 `name="txswx-ehcache"` 这个地方，不同项目修改成不同的名称就好。

同理hibernate相应ehcache的配置文件也改一下名称。



再次启动 OK

