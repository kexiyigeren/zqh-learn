# HBase

### 简介

HBase是一种分布式、可扩展、支持海量数据存储的NoSQL数据库.

**数据模型**

逻辑上，HBase 的数据模型同关系型数据库很类似，数据存储在一张表中，有行有列。但从 HBase 的底层物理存储结构（K-V）来看，HBase 更像是一个 multi-dimensional map。

**逻辑结构**

![image-20210610091813418](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210610091813418.png)

**物理存储结构**

![](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210610092030346.png)

**数据模型**

- **Name Space**

  命名空间，类似于关系型数据库的database概念，每个命名空间下有多个表。HBase两个自带的命名空间，分别是hbase和default，hbase中存放的是HBase内置的表，default表是用户默认使用的命名空间。

- **Table**

  类似于关系型数据库的表概念。不同的是，HBase定义表时只需要声明列族即可，不需要声明具体的列。这意味着，往HBase写入数据时，字段可以动态、按需指定。因此，和关系型数据库相比，HBase能够轻松应对字段变更的场景。

- **Row**

  HBase表中的每行数据都由一个**RowKey**和多个**Column**（列）组成，数据是按照RowKey的字典顺序存储的，并且查询数据时只能根据RowKey进行检索，所以RowKey的设计十分重要。

- **Column**

  HBase中的每个列都由Column Family(列族)和Column Qualifier（列限定符）进行限定，例如info：name，info：age。建表时，只需指明列族，而列限定符无需预先定义。

- **Time** **Stamp**

  用于标识数据的不同版本（version），每条数据写入时，系统会自动为其加上该字段，其值为写入HBase的时间。

- **Cell**

  由{rowkey, column Family：column Qualifier, time Stamp} 唯一确定的单元。cell中的数据全部是字节码形式存贮。

**HBase基本架构**

![image-20210610093253038](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210610093253038.png)

**架构角色：**

- **Region** **Server**

  Region Server为 Region的管理者，其实现类为HRegionServer，主要作用如下:

  对于数据的操作：get, put, delete；

  对于Region的操作：splitRegion、compactRegion。

- **Master**

  Master是所有Region Server的管理者，其实现类为HMaster，主要作用如下：

  对于表的操作：create, delete, alter

  对于RegionServer的操作：分配regions到每个RegionServer，监控每个RegionServer的状态，负载均衡和故障转移。

- **Zookeeper**

  HBase通过Zookeeper来做master的高可用、RegionServer的监控、元数据的入口以及集群配置的维护等工作。

- **HDFS**

  HDFS为Hbase提供最终的底层数据存储服务，同时为HBase提供高可用的支持.

###  入门

#### 安装部署

- 1 首先确保Zookeeper集群的正常部署，请使用群起脚本或在各节点分别启动。

  ```shell
  bin/zkServer.sh start
  ```

- 2 Hadoop集群正常部署并启动。

  ```shell
  sbin/start-dfs.sh
  sbin/start-yarn.sh
  ```

- HBase的解压

  - 解压Hbase到指定目录：
  - 配置环境变量

- HBase的配置文件

  - hbase-env.sh修改内容

    ```sh
    export HBASE_MANAGES_ZK=false
    ```

  - hbase-site.xml修改内容

    ```xml
    <configuration>
        <property>
            <name>hbase.rootdir</name>
            <value>hdfs://hadoop102:8020/hbase</value>
        </property>
    
        <property>
            <name>hbase.cluster.distributed</name>
            <value>true</value>
        </property>
    
        <property>
            <name>hbase.zookeeper.quorum</name>
            <value>hadoop102,hadoop103,hadoop104</value>
        </property>
    
        <property>
            <name>hbase.unsafe.stream.capability.enforce</name>
            <value>false</value>
        </property>
        
        <property>
            <name>hbase.wal.provider</name>
            <value>filesystem</value>
        </property>
    </configuration>
    ```

  - 修改regionservers

    ```
    hadoop102
    hadoop103
    hadoop104
    ```

- HBase远程发送到其他集群

- HBase服务的启动

  - 单点启动

    ```sh
    bin/hbase-daemon.sh start master
    bin/hbase-daemon.sh start regionserver
    ```

    提示：如果集群之间的节点时间不同步，会导致regionserver无法启动，抛出ClockOutOfSyncException异常。

    修复提示：

    a、同步时间服务

    请参看帮助文档：《尚硅谷大数据技术之Hadoop入门》

    b、属性：hbase.master.maxclockskew设置更大的值

    ```xml
    <property>
        <name>hbase.master.maxclockskew</name>
        <value>180000</value>  
        <description>Time difference of regionserver from master</description>
    </property>
    ```

  - 群启

    ```sh
    # 启动服务
    bin/start-hbase.sh 
    # 停止服务
    bin/stop-hbase.sh
    ```

- 查看HBase页面

  启动成功后，可以通过“host:port”的方式来访问HBase管理页面，例如：

  [http://hadoop102:16010](http://linux01:16010) 

#### 高可用(可选)

在HBase中HMaster负责监控HRegionServer的生命周期，均衡RegionServer的负载，如果HMaster挂掉了，那么整个HBase集群将陷入不健康的状态，并且此时的工作状态并不会维持太久。所以HBase支持对HMaster的高可用配置。

- 关闭HBase集群（如果没有开启则跳过此步）

  ```sh
  bin/stop-hbase.sh
  ```

- 在conf目录下创建backup-masters文件

  ```sh
  touch conf/backup-masters
  ```

- 在backup-masters文件中配置高可用HMaster节点

  ```sh
  echo hadoop103 > conf/backup-masters
  ```

- 将整个conf目录scp到其他节点

  ```sh
  scp -r conf/ hadoop103:/opt/module/hbase/
  scp -r conf/ hadoop104:/opt/module/hbase/
  ```

- **打开页面测试查看** [http://hadoop102:16010](http://hadoop102:16010) 

#### HBase Shell操作

...... 待补充

### HBase进阶

#### RegionServer 架构

![image-20210610102843088](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210610102843088.png)

- **StoreFile**

  保存实际数据的物理文件，StoreFile以Hfile的形式存储在HDFS上。每个Store会有一个或多个StoreFile（HFile），数据在每个StoreFile中都是有序的。

- **MemStore**

  写缓存，由于HFile中的数据要求是有序的，所以数据是先存储在MemStore中，排好序后，等到达刷写时机才会刷写到HFile，每次刷写都会形成一个新的HFile。

- **WAL**

  由于数据要经MemStore排序后才能刷写到HFile，但把数据保存在内存中会有很高的概率导致数据丢失，为了解决这个问题，数据会先写在一个叫做Write-Ahead logfile的文件中，然后再写入MemStore中。所以在系统出现故障的时候，数据可以通过这个日志文件重建。

- **BlockCache**

  读缓存，每次查询出的数据会缓存在BlockCache中，方便下次查询。

#### 写流程

![image-20210610105020064](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210610105020064.png)

**流程**

- 1）Client先访问zookeeper，获取hbase:meta表位于哪个Region Server。

- 2）访问对应的Region Server，获取hbase:meta表，根据读请求的namespace:table/rowkey，查询出目标数据位于哪个Region Server中的哪个Region中。并将该table的region信息以及meta表的位置信息缓存在客户端的meta cache，方便下次访问。

- 3）与目标Region Server进行通讯；

- 4）将数据顺序写入（追加）到WAL；

- 5）将数据写入对应的MemStore，数据会在MemStore进行排序；

- 6）向客户端发送ack；

- 7）等达到MemStore的刷写时机后，将数据刷写到HFile。

#### MemStore Flush

![image-20210610110258703](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210610110258703.png)

**MemStore刷写时机：**

触发MemStore刷写的机制大概分为：人为手动触发、HBase定时触发、HLog数量限制触发，其他事件触发（Compact、Split、Truncate等）、内存限制触发。其中内存限制触发细分为：MemStore级别限制触发、Region级别限制触发、RegionServer级别限制触发。

1. **Region 中所有 MemStore 占用的内存超过相关阈值**

   当某个memstore的大小达到了**hbase.hregion.memstore.flush.size（默认值128M）**，其所在region的所有memstore都会刷写。

   但是如果我们的数据增加得很快，达到了 **hbase.hregion.memstore.flush.size * hbase.hregion.memstore.block.multiplier** 的大小，**hbase.hregion.memstore.block.multiplier 默认值为4**，也就是128*4=512MB的时候，那么除了触发 MemStore 刷写之外，HBase 还会在刷写的时候同时阻塞所有写入该 Store 的写请求！这时候如果你往对应的 Store 写数据，会出现 RegionTooBusyException 异常。

2. **整个 RegionServer 的 MemStore 占用内存总和大于相关阈值**

   当region server中memstore的总大小达到

   <font color=red>**java_heapsize  * hbase.regionserver.global.memstore.size（默认值0.4）* hbase.regionserver.global.memstore.size.lower.limit(默认值0.95)**</font>,region会按照其所有memstore的大小顺序（由大到小）依次进行刷写。直到region server中所有memstore的总大小减小到上述值以下。

   如果此时数据写入吞吐量依然很大，导致该RegionServer种所有MemStore的大小加和超过该RegionServer全局水位阈值<font color=red>java_heapsize * hbase.regionserver.global.memstore.size </font>值大小，RegionServer会阻塞写请求，直到MemStore刷写大小将到低水位阈值。

3. **定期自动刷写**

   到达自动刷写的时间，也会触发memstore flush。自动刷新的时间间隔由该属性进行配置<font color=red>**hbase.regionserver.optionalcacheflushinterval**（默认1小时）</font>。

4. **WAL数量大于相关阈值**

   WAL（Write-ahead log，预写日志）用来解决宕机之后的操作恢复问题的。数据到达 Region 的时候是先写入 WAL，然后再被写到 Memstore 的。如果 WAL 的数量越来越大，这就意味着 MemStore 中未持久化到磁盘的数据越来越多。当 RS 挂掉的时候，恢复时间将会变成，所以有必要在 WAL 到达一定的数量时进行一次刷写操作。

   当WAL文件的数量超过<font color=red>**hbase.regionserver.maxlogs**</font>，region会按照时间顺序依次进行刷写，直到WAL文件数量减小到该参数值以下(<font color=red>该属性名已经废弃，现无需手动设置，最大值为32</font>.

5. **人为手动触发**

   通过 shell 命令`flush 'tablename'`或者`flush ‘regionname’`分别对整表所有region和具体一个Region进行flush。

6. **其他事件触发**

   在执行Region的合并、分裂、快照以及HFile的Compact等前会执行刷写

#### 读流程

- **整体流程**

  ![image-20210610120054869](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210610120054869.png)

- **Merge细节**

  ![](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210610120148944.png)

**读流程:**

1. Client先访问zookeeper，获取hbase:meta表位于哪个Region Server。
2. 访问对应的Region Server，获取hbase:meta表，根据读请求的namespace:table/rowkey，查询出目标数据位于哪个Region Server中的哪个Region中。并将该table的region信息以及meta表的位置信息缓存在客户端的meta cache，方便下次访问。
3. 与目标Region Server进行通讯.
4. 分别在MemStore和Store File（HFile）中查询目标数据，并将查到的所有数据进行合并。此处所有数据是指同一条数据的不同版本（time stamp）或者不同的类型（Put/Delete）。
5. 将查询到的新的数据块（Block，HFile数据存储单元，默认大小为64KB）缓存到Block Cache。
6. 将合并后的最终结果返回给客户端。

#### StoreFile Compaction

由于memstore每次刷写都会生成一个新的HFile，且同一个字段的不同版本（timestamp）和不同类型（Put/Delete）有可能会分布在不同的HFile中，因此查询时需要遍历所有的HFile。为了减少HFile的个数，以及清理掉过期和删除的数据，会进行StoreFile Compaction。

Compaction分为两种，分别是**Minor Compaction**和**Major Compaction**。Minor Compaction会将临近的若干个较小的HFile合并成一个较大的HFile，并不清理过期和删除的数据。Major Compaction会将一个Store下的所有的HFile合并成一个大HFile，并且**会**清理掉所有过期和删除的数据。

![image-20210610151009698](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210610151009698.png)

#### Region Split

默认情况下，每个Table起初只有一个Region，随着数据的不断写入，Region会自动进行拆分。刚拆分时，两个子Region都位于当前的Region Server，但处于负载均衡的考虑，HMaster有可能会将某个Region转移给其他的Region Server。

**Region Split时机：**

当1个region中的某个Store下所有StoreFile的总大小超过

<font color=red>**Min(initialSize * R^3 ,hbase.hregion.max.filesize)**</font>，该Region就会进行拆分。其中initialSize的默认值为<font color=red>**2*hbase.hregion.memstore.flush.size**</font>，R为当前Region Server中属于该Table的Region个数（0.94版本之后）。

具体的切分策略为：

第一次split：1^3 * 256 = 256MB 

第二次split：2^3 * 256 = 2048MB 

第三次split：3^3 * 256 = 6912MB 

第四次split：4^3 * 256 = 16384MB > 10GB，因此取较小的值10GB 

后面每次split的size都是10GB了。

**Hbase 2.0引入了新的split策略**：如果当前RegionServer上该表只有一个Region，按照2 * hbase.hregion.memstore.flush.size分裂，否则按照hbase.hregion.max.filesize分裂。

![image-20210610152309076](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210610152309076.png)

### HBase API

新建项目后在pom.xml中添加依赖：

```xml
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-server</artifactId>
    <version>2.0.5</version>
</dependency>

<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-client</artifactId>
    <version>2.0.5</version>
</dependency>
```

#### DDL

##### **判断表是否存在**

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.NamespaceDescriptor;
import org.apache.hadoop.hbase.NamespaceExistException;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;

public class HBase_DDL {

    //TODO 判断表是否存在
    public static boolean isTableExist(String tableName) throws IOException {

        //1.创建配置信息并配置
        Configuration configuration = HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.quorum", "hadoop102,hadoop103,hadoop104");

        //2.获取与HBase的连接
        Connection connection = ConnectionFactory.createConnection(configuration);

        //3.获取DDL操作对象
        Admin admin = connection.getAdmin();

        //4.判断表是否存在操作
        boolean exists = admin.tableExists(TableName.valueOf(tableName));

        //5.关闭连接
        admin.close();
        connection.close();

        //6.返回结果
        return exists;
    }

}
```

##### 创建表

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.NamespaceDescriptor;
import org.apache.hadoop.hbase.NamespaceExistException;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;

public class HBase_DDL {

    //TODO 创建表
    public static void createTable(String tableName, String... cfs) throws IOException {

        //1.判断是否存在列族信息
        if (cfs.length <= 0) {
            System.out.println("请设置列族信息！");
            return;
        }

        //2.判断表是否存在
        if (isTableExist(tableName)) {
            System.out.println("需要创建的表已存在！");
            return;
        }

        //3.创建配置信息并配置
        Configuration configuration = HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.quorum", "hadoop102,hadoop103,hadoop104");

        //4.获取与HBase的连接
        Connection connection = ConnectionFactory.createConnection(configuration);

        //5.获取DDL操作对象
        Admin admin = connection.getAdmin();

        //6.创建表描述器构造器
        TableDescriptorBuilder tableDescriptorBuilder = TableDescriptorBuilder.newBuilder(TableName.valueOf(tableName));

        //7.循环添加列族信息
        for (String cf : cfs) {
            ColumnFamilyDescriptorBuilder columnFamilyDescriptorBuilder = ColumnFamilyDescriptorBuilder.newBuilder(Bytes.toBytes(cf));
            tableDescriptorBuilder.setColumnFamily(columnFamilyDescriptorBuilder.build());
        }

        //8.执行创建表的操作
        admin.createTable(tableDescriptorBuilder.build());

        //9.关闭资源
        admin.close();
        connection.close();
    }
}
```

##### 删除表

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.NamespaceDescriptor;
import org.apache.hadoop.hbase.NamespaceExistException;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;

public class HBase_DDL {

    //TODO 删除表
    public static void dropTable(String tableName) throws IOException {

        //1.判断表是否存在
        if (!isTableExist(tableName)) {
            System.out.println("需要删除的表不存在！");
            return;
        }

        //2.创建配置信息并配置
        Configuration configuration = HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.quorum", "hadoop102,hadoop103,hadoop104");

        //3.获取与HBase的连接
        Connection connection = ConnectionFactory.createConnection(configuration);

        //4.获取DDL操作对象
        Admin admin = connection.getAdmin();

        //5.使表下线
        TableName name = TableName.valueOf(tableName);
        admin.disableTable(name);

        //6.执行删除表操作
        admin.deleteTable(name);

        //7.关闭资源
        admin.close();
        connection.close();
    }

}
```

##### **创建命名空间**

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.NamespaceDescriptor;
import org.apache.hadoop.hbase.NamespaceExistException;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;

public class HBase_DDL {

    //TODO 创建命名空间
    public static void createNameSpace(String ns) throws IOException {

        //1.创建配置信息并配置
        Configuration configuration = HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.quorum", "hadoop102,hadoop103,hadoop104");

        //2.获取与HBase的连接
        Connection connection = ConnectionFactory.createConnection(configuration);

        //3.获取DDL操作对象
        Admin admin = connection.getAdmin();

        //4.创建命名空间描述器
        NamespaceDescriptor namespaceDescriptor = NamespaceDescriptor.create(ns).build();

        //5.执行创建命名空间操作
        try {
            admin.createNamespace(namespaceDescriptor);
        } catch (NamespaceExistException e) {
            System.out.println("命名空间已存在！");
        } catch (Exception e) {
            e.printStackTrace();
        }

        //6.关闭连接
        admin.close();
        connection.close();

    }
}
```

#### DML

##### 插入数据

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;

public class HBase_DML {

    //TODO 插入数据
    public static void putData(String tableName, String rowKey, String cf, String cn, String value) throws IOException {

        //1.获取配置信息并设置连接参数
        Configuration configuration = HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.quorum", "hadoop102,hadoop103,hadoop104");

        //2.获取连接
        Connection connection = ConnectionFactory.createConnection(configuration);

        //3.获取表的连接
        Table table = connection.getTable(TableName.valueOf(tableName));

        //4.创建Put对象
        Put put = new Put(Bytes.toBytes(rowKey));

        //5.放入数据
        put.addColumn(Bytes.toBytes(cf), Bytes.toBytes(cn), Bytes.toBytes(value));

        //6.执行插入数据操作
        table.put(put);

        //7.关闭连接
        table.close();
        connection.close();
    }

}
```

##### 单条数据查询

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;

public class HBase_DML {

    //TODO 单条数据查询(GET)
    public static void getDate(String tableName, String rowKey, String cf, String cn) throws IOException {

        //1.获取配置信息并设置连接参数
        Configuration configuration = HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.quorum", "hadoop102,hadoop103,hadoop104");

        //2.获取连接
        Connection connection = ConnectionFactory.createConnection(configuration);

        //3.获取表的连接
        Table table = connection.getTable(TableName.valueOf(tableName));

        //4.创建Get对象
        Get get = new Get(Bytes.toBytes(rowKey));
        // 指定列族查询
        // get.addFamily(Bytes.toBytes(cf));
        // 指定列族:列查询
        // get.addColumn(Bytes.toBytes(cf), Bytes.toBytes(cn));

        //5.查询数据
        Result result = table.get(get);

        //6.解析result
        for (Cell cell : result.rawCells()) {
            System.out.println("CF:" + Bytes.toString(cell.getFamilyArray()) +
                    ",CN:" + Bytes.toString(cell.getQualifierArray()) +
                    ",Value:" + Bytes.toString(cell.getValueArray()));
        }

        //7.关闭连接
        table.close();
        connection.close();

    }

}
```

##### **扫描数据**

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;

public class HBase_DML {

    //TODO 扫描数据(Scan)
    public static void scanTable(String tableName) throws IOException {

        //1.获取配置信息并设置连接参数
        Configuration configuration = HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.quorum", "hadoop102,hadoop103,hadoop104");

        //2.获取连接
        Connection connection = ConnectionFactory.createConnection(configuration);

        //3.获取表的连接
        Table table = connection.getTable(TableName.valueOf(tableName));

        //4.创建Scan对象
        Scan scan = new Scan();

        //5.扫描数据
        ResultScanner results = table.getScanner(scan);

        //6.解析results
        for (Result result : results) {
            for (Cell cell : result.rawCells()) {
                System.out.println("CF:" + Bytes.toString(cell.getFamilyArray()) +
                        ",CN:" + Bytes.toString(cell.getQualifierArray()) +
                        ",Value:" + Bytes.toString(cell.getValueArray()));
            }
        }

        //7.关闭资源
        table.close();
        connection.close();

    }

}
```

##### **删除数据**

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;

public class HBase_DML {

    //TODO 删除数据
    public static void deletaData(String tableName, String rowKey, String cf, String cn) throws IOException {

        //1.获取配置信息并设置连接参数
        Configuration configuration = HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.quorum", "hadoop102,hadoop103,hadoop104");

        //2.获取连接
        Connection connection = ConnectionFactory.createConnection(configuration);

        //3.获取表的连接
        Table table = connection.getTable(TableName.valueOf(tableName));

        //4.创建Delete对象
        Delete delete = new Delete(Bytes.toBytes(rowKey));

        // 指定列族删除数据
        // delete.addFamily(Bytes.toBytes(cf));
        // 指定列族:列删除数据(所有版本)
        // delete.addColumn(Bytes.toBytes(cf), Bytes.toBytes(cn));
        // 指定列族:列删除数据(指定版本)
        // delete.addColumns(Bytes.toBytes(cf), Bytes.toBytes(cn));

        //5.执行删除数据操作
        table.delete(delete);

        //6.关闭资源
        table.close();
        connection.close();

}

}
```

### HBase优化

#### 预分区



#### RowKey设计

一条数据的唯一标识就是rowkey，那么这条数据存储于哪个分区，取决于rowkey处于哪个一个预分区的区间内，设计rowkey的主要目的 ，就是让数据均匀的分布于所有的region中，在一定程度上防止数据倾斜。接下来我们就谈一谈rowkey常用的设计方案。

1. **生成随机数、hash、散列值**
2. **字符串反转**
3. **字符串拼接**

#### 内存优化

HBase操作过程中需要大量的内存开销，毕竟Table是可以缓存在内存中的，但是不建议分配非常大的堆内存，因为GC过程持续太久会导致RegionServer处于长期不可用状态，一般16~36G内存就可以了，如果因为框架占用内存过高导致系统内存不足，框架一样会被系统服务拖死。

#### 基础优化

##### **Zookeeper会话超时时间**

hbase-site.xml

属性：zookeeper.session.timeout

解释：默认值为90000毫秒（90s）。当某个RegionServer挂掉，90s之后Master才能察觉到。可适当减小此值，以加快Master响应，可调整至60000毫秒。

##### **设置RPC监听数量**

hbase-site.xml

属性：hbase.regionserver.handler.count

解释：默认值为30，用于指定RPC监听的数量，可以根据客户端的请求数进行调整，读写请求较多时，增加此值。

##### **手动控制Major** **Compaction**

hbase-site.xml

属性：hbase.hregion.majorcompaction

解释：默认值：604800000秒（7天）， Major Compaction的周期，若关闭自动Major Compaction，可将其设为0

##### **优化HStore文件大小**

hbase-site.xml

属性：hbase.hregion.max.filesize

解释：默认值10737418240（10GB），如果需要运行HBase的MR任务，可以减小此值，因为一个region对应一个map任务，如果单个region过大，会导致map任务执行时间过长。该值的意思就是，如果HFile的大小达到这个数值，则这个region会被切分为两个Hfile。

##### **优化HBase客户端缓存**

hbase-site.xml

属性：hbase.client.write.buffer

解释：默认值2097152bytes（2M）用于指定HBase客户端缓存，增大该值可以减少RPC调用次数，但是会消耗更多内存，反之则反之。一般我们需要设定一定的缓存大小，以达到减少RPC次数的目的。

##### 指定scan.next扫描HBase所获取的行数

hbase-site.xml

属性：hbase.client.scanner.caching

解释：用于指定scan.next方法获取的默认行数，值越大，消耗内存越大。

##### BlockCache占用RegionServer堆内存的比例

hbase-site.xml

属性：hfile.block.cache.size

解释：默认0.4，读请求比较多的情况下，可适当调大

##### MemStore占用RegionServer堆内存的比例

hbase-site.xml

属性：hbase.regionserver.global.memstore.size

解释：默认0.4，写请求较多的情况下，可适当调大

### 整合Phoenix

#### Phoenix简介

##### Phoenix定义

Phoenix是HBase的开源SQL皮肤。可以使用标准JDBC API代替HBase客户端API来创建表，插入数据和查询HBase数据。

##### Phoenix特点

1）容易集成：如Spark，Hive，Pig，Flume和Map Reduce；

2）操作简单：DML命令以及通过DDL命令创建和操作表和版本化增量更改；

3）支持HBase二级索引创建。

#### Phoenix快速入门

##### Phoenix安装部署

##### Phoenix Shell操作

##### Phoenix JDBC操作

1. **启动query erver**

   ```sh
   $ queryserver.py start
   ```

2. **创建项目并导入依赖**

   ```xml
   <dependencies>
       <!-- https://mvnrepository.com/artifact/org.apache.phoenix/phoenix-queryserver-client -->
       <dependency>
           <groupId>org.apache.phoenix</groupId>
           <artifactId>phoenix-queryserver-client</artifactId>
           <version>5.0.0-HBase-2.0</version>
       </dependency>
   </dependencies>
   ```

3. **编写代码**

```java
package com.atguigu;

import java.sql.*;
import org.apache.phoenix.queryserver.client.ThinClientUtil;

public class PhoenixTest {
public static void main(String[] args) throws SQLException {

        String connectionUrl = ThinClientUtil.getConnectionUrl("hadoop102", 8765);
        System.out.println(connectionUrl);
        Connection connection = DriverManager.getConnection(connectionUrl);
        PreparedStatement preparedStatement = connection.prepareStatement("select * from student");

        ResultSet resultSet = preparedStatement.executeQuery();

        while (resultSet.next()) {
            System.out.println(resultSet.getString(1) + "\t" + resultSet.getString(2));
        }

        //关闭
        connection.close();

}
}
```

#### Phoenix二级索引

##### 二级索引配置文件

添加如下配置到HBase的HRegionserver节点的hbase-site.xml

```xml
<!-- phoenix regionserver 配置参数-->
    <property>
        <name>hbase.regionserver.wal.codec</name>
        <value>org.apache.hadoop.hbase.regionserver.wal.IndexedWALEditCodec</value>
    </property>

    <property>
        <name>hbase.region.server.rpc.scheduler.factory.class</name>
        <value>org.apache.hadoop.hbase.ipc.PhoenixRpcSchedulerFactory</value>
        <description>Factory to create the Phoenix RPC Scheduler that uses separate queues for index and metadata updates</description>
    </property>

    <property>
        <name>hbase.rpc.controllerfactory.class</name>
        <value>org.apache.hadoop.hbase.ipc.controller.ServerRpcControllerFactory</value>
        <description>Factory to create the Phoenix RPC Scheduler that uses separate queues for index and metadata updates</description>
    </property>
```

##### 全局二级索引

Global Index是默认的索引格式，创建全局索引时，会在HBase中建立一张新表。也就是说索引数据和数据表是存放在不同的表中的，因此全局索引适用于<font color=red>**多读少写**</font>的业务场景。

写数据的时候会消耗大量开销，因为索引表也要更新，而索引表是分布在不同的数据节点上的，跨节点的数据传输带来了较大的性能消耗。

在读数据的时候Phoenix会选择索引表来降低查询消耗的时间。

1. **创建单个字段的全局索引**

   ```sql
   CREATE INDEX my_index ON my_table (my_col);
   ```

   ![image-20210610164524022](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210610164524022.png)

   ![image-20210610164546072](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210610164546072.png)

2. **创建携带其他字段的全局索引**

   ```sql
   CREATE INDEX my_index ON my_table (v1) INCLUDE (v2);
   ```

   ![image-20210610164815070](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210610164815070.png)

##### 本地二级索引

Local Index适用于<font color = red>**写操作频繁**</font>的场景。

索引数据和数据表的数据是存放在同一张表中（且是同一个Region），避免了在写操作的时候往不同服务器的索引表中写索引带来的额外开销。

```sql
CREATE LOCAL INDEX my_index ON my_table (my_column);
```

![image-20210610165038346](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210610165038346.png)

### 与Hive的集成

#### HBase与Hive的对比

##### Hive

**(1) 数据仓库**

Hive的本质其实就相当于将HDFS中已经存储的文件在Mysql中做了一个双射关系，以方便使用HQL去管理查询。

(**2) 用于数据分析、清洗**

Hive适用于离线的数据分析和清洗，延迟较高。

**(3) 基于HDFS、MapReduce**

Hive存储的数据依旧在DataNode上，编写的HQL语句终将是转换为MapReduce代码执行。

##### HBase

**(1) 数据库**

是一种面向列族存储的非关系型数据库。

**(2) 用于存储结构化和非结构化的数据**

适用于单表非关系型数据的存储，不适合做关联查询，类似JOIN等操作。

**(3) 基于HDFS**

数据持久化存储的体现形式是HFile，存放于DataNode中，被ResionServer以region的形式进行管理。

**(4) 延迟较低，接入在线业务使用**

面对大量的企业数据，HBase可以直线单表大量数据的存储，同时提供了高效的数据访问速度。

####  HBase与Hive集成使用

在hive-site.xml中添加zookeeper的属性，如下：

```xml
    <property>
        <name>hive.zookeeper.quorum</name>
        <value>hadoop102,hadoop103,hadoop104</value>
    </property>

    <property>
        <name>hive.zookeeper.client.port</name>
        <value>2181</value>
    </property>
```

##### 案例一

建立Hive表，关联HBase表，插入数据到Hive表的同时能够影响HBase表。

分步实现：

1. 在Hive中创建表同时关联HBase

   ```sql
   CREATE TABLE hive_hbase_emp_table(
   empno int,
   ename string,
   job string,
   mgr int,
   hiredate string,
   sal double,
   comm double,
   deptno int)
   STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
   WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,info:ename,info:job,info:mgr,info:hiredate,info:sal,info:comm,info:deptno")
   TBLPROPERTIES ("hbase.table.name" = "hbase_emp_table");
   ```

   **提示**：完成之后，可以分别进入Hive和HBase查看，都生成了对应的表

2. 在Hive中创建临时中间表，用于load文件中的数据

   **提示**：不能将数据直接load进Hive所关联HBase的那张表中.

   ```sql
   CREATE TABLE emp(
   empno int,
   ename string,
   job string,
   mgr int,
   hiredate string,
   sal double,
   comm double,
   deptno int)
   row format delimited fields terminated by '\t';
   ```

3. 向Hive中间表中load数据

   ```sql
   hive> load data local inpath '/home/admin/softwares/data/emp.txt' into table emp;
   ```

4. 通过insert命令将中间表中的数据导入到Hive关联Hbase的那张表中

   ```sql
   hive> insert into table hive_hbase_emp_table select * from emp;
   ```

5. 查看Hive以及关联的HBase表中是否已经成功的同步插入了数据

   Hive：

   ```sql
   hive> select * from hive_hbase_emp_table;
   ```

   HBase：

   ```sh
   Hbase> scan 'hbase_emp_table'
   ```

##### 案例二

在HBase中已经存储了某一张表hbase_emp_table，然后在Hive中创建一个外部表来关联HBase中的hbase_emp_table这张表，使之可以借助Hive来分析HBase这张表中的数据。

**注：**该案例2紧跟案例1的脚步，所以完成此案例前，请先完成案例1。

分步实现：

1. 在Hive中创建外部表

   ```sql
   CREATE EXTERNAL TABLE relevance_hbase_emp(
   empno int,
   ename string,
   job string,
   mgr int,
   hiredate string,
   sal double,
   comm double,
   deptno int)
   STORED BY 
   'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
   WITH SERDEPROPERTIES ("hbase.columns.mapping" = 
   ":key,info:ename,info:job,info:mgr,info:hiredate,info:sal,info:comm,info:deptno") 
   TBLPROPERTIES ("hbase.table.name" = "hbase_emp_table");
   ```

2. 关联后就可以使用Hive函数进行一些分析操作了

   ```sql
   hive (default)> select * from relevance_hbase_emp;
   ```

   

