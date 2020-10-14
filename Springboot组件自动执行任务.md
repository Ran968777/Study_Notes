# Springboot启动时自动执行任务的几种方法：

​	介绍 ：我在做一个物联网项目的时候，有个需求是系统要监听udp的某个端口，因为设备会通过udp将报警发出。所以就要求系统在启动的同时启动一个udpServer的组件，下面就是针对在Springboot启动的同时执行组件任务的几种方法。

- ApplicationRunner 的方式

```java
/**
 * 描述:
 * @create 2020-10-14 11:24
 */
@Component
public class UDPServer implements ApplicationRunner {
 private final static Logger logger = getLogger(UDPServer.class);
    @Override
    public void run(ApplicationArguments args) throws Exception {
        logger.info("udp服务已启动");
    }
}
```

- CommandLineRunner的方式

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

/**
 * 描述:
 *
 * @author Ran
 * @create 2020-10-14 11:33
 */
@Component
public class UDPServer2 implements CommandLineRunner {
 private final static Logger logger = LoggerFactory.getLogger(UDPServer2.class);
    @Override
    public void run(String... args) throws Exception {
        logger.info("udp服务2已启动");
    }
}
```

- PostConstruct的方式

这种方式是在组件初始化的时候就开始运行了，跟上面两种方式不太一样，但也能达到效果。

```java
import org.slf4j.Logger;
import org.springframework.stereotype.Component;
import javax.annotation.PostConstruct;
import static org.slf4j.LoggerFactory.getLogger;

/**
 * 描述:
 *
 * @author Ran
 * @create 2020-10-14 11:37
 */
@Component
public class UDPServer3 {
    private final static Logger logger = getLogger(UDPServer3.class);

    @PostConstruct
    public void doServer() {
        logger.info("udp3服务已启动");
    }
}
```

启动后的日志如下：

```verilog
2020-10-14 11:45:12.668  INFO 7136 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2020-10-14 11:45:12.675  INFO 7136 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2020-10-14 11:45:12.675  INFO 7136 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.38]
2020-10-14 11:45:12.728  INFO 7136 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2020-10-14 11:45:12.729  INFO 7136 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 623 ms
2020-10-14 11:45:12.760  INFO 7136 --- [           main] c.r.s.springboot.component.UDPServer3    : udp3服务已启动
2020-10-14 11:45:12.842  INFO 7136 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-10-14 11:45:12.951  INFO 7136 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-10-14 11:45:12.958  INFO 7136 --- [           main] c.r.s.springboot.SpringbootApplication   : Started SpringbootApplication in 1.128 seconds (JVM running for 1.682)
2020-10-14 11:45:12.960  INFO 7136 --- [           main] c.r.s.springboot.component.UDPServer     : udp服务已启动
2020-10-14 11:45:12.960  INFO 7136 --- [           main] c.r.s.springboot.component.UDPServer2    : udp服务2已启动

```

与此同时，我注意到还可以用`Order`注解来定义任务执行的顺序，在UDPServer2加上`@Order(1)`，在UDPServer加上`@Order(10)`后。启动的效果就不一样了。  

```verilog
2020-10-14 13:14:19.864  INFO 23932 --- [           main] c.r.s.springboot.component.UDPServer2    : udp服务2已启动
2020-10-14 13:14:19.864  INFO 23932 --- [           main] c.r.s.springboot.component.UDPServer     : udp服务已启动
```

所以我又去研究了下`Order`注解，该注解在源码中有这么两句话：

```java
/*
 Lower values have higher priority. 数值越小优先级越高。
 please be aware that they do not influence singleton startup
 order which is an orthogonal concern determined by dependency relationships and
 {@code @DependsOn} declarations (influencing a runtime-determined dependency graph).数值不会影响bean的初始化顺序
*/                                                                                                                                                                           
```

 为了验证，我又将代码进行了修改 ，并且执行结如下：

```java
/**
 * 描述:
 *
 * @create 2020-10-14 11:24
 */
@Component
@Order(10)
public class UDPServer implements ApplicationRunner {
    private final static Logger logger = getLogger(UDPServer.class);

    public UDPServer() {
        logger.info("UDPServer的构造方法已执行");
    }

    @Override
    public void run(ApplicationArguments args) throws Exception {
        logger.info("udp服务已启动");
    }

}

@Component
@Order(1) 
public class UDPServer2 implements CommandLineRunner {
    private final static Logger logger = LoggerFactory.getLogger(UDPServer2.class);

    public UDPServer2() {
        logger.info("UDPServer2的构造方法已执行");
    }

    @Override
    public void run(String... args) throws Exception {
        logger.info("udp服务2已启动");
    }
}

// 执行结果
2020-10-14 13:29:34.419  INFO 19196 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1118 ms
2020-10-14 13:29:34.448  INFO 19196 --- [           main] c.r.s.springboot.component.UDPServer     : UDPServer的构造方法已执行
2020-10-14 13:29:34.448  INFO 19196 --- [           main] c.r.s.springboot.component.UDPServer2    : UDPServer2的构造方法已执行
2020-10-14 13:29:34.449  INFO 19196 --- [           main] c.r.s.springboot.component.UDPServer3    : udp3服务已启动
2020-10-14 13:29:34.527  INFO 19196 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-10-14 13:29:34.631  INFO 19196 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-10-14 13:29:34.637  INFO 19196 --- [           main] c.r.s.springboot.SpringbootApplication   : Started SpringbootApplication in 3.217 seconds (JVM running for 6.735)
2020-10-14 13:29:34.638  INFO 19196 --- [           main] c.r.s.springboot.component.UDPServer2    : udp服务2已启动
2020-10-14 13:29:34.638  INFO 19196 --- [           main] c.r.s.springboot.component.UDPServer     : udp服务已启动
    
```

​                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              

