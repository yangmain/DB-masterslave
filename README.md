# 数据库的读写分离

## 使用方法

RWDataSource dataSource=new RWDataSource(dbType,writeDataSource,readDataSource);
这样就可以获取到dataSource实例，根据传给statement或preparedStatement的sql语句，来决定是走write库，还是read库.


## 基本原理
即RWDataSource作为 主库与从库的代理，产生代理的conntion,
从而产生代理的statement，然后
通过拦截 sql 来判断是走主库还是从库,然后获取真正的库，从而获取真的connection
来执行相应的sql操作;

它的思路启发于mycat的HA-DataSource,这个项目是作为mycat的ha功能。本项目代码是在这个代码基础上进行整改而成，即原先ha的功能，更改为读写分离的功能.
因为我原先给项目组提供的读写分离方案是利用spring的切面，进行动态路由，但是在实际使用过程中出现，对于方法是否应当执行切面操作没有定论，因为业务代码太复杂。
所以一直思考能否从更底层的代码切入来进行数据源的路由,幸好遇到了ha-datasource这个项目。如果说spring的切面进行动态路由是在第3层，则进行sql拦截即为第5或6
层。这样对于业务的侵入性更少，从使用的方便性来说这个更好。
但是这个方案依然没有考虑主从复制之间的延迟问题，所以它的适用性还是很有局限的.
 
## 缺点
本组件只支持一主一从的数据库的读写分离。对于多从情况可考虑通过haproxy(lvs)作为多从的负载均衡器，来构建一主一从，而关于haproxy可以通过keepalive作失效转移;对于主服务的失效转移可考虑keepalive虚拟ip方案,在此不详述。
另本组件对于数据库主从之间的复制的时间延迟没有考虑。
