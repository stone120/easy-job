在Java工程师平常的开发过程中，由于业务的不同，可能关注的点有很多不一样的地方，但是在基础层面还是有一些共性的。此文概括了在Java开发、测试、部署、工程化方面一些需要注意的地方，供大家参考。

# 1. 将一些需要变动的配置写在属性文件中
比如，没有把一些需要并发执行时使用的线程数设置成可在属性文件中配置。那么你的程序无论在DEV环境中，还是TEST环境中，都可以顺畅无阻地运行，但是一旦部署在PROD上，把它作为多线程程序处理更大的数据集时，就会抛出IOException，原因也许是线上环境并发造成也许是其他。如果线程数目可以在属性文件中配置，那么使它成为一个单线程应用程序就变得十分容易了。我们不再需要为了解决问题而反复地部署和测试应用了。这种方法也同样适用于配置 URL、服务器和端口号等。

这里推荐使用属性文件外化这些配置，文件格式使用properties、yaml、hocon、json都可以。下面的类实现了对这些格式的文件的spring注入支持，包括占位符支持。

https://github.com/superhj1987/awesome-libs/blob/master/src/main/java/me/rowkey/libs/spring/config/AwesomePropertyPlaceholderConfigurer.java

# 2. 测试中尽可能模拟线上环境
生产过程中一个典型的场景就是只使用1到3个帐户进行测试，而这个数量本应是1000到2000个的。在做性能测试时，使用的数据必须是真实并且未经裁剪的。不贴近真实环境的性能测试，可能会带来不可预料的性能、拓展和多线程问题。这里也可以采取预发布环境的方式来解决部分问题。

# 3. 对于所有外部调用以及内部服务都要做容错处理
不管是RPC调用还是对于第三方服务的调用，都不能想当然的认为可用性是100%的。不允许出现服务调用超时和重试，将会对应用程序的稳定性和性能造成不利的影响。

# 4. 安全设计上一个系统要遵循最小权限原则
网络服务随处可见，从而使得黑客可以轻易地利用它进行拒绝服务攻击。所以，设计系统时，需要遵循“最小权限”原则，采用白名单等方式。

# 5. 需要提供以下文档
编写单元测试文档并使其拥有良好的代码覆盖率。
高层次的设计图：描述了所有的组件，交互和结构。
详细的设计图：具体到代码层面的设计，以及一些关键逻辑的流程。
系统组成文档：说明系统的所有组成文件、配置文件等。
数据库层面的dml以及ddl文档，尤其是sql查询语句需要经过dba或者核心开发人员的review才能够上线。
不仅仅对于传统的开发流程，即使对于敏捷开发，这些文档也是必不可少的，否则在后续的维护、交接上会带来很大的不便。

# 6. 做好系统关键功能的监控、错误恢复、备份等
对于系统一些至关重要的功能模块要做好对其的监控，防止其影响系统的运行,造成不可估算的损失。另外，如果可以，监控到故障后去去试图恢复，恢复失败再发送告警。对于一些很重要的数据文件，还要做到冗余备份，防止发生一些突然故障造成数据丢失。

# 7. 数据库设计时设计一些便于追踪历史、整理的列
比如create_time、update_time可以说明记录的创建和更新时间。create_by、update_by可以说明记录是由谁创建和更新的。

此外，删除记录有时候并非真正删除，这时需要设计表示此记录状态的列，如可以取‘Active’或‘Inactive’的 ‘status’列。

# 8. 制定好项目回滚计划
新的功能上线时，如果发生故障，没有一份回滚计划，那么可能会手忙脚乱而造成线上服务一段时间不可用。有一个良好的回滚计划，可以让你能够有条不紊的执行相关操作，在可控时间内将系统恢复到一个可运行的状态。

# 9. 项目上线前要做好量化分析
对于项目中用到的内存、数据库、文件、缓存等，要做好量化分析。预估出未来一段时间的空间占用，给运维分配机器时一个参考。防止，由于数据量增长过快，导致存储不够。这一点是非常重要的，不然很容易造成线上服务不可用。

# 10. 制定好系统的部署计划。
系统部署的平台是一个至关重要的部分。对于部署平台的描述，不能仅限于一台服务器、两个数据库这个层面，至少需要包括

操作系统的特定版本，JVM等。
有多少内存（包括物理内存，JVM堆内存，JVM栈内存和JVM永久代的空间）。
CPU（内核数）。
负载均衡器，需要的节点数、节点类型，比如是Active-Standby型还是Active-Active型。
文件系统要求，例如，你的应用程序可能会收集生成的日志并将其保存很长的周期，之后才进行归档。这样的话，你就需要有足够的硬盘空间。

# 11. 选择最合适的工具/技术
很多情况下，开发者会在生产系统中使用一门想要学习的语言或某种工具。通常这不是最好的选择。比如，为已经实际上是关系型的数据使用NoSQL数据库。不管是语言还是工具，都有其适用的场景。不能求新，也不能以“自我”为标准。

# 12. 在一些关键技术领域具有充足的知识储备。
设计模式
JVM调优
多线程“并发问题”
事务问题，包括分布式事务
性能问题，包括GC、计算等
缓存

# 13. Java编码注意点
谨慎对待Java的循环遍历

对于集合类如List的遍历可以使用两种方式，如下：
``` java
  private final List<User> userList;
  for(User u : userList) {    
      ...
  }

  private final List<User> userList;
  for(int i = 0;i < userList.size() ; i++) {   
      User u = userList.get(i); 
      ...
  }
```

前者多了创建迭代器的开销，但是取元素的时候是O(1)复杂度(getNext、hasNext())。后者没有迭代器的开销，能够节省一部分开销，但是获取元素的地方会因List的具体实现类不同而不同，如LinkedList就会为O(n)、ArrayList为O(1)。要根据业务目标来选择合适的用法。

在初始化的时候预估集合的大小

这个主要是对HashMap说的。对于HashMap来说主要有两个影响性能的因素：初始大小和加载因子，当哈希表的大小达到初始大小和加载因子的乘积的时候，哈希表会进行rehash操作会影响性能。因此在初始化HashMap时要尽量能够预估其大小并做设置，如：
``` java
  new HashMap<String, User>((int)Math.ceil(userList.size() / 0.7)); 
```
延迟表达式的计算

Java中，如果方法的参数是一个表达式，那么都会在方法调用之前进行计算。因此，对于一些非常耗资源的操作，要尽量让其延迟化计算。如：
``` java
  class User{
      private String name;
      private Address address;

      public int compareTo(User other){
           return ComparisonChain.start()
              .compare(this.name, other.getName())
              .compare(this.address.toString(), user1.getAddress().toString())
              .result();
      }
  }
```
使用guava的ComparisonChain能够在第一个非0比较结果时就结束整个比较操作，但上面的代码中address的toString方法仍然会执行。可以通过使用单独的Comparator来使其延迟计算。
``` java
  return ComparisonChain.start()
              .compare(this.name, other.getName())
              .compare(this.address, user1.getAddress(), new Comparator<Address>(){
                  @Override
                  public int compare(Address a1, Address a2) {
                      return a1.toString().compareTo(a2.toString());
                  }
              })
              .result();
```
提前编译正则表达式

Java中的正则表达式运算是非常耗资源的，因此JDK中有正则参数的方法调用都最好提前编译正则表达式。如String的replaceAll方法，其第一个参数是一个正则表达式，可以通过缓存Pattern来避免每次都创建模式。
``` java
  private final Pattern pattern = Pattern.compile("<regx>"); //全局缓存
  ...
  pattern.matcher(“<text>”).replaceAll("<replacement>"); 
  ...
```