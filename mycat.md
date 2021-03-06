
## myCat

![img](imgs/db/mycat_arch.jpg)

mycat DAL中间件 数据访问层(Data Access Layer)

#### :dvd: some questions :question:
- mycat能做什么，优点是什么
- 逻辑表&逻辑库？schema
- mod水平切分，其他手段？
- 垂直切分？
- 自动处理切分？策略？
- 父子关系，外键关联关系，相当于垂直切分，ER表
- mycat有什么缺点


```
           |-> mysql
           |
- mycat---|-> mysql
           |
           |-> mysql
```

- 遗留问题
   mysql之间要keepalived吗:question:
   benchmark性能测试怎么做:question:


---
- <b>优势</b>
  [要点都在这了:joy:](http://www.mycat.io/)
  - 读写分离
  - 负载均衡，HA
  - 分表分库，主要是水平切分
  - 其他：基于NIO-并发高，
  - 官方规划2.0版本<b>:star:通过Mycat Balance 替代第三方的Haproxy，LVS等第三方高可用，完整的兼容Mycat集群节点的动态上下线。</b>
- <b>缺点</b>
  - 事务性XA较弱，:question:事务性
    关于分布式事务性的描述参考[分布式事务](https://www.cnblogs.com/zengkefu/p/5742617.html)
  - 分页查询，limit n，只查到某个分片上的topn，或者后区间数据可能落在不同分片上，可能查询缓慢，DBA可优化
    [参](https://www.cnblogs.com/leeSmall/p/9539370.html)

#### :dvd: 垂直切分
   按业务规划细分
   不缩表，减少字段数
   不能算myCat的优势

#### :dvd: 水平切分&规则
缩表，减少记录数
1. 枚举法：
   通过在配置文件中配置可能的枚举id，自己配置分片，适用于按照省份或者区县来拆分数据类业务
```
  <rule>
  <columns>province_id</columns>
  <algorithm>enum-int</algorithm>
  </rule>
```
2. 固定分片hash算法
```
  <tableRule name="rule1">
      <rule>
        <columns>user_id</columns>
        <algorithm>func1</algorithm>
      </rule>
  </tableRule>
  <function name="func1" class="org.opencloudb.route.function.PartitionByLong">
    <property name="partitionCount">2,1</property>
    <property name="partitionLength">256,512</property>
  </function>

  本例的分区策略：希望将数据水平分成3份，前两份各占25%，第三份占50%。（故本例非均匀分区）
  |<---------------------1024------------------------>|
  |<----256--->|<----256--->|<----------512---------->|
  | partition0 | partition1 | partition2 |
```
3. 范围约定
  预先制定可能的id范围到某个分片
```
  <tableRule name="auto-sharding-long">
      <rule>
        <columns>user_id</columns>
        <algorithm>rang-long</algorithm>
      </rule>
  </tableRule>
```
4. 求模法
```
      <columns>user_id</columns>
      <algorithm>mod-long</algorithm>
```
5. 日期列分区法
   起始时间
   分区时间间隔
6. 通配取模 
7. ASCII码求模通配
   %n值分n个分区
8. 编程指定
   自定义算法
9. 字符串拆分hash解析
10. 一致性hash

#### :dvd: myCat HA

- keepalived & haproxy装在一起
- keepalived 负责抢VIP
- haproxy 负责分发给可用myCat

![img](imgs/db/myCatHa1.png)
![img](imgs/db/myCatHa.png)
1. HAProxy实现了Mycat多节点的集群高可用和负载均衡，而HAProxy自身的高可用则可以通过Keepalived来实现。因此，HAProxy主机上要同时安装HAProxy和Keepalived，Keepalived负责为该服务器抢占vip（虚拟ip，图中的192.168.209.130），抢占到vip后，对该主机的访问可以通过原来的ip（192.168.209.135）访问，也可以直接通过vip（192.168.209.130）访问。
2. Keepalived抢占vip有优先级，在keepalived.conf配置中的priority属性决定。但是一般哪台主机上的Keepalived服务先启动就会抢占到vip，即使是slave，只要先启动也能抢到（要注意避免Keepalived的资源抢占问题）。
3. HAProxy负责将对vip的请求分发到Mycat集群节点上，起到负载均衡的作用。同时HAProxy也能检测到Mycat是否存活，HAProxy只会将请求转发到存活的Mycat上。
4. 如果Keepalived+HAProxy高可用集群中的一台服务器宕机，集群中另外一台服务器上的Keepalived会立刻抢占vip并接管服务，此时抢占了vip的HAProxy节点可以继续提供服务。
5. 如果一台Mycat服务器宕机，HAPorxy转发请求时不会转发到宕机的Mycat上，所以Mycat依然可用。
综上：Mycat的高可用及负载均衡由HAProxy来实现，而HAProxy的高可用，由Keepalived来实现。<br>
[myCat HA reference](https://blog.csdn.net/l1028386804/article/details/76397064)

### :dvd: 附图
![img](imgs/db/1508220824685.png) <br><br><br>
![img](imgs/db/1508224104762.png) <br><br><br>
![img](imgs/db/1508224116977.png) <br><br><br>
![img](imgs/db/1508224133068.png) <br><br><br>
![img](imgs/db/1508224143652.png) <br><br><br>
![img](imgs/db/1508224155090.png) <br><br><br>
![img](imgs/db/1508224163862.png) <br><br><br>
![img](imgs/db/1508224175479.png) <br><br><br>
![img](imgs/db/1508224183405.png) <br><br><br>
![img](imgs/db/1508224195432.png) <br><br><br>
![img](imgs/db/1508232831054.png) <br><br><br>
![img](imgs/db/1508233065722.png) <br><br><br>
![img](imgs/db/1508233301178.png) <br><br><br>
![img](imgs/db/1508234022148.png) <br><br><br>
![img](imgs/db/1508234196383.png) <br><br><br>
![img](imgs/db/1508234209931.png) <br><br><br>
![img](imgs/db/1508234224944.png) <br><br><br>
![img](imgs/db/1508234232978.png) <br><br><br>
![img](imgs/db/1508234240747.png) <br><br><br>
![img](imgs/db/1508234256389.png) <br><br><br>
![img](imgs/db/1508234267301.png) <br><br><br>
![img](imgs/db/1508234279553.png) <br><br><br>
![img](imgs/db/1508234313005.png) <br><br><br>
![img](imgs/db/1508234321343.png) <br><br><br>
![img](imgs/db/1508234483676.png) <br><br><br>
![img](imgs/db/1508234508872.png) <br><br><br>
![img](imgs/db/1508234525067.png) <br><br><br>
![img](imgs/db/1508234545534.png) <br><br><br>
![img](imgs/db/1508234554521.png) <br><br><br>
