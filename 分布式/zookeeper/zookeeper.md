## 1. Zookeeper介绍

### 1.1 什么是Zookeeper

+ ZooKeeper 是⼀种分布式 <font color=red>**`协调服务`**</font>，⽤于管理大型主机。

  > 在分布式环境中 <font color=red>**`协调`**</font> 和 <font color=red>**`管理`**</font> 服务是 一个复杂的过程。ZooKeeper 通过其简单的架构和 API 解决了这个问题。ZooKeeper 允许开 发⼈员专注于核心应⽤程序逻辑，⽽不必担⼼应⽤程序的分布式特性。
  >

### 1.2 Zookeeper的应用场景

#### 1.2.1 分布式协调组件

+ 分布式协调组件

  > ![image-20220523214322643](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220523214322643.png)
=======
  > 在分布式系统中，需要有 zookeeper 作为分布式协调组件，协调分布式系统中的 <font color=red>**`状态`**</font>。

#### 1.2.1 分布式锁

+ zk在实现分布式锁上，可以做到强⼀致性，关于分布式锁相关的知识，在之后的 ZAB 协议中介 绍。

#### 1.2.3 无状态化的实现

+ 如下图所示：

  > 用户在登录的时候，如果将登录信息存放在服务器 1 上是不合适的，因为如果用户下一次登录的时候，通过 nginx 有可能登录到系统 2，3 上。所以我们统一的将登录信息放在 zk 上，那么服务器 1，2，3 就无需关系用户到底在哪台服务器上登录了，因为登录的状态信息在 zk 上维护。这个时候就实现了系统1，2，3 的无状态化。
  >
  > ![image-20220523214443891](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220523214443891.png)

## 2. 搭建zk服务器

+ 下载地址：https://www.apache.org/dyn/closer.lua/zookeeper/zookeeper-3.8.0/apache-zookeeper-3.8.0-bin.tar.gz

+ 安装 zk: 解压缩即可

  > 我们的安装目录是：/root/zk
  >
  > ```shell
  > [root@localhost zk]# tar -zxvf apache-zookeeper-3.8.0-bin.tar.gz 
  > ```
  >
  > 解压缩之后删除 zk 安装包
  >
  > ```shell
  > [root@localhost zk]# rm -rf apache-zookeeper-3.8.0-bin.tar.gz 
  > ```

+ 基本目录概述

  > ![image-20220523224611703](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220523224611703.png)
  >
  > ```shell
  > [root@localhost apache-zookeeper-3.8.0-bin]# cd bin/
  > ```
  >

### 2.1 zoo.cfg 配置文件说明

+ zk 为我们提供了一个模板配置文件 zoo_sample.cf, 配置文件主要参数说明

  > ```properties
  > # zookeeper时间配置中的基本单位 (毫秒)
  > tickTime=2000
  > 
  > # 允许follower初始化连接到leader最⼤时⻓，它表示tickTime时间倍数
  > # 即:initLimit*tickTime
  > initLimit=10
  > 
  > # 允许follower 与 leader 数据同步最⼤时⻓, 它表示 tickTime 时间倍数
  > syncLimit=5
  > 
  > #zookeper 数据存储⽬录及⽇志保存⽬录(如果没有指明dataLogDir，则⽇志也保存在这个⽂件中)
  > dataDir=/tmp/zookeeper
  > 
  > #对客户端提供的端⼝号
  > clientPort=2181
  > 
  > #单个客户端与 zookeeper 最⼤并发连接数
  > maxClientCnxns=60
  > 
  > # 保存的数据快照数量，之外的将会被清除
  > autopurge.snapRetainCount=3
  > 
  > #⾃动触发清除任务时间间隔，⼩时为单位。默认为0，表示不⾃动清除。
  > autopurge.purgeInterval=1
  > ```

### 2.2 zk服务器的操作命令

+ 重命名 conf 中的文件zoo_sample.cfg -> zoo.cfg

  > ```shell
  > [root@localhost conf]# mv zoo_sample.cfg zoo.cfg
  > ```

+ 修改 zoo.cfg 中的数据存储目录

  > ```properties
  > [root@localhost conf]# vi zoo.cfg
  > 
  > dataDir=dataDir=/root/zk/apache-zookeeper-3.8.0-bin/zkData
  > ```

+ 启动 zk 服务器

  > ```shell
  > [root@localhost bin]# ./zkServer.sh start ../conf/zoo.cfg 
  > ```
  >
  > ![image-20220523231045407](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220523231045407.png)

+ 查看 zk 服务器状态

  > ```sh
  > [root@localhost bin]# ./zkServer.sh status ../conf/zoo.cfg 
  > ```
  >
  > ![image-20220523231247812](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220523231247812.png)

+ 停止 zk 服务器

  > ```sh
  > [root@localhost bin]# ./zkServer.sh stop ../conf/zoo.cfg 
  > ```
  >
  > ![image-20220523231437315](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220523231437315.png)

## 3. zk内部的数据模型

### 3.1 zk是如何保存数据的

+ zk 中的数据是保存在 <font color=red>**`节点`**</font> 上的，节点就是 <font color=red>**`znode`**</font>，多个 <font color=red>**`znode`**</font> 之间构成⼀颗 <font color=red>**`树`**</font> 的目录结构。 

  > Zookeeper 的数据模型是什么样子呢？它很像数据结构当中的 <font color=red>**`树`**</font>，也很像文件系统的 <font color=red>**`目录`**</font>。
  >
  > <img src="https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220523232258556.png" alt="image-20220523232258556" style="zoom: 33%;" />

+ 树是由 <font color=red>**`节点`**</font> 所组成，Zookeeper 的数据存储也同样是基于 <font color=red>**`节点`**</font>，这种节点叫做<font color=red> **`Znode`**</font> 但是，不同于树的节点，Znode 的引用方

  式是 <font color=red>**`路径`**</font> 引用，类似于 <font color=red>**`文件路径`**</font>

  > ```properties
  > /动物/猫
  > /汽⻋/宝⻢
  > ```
  >
  > 这样的层级结构，让每⼀个 <font color=red>**`Znode`**</font> 节点拥有 <font color=red>**`唯⼀`**</font> 的路径，就像命名空间⼀样对不同信息作出 清晰的 <font color=red>**`隔离`**</font>。

### 3.2 znode 结构

+ zk 中的 znode，包含了四个部分：

  > 1. <font color=blue>**data**</font>：保存数据
  > 2. <font color=blue>**acl**</font>：权限，定义了什么样的用户能够操作这个节点，且能够进行怎样的操作。
  >    + <font color=pink>**c** </font>: create 创建权限，允许在该节点下创建子节点
  >    + <font color=pink>**w**</font> ：write 更新权限，允许更新该节点的数据 
  >    + <font color=pink>**r** </font>：read 读取权限，允许读取该节点的内容以及子节点的列表信息 
  >    + <font color=pink>**d**</font> ：delete 删除权限，允许删除该节点的子节点 
  >    + <font color=pink>**a**</font> ：admin 管理者权限，允许对该节点进行 acl 权限设置
  > 3. <font color=blue>**stat**</font>：描述当前znode的元数据
  > 4. <font color=blue>**child**</font>：当前节点的⼦节点

### 3.3 znode 的类型及创建

+ 客户端简单操作

  > 1. 启动一个客户端
  >
  >    ```sh
  >    [root@localhost bin]# ./zkCli.sh 
  >    ```
  >
  > 2. 查询根节点下所有节点
  >
  >    ```sh
  >    [zk: localhost:2181(CONNECTED) 0] ls /
  >    ```
  >
  >    ![image-20220524084451605](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524084451605.png)

1. <font color=blue>**持久节点**</font>: 创建出的节点，在会话结束后依然存在。保存数据 

   > 1. 在根节点创建一个持久节点 test1
   >
   >    ```shell
   >    [zk: localhost:2181(CONNECTED) 2] create /test1
   >    ```
   >
   >    ![image-20220524084732696](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524084732696.png)
   >
   > 2. 查询所创建的节点
   >
   >    ```shell
   >    [zk: localhost:2181(CONNECTED) 4] ls /
   >    ```
   >
   >    ![image-20220524084942613](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524084942613.png)

2. <font color=blue>**持久序号节点**</font>: 创建出的节点，根据先后顺序，会在节点之后带上⼀个数值，越后执行数值 <font color=red>**`越大`**</font>

   > 1. 适⽤于 <font color=red>**`分布式锁`**</font> 的应⽤场景 - <font color=red>**`单调递增`**</font>
   >
   > 2. 继续创建一个节点 test1(上面我们已经创建了 test1)
   >
   >    ![image-20220524085140004](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524085140004.png)
   >
   > 3. 创建一个持久需要节点：加参数 -s, 执行下面的命令 4 次
   >
   >    ```sh
   >    [zk: localhost:2181(CONNECTED) 6] create -s /test1 
   >    ```
   >
   >    ![image-20220524085724439](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524085724439.png)

3. <font color=blue>**临时节点**</font>：临时节点是在会话结束后，自动被 <font color=red>**`删除`**</font> 的

   > 1. 创建一个零时节点：使用参数 -e
   >
   >    ```shell
   >    [zk: localhost:2181(CONNECTED) 11] create -e /test2
   >    ```
   >
   >    ![image-20220524090446614](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524090446614.png)
   >
   > 2. 新开一个 xshell 会话 <font color=red>`session2`</font>，将 1 中使用的 xshell <font color=red>`session1`</font> 会话关闭，过30 秒，使用 session2 会话查询节点
   >
   >    ![image-20220524090739139](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524090739139.png)
   >
   >    我们发现 <font color=red>`test2`</font> 这个节点不在了，因为与之相连的 session1 会话已经关闭了，test2 节点会自动被 <font color=red>**`删除`**</font>。通过这个特性 zk 可以实现服务 <font color=red>**`注册`**</font> 与 <font color=red>**`发现`**</font> 的效果。那么临时节点是如何维持心跳呢?![image-20220523234415478](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220523234415478.png)
   >
   
4. <font color=blue>**临时序号节点**</font>：跟持久序号节点相同，适⽤于<font color=red> **`临时`**</font> 的 <font color=red>**`分布式锁`**</font>。

   > 1. 创建临时需要节点：加参数 -s -e, 或者 -es
   >
   >    ```sh
   >    [zk: localhost:2181(CONNECTED) 3] create -es /test3
   >    ```
   >
   >    ![image-20220524091702029](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524091702029.png)

5. <font color=blue>**Container节点(3.5.3版本新增)**</font>：Container 容器节点，当容器中没有任何子节点，该容器节点会被 zk 定期删除(60s)。

   > 1. 创建容器节点：使用参数 -c
   >
   >    ```shell
   >    [zk: localhost:2181(CONNECTED) 5] create -c /mycontainer
   >    ```
   >
   >    ![image-20220524092038162](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524092038162.png)
   >
   > 2. 在容器节点下，继续创建 node1, node2
   >
   >    ```shell
   >    [zk: localhost:2181(CONNECTED) 9] create /mycontainer/node1
   >    [zk: localhost:2181(CONNECTED) 9] create /mycontainer/node2
   >    ```
   >
   >    ![image-20220524092338224](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524092338224.png)
   >
   > 3. 删除 /mycontainer 下的 node1, node2 过一段时间再看看 mycontainer 节点是否存在
   >
   >    ```shell
   >    [zk: localhost:2181(CONNECTED) 12] delete /mycontainer/node1
   >    [zk: localhost:2181(CONNECTED) 12] delete /mycontainer/node2
   >    ```
   >
   >    ![image-20220524092647312](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524092647312.png)
   >
   >    mycontainer 节点已经不存在了！

6. <font color=blue>**TTL节点**</font>：可以指定节点的到期时间，到期后被 zk <font color=red>**`定时`**</font> 删除。只能通过系统配置

   > ```properties
   > ## 开启
   > zookeeper.extendedTypesEnabled=true
   > ```

## 4. zk的数据持久化

+ zk 的数据是运⾏在 <font color=red>`内存`</font> 中，zk 提供了两种持久化机制：<font color=red>`事务日志`</font>，<font color=red>`数据快照`</font>

  ![image-20220524093256175](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524093256175.png)

### 4.1 事务日志

+ zk 把执行的 <font color=red>**`命令`**</font> 以日志形式保存在 <font color=red>`dataLogDir`</font> 指定的路径中的文件中(如果没有指定 <font color=red>`dataLogDir`</font>，则按 <font color=red>`dataDir`</font> 指定的路径)

### 4.2 数据快照

+ zk 会在⼀定的时间 <font color=red>**`间隔`**</font> 内做⼀次内存数据的快照，把该时刻的内存数据保存在 <font color=red>**`快照⽂件`**</font> 中。

### 4.3 恢复顺序

+ 先快照，后日志

  > zk 通过两种形式的 <font color=red>**`持久化`**</font>，在恢复时先 <font color=red>**`恢复快照⽂`**</font> 件中的数据到内存中，再⽤ <font color=red>**`⽇志⽂件`**</font> 中的 数据做 <font color=red>`增量`</font> 恢复，这样的恢复速度更快。
  >

## 5. zk客户端(zkCli)的使用

### 5.1 创建节点

+ 之前我们已经在 znode 类型及创建这一小结讲过了如何创建各种类型的节点

+ 创建节点，并添加数据

  > 另外讲一点，之前创建节点，只是单纯的创建了个节点，我们可以在创建节点的时候添加数据
  >
  > ```sh
  > [zk: localhost:2181(CONNECTED) 16] create /test4 郝伟
  > ```
  >
  > ![image-20220524094040400](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524094040400.png)
  >
  > 获取节点数据
  >
  > ```shell
  > [zk: localhost:2181(CONNECTED) 18] get /test4
  > ```
  >
  > ![image-20220524094142510](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524094142510.png)

### 5.2 查询节点

#### 5.2.1 普通查询(ls)

+ 先创建这样一个多节点 /test5/sub1/sub2

  > ```shell
  > [zk: localhost:2181(CONNECTED) 20] create /test5
  > Created /test5
  > [zk: localhost:2181(CONNECTED) 21] create /test5/sub1
  > Created /test5/sub1
  > [zk: localhost:2181(CONNECTED) 22] create /test5/sub1/sub2
  > Created /test5/sub1/sub2
  > [zk: localhost:2181(CONNECTED) 23] 
  > ```

+ 查询 sub2 节点：使用 <font color=red>`ls`</font> 命令

  > ```shel
  > [zk: localhost:2181(CONNECTED) 23] ls /test5/sub1/sub2 
  > ```
  >
  > 每次都需要写 <font color=red>`/test5/sub1/sub2`</font> 这么长一串，很麻烦

+ 查询加参数 <font color=red>`-R`</font> : 递归的将 /test5 下的所有节点都查 询出来

  > ```shell
  > [zk: localhost:2181(CONNECTED) 24] ls -R /test5
  > /test5
  > /test5/sub1
  > /test5/sub1/sub2
  > [zk: localhost:2181(CONNECTED) 25] 
  > ```
  >
  > ![image-20220524094925071](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524094925071.png)

#### 5.2.2 获取节点数据(get)

+ 使用 get 命令获取存放在节点中的数据

  > ```shell
  > [zk: localhost:2181(CONNECTED) 27] get /test4
  > ```
  >
  > ![image-20220524100721874](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524100721874.png)

+ 获取节点详细信息 get -s

  > ```shell
  > [zk: localhost:2181(CONNECTED) 29] get -s /test4
  > ```
  >
  > ![image-20220524100908875](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524100908875.png)

+ 节点信息说明

  > ```properties
  > cZxid: 创建节点的事务ID
  > mZxid：修改节点的事务ID
  > pZxid：添加和删除⼦节点的事务ID
  > ctime：节点创建的时间
  > mtime: 节点最近修改的时间
  > dataVersion: 节点内数据的版本，每更新⼀次数据，版本会+1
  > aclVersion: 此节点的权限版本
  > ephemeralOwner: 如果当前节点是临时节点，该值是当前节点所有者的session
  > id。如果节点不是临时节点，则该值为零。
  > dataLength: 节点内数据的⻓度
  > numChildren: 该节点的⼦节点个数
  > ```

### 5.3 删除节点(delete)

#### 5.3.1 普通删除

+ 删除单节点 : delete

  > 这个节点下面没有任何节点
  >
  > ```shell
  > [zk: localhost:2181(CONNECTED) 32] delete /test1
  > ```

+ 删除层级节点

  > 删除带有子节点的节点
  >
  > ```shell
  > [zk: localhost:2181(CONNECTED) 38] ls -R /test5
  > ```
  >
  > ![image-20220524101613230](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524101613230.png)
  >
  > test5 节点下有 sub1, sub1 下有 sub2，删除 test5
  >
  > ```shell
  > [zk: localhost:2181(CONNECTED) 39] delete /test5
  > ```
  >
  > ![image-20220524101736683](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524101736683.png)
  >
  > test5 不是空节点，无法删除。使用 deleteall 删除 test5 以及下面所有的子节点
  >
  > ```shell
  > [zk: localhost:2181(CONNECTED) 40] deleteall /test5
  > ```

#### 5.3.2 乐观锁删除

+ 删除的命令中有个参数 v, 我们来探究以下: delete -v 

1. 先来创建一个节点 vtest1

   > ```shell
   > [zk: localhost:2181(CONNECTED) 42] create /vtest1
   > ```

2. 删除节点带版本号，随便给一个版本号，比如 1

   > ```shell
   > [zk: localhost:2181(CONNECTED) 43] delete -v 1 /vtest1 
   > ```
   >
   > ![image-20220524102229713](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524102229713.png)
   >
   > 报错：版本号是无效的

3. 获取 vtest1 的信息

   > ```shell
   > [zk: localhost:2181(CONNECTED) 38] ls -R /test5
   > ```
   >
   > ![image-20220524102432994](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524102432994.png)

4. 什么是数据版本号呢？看看接下来的操作：给 vtest1 节点中存数据 “abccc”

   > ```shell
   > [zk: localhost:2181(CONNECTED) 45] set /vtest1 abccc
   > ```
   >
   > ![image-20220524102737944](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524102737944.png)
   >
   > 只要我们 set 一次值，那么数据版本就会 +1，继续 set 一次值 “hhhh”
   >
   > ```shell
   > [zk: localhost:2181(CONNECTED) 49] set /vtest1 hhhh
   > ```
   >
   > ![image-20220524104816266](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524104816266.png)

5. 接下来我们继续删除版本 0

   > ```shell
   > [zk: localhost:2181(CONNECTED) 48] delete -v 0 /vtest1 
   > ```
   >
   > ![image-20220524103710499](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524103710499.png)
   >
   > 还是无法删除

6. 乐观锁删除

   > + vtest1 节点的数据版本现在是 1，我们作 delete -v 0 删除时，传入了版本号，zk 会拿着传入的版本号与 vtest1 当前的本号做对比，如果版本号不相同则无法删除(或者其他操作)。
   >
   > + 简单来说：<font color=red>**乐观锁是验证，不是阻止**</font>
   >
   > + 加版本号主要是应对并发场景下，数据一致性问题！

### 5.4 节点权限设置

#### 5.4.1 环境准备

+ 连接 zk 用的是 xshell

+ 使用 xshell 新开两个连接，命名为 session1, session2

  > ![image-20220524111801225](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524111801225.png)

+ session1, session2 分别启动 zk 客户端

  > ```shell
  > [root@localhost bin]# ./zkCli.sh 
  > ```

#### 5.4.2 设置节点权限

+ 注册当前会话的账号和密码：

  > 命令格式：
  >
  > ```shell
  > addauth digest xiaowang:123456
  > ```
  >
  > 给 session1 设置用户名/密码为 -- xiaowang/123456
  >
  > ```shell
  > [zk: localhost:2181(CONNECTED) 0] addauth digest xiaowang:123456
  > ```

+ 创建节点并设置权限

  > 命令格式
  >
  > ```shell
  > create /test-node abcd auth:xiaowang:123456:cdwra
  > ```
  >
  > 创建节点，并且设置用户/密码，并且指定操作权限
  >
  > ```shell
  > [zk: localhost:2181(CONNECTED) 1] create /test-node abcd auth:xiaowang:123456:cdwra
  > ```

+ 在 session2 中查询 test-node 节点

  > ```shell
  > [zk: localhost:2181(CONNECTED) 0] ls /test-node 
  > ```
  >
  > ![image-20220524112901283](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524112901283.png)
  >
  > ```shell
  > [zk: localhost:2181(CONNECTED) 1] get /test-node
  > ```
  >
  > ![image-20220524113015746](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524113015746.png)
  >
  > 提示我们权限不足，无法查看节点 test-node

+ 在另⼀个会话中必须先使用账号密码，才能拥有操作该节点的权限

  > 在 session2 中给 test-node 也添加相同的用户名/密码/操作权限
  >
  > ```shell
  > [zk: localhost:2181(CONNECTED) 0] addauth digest xiaowang:123456
  > ```
  >
  > 再次查看
  >
  > ```shell
  > [zk: localhost:2181(CONNECTED) 1] get /test-node
  > ```
  >
  > ![image-20220524113328772](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524113328772.png)

## 6. Curator客户端的使用

### 6.1 Curator介绍

+ Curator是 Netflix 公司开源的⼀套 zookeeper 客户端框架，Curator 是对 Zookeeper <font color=red>**`支持最好`**</font> 的客户端框架。

  > Curator 封装了大部分 Zookeeper 的功能，比如 <font color=red>**`Leader 选举`**</font>、<font color=red>**`分布式锁`**</font>等，减 少了技术⼈员在使用Zookeeper时的底层细节开发工作。

### 6.2 环境搭建

1. 引入Curator依赖

   > ```xml
   > <!--Curator-->
   >  <dependency>
   >  <groupId>org.apache.curator</groupId>
   >  <artifactId>curator-framework</artifactId>
   >  <version>2.12.0</version>
   >  </dependency>
   >  <dependency>
   >  <groupId>org.apache.curator</groupId>
   >  <artifactId>curator-recipes</artifactId>
   >  <version>2.12.0</version>
   >  </dependency>
   >  <!--Zookeeper-->
   >  <dependency>
   >  <groupId>org.apache.zookeeper</groupId>
   >  <artifactId>zookeeper</artifactId>
   >  <version>3.7.14</version>
   >  </dependency>
   > ```

2. application.properties 配置⽂件

   > ```properties
   > curator.retryCount=5
   > curator.elapsedTimeMs=5000
   > curator.connectString=172.16.253.35:2181
   > curator.sessionTimeoutMs=60000
   > curator.connectionTimeoutMs=5000
   > ```

3. 注入配置Bean

   > ```java
   > @Data
   > @Component
   > @ConfigurationProperties(prefix = "curator")
   > public class WrapperZK {
   >        private int retryCount;
   >        private int elapsedTimeMs;
   >        private String connectString;
   >        private int sessionTimeoutMs;
   >        private int connectionTimeoutMs;
   > }
   > ```

4. 注入 CuratorFramework

   > ```java
   > @Configuration
   > public class CuratorConfig {
   >      @Autowired
   >      WrapperZK wrapperZk;
   >        @Bean(initMethod = "start")
   >        public CuratorFramework curatorFramework() {
   >            return CuratorFrameworkFactory.newClient(
   >                wrapperZk.getConnectString(),
   >                wrapperZk.getSessionTimeoutMs(),
   >                wrapperZk.getConnectionTimeoutMs(),
   >                new RetryNTimes(wrapperZk.getRetryCount(),
   >                                wrapperZk.getElapsedTimeMs()));
   >      }
   > }
   > ```

### 6.3 创建节点

+ 默认创建的是持久节点

  > ```java
  > @Autowired
  >  CuratorFramework curatorFramework; 
  > 
  >  @Test
  >  void createNode() throws Exception {
  >      //添加持久节点
  >      String path = curatorFramework.create().forPath("/curator-node");
  >      //添加临时序号节点
  >      String path1 =
  >      curatorFramework.create().withMode(CreateMode.EPHEMERAL_SEQUENTIAL).forPath("/curator-node", "some-data".getBytes());
  >      System.out.println(String.format("curator create node :%s 
  >      successfully.",path));
  >      System.in.read();
  >  }
  > ```

### 6.4 获得节点数据

```java
@Test
public void testGetData() throws Exception {
    byte[] bytes = curatorFramework.getData().forPath("/curator-node");
    System.out.println(new String(bytes));
}
```

### 6.5 修改节点数据

---

```java
@Test
 public void testSetData() throws Exception {
     curatorFramework.setData().forPath("/curatornode","changed!".getBytes());
     byte[] bytes = curatorFramework.getData().forPath("/curator-node");
     System.out.println(new String(bytes));
 }
```

### 6.6 创建节点同时创建父节点

```java
@Test
public void testCreateWithParent() throws Exception {
    String pathWithParent="/node-parent/sub-node-1";
    String path 
        = curatorFramework.create().creatingParentsIfNeeded().forPath(pathWithParent);
    System.out.println(String.format("curator create node :%s successfully.",path));
}
```

### 6.7 删除节点

```java
@Test
 public void testDelete() throws Exception {
    String pathWithParent="/node-parent";
	curatorFramework.delete().guaranteed().deletingChildrenIfNeeded().forPath(pathWithParent);
 }
```

## 7. zk实现分布式锁

### 7.1 zk中锁的种类

1. <font color=red>**读锁**</font>: 读锁也成为共享锁，大家都可以读，要想上读锁的前提 --> 之前的锁没有写锁

2. <font color=red>**写锁**</font>: 只有得到写锁的才能写。要想上写锁的前提是 --> 之前没有任何锁。

### 7.2 zk如何上读锁

+ 创建⼀个 <font color=red>**`临时序号`**</font> 节点，节点的数据是 <font color=red>**`read`**</font>，表示是 <font color=red>**`读锁`**</font>

  > 临时序号节点：会话结束后，该节点会被删除

+ 获取当前 zk 中序号比自己小的所有节点；判断最小节点是否是读锁：

  > + 如果不是读锁的话，则上锁失败，为最小节点设置监听。阻塞等待，zk 的 watch 机制 会当最小节点发生变化时通知当前节点，于是再执行第二步的流程
  >
  > + 如果是读锁的话，则上锁成功
  >
  > <img src="https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524125449195.png" alt="image-20220524125449195" style="zoom:33%;" />

### 7.2 如何上写锁

+ 创建⼀个 <font color=red>**`临时序号`**</font> 节点，节点的数据是 <font color=red>`write`</font>，表示是 <font color=red>**`写锁`**</font>

+ 获取 zk 中所有的子节点；判断自己是否是最小的节点：

  > + 如果是，则上写锁成功
  >
  > + 如果不是，说明前⾯还有锁，则上锁失败，监听最小的节点，如果最小节点有变化， 则回到第二步。
  >
  >   <img src="https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524150401445.png" alt="image-20220524150401445" style="zoom:33%;" />

### 7.3 羊群效应

+ 如果用上述的上锁方式，只要有节点发生变化，就会触发其他节点的监听事件，这样的话对 zk 的压力非常大，这就是 <font color=red>**`羊群效应`**</font>。可以调整成 <font color=red>**`链式`**</font> 监听。解决这个问题。

  > <img src="https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524152615295.png" alt="image-20220524152615295" style="zoom:33%;" />

### 7.4 curator实现读写锁

+ 获取读锁

  > ```java
  > @Test
  >  void testGetReadLock() throws Exception {
  >      // 读写锁
  >      InterProcessReadWriteLock interProcessReadWriteLock=new InterProcessReadWriteLock(client, "/lock1");
  >      // 获取读锁对象
  >      InterProcessLock interProcessLock = interProcessReadWriteLock.readLock();
  >      System.out.println("等待获取读锁对象!");
  >      // 获取锁
  >      interProcessLock.acquire();
  >      for (int i = 1; i <= 100; i++) {
  >            Thread.sleep(3000);
  >            System.out.println(i);
  >      }
  >      // 释放锁
  >      interProcessLock.release();
  >      System.out.println("等待释放锁!");
  >  }
  > ```

+ 获取写锁

  > ```java
  >  @Test
  >  void testGetWriteLock() throws Exception {
  >        // 读写锁
  >        InterProcessReadWriteLock interProcessReadWriteLock = 
  >            new InterProcessReadWriteLock(client, "/lock1");
  >        // 获取写锁对象
  >        InterProcessLock interProcessLock = interProcessReadWriteLock.writeLock();
  >        System.out.println("等待获取写锁对象!");
  > 
  >        // 获取锁
  >        interProcessLock.acquire();
  >        for (int i = 1; i <= 100; i++) {
  >            Thread.sleep(3000);
  >            System.out.println(i);
  >        }
  >      
  >        // 释放锁
  >        interProcessLock.release();
  >        System.out.println("等待释放锁!");
  >  }
  > ```


## 8. zk的watch机制

### 8.1 watch机制介绍

+ 什么是 watch 机制

  > 我们可以把 <font color=red>`Watch`</font> 理解成是注册在特定 <font color=red>`Znode`</font> 上的 <font color=red>**`触发器`**</font>。当这个 Znode 发生 <font color=red>**`改变`**</font>，也就 是调用了 <font color=red>`create`</font>， <font color=red>`delete`</font>， <font color=red>`setData`</font> 方法的时候，将会触发 Znode 上注册的对应事件，请求 Watch 的客户端会接收到 <font color=red>**`异步通知`**</font>。
  >
  
+ 具体交互过程如下：

  > + 客户端调用 <font color=red>`getData`</font> 方法，watch 参数是 true。服务端接到请求，返回节点数据，并 且在对应的哈希表里插入被 Watch 的 <font color=red>**`Znode 路径`**</font>，以及 <font color=red>**`Watcher 列表`**</font>。 
  >
  > + 当被 Watch 的 Znode 已删除，服务端会查找哈希表，找到该 Znode 对应的所有 Watcher，<font color=red>**`异步`**</font> 通知客户端，并且删除哈希表中对应的 Key-Value。
  >
  >   ![image-20220524202742750](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524202742750.png)
  
+ 客户端使⽤了 NIO 通信模式监听服务端的调⽤。

### 8.2 zkCli客户端使用watch

+ 如何监听：使用 参数 -w

  > ```shell
  > create /test xxx
  > get -w /test ⼀次性监听节点
  > ls -w /test 监听⽬录,创建和删除⼦节点会收到通知。⼦节点中新增节点不会收到通知
  > ls -R -w /test 对于⼦节点中⼦节点的变化，但内容的变化不会收到通知
  > ```

+ 举例

  > + session1 创建节点 testwatch
  >
  >   ```shell
  >   [zk: localhost:2181(CONNECTED) 0] create /testwatch
  >   ```
  >
  > + session1 监听 testwatch 节点
  >
  >   ```shell
  >   [zk: localhost:2181(CONNECTED) 1] get -w /testwatch 
  >   ```
  >
  > + session2 给节点 testwatch 设置值 “bbb”
  >
  >   ```shell
  >   [zk: localhost:2181(CONNECTED) 0] set /testwatch bbb
  >   ```
  >
  >   观察 session1 中的监听结果
  >
  >   ![wwww](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/wwww.gif)
  >
  >   **WatchedEvent state:SyncConnected type:NodeDataChanged path:/testwatch**
  >
  > + session2 中再次给 testwatch 设置值 “aaa”, 这个时候 session1 中监听不到 testwatch 的变化了，因为 -w 是一次性监听；如何持续监听呢？====> session1 在每次获取 testwatch 的数据的时候都加上 -w 参数
  >
  >   ![image-20220524205520719](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524205520719.png)

### 8.3 curator客户端使用watch

+ 直接上代码

  > ```java
  >  @Test
  >  public void addNodeListener() throws Exception {
  >  
  >        NodeCache nodeCache = new NodeCache(curatorFramework, "/curator-node");
  >        nodeCache.getListenable().addListener(new NodeCacheListener() {
  >            @Override
  >            public void nodeChanged() throws Exception {
  >                log.info("{} path nodeChanged: ","/curator-node");
  >                printNodeData();
  >            }
  >        });
  >      
  >        nodeCache.start();
  >        System.in.read(); 
  >   }
  > 
  >  public void printNodeData() throws Exception {
  >        byte[] bytes = curatorFramework.getData().forPath("/curator-node");
  >        log.info("data: {}",new String(bytes));
  >  }
  > ```

## 9. Zookeeper集群

### 9.1 集群角色

+ zookeeper 集群中的节点有三种角色

  > + <font color=red>**Leader**</font>：处理集群的所有事务请求，<font color=red>**`集群中只有⼀个 Leader`**</font>。 
  > + <font color=red>**Follower**</font>：只能处理读请求，<font color=red>**`参与 Leader 选举`**</font>。 
  > + <font color=red>**Observer**</font>：只能处理读请求，提升集群读的性能，但 <font color=red>**`不能参与 Leader 选举`**</font>。
  >
  > ![image-20220524215727140](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524215727140.png)

### 9.2 集群搭建

+ 搭建 4 个节点，其中一个节点为 Observer

1. 创建 4 个节点的 myid，并设值

   > zkData 目录是我们存放数据文件的目录
   >
   > ![image-20220524220404817](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524220404817.png)
   >
   > 1. 新建四个目录  zk1, zk2, zk3, zk4![image-20220524220552045](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524220552045.png)
   >
   > 2. 分别在文件夹 zk1, zk2, zk3, zk4 中新建文件 myid，如下是四个 myid 文件的内容：
   >
   >    zk1/myid  -- 1
   >
   >    zk2/myid  -- 2
   >
   >    zk3/myid  -- 3
   >
   >    zk4/myid  -- 4
   >
   >    myid 文件中的数字就是这台服务器的 id，唯一标识一台 zk 服务器，不能重复

2. 编写 4 个 zoo.cfg

   > ![image-20220524222049965](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524222049965.png)
   >
   > ![image-20220524222247865](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524222247865.png)
   >
   > + 修改 zoo1,2,3,4.cfg 中的 dataDir, clientPort, server 配置
   >
   >   ```properties
   >   # 修改对应的zk1 zk2 zk3 zk4
   >   dataDir=/root/zk/apache-zookeeper-3.8.0-bin/zkData/zk1
   >         
   >   # 修改对应的端⼝ 2181 2182 2183 2184
   >   clientPort=2181
   >         
   >   # 2001为集群通信端⼝，3001为集群选举端⼝，observer表示不参与集群选举
   >   server.1=127.0.0.1:2001:3001
   >   server.2=127.0.0.1:2002:3002
   >   server.3=127.0.0.1:2003:3003
   >   server.4=127.0.0.1:2004:3004:observer
   >   ```

3. 启动 4 台 zk server

   > ```shell
   > [root@localhost bin]# ./zkServer.sh start ../conf/zoo1.cfg 
   > [root@localhost bin]# ./zkServer.sh start ../conf/zoo2.cfg 
   > [root@localhost bin]# ./zkServer.sh start ../conf/zoo3.cfg 
   > [root@localhost bin]# ./zkServer.sh start ../conf/zoo4.cfg 
   > ```
   >
   > ![image-20220524223842238](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524223842238.png)

4. 查看每台 zk 服务器的角色信息

   > ```shell
   > [root@localhost bin]# ./zkServer.sh start ../conf/zoo1.cfg 
   > ```
   >
   > ![image-20220524224317317](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524224317317.png)
   >
   > 第二台是 leader, 1, 3 是 follower, 第四台是 observer 

### 9.3 连接zk集群

+ 使用 zkCli 客户端连接集群

  > ```shell
  > [root@localhost bin]# ./zkCli.sh -server 127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183
  > ```
  >
  > 为什么要写三台服务器呢？其实连接一台也可以，那么如果连接的这一台服务器挂了，那么就真挂了。连接三台，如果其中有一台挂了，自动会选举 leader 连接，抱枕集群的高可用性！
  >
  > ![image-20220524235712452](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220524235712452.png)
  >
  > 通过查看客户端启动日志，我们发现连接到的是 2183 这台服务器

## 10. ZAB协议

### 10.1 什么是ZAB协议？

+ zk 是分布式的鼻祖！！！

  > zookeeper 作为非常重要的 <font color=red>**`分布式协调`**</font> 组件，需要进行集群部署，集群中会以 <font color=red>**`⼀主多从`**</font> 的形式 进行部署。zookeeper 为了保证数据的 <font color=red>**`⼀致性`**</font>，使用了 <font color=red>`ZAB(Zookeeper Atomic Broadcast)`</font> 协议，这个协议解决了Zookeeper的崩溃恢复和主从数据同步的问题。
  >
  > ![image-20220525085934233](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220525085934233.png)

### 10.2 四种节点状态

+ ZAB 协议定义的四种 <font color=red>**`节点`**</font> 状态

  > 1. <font color=blue>**Looking**</font> ：服务器处于寻找leader的状态；
  > 2. <font color=blue>**Following**</font> ：Follower 节点(从节点)所处的状态。 
  > 3. <font color=blue>**Leading**</font> ：Leader 节点(主节点)所处状态。 
  > 4. <font color=blue>**Observing**</font>：观察者节点所处的状态

### 10.3 集群上线时Leader选举

+ 首先我们来看看一个班级要选班长的大致过程

  > 1. 当然刚开始投票，我们会认为我们自己最强，希望自己当班长，所以先投自己一票
  > 2. 和其他人交流，认为张xx最强，所以投张xx票
  > 3. 最后得票最多的那个人就是班长。
  >
  > + 这里有个问题，你凭什么认为 张xx 能力最强，标准是什么？学习能力强？人好？运动好？

+ zookeeper 两个重要属性：<font color=red>`zxid`</font>, <font color=red>`myid`</font>

  > + 和选举班长类似，看哪个 zk 节点的票数多，那么就是 leader。同样面对一个问题，比如三个 zk 服务器，我们用什么标准来判断到底哪台 zk节点能力最强呢？
  >
  >   ![image-20220525152328854](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220525152328854.png)
  >
  > + zk1, zk2, zk3 没有及时同步数据，那么导致三台服务器中的数据有可能不一样，暂且认为 zk1 中的数据是最新的。那么 zk1 最适合当班长了。我们如何判断 zk1 中的数据是否是最新的呢？每个 zk 节点都有一个 <font color=red>**`zxid 属性(事务 id)`**</font>，比如所 zk1 接收到了 100 次请求，每次请求都修改了数据，每次修改数据 <font color=red>**`zxid 都会 +1`**</font>。而 zk2 接收了 50 次请求，zxid = 50
  >
  >   ![image-20220525153421394](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220525153421394.png)
  >
  >   总结：<font color=red>**`zxid 越大`**</font>，标识数据最新，越适合当班长
  >
  > + 如果 zk1, zk2, zk3 中的数据都是一模一样的，也就是说 zxid 都是 100，那么如何选呢？这个时候我们知道每台 zk 节点都有一个唯一标识 <font color=red>`myid`</font>，选择 <font color=red>`myid`</font> 大的作为班长
  >
  >   ![image-20220525153744018](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220525153744018.png)
  >
  >   总结：<font color=red>**zxid 一样，myid 大的作为班长**</font>
  
+ 投票箱

  > 班级选举班长时，整个班级讲台上放一个投票箱，但是 zk 不一样，<font color=red>**`每一个 zk 节点内部维护一个投票箱数据结构`**</font>

+ 集群上线时的 Leader 选举过程

  > + Zookeeper 集群中的节点在上线时，将会进入到 <font color=red>`Looking`</font> 状态，也就是选举 Leader 的状态，这个状态具体会发生什么？我们现在有三台 zk 服务器：<font color=red>`zk1(myid =1, zxid = 100)`</font>, <font color=red>`zk2(myid = 2, zxid = 100)`</font>, <font color=red>`zk3(myid = 3, zxid = 100)`</font>
  >
  > 
  >1. 首先启动 zk1: 启动之后 <font color=red>`首先投自己一票`</font>，并且在投票箱中记录以下选了自己(myid = 1, zxid = 100)。并且将投的票放在投票箱内。在 <font color=red>`zoo1.cfg`</font> 配置文件中，zk1 是知道这个集群还有其他两台 zk2, zk3 节点，就发消息给另外两台，因为 zk2, zk3, 还未启动，消息肯定发不出去。所以会等待其他节点的启动
  > 
  >   ![image-20220525173316790](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220525173316790.png)
  > 
  >2. 启动 zk2 : zk2 启动之后 <font color=red>**`首先也投自己一票`**</font>，并将投票结果放入投票箱中
  > 
  >   ![image-20220525164441463](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220525164441463.png)
  > 
  >3. zk1, zk2 都启动了，这个时候可以 <font color=red>`建立连接进行沟通了`</font>，假设：zk2 先发送自己的选票给 zk1, zk1的选票进行 <font color=red>`pk`</font>。
  > 
  >   + pk 过程如下：
  > 
  >     1. 首先比较 <font color=red>`zxid`</font>, 因为 <font color=red>`zk1.zxid = zk2.zxid`</font> 所以打了个平手
  > 
  >     2. 继续比较 <font color=red>`myid`</font>, 因为 <font color=red>`zk2.myid > zk1.myid`</font>, 所以 zk2 胜出，所以 zk1 给 zk2(myid = 2, zxid = 100)投了票
  > 
  >     3. 将投的票 <font color=red>`(myid = 2, zxid = 100)`</font> 结果放到投票箱中
  >   
  >        ![image-20220525174628345](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220525174628345.png)
  > 
  >   注意：<font color=blue>**zk1 的投票箱中的两张票虽然一样，但是是不同节点投的**</font>。
  > 
  >   zk1 改了选票(<font color=red>`将 (myid = 1, zxid = 100)改为了 (myid = 2 , zxid = 100)`</font>), 会将改票的结果发送到其他节点上去：
  > 
  >   ![image-20220525180327737](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220525180327737.png)
  > 
  >   zk2 接收到 zk1 发的选票信息，进行同样的 pk。zk2 选择了自己，然后将选票放入了投票箱。
  > 
  >3. 至此投票结束, leader 是 zk2：为什么呢？选举原则 ---> 选票数大于集群节点数的一半，停止投票。
  >    + zk2 获得的票数 2
  >   + 集群数量 3， 半数 3 / 2 = 1.5 (有一个问题：zk3 都没还有启动，为什么 zk1, zk2 知道有三台服务器？--zoo.cfg 中的配置)
  >    + 2 > 1.5 ，不再进行投票，选举 zk2 为 leader, zk1 为 follower
  >   
  > 4. 启动 zk3, 发现集群中已经有 leader(zk2)了, 那么 zk3 标识为 follower。
  
+ 集群奇数个节点

  > + Zookeeper 要想启用，则成功的节点必须＞总结点 / 2。
  > + 假设总共 <font color=red>`5`</font> 个节点，则可以 <font color=red>`宕机 2台`</font>，因为<font color=red>`(5 - 2)> 5 / 2`</font>，可以正常选举 leader
  > + 如果总共有 <font color=red>`6`</font> 个节点，也只能 <font color=red>`宕机 2台`</font>，如果 <font color=red>`宕机 3 台`</font> 会导致 <font color=red>`(6 - 3) > 6 / 2` </font>无法成立，集群失效无法选举 leader。
  > + 所以占用服务器空间利用成本角度考虑，建议 zk 集群节点一定是为 <font color=red>`奇数`</font>。

+ 集群最少 3 个节点

  > + 3 个节点的 cluster 可以挂掉一个节点(leader 可以得到 2 票 > 1.5)
  > + 2 个节点的 cluster 就不能挂掉任何一个节点了(leader 可以得到 1 票 <= 1)

### 10.4 崩溃恢复时的Leader选举

+ Leader 宕机了选举

  > Leader 建立完后，Leader <font color=red>**`周期性`**</font> 地不断向 Follower发送 <font color=red>`⼼跳`</font>(<font color=red>`ping命令，没有内容的 socket`</font>)。当 Leader 崩溃后，Follower 发现 socket 通道已 <font color=red>**`关闭`**</font>，于是 Follower 开始进入到 <font color=red>`Looking`</font> 状态，重新回到上⼀节中的Leader选举过程，此时 <font color=red>`集群不能对外提供服务`</font>。
  >

### 10.5 主从之间的数据同步

![image-20220525220645080](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220525220645080.png)

+ 说明：

  > - Client1 有可能连接到的是 follower, 而不一定是 leader, 这个时候 follower 接收到写数据的请求时，会将写请求转给 leaeder
  > - ack : 写数据操作成功后返回通知
  > - 第 3 步中，leader 发送数据给所有的 follower,这个过程也称为广播
  > - 第 6 步中，leader 首先自己 commit, 将 数据写入内存，然后再发送 commit 请求给所有的 follower.

+ Zookeeper 中的 NIO 与 BIO 的应⽤

  > + NIO 
  >   1. ⽤于被客户端连接的2181端⼝，使⽤的是NIO模式与客户端建⽴连接 
  >   2. 客户端开启Watch时，也使⽤NIO，等待Zookeeper服务器的回调
  > + BIO
  >   1. 集群在选举时，多个节点之间的投票通信端⼝，使⽤ BIO 进⾏通信。

## 11. CAP理论

### 11.1 CAP 定理

+ 什么是 CAP

  > + 2000 年 7 ⽉，加州大学伯克利分校的 Eric Brewer 教授在 ACM PODC 会议上提出 CAP 猜 想。2 年后，麻省理⼯学院的 Seth Gilbert 和 Nancy Lynch 从理论上证明了 CAP。之后， CAP 理论正式成为 <font color=red>`分布式计算领域`</font> 的公认定理。
  >
  > + CAP 理论为：⼀个分布式系统最多只能同时满足 <font color=red>`⼀致性(Consistency)`</font>、<font color=red>`可⽤性 (Availability)`</font>和 <font color=red>`分区容错性(Partition tolerance)`</font>这三项中的两项。
  
+ ⼀致性(Consistency)

  > ⼀致性指  <font color='blue'>`“all nodes see the same data at the same time”`</font>，即更新操作成功并返回客户端 完成后，所有节点在同⼀时间的数据完全 <font color=red>**`⼀致`**</font>。
  >
  
+ 可⽤性(Availability)

  > 可⽤性指 <font color='blue'>`“Reads and writes always succeed”`</font>，即服务⼀直可用，而且是正常响应时间。

+ 分区容错性(Partition tolerance)

  > 分区容错性指 <font color='blue'>`“the system continues to operate despite arbitrary message loss or failure of part of the system”`</font>，即分布式系统在遇到某节点或网络分区故障的时候，仍然能够对外提供满足⼀致性或可用的服务。——避免单点故障，就要进行冗余部署，冗余部署相当于是服务的分区，这样的分区就具备了容错性。
  >

### 11.2 CAP权衡

+ 通过 CAP 理论，我们知道无法同时满足 <font color=red>`一致性`</font>、<font color=red>`可用性`</font> 和 <font color=red>**`分区容错性`**</font> 这三个特性，那要舍弃 哪个呢？ 

  > + 对于多数大型互联网应用的场景，主机众多、部署分散，而且现在的集群规模越来越大，所以节点故障、网络故障是常态，而且要保证服务可用性达到 N 个 9，即保证 P 和 A，舍弃 C(退而求其次保证最终⼀致性)。虽然某些地方会影响客户体验，但没达到造成用户流程的严重程度。 
  >
  > + 对于涉及到钱财这样不能有⼀丝让步的场景，C 必须保证。网络发生故障宁可停止服务，这是保证 CA，舍弃 P。貌似这几年国内银行业发生了不下 10 起事故，但影响面不大，报到也不 多，广大群众知道的少。
  >
  > + 还有⼀种是保证 CP，舍弃 A。例如网络故障是只读不写。 孰优孰略，没有定论，只能根据场景定夺，适合的才是最好的。
  >
  >   ![image-20220526224954627](https://github.com/kuanrongdelvdou/StudyNote/blob/main/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper/image/image-20220526224954627.png)

### 11.3 BASE 理论

+ base 理论的提出

  > eBay 的架构师 Dan Pritchett 源于对⼤规模分布式系统的实践总结，在 ACM 上发表文章提出 BASE 理论，BASE 理论是对 CAP 理论的延伸，核心思想是即使无法做到强⼀致性(Strong Consistency，CAP 的⼀致性就是强⼀致性)，但应⽤可以采用适合的放式达到 <font color=red>**`最终⼀致性 (Eventual Consitency)`**</font>。
  >
  
+ 基本可用(Basically Available)

  > 基本可⽤是指分布式系统在出现故障的时候，允许损失部分可用性，即保证核心可⽤。 电商大促时，为了应对访问量激增，部分用户可能会被引导到降级页面，服务层也可能只提供降级服务。这就是损失部分可用性的体现。
  >
  
+ 软状态(Soft State)

  > 软状态是指允许系统存在中间状态，而该中间状态不会影响系统整体可⽤性。分布式存储中 ⼀般⼀份数至少会有 <font color=red>`三个副本`</font>，允许不同节点间副本同步的延时就是软状态的体现。mysql replication 的异步复制也是⼀种体现。
  >
  
+ 最终⼀致性(Eventual Consistency)

  > 最终⼀致性是指系统中的所有数据副本经过⼀定时间后，最终能够达到⼀致的状态。<font color=red>`弱⼀致性`</font> 和 <font color=red>`强⼀致性`</font> 相反，最终⼀致性是弱⼀致性的⼀种特殊情况。
  >

### 11.4 Zookeeper追求的一致性

Zookeeper 在数据同步时，追求的并不是强⼀致性，而是顺序一致性(事务id的单调递增)。
