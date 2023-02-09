## 一、介绍

**HDFS:分布式的文件系统 Hadoop Distributed File System**

在Hadoop生态圈中，HDFS属于底层基础，负责存储文件。

**HDFS的优点：**

高容错性。提供了容错和恢复机制，副本丢失后，自动恢复。

高可靠性。数据自动保存多个副本，通过多副本提高可靠性。

适合大数据处理。可以处理超大文件，比如 TB级甚至PB级 的文件。

适合批处理。移动计算而非移动数据；数据位置暴露给计算框架。

支持流式数据访问。一次性写入，多次读取（一个数据集一旦生成，就会被复制分发到不同的存储节点，各节点可以进行读取/访问）；保证数据一致性。

低成本运行。可以运行在低成本的硬件之上。



**HDFS的缺点：**

不适合处理 低延迟 的数据访问。比如用户 要求时间比较短 的低延迟应用（主要处理高数据吞吐量的应用）。

不适合处理 大量的小 文件。会造成寻址时间超过读取时间；会占用NameNode大量内存，因为NameNode把文件系统的元数据存放在内存中（文件系统的容量由NameNode的大小决定），小文件太多会消耗NameNode的内存。

不适合 并发写入。一个文件只能有一个写入者，HDFS暂不支持多个用户对同一个文件的写操作。

不适合 任意修改 文件。仅支持append(附加)，不支持在文件的任意位置进行修改。

HDFS视硬件错误为常态，硬件服务器随时有可能发生故障。 为了容错，文件的所有 block 都会有副本。









**HDFS的组成架构图及各部分功能如下所示：**

![img](https://cdn.nlark.com/yuque/0/2021/png/22463628/1636279481455-26ced254-e9a8-4771-9794-0498e49aab39.png)

hdfs将所有的文件全部抽象成为block块来进行存储，不管文件大小，全部一视同仁都是以block块的统一大小和形式进行存储，方便我们的分布式文件系统对文件的管理



HDFS数据块默认大小为128M（Hadoop2.2之前为64M），将HDFS的数据块设置得很大的目的是什么？（传统数据块只有512个字节）

答：为了减少寻址开销，让HDFS的文件传输时间由传输速率决定（如果块设置得足够大，从磁盘 传输数据的时间 会明显大于 定位这个块开始位置 所需的时间）。



## 二、HDFS 设计原理

![img](HDFS.assets/hdfsarchitecture.png)

### HDFS 架构

HDFS 遵循主/从架构，由单个 NameNode(NN) 和多个 DataNode(DN) 组成：

- **NameNode** : 负责执行有关 `文件系统命名空间` 的操作，例如打开，关闭、重命名文件和目录等。它同时还负责集群元数据的存储，记录着文件中各个数据块的位置信息。
- **DataNode**：负责提供来自文件系统客户端的读写请求，执行块的创建，删除等操作。



**NameNode节点**

当用户访问数据文件时，为了保证能够读取到每一个数据块， HDFS有一个专门 **负责保存文件属性信息**的节点，这个节点就是 **NameNode** 节点（即 名称节点 ）。NameNode节点 是HDFS的管理者，负责保存和管理HDFS的元数据。

节点职责：

**① 管理维护HDFS的命名空间**
NameNode管理HDFS系统的命名空间，维护文件系统树以及文件系统树中所有文件的元数据。管理这些信息的的文件分别是 **edits**（操作日志文件） 和 **fsimage**（命名空间镜像文件） 。

其中，FsImage镜像文件用于存储整个文件系统命名空间的信息，EditLog日志文件用于持久化记录文件系统元数据发生的变化。当NameNode启动的时候，FsImage镜像文件就会被加载到内存中，然后对内存里的数据执行记录的操作，以确保内存所保留的数据处于最新的状态，这样就加快了元数据的读取和更新操作。

HDFS 的 `文件系统命名空间` 的层次结构与大多数文件系统类似 (如 Linux)， 支持目录和文件的创建、移动、删除和重命名等操作，支持配置用户和访问权限，但不支持硬链接和软连接。`NameNode` 负责维护文件系统名称空间，记录对名称空间或其属性的任何更改。

**② 管理DataNode上的数据块**
负责管理数据块上所有的元数据信息（管理DataNode上数据块的均衡，维持副本数量）。

**③ 接收客户端的请求**
接收客户端文件上传、下载、创建目录等的请求。





**DataNode节点**

HDFS首先把大文件切分成若干个小的数据块，再把这些数据块写入不同的节点，这个 **负责保存文件数据**的节点就是 **DataNode** 节点（即 数据节点 ）。DataNode节点 负责存储数据，把Block(数据块)以Linux文件的形式保存在磁盘上，并根据Block标识和字节范围来读写块数据。

节点职责：

① 保存数据块

一个数据块会在多个DataNode进行冗余备份（在某一个DataNode最多只有一个备份）。

② 负责客户端对数据块的IO请求

在客户端执行写操作时，DataNode之间会相互通信，保证写操作的一致性。

③ 定期和NameNode进行心跳通信，接受NameNode的指令

如果NameNode节点10分钟没有收到DataNode的心跳信息，就会将其上的数据块复制到其他DataNode节点。



**SecondaryNameNode节点**

HDFS有一个定期创建命名空间的检查点(CheckPoint)操作的节点，也就是SecondaryNameNode节点（即 第二名称节点）。

出于可靠性考虑，SecondaryNameNode节点与NameNode节点通常运行在不同的机器上，且SecondaryNameNode节点与NameNode节点的内存要一样大。

节点职责

SecondaryNameNode节点 定期把NameNode的 **fsimage** 和 **edits** 下载到本地，再将它们加载到内存并进行合并，最后把合并后新的 fsimage 返回NameNode （这个过程称为**检查点**）。

![img](https://cdn.nlark.com/yuque/0/2021/png/22463628/1636280030572-28818db4-a0dc-4bce-abf3-b66c28549d2a.png)

**作用：**

**① 防止edits过大**
定期合并 fsimage 和 edits 文件，使 edits 大小保持在限制范围内。这样做减少了重新启动NameNode时合并 fsimage 和 edits 耗费的时间，从而减少了NameNode启动的时间。

**② 做冷备份**
对一定范围内数据做快照性备份，在NameNode失效时能恢复部分 fsimage 。

### 副本节点选择

**Hadoop2.7.2副本节点选择**
第一个副本在client所处的节点上。如果客户端在集群外，随机选一个。
第二个副本和第一个副本位于相同机架，随机节点。
第三个副本位于不同机架，随机节点。

为了最大限度地减少带宽消耗和读取延迟，HDFS 在执行读取请求时，优先读取距离读取器最近的副本。如果在与读取器节点相同的机架上存在副本，则优先选择该副本。如果 HDFS 群集跨越多个数据中心，则优先选择本地数据中心上的副本。



### 架构的稳定性

1. 心跳机制和重新复制

每个 DataNode 定期向 NameNode 发送心跳消息，如果超过指定时间没有收到心跳消息，则将 DataNode 标记为死亡。NameNode 不会将任何新的 IO 请求转发给标记为死亡的 DataNode，也不会再使用这些 DataNode 上的数据。 由于数据不再可用，可能会导致某些块的复制因子小于其指定值，NameNode 会跟踪这些块，并在必要的时候进行重新复制。

2. 数据的完整性

由于存储设备故障等原因，存储在 DataNode 上的数据块也会发生损坏。为了避免读取到已经损坏的数据而导致错误，HDFS 提供了数据完整性校验机制来保证数据的完整性，具体操作如下：

当客户端创建 HDFS 文件时，它会计算文件的每个块的 `校验和`，并将 `校验和` 存储在同一 HDFS 命名空间下的单独的隐藏文件中。当客户端检索文件内容时，它会验证从每个 DataNode 接收的数据是否与存储在关联校验和文件中的 `校验和` 匹配。如果匹配失败，则证明数据已经损坏，此时客户端会选择从其他 DataNode 获取该块的其他可用副本。

3.元数据的磁盘故障

`FsImage` 和 `EditLog` 是 HDFS 的核心数据，这些数据的意外丢失可能会导致整个 HDFS 服务不可用。为了避免这个问题，可以配置 NameNode 使其支持 `FsImage` 和 `EditLog` 多副本同步，这样 `FsImage` 或 `EditLog` 的任何改变都会引起每个副本 `FsImage` 和 `EditLog` 的同步更新。

4.支持快照

快照支持在特定时刻存储数据副本，在数据意外损坏时，可以通过回滚操作恢复到健康的数据状态。

## HDFS 常用 shell 命令

**1. 显示当前目录结构**

```
# 显示当前目录结构
hadoop fs -ls  <path>
# 递归显示当前目录结构
hadoop fs -ls  -R  <path>
# 显示根目录下内容
hadoop fs -ls  /
```

**2. 创建目录**

```
# 创建目录
hadoop fs -mkdir  <path> 
# 递归创建目录
hadoop fs -mkdir -p  <path>  
```

**3. 删除操作**

```
# 删除文件
hadoop fs -rm  <path>
# 递归删除目录和文件
hadoop fs -rm -R  <path> 
```

**4. 从本地加载文件到 HDFS**

```
# 二选一执行即可
hadoop fs -put  [localsrc] [dst] 
hadoop fs - copyFromLocal [localsrc] [dst] 
```

**5. 从 HDFS 导出文件到本地**

```
# 二选一执行即可
hadoop fs -get  [dst] [localsrc] 
hadoop fs -copyToLocal [dst] [localsrc] 
```

**6. 查看文件内容**

```
# 二选一执行即可
hadoop fs -text  <path> 
hadoop fs -cat  <path>  
```

**7. 显示文件的最后一千字节**

```
hadoop fs -tail  <path> 
# 和Linux下一样，会持续监听文件内容变化 并显示文件的最后一千字节
hadoop fs -tail -f  <path> 
```

**8. 拷贝文件**

```
hadoop fs -cp [src] [dst]
```

**9. 移动文件**

```
hadoop fs -mv [src] [dst] 
```

**10. 统计当前目录下各文件大小**

- 默认单位字节
- -s : 显示所有文件大小总和，
- -h : 将以更友好的方式显示文件大小（例如 64.0m 而不是 67108864）

```
hadoop fs -du  <path>  
```

**11. 合并下载多个文件**

- -nl 在每个文件的末尾添加换行符（LF）
- -skip-empty-file 跳过空文件

```
hadoop fs -getmerge
# 示例 将HDFS上的hbase-policy.xml和hbase-site.xml文件合并后下载到本地的/usr/test.xml
hadoop fs -getmerge -nl  /test/hbase-policy.xml /test/hbase-site.xml /usr/test.xml
```

**12. 统计文件系统的可用空间信息**

```
hadoop fs -df -h /
```

**13. 更改文件复制因子**

```
hadoop fs -setrep [-R] [-w] <numReplicas> <path>
```

- 更改文件的复制因子。如果 path 是目录，则更改其下所有文件的复制因子
- -w : 请求命令是否等待复制完成

```
# 示例
hadoop fs -setrep -w 3 /user/hadoop/dir1
```

**14. 权限控制**

```
# 权限控制和Linux上使用方式一致
# 变更文件或目录的所属群组。 用户必须是文件的所有者或超级用户。
hadoop fs -chgrp [-R] GROUP URI [URI ...]
# 修改文件或目录的访问权限  用户必须是文件的所有者或超级用户。
hadoop fs -chmod [-R] <MODE[,MODE]... | OCTALMODE> URI [URI ...]
# 修改文件的拥有者  用户必须是超级用户。
hadoop fs -chown [-R] [OWNER][:[GROUP]] URI [URI ]
```

**15. 文件检测**

```
hadoop fs -test - [defsz]  URI
```

可选选项：

- -d：如果路径是目录，返回 0。
- -e：如果路径存在，则返回 0。
- -f：如果路径是文件，则返回 0。
- -s：如果路径不为空，则返回 0。
- -r：如果路径存在且授予读权限，则返回 0。
- -w：如果路径存在且授予写入权限，则返回 0。
- -z：如果文件长度为零，则返回 0。

```
# 示例
hadoop fs -test -e filename
```