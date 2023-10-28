# 第一章、 RPC基础知识

## 1.1 软件架构

### 1.1.1 单一应用架构

当网站流量很小时，应用规模小时，只需一个应用，将所有功能都部署在一起，以减少部署服务器数量和成本。
此时，用于简化增删改查工作量的数据访问框架(ORM)是关键。数据库的处理时间影响应用的性能。

![image.png](https://s2.loli.net/2023/10/27/Uv71Rtiexw5KFmS.png)

这种结构的应用适合小型系统，小型网站，或者企业的内部系统，用户较少，请求量不大，对请求的处理时间没有太高的要求。
将所有功能都部署到一个服务器，简单易用。开发项目的难度低。

    缺点:
    1、性能扩展比较困难
    2、不利于多人同时开发
    3、不利于升级维护
    4、整个系统的空间占用比较大

### 1.1.2 分布式服务架构

当应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。
此时，用于提高业务复用及整合的 **分布式服务框架(RPC)** 是关键。分布式系统将服务作为独立的应用，实现服务共享和重用。

![image.png](https://s2.loli.net/2023/10/27/572bD6rQsmdYGjU.png)

## 1.2 分布式系统

### 1.2.1 什么是分布式系统

分布式系统是若干独立计算机（服务器）的集合，这些计算机对于用户来说就像单个相关系统，分布式系统（distributed
system）是建立在网络之上的服务器端一种结构。
分布式系统中的计算机可以使用不同的操作系统，可以运行不同应用程序提供服务，将服务分散部署到多个计算机服务器上。

### 1.2.2 RPC

RPC (Remote Procedure Call) 是指远程过程调用，是一种进程间通信方式，是一种技术思想，而不是规范。
它允许程序调用另一个地址空间（网络的另一台机器上）的过程或函数，而不用开发人员显式编码这个调用的细节。
调用本地方法和调用远程方法一样。

RPC 的实现方式可以不同。例如 java 的 rmi, spring 远程调用等。

RPC 概念是在上世纪 80 年代由 Brue Jay Nelson(布鲁·杰伊·纳尔逊)提出。
使用 PRC 可以将本地的调用扩展到远程调用（分布式系统的其他服务器）。

    RPC 的特点:
    简单：使用简单，建立分布式应用更容易。
    高效：调用过程看起来十分清晰，效率高。
    通用：进程间通讯的方式，有通用的规则。

### 1.2.3 RPC 基本原理

![image.png](https://s2.loli.net/2023/10/27/ZTOMdYR7asoSGNt.png)

1. 调用方 client 要使用右侧 server 的功能（方法），发起对方法的调用。
2. client stub 是 PRC 中定义的存根，看做是 client 的助手。stub 把要调用的方法参数进行序
   列化，方法名称和其他数据包装起来。
3. 通过网络socket(网络通信的技术)，把方法调用的细节内容发送给右侧的 server
4. server 端通过 socket 接收请求的方法名称，参数等数据，传给 stub。
5. server 端接到的数据由 server stub(server 的助手)处理，调用 server 的真正方法，处理业务
6. server 方法处理完业务，把处理的结果对象（Object）交给了助手，助手把 Object 进行序
   列化，对象转为二进制数据。
7. server 助手二进制数据交给网络处理程序
8. 通过网络将二进制数据，发送给 client。
9. client 接数据，交给 client 助手。
10. client 助手，接收数据通过反序列化为 java 对象（Object），作为远程方法调用结果。转换为Markdown格式

# 第二章、 Dubbo框架

## 2.1 dubbo 概述

Apache Dubbo (incubating) |ˈdʌbəʊ| 是一款高性能、轻量级的开源 Java RPC 框架，它提供了三大核心能力：面向接口的远程方法调用，智能容错和负载均衡，以及服务自动注册和发现。

Dubbo 是一个分布式服务框架，致力于提供高性能和透明化的 RPC 远程服务调用方案、服务治理方案。

官网地址: [Apache Dubbo 中文](https://cn.dubbo.apache.org/)

![image.png](https://s2.loli.net/2023/10/28/BLYxofQwKcl7CEs.png)

面向接口代理：调用接口的方法，在 A 服务器调用 B 服务器的方法，由 dubbo 实现对 B 的调用，无需关心实现的细节，就像 MyBatis 访问
Dao 的接口，可以操作数据库一样。不用关心 Dao 接口方法的实现。这样开发是方便，舒服的。

## 2.2 基本框架

![image.png](https://s2.loli.net/2023/10/28/3osyJT2POHmrWXZ.png)

- 服务提供者（Provider）：暴露服务的服务提供方，服务提供者在启动时，向注册中心注册自己提供的服务。
- 服务消费者（Consumer）: 调用远程服务的服务消费方，服务消费者在启动时，向注册中心订阅自己所需的服务，服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
- 注册中心（Registry）：注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者
- 监控中心（Monitor）：服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心

调用关系说明:

服务容器负责启动，加载，运行服务提供者。

服务提供者在启动时，向注册中心注册自己提供的服务。

服务消费者在启动时，向注册中心订阅自己所需的服务。

注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。

服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用， 如果调用失败，再选另一台调用。

服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

## 2.3 dubbo 支持的协议

支持多种协议：dubbo , hessian , rmi , http, webservice , thrift , memcached , redis。dubbo 官方推荐使用 dubbo 协议。dubbo
协议默认端口 20880

使用 dubbo 协议，spring 配置文件加入：

`<dubbo:protocol name="dubbo" port="20880" />`

## 2.4 直连方式 dubbo

点对点的直连项目:消费者直接访问服务提供者，没有注册中心。消费者必须指定服务提供者的访问地址（url）。

消费者直接通过 url 地址访问固定的服务提供者。这个 url 地址是不变的。

![image.png](https://s2.loli.net/2023/10/28/vXIMTSNuAG4Zq3x.png)

### 2.4.1 实现目标

用户访问 ------>【商品网站服务】访问-----> 【用户服务】

### 2.4.2 实现方式

#### (1) 创建服务提供者：用户服务

项目名称: link-userservice-provider

pom.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.softeem.dubbo</groupId>
    <artifactId>001-link-userservice-provider</artifactId>
    <version>1.0.0</version>
    <packaging>war</packaging>

    <dependencies>
        <!--Spring依赖-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>4.3.16.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>4.3.16.RELEASE</version>
        </dependency>

        <!--dubbo依赖-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.6.2</version>
        </dependency>
    </dependencies>


    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

```

创建用户实体类：User

```java
//实体类对象必须要实现序列化接口,否则不法在网络间传递
public class User implements Serializable {
    private Integer id;
    private String username;
    private Integer age;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```

新建用户服务接口：UserService

```java
public interface UserService {

    /**
     * 根据用户标识获取用户信息
     * @param id
     * @return
     */
    User queryUserById(Integer id);
}
```

新建接口的实现类：UserServiceImpl

```java
public class UserServiceImpl implements UserService {

    @Override
    public User queryUserById(Integer id) {

        User user = new User();
        user.setId(id);
        user.setUsername("XiaoMing");
        user.setAge(18);

        return user;
    }
}
```

创建 dubbo 配置文件: userservce-provider.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo
       http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <!--服务提供者声明名称:必须保证服务名称的唯一性,它的名称是dubbo内部使用的唯一标识-->
    <dubbo:application name="001-link-userservice-provider"/>
    <!--访问服务协议的名称及端口号,dubbo官方推荐使用的是dubbo协议,端口号默认为20880-->
    <!--
        name:指定协议的名称
        port:指定协议的端口号(默认为20880)
    -->
    <dubbo:protocol name="dubbo" port="20880"/>
    <!--
        暴露服务接口->dubbo:service
        interface:暴露服务接口的全限定类名
        ref:接口引用的实现类在spring容器中的标识
        registry:如果不使用注册中心,则值为:N/A[直连方式]
    -->
    <dubbo:service interface="com.softeem.dubbo.service.UserService" ref="userService" registry="N/A"/>

    <!--将接口的实现类加载到spring容器中-->
    <bean id="userService" class="com.softeem.dubbo.service.impl.UserServiceImpl"/>

</beans>
```

安装本地 jar 到 maven 仓库:

服务接口中的方法要给消费者使用，消费者项目需要知道接口名称和接口中的方法名称、参数等。这些信息服务提供者才知道。需要把接口的
class 文件打包为 jar .

服务接口项目的类文件打包为 jar， 安装到 maven 仓库，仓库中的提供者 jar 可以被消费者使用。

使用 idea 的 maven 窗口执行 install

#### (2)创建服务消费者：用户网站

duboo 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo
       http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <!--声明服务消费者的名称:保证唯一性-->
    <dubbo:application name="002-link-consumer"/>
    <!--
        引用远程服务接口:
        id:远程服务接口对象名称
        interface:调用远程接口的全限定类名
        url:访问服务接口的地址
        registry:不使用注册中心,值为:N/A
    -->
    <dubbo:reference id="userService"
                     interface="com.softeem.dubbo.service.UserService"
                     url="dubbo://localhost:20880"
                     registry="N/A"/>

</beans>
```

## 2.5 dubbo 常用标签

Dubbo 中常用标签。分为三个类别：公用标签，服务提供者标签，服务消费者标签。

**1. 公用标签 (Both 服务提供者 and 服务消费者)**

这部分的标签对于服务提供者和服务消费者都是通用的。

- **应用信息配置**

  ```xml
  <dubbo:application name="服务的名称"/>
  ```

  **说明：** 这个标签用于配置应用的名称。

- **注册中心配置**

  ```xml
  <dubbo:registry address="ip:port" protocol="协议"/>
  ```

  **说明：** 这个标签用于配置注册中心的地址和所使用的协议。

---

**2. 服务提供者标签**

这部分的标签主要用于服务提供者配置。

- **暴露的服务配置**

  ```xml
  <dubbo:service interface="服务接口名" ref="服务实现对象 bean"/>
  ```

  **说明：** 该标签用于配置要暴露的服务。其中，`interface` 属性表示服务的接口名，`ref` 属性表示服务的实现对象 bean 的引用。

---

**3. 服务消费者标签**

这部分的标签主要用于服务消费者配置。

- **远程服务引用配置**

  ```xml
  <dubbo:reference id="服务引用 bean 的 id" interface="服务接口名"/>
  ```

  **说明：** 该标签用于配置服务消费者引用的远程服务。其中，`id` 属性表示服务引用的 bean 的 id，`interface` 属性表示要引用的远程服务的接口名。

# 第三章、Zookeeper 注册中心

## 3.1 注册中心的概述

对于服务提供方，它需要发布服务，而且由于应用系统的复杂性，服务的数量、类型也不断膨胀；对于服务消费方，它最关心如何获取到它所需要的服务，而面对复杂的应用系统，
需要管理大量的服务调用。

而且，对于服务提供方和服务消费方来说，他们还有可能兼具这两种角色，即需要提供服务，有需要消费服务。
通过将服务统一管理起来，可以有效地优化内部应用对服务发布、使用的流程和管理。
服务注册中心可以通过特定协议来完成服务对外的统一。

      Dubbo 提供的注册中心有如下几种类型可供选：
      Multicast 注册中心：组播方式。
      Redis 注册中心：使用 Redis 作为注册中心。
      Simple 注册中心：就是一个 dubbo 服务。作为注册中心。提供查找服务的功能。
      Zookeeper 注册中心：使用 Zookeeper 作为注册中心推荐使用 Zookeeper 注册中心。

## 3.2 注册中心工作方式

![image.png](https://s2.loli.net/2023/10/28/dZqbCtpfXTQ8DI9.png)

## 3.3 Zookeeper 注册中心

Zookeeper 是一个高性能的，分布式的，开放源码的分布式应用程序协调服务。
简称 zk。Zookeeper 是翻译管理是动物管理员。
可以理解为 windows 中的资源管理器或者注册表。
他是一个树形结构。这种树形结构和标准文件系统相似。
ZooKeeper 树中的每个节点被称为Znode。
和文件系统的目录树一样，ZooKeeper 树中的每个节点可以拥有子节点。
每个节点表示一个唯一服务资源。Zookeeper 运行需要 java 环境。

官网下载地址: [Apache ZooKeeper](http://zookeeper.apache.org/)

### 3.3.1 Windows 平台 Zookeeper 安装，配置

下载的文件zookeeper-3.5.4-beta.tar.gz. 解压后到目录就可以了，例如 d:/servers/ zookeeper- 3.5.4

修改 zookeeper-3.5.4/conf/ 目录下配置文件

复制 zoo-sample.cfg 改名为 zoo.cfg

![image.png](https://s2.loli.net/2023/10/28/wfTt5FkzcIygJSi.png)

文件内容：

![image.png](https://s2.loli.net/2023/10/28/yM9IOmLAfRFGvib.png)

tickTime:心跳的时间，单位毫秒. Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime
时间就会发送一个心跳。表明存活状态。

dataDir:数据目录，可以是任意目录。存储 zookeeper 的快照文件、pid 文件，默认为/tmp/zookeeper，建议在 zookeeper 安装目录下创建
data 目录，将 dataDir 配置改为/usr/local/zookeeper-3.4.10/data

clientPort:客户端连接 zookeeper 的端口，即 zookeeper 对外的服务端口，默认为 2181

需要配置内容：

      dataDir : zookeeper 数据的存放目录
      admin.serverPort=8888(原因：zookeeper 3.5.x 占用 8080)

### 3.3.2 Linux 平台 Zookeeper 安装、配置

Zookeeper 的运行需要 jdk。使用前 Linux 系统要安装好 jdk.

①：上传 zookeeper-3.5.4-beta.tar.gz.并解压解压文件 zookeeper-3.5.4-beta.tar.gz.

执行命令：`tar -zxvf zookeeper-3.5.4-beta.tar.gz. -C /usr/local/`

②：配置文件

在 zookeeper 的 conf 目录下，将 zoo_sample.cfg 改名为 zoo.cfg，`cp zoo_sample.cfg zoo.cfg`

zookeeper 启动时会读取该文件作为默认配置文件

![image.png](https://s2.loli.net/2023/10/28/piHmhjyBf2nvKD6.png)

③：启动 Zookeeper

启动（切换到安装目录的 bin 目录下）：`./zkServer.sh start`

![image.png](https://s2.loli.net/2023/10/28/4sjV7e3Nf5ctyuO.png)

④：关闭 Zookeeper

关闭（切换到安装目录的 bin 目录下）：`./zkServer.sh stop`

![image.png](https://s2.loli.net/2023/10/28/BL6VoP73gmbcGQT.png)

### 3.4 zookeeper 的只用

引人zookeeper依赖

```xml

<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>4.1.0</version>
</dependency>
```

配置文件只中指定zookeeper的协议和端口号

![image.png](https://s2.loli.net/2023/10/28/GlqMCpzR1c57vwo.png)

### 3.5 注册中心的健壮性

- 监控中心宕掉不影响使用，只是丢失部分采样数据
- 注册中心仍能通过缓存提供服务列表查询，但不能注册新服务
- 服务提供者无状态，任意一台宕掉后，不影响使用
- 服务提供者全部宕掉后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复

# 第四章、dubbo的配置

