---
title: Freemarker cache
date: 2017-07-21 16:03:26
tags: [java, freemarker]
categories: 技术
---

## 模板缓存

<!-- more -->

### 说明
FreeMarker 是会缓存模板的(假设使用 Configuration 对象的方法来创建 Template 对象)。这就是说当调用 getTemplate方法时，FreeMarker不但返回了 Template 对象，而且还会将它存储在缓存中， 当下一次再以相同(或相等)路径调用 getTemplate 方法时， 那么它只返回缓存的 Template 实例， 而不会再次加载和解析模板文件了。

如果更改了模板文件，当下次调用模板时，FreeMarker 将会自动重新载入和解析模板。 然而，要检查模板文件是否改变内容了是需要时间的，有一个 Configuration 级别的设置被称作"更新延迟"，它可以用来配置这个时间。 这个时间就是从上次对某个模板检查更新后，FreeMarker再次检查模板所要间隔的时间。 其默认值是5秒。如果想要看到模板立即更新的效果，那么就要把它设置为0。 要注意某些模板加载器也许在模板更新时可能会有问题。 例如，典型的例子就是在基于类加载器的模板加载器就不会注意到模板文件内容的改变。

当调用了 getTemplate 方法时， 与此同时FreeMarker意识到这个模板文件已经被移除了，所以这个模板也会从缓存中移除。 如果Java虚拟机认为会有内存溢出时，默认情况它会从缓存中移除任意模板。 此外，你还可以使用 Configuration 对象的 clearTemplateCache 方法手动清空缓存。

何时将一个被缓存了的模板清除的实际应用策略是由配置的属性 cache_storage 来确定的，通过这个属性可以配置任何 CacheStorage 的实现。对于大多数用户来说， 使用 freemarker.cache.MruCacheStorage 就足够了。 这个缓存存储实现了二级最近使用的缓存。在第一级缓存中， 组件都被强烈引用到特定的最大数目(引用次数最多的组件不会被Java虚拟机抛弃， 而引用次数很少的组件则相反)。当超过最大数量时， 最近最少使用的组件将被送至二级缓存中，在那里它们被很少引用， 直到达到另一个最大的数目。引用强度的大小可以由构造方法来指定。 例如，设置强烈部分为20，轻微部分为250：

cfg.setCacheStorage(new freemarker.cache.MruCacheStorage(20, 250))
或者，使用 MruCacheStorage 缓存， 它是默认的缓存存储实现：

cfg.setSetting(Configuration.CACHE_STORAGE_KEY, "strong:20, soft:250");
当创建了一个新的 Configuration 对象时， 它使用一个 strongSizeLimit 值为0的 MruCacheStorage 缓存来初始化， softSizeLimit 的值是 Integer.MAX_VALUE (也就是在实际中，是无限大的)。但是使用非0的 strongSizeLimit 对于高负载的服务器来说也许是一个更好的策略，对于少量引用的组件来说， 如果资源消耗已经很高的话，Java虚拟机往往会引发更高的资源消耗， 因为它不断从缓存中抛出经常使用的模板，这些模板还不得不再次加载和解析

### spring示例

``` xml
<bean id="freemarkerMailConfiguration" class="org.springframework.ui.freemarker.FreeMarkerConfigurationFactoryBean">
    <property name="templateLoaderPaths" value="classpath:emailtemplates/task,classpath:emailtemplates/user"/>
    <!-- Activate the following to disable template caching -->
    <property name="freemarkerSettings" value="cache_storage=freemarker.cache.NullCacheStorage" />
</bean>

```

自定义 freemarkerConfigure

``` xml
<bean id="freemarkerConfig" class="com.xxxx.framework.web.EasyFreeMarkerConfigurer">
    <property name="freemarkerSettings">
        <props>
            <prop key="cache_storage">freemarker.cache.NullCacheStorage</prop>
        </props>
    </property>

```

``` java
public class EasyFreeMarkerConfigurer extends FreeMarkerConfigurer {
    // ...
    public void setfreemarkerSettings(Properties settings) {
        // ... user defined 
        super.setFreemarkerSettigs(settings);
    }
}
```