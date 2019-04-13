# cacheservice
cache service 
1. 需求

must have：
屏蔽业务层缓存访问的复杂性包括（网络问题如超时&重试&failover&连接池）
屏蔽多数据源（mongo、redis）
屏蔽运维操作给业务的影响（升级版本、迁移服务、扩缩容服务节点）
监控埋点（性能、qps、命中率、hit rate、报文大小等）

nice to have：
二进制序列化协议
性能优化（根据监控情况做优化，比如本地缓存、拦截报文太大的请求）
分布式锁 （单独提供，或者在cache service里提供）

2. 使用方视角

对cache service的依赖形式
jar包依赖或者proxy依赖
比较：jar包形式难度小，proxy依赖难度大。两者的差异在其他中间件的场景里都是类似的，比如sharding-jdbc vs mycat
jar形式（client jar + 缓存集群）
采用简单的get、put、delete访问缓存，类似下面这样：
xxxCacheFactory.getInstance().get()
xxxCacheFactory.getInstance().put()
xxxCacheFactory.getInstance().delete()
上面是简单的key设计，可以考虑大key(major key)+小key(minor key)的设计，xxxCacheFactory.getInstance().get(majorKeyId).get(minorKeyId)
proxy形式（client jar + server + 缓存集群）
proxy形式也需要client jar包，类似纯粹的jar形式


3. 提供方视角

需求参考需求部分
access cache，简单的访问接口，不用关注连接信息（ip、port、一系列连接池参数），不用考虑释放
LocalCache，具有本地缓存功能
access statistics监控埋点
模块分工
meta server

记录了每个业务的连接配置，根据业务诉求来映射集群（appid、配置等级（取决于集群搭建比如1g/2g/4g）、缓存server ip+port），这类配置用yaml比json更灵活
如果需要做二进制序列化协议，报文的schema也放在这里

clent jar：
连接不同的缓存服务器（比如mongo、redis）
暴露get、put、delete接口
本地缓存
埋点
sharding（可选）
二进制协议（可选）

4. 可能有用的参考资料：
阿里的tair，c++写的，proxy模式，有java客户端 https://github.com/alibaba/tair-java-client
codis
