

- session复制

  优点：web-server(Tomcat)原生支持，修改配置文件即可

  缺点：

  1. session同步需要数据传输，占用网络带宽，降低服务器集群的业务能力
  2. 任意一台web-server保存的数据都是所有web-server的session总和
  3. 大型分布式集群情况下，由于web-server都全量保存数据

- 客户端存储（不推荐）

  优点：服务器不存储session，用户保存自己的session信息到cookie中。节省服务端资源。

  缺点：

  1. 每次http请求，携带用户在cookie中完整信息，浪费网络带宽
  2. session数据放在cookie中，cookie长度有限制（4K），不能保存大量信息
  3. session数据放在cookie中，存在泄露、篡改等安全隐患

- Hash一致性

  优点：

  1. 只需要修改nginx配置，不需要修改应用代码
  2. 负载均衡，只要hash属性的值分布式均匀的，多台web-server的负载就是均匀的
  3. 可以支持web-server水平扩展

  缺点：

  1. 若web-server重启可能会导致部分session丢失，影响业务
  2. 如果web-server水平扩展，rehash后session重新分步，也会有部分用户路由不到正确的session

- 统一存储

  优点：

  1. 无安全隐患
  2. 可水平扩展，数据库/缓存水平切分即可
  3. web-server重启或扩容都不会session丢失

  缺点：

  1. 增加一次网络io的时间，如将所有的getSeesion方法替换为从Redis中查的方式。redis获取数据比从内存中获取慢的多
  2. 可用SpringSession完美解决

