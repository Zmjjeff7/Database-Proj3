#### 数据库原理Project3:

# OpenGauss与PostgreSQL数据库的对比报告

郭振弘 12310817

代码和数据等额外资源在git仓库：[Zmjjeff7/Database-Proj3](https://github.com/Zmjjeff7/Database-Proj3)

**摘要**

OpenGauss是华为最新以PostgreSQL为基础打造的一个开源关系数据库，官方称它做了性能等各方面的优化且专为处理复杂的数据密集型工作负载而设计。本报告便以OpenGauss和PostgresSQL这两个关系数据库为例试着比较这两者的性能等各方面因素，例如查询性能、可靠性、安全性等，并最终给出OpenGauss数据库的优劣势，同时也可以探究评判数据库优劣时都可以有什么指标。

**关键词：**OpenGauss，PostgreSQL，关系数据库，性能比较

### 一、引言

#### 研究背景及目的

随着科技的不断发展，我们现在正处于快节奏的信息化时代，网络上各种来源的数据也是越来越多，这些数据对于科研或者企业经营等方面的决策的影响也自然会越来越大。于是如今对于高效、可靠的数据管理解决方案的需求愈发迫切。这就需要先理清楚一个数据库的好坏与否到底都要由什么样的一些因素或测试结果去衡量，以便分清楚哪些数据库是更优秀的。现如今在众多的选择中，关系型数据库管理系统（RDBMS）因其结构化查询语言（SQL）的支持、事务处理的ACID特性以及成熟的生态系统而占据重要地位。然而，在面对高并发、大数据量的工作负载时，不同RDBMS之间的性能差异变得尤为显著。因此，了解和评估这些数据库在实际工作环境中的表现，对于确保业务连续性和数据安全至关重要。

#### 比较对象简介

- **PostgreSQL**: 作为全球最著名的免费开源关系型数据库之一，PostgreSQL有着强大的适应性、良好的可扩展性及对标准的高度遵循。它不仅支持ACID特性以保证交易处理的一致性和可靠性，还提供了高度的灵活性，允许用户定义自定义函数、类型，甚至添加新的索引类型。此外，PostgreSQL以其对多种高级数据类型的原生支持（如JSON、数组等），以及庞大的社区支持网络而著称，这使得它成为了一个稳定且文档齐全的开源RDBMS选项。

- **OpenGauss**: OpenGauss是由华为在Postgresql数据库的基础上开发并开源的关系型数据库管理系统（RDBMS），在性能、安全性和可靠性等多方面做了改进和优化。在OpenGauss的概述幻灯片中介绍到其专为处理复杂的数据密集型工作负载而设计，适用于金融系统、电信行业以及大型电子商务平台等领域。该数据库系统承诺能够在高并发、大数据量的环境中保持高效运作，并且具备自动故障切换和恢复的能力，确保了服务的连续性和数据的安全性。

作为和PostgreSQL同样已经开源的数据库，OpenGauss现在承诺其在PostgreSQL的基础上在高负载的工作情形下能够提供更好的性能。于是若经过我们的测试对比研究发现它的确全方面优于PostgreSQL，那么OpenGauss就会是一个面向应用场景优化数据库的很好的样例并且值得考虑广泛运用。若有些缺点被测试出来了也可以为未来数据库的发展升级换代提供一些优化的方向或思路。

### 二、评估准则

在评判一个数据库的好坏时若希望尽可能的全面、客观，我们应该要考虑到以下几个关键方面：

#### 1. 查询性能 (Query Performance)

- **查询响应时间**：即是数据库从发出查询请求到输出完整结果所用的时间。其中包括简单查询和复杂查询（例如多表连接、子查询和聚合操作）。快速的查询响应时间对于提高用户体验至关重要。而且在本学期Proj1中尝试用Java或者C++实现数据库的一些查询功能时就不难发现，就像于老师课上指出的一样，现如今性能方面的提升有一大瓶颈就是数据输入和输出（IO，数据读写）的效率，所以评估一个数据库对于这方面的优化如何是很有意义的。
- **事务处理能力（TPS, Transactions Per Second）**：指每秒钟可以完成的事务数量，这是评估数据库在高并发环境下的处理效率的重要指标。
- **并发处理能力**：考察数据库在多个用户同时访问时的表现，特别是它如何管理资源分配以确保所有请求都能得到及时响应。

#### 2. 可靠性 (Reliability)

- **故障恢复能力**：评估数据库在发生硬件或软件故障后自动恢复的能力，包括预防数据丢失、自动故障切换机制以及灾难恢复方案等。
- **数据一致性**：确保所有提交的数据在工作期间保持一致，即使异常也能维持ACID特性（原子性、一致性、隔离性和持久性）。
- **系统稳定性**：通过长时间运行测试来验证数据库能否稳定工作而不出现意外崩溃或性能下降的问题。稳定的系统能够保证业务连续性，同时也可以更好地避免数据丢失或损坏等意外发生。

#### 3. 安全性 (Security)

- **SQL注入防护**：检查数据库是否内置防御机制防止SQL注入攻击。这是一种常见的安全威胁，可能造成敏感信息泄露或数据篡改。
- **权限管理**：评估细粒度权限控制功能，允许管理员精确设置用户对不同对象（如表、视图、存储过程）的操作权限，并且支持多种认证方式（如LDAP、Kerberos）。

#### 4. 扩展性 (Scalability)

- **水平扩展**：考察数据库通过增加更多节点来提升整体性能的能力，特别是在分布式环境中，需要考虑集群管理和负载均衡策略。
- **垂直扩展**：评估数据库通过增加现有服务器上的CPU、内存等硬件资源来提高性能的能力，重点在于系统的最大负载能力和相应的性能变化。

#### 5. 兼容性 (Compatibility)

- **SQL标准支持**：衡量数据库对SQL语言标准的支持程度，包括最新的ANSI/ISO SQL规范。良好的SQL标准支持有助于简化应用程序开发和迁移。
- **集成能力**：评估与其他系统（如操作系统、编程语言或其他数据库产品）的兼容性和互操作性，确保可以无缝集成到现有的IT基础设施中。此外，还应考虑API接口的丰富性和灵活性。

虽然以上有些指标在个人本地可能较难完成测试，但这些都是企业等大型用户在选择数据库时需要考虑到的因素。

### 三、研究方法

#### 硬件配置：

处理器：12th Gen Intel(R) Core(TM) i7-12700H (20 CPUs), ~2.3GHz

内存：40960MB RAM

显卡：NVIDIA GeForce RTX 3060 Laptop GPU

#### 软件配置：

Windows 11 家庭中文版64位（10.0，版本26100）（配置了docker和wsl2-ubuntu22.04）

Docker Desktop

PostgreSQL （用docker配置到Ubuntu22.04环境下，port为5432）

OpenGauss （用docker分别配置到Win和Ubuntu22.04环境下，port分别为15432和35432）

JetBrains DataGrip 2024.2.2

BenchmarkSQL-5.0

#### **数据库部署：**

借鉴助教提供的教程：

```cmd
$ docker pull docker.1panel.live/enmotech/opengauss:3.0.0 

$ docker run --name project3-opengauss --privileged=true -d -e GS_PASSWORD=Gzh12310817* -v ~/DBProj3/opengauss:/var/lib/opengauss -u root -p 35432:5432 docker.1panel.live/enmotech/opengauss:3.0.0

$ docker pull docker.1panel.live/ubuntu/postgres:14-22.04_beta

$ docker run -d --name project3-postgres -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=Gzh12310817* -e POSTGRES_DB=postgres -e PGDATA=/var/lib/postgresql/data/pgdata -p 5432:5432 -v ~/DBProj3/postgres:/var/lib/postgresql/data docker.1panel.live/ubuntu/postgres:14-22.04_beta
```

或者其中PostgreSQL可以直接在Ubuntu-22.04中部署：

```cmd
$ sudo apt update
$ sudo apt install postgresql postgresql-contrib
$ sudo systemctl start postgresql
$ sudo systemctl enable postgresql
$ sudo -u postgres psql
$ \password postgres //设置数据库用户postgres的密码
```

部署完在DataGrip根据设定好的参数添加数据源即可连接到数据库:

<img src="E:\DBProj3\Database-Proj3\数据库连接.png" alt="数据库连接" style="zoom:50%;" />

#### 测试工具BenchmarkSQL：

##### 简介：

BenchmarkSQL 是一个用于测试关系型数据库性能的开源工具，特别适用于OLTP（在线事务处理）场景。它通过模拟真实的业务工作负载来评估数据库系统的性能，包括事务吞吐量、响应时间和并发支持能力等。它的主要功能有：

多线程执行: 支持多线程并发执行，可以模拟大量用户同时访问数据库的情况。

多种工作负载模式: 提供了多种预定义的工作负载模式，可以根据具体需求选择合适的模式;可以自定义新的工作负载模式。

详细的统计报告: 生成详细的性能报告，包括事务成功率、平均响应时间、最大和最小响应时间等。

灵活的配置选项: 支持丰富的配置选项，允许用户调整各种参数以适应不同的测试环境和需求。

支持多种数据库: 最初是为 PostgreSQL设计，但可以通过配置支持其他兼容 JDBC 的数据库系统，如 MySQL、Oracle 等。

##### 工具部署:

先从https://sourceforge.net/projects/benchmarksql/下载benchmarksql压缩包，我放在目录~/DBProj3下

```cmd
$ cd ~/DBProj3
$ unzip benchmarksql-5.0.zip
$ cd benchmarksql-5.0
$ sudo apt-get install ant
$ //编译benchmarksql
$ ant 
$ //更新postgres JDBC驱动(非常重要，否则连不上PostgreSQL)
$ cd lib/postgres
$ mv postgresql-9.3-1102.jdbc41.jar postgresql-9.3-1102.jdbc41.jar_20230428
$ wget https://jdbc.postgresql.org/download/postgresql-42.6.0.jar
$ //创建压测用户和库
$ sudo -u postgres createdb benchmarkdb
$ sudo -u postgres psql -c "CREATE USER benchmarkuser WITH PASSWORD '410329';"
$ sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE benchmarkdb TO benchmarkuser;"
$ //修改配置文件
$ cd ~/DBProj3/benchmarksql-5.0/run
$ vim postgres.properties
//文件内参数照如下格式按要求修改
db.url=jdbc:postgresql://localhost:5432/benchmarkdb // 待测数据库的端口及压测库名称
db.user=benchmarkuser // 压测用户名
db.password=410329 // 压测用户密码
terminals=1 // 测试终端数量
//To run specified transactions per terminal- runMins must equal zero
runTxnsPerTerminal=0 
//To run for specified minutes- runTxnsPerTerminal must equal zero
runMins=60 // 测试时长（单位为min）
$ //部署结束
```

#### 实验策略：

##### BenchmarkSQL:

我们测试以下几组情况：

case 1: 数据库-由docker部署在win11系统的OpenGauss，测试终端数量-1，测试时长-60

case 2: 数据库-由docker部署在win11系统的OpenGauss，测试终端数量-4，测试时长-60

case 3: 数据库-由docker部署在win11系统的OpenGauss，测试终端数量-8，测试时长-60

case 4: 数据库-由docker部署在win11系统的OpenGauss，测试终端数量-1，测试时长-180

case 5: 数据库-由docker部署在Ubuntu系统的OpenGauss，测试终端数量-1，测试时长-60

case 6: 数据库-由docker部署在Ubuntu系统的OpenGauss，测试终端数量-4，测试时长-60

case 7: 数据库-由docker部署在Ubuntu系统的OpenGauss，测试终端数量-8，测试时长-60

case 8: 数据库-由docker部署在Ubuntu系统的OpenGauss，测试终端数量-1，测试时长-180

case 9: 数据库-部署在Ubuntu系统的PostgreSQL，测试终端数量-1，测试时长-60

case 10: 数据库-部署在Ubuntu系统的PostgreSQL，测试终端数量-4，测试时长-60

case 11: 数据库-部署在Ubuntu系统的PostgreSQL，测试终端数量-8，测试时长-60

case 12: 数据库-部署在Ubuntu系统的PostgreSQL，测试终端数量-1，测试时长-180

（经实践，测试终端数量不可以是16，测试终端数量也可视为并发数）

设定好配置文件后开始压测：

```cmd
$ ./runDatabaseDestroy.sh postgres.properties
$ ./runDatabaseBuild.sh postgres.properties
$ ./runBenchmark.sh postgres.properties
$ //压测开始
$ //压测结束后run文件夹下会多出一个my_result开头的文件夹
$ ./generateReports.sh my_result_* // 生成最后的html格式的报告文件
```

运行示例：

![屏幕截图 2024-12-22 190530](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 190530.png)

### 四、实验结果及分析

##### 实验结果格式阐述：

Transaction Type（事务类型）代表了模拟的TPC-C基准测试中的五种核心事务。每种事务都模拟了商业应用中可能遇到的操作，以评估数据库在不同工作负载下的性能。

表格列出了不同类型事务（如NEW_ORDER, PAYMENT等）的统计信息，包括：

- Latency (延迟)：列出90百分位数（90th %）和最大值（Maximum），表示大部分请求（90%）的响应时间和最慢的响应时间。
- Count (计数)：执行成功的事务数量。
- Percent (百分比)：各类型事务占总事务的比例。
- Rollback (回滚)：事务失败并被回滚的比例。
- Errors (错误)：事务处理中遇到的错误次数。
- Skipped (跳过)：某些情况下可能跳过的事务数量。

tpmC 是每分钟完成的新订单事务数量，这是TPC-C基准测试中的关键性能指标之一。
tpmTotal 表示所有类型事务的总吞吐量（每分钟的事务总数），这可以用来评估系统的整体性能。

##### 实验结果:

##### case 1

![屏幕截图 2024-12-22 164126](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 164126.png)

![屏幕截图 2024-12-22 164139](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 164139.png)

##### case 2

![屏幕截图 2024-12-22 165330](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 165330.png)

![屏幕截图 2024-12-22 165354](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 165354.png)

##### case 3

![屏幕截图 2024-12-22 165529](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 165529.png)

![屏幕截图 2024-12-22 165544](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 165544.png)

##### case 4

![屏幕截图 2024-12-22 165928](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 165928.png)

![屏幕截图 2024-12-22 165947](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 165947.png)

##### case 5

![屏幕截图 2024-12-22 170550](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 170550.png)

![屏幕截图 2024-12-22 170604](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 170604.png)

##### case 6

![屏幕截图 2024-12-22 170754](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 170754.png)

![屏幕截图 2024-12-22 170805](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 170805.png)

##### case 7

![屏幕截图 2024-12-22 171531](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 171531.png)

![屏幕截图 2024-12-22 171841](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 171841.png)

##### case 8

![屏幕截图 2024-12-22 185144](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 185144.png)

![屏幕截图 2024-12-22 185156](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 185156.png)

##### case 9

![屏幕截图 2024-12-22 172210](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 172210.png)

![屏幕截图 2024-12-22 172222](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 172222.png)

##### case 10

![屏幕截图 2024-12-22 172243](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 172243.png)

![屏幕截图 2024-12-22 172254](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 172254.png)

##### case 11

![屏幕截图 2024-12-22 172323](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 172323.png)

![屏幕截图 2024-12-22 172343](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 172343.png)

##### case 12

![屏幕截图 2024-12-22 172555](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 172555.png)

![屏幕截图 2024-12-22 172609](C:\Users\Jeff\Pictures\Screenshots\屏幕截图 2024-12-22 172609.png)

（还有很多额外实验结果在git仓库中）



##### 实验结果分析：

回忆一下每种case:

case 1-4: 数据库-由docker部署在win11系统的OpenGauss，测试终端数量-1、4、8、1，测试时长-60、60、60、180

case 5-8: 数据库-由docker部署在Ubuntu系统的OpenGauss，测试终端数量-1、4、8、1，测试时长-60、60、60、180

case 9-12: 数据库-部署在Ubuntu系统的PostgreSQL，测试终端数量-1、4、8、1，测试时长-60、60、60、180



以下是NEW_ORDER的Latency数据：

| case | latency(90%) | latency(max) |
| ---- | ------------ | ------------ |
| 1    | 0.039        | 0.190        |
| 2    | 0.034        | 0.206        |
| 3    | 0.046        | 0.217        |
| 4    | 0.036        | 0.526        |
| 5    | 0.026        | 0.356        |
| 6    | 0.030        | 0.323        |
| 7    | 0.043        | 0.398        |
| 8    | 0.028        | 0.753        |
| 9    | 0.011        | 0.654        |
| 10   | 0.009        | 0.457        |
| 11   | 0.011        | 0.402        |
| 12   | 0.011        | 0.692        |

若看一整列Latency(90%)的数据不难发现PostgreSQL的数据要明显比OpenGauss的小,并且由docker部署在Ubuntu的OpenGauss会比部署在win11的要略快一点。而为了消除PostgreSQL在本地wsl部署的这一因素的影响，我用同样通过docker部署在win11的PostgreSQL用同样的参数做测试（结果在仓库中有），发现Latency（90%）变成了0.031，比对应的0.039还是要小，不过差距已经很小。这说明**本地部署的请求延迟确实会比用docker部署的要小**，而且PostgreSQL的大多数请求延迟会比OpenGauss的要小。

但有意思的是在Latency（max）的数据下OpenGauss的数据整体要比PostgreSQL的要小不少，这说明OpenGauss的查询延迟虽然可能比PostgreSQL要略慢一点，但是整体查询延迟要更稳定，波动区间会更小，**这使得OpenGauss在面临庞大的数据量处理时表现出的性能可能会更加稳定高效**，而对于PostgreSQL只要在长时间运行中有请求的延迟慢很多就会一起影响到后面的查询。

如果单独对比1和4，5和8，9和11这一点会更明显，可以看出来PostgreSQL的最大延迟随着运行时间的增长变化不大，都会比OpenGauss的最大延迟要大。而对于OpenGauss，最大延迟是随着运行时间变长才会慢慢变大，这说明OpenGauss的稳定性比起PostgreSQL会更胜一筹，进而在数据处理性能效率上也会更好。

另外还有一个趋势可以单独对比1、2、3或5、6、7或9、10、11得到，也就是**随着的并发数的增大查询延迟会呈现一个下凹状的变化（先变小再变大）**，这可能是因为查询终端的变多最开始会使得数据库对数据的处理速度变快，但随着数量继续增多、并发数变大，数据库的性能会因为多线程的查询而受到影响从而增大了查询延迟。



下面是NEW_ORDER的Rollback数据：

| case | Rollback(%) |
| ---- | ----------- |
| 1    | 1.131       |
| 2    | 0.985       |
| 3    | 1.116       |
| 4    | 1.004       |
| 5    | 1.042       |
| 6    | 0.823       |
| 7    | 1.207       |
| 8    | 0.947       |
| 9    | 1.306       |
| 10   | 1.039       |
| 11   | 0.998       |
| 12   | 0.883       |

首先比较容易得到的结论是部署在Ubuntu的OpenGauss的回滚率基本要小于部署在win11的OpenGauss。其次与查询延迟类似，回滚率随着并发数的增大会有先变小再变大的趋势，原因应该是类似的。而整体上来讲OpenGauss的回滚率要比PostgreSQL要小（可参考仓库中其他结果）。



最后分析tpmTotal的统计图象：

case 8:

![tpm_nopm](E:\DBProj3\Database-Proj3\实验结果\my_result_2024-12-22_152900_og_1_3_180\tpm_nopm.png)

case 12:

![tpm_nopm](E:\DBProj3\Database-Proj3\实验结果\my_result_2024-12-22_090555_p_1_0_180\tpm_nopm.png)

先将case1、2、3、4分别和case5、6、7、8对比可以看到其变化图像形状和趋势等基本一致，可见配置环境对该指标的影响不大。我们再把case5、6、7、8分别和case9、10、11、12对比，可以得到**OpenGauss在运行时的事务吞吐量随着时间的变化会更加有规律**，并且波动幅度也会稍小一些，从而显示出其比PostgreSQL在运行时会更加稳定。特别是在对比case8和case12时会注意到，**PostgreSQL在长达3小时的测试时长中吞吐量的波动明显变的越来越大（且是向下波动的幅度更大**），**而OpenGauss的吞吐量始终很稳定**。这充分体现了OpenGauss在运行时的**可靠性**是要优于PostgreSQL的。

另外，在OpenGauss的官方讲解PPT上也有一个BenchmarkSQL的结果图（见下）

![微信图片_20241222215829](E:\DBProj3\Database-Proj3\微信图片_20241222215829.png)

由图上参数可知它在测试时的数据量比我本地应该是要大很多的，并且一个小时内其吞吐量像我的结果中的case 12一样有着明显的下降，不过在大概前40分钟内都还是十分稳定的。由此也可以侧面证实我们实验结果以及分析的合理性。

### 五、实验总结与反思

经过这次实验，感觉OpenGauss的一大优势就是**稳定**，无论是查询延迟还是事务吞吐量都在保证了较高性能的前提下做到了能随着长时间的运作将高性能维持住。这一点感觉很符合需要让数据库服务器**长期处于正常高效运作状态下处理巨量数据**的企业等大型用户，所以OpenGauss关于其适合大型电子商务平台等领域的言论并非空穴来风。而OpenGauss的劣势便在于它还不够成熟，不像PostgreSQL这种经典关系数据库已经通过时间的变迁来证明了其可用性，OpenGauss在安全性、扩展性以及功能完整性等方面还有待完整的测试优化才有可能真正成为一个能被广泛采纳的数据库。相比较下PostgreSQL就更加适合个人短期的使用，效率高并且由于处理的数据量偏小一般不会因为长时间运行或者访问量过大而对使用性能造成比较大的影响。

但这次实验中有一些我们前面提及到的因素还并未研究到，比如数据库对于资源的利用率，数据库的可扩展性以及安全性等。并且该实验中使用的BenchmarkSQL根据其代码内容还应该有统计CPU利用率等功能在项目过程中最终并未实现，并且已经得到的结果数据也并未所有都有分析到，还有一些跟我们的结论呈对立的特殊结果。

即使这次项目完结，后续我们依然可以尝试去增加并发数或者压测数据量、进行多次重复实验、记录压测时数据库占用内存等资源的大小、改变数据库在运行或者压力测试时使用的CPU核心数或者节点数并检测其性能变化以及配置更多不同的数据库压测工具（例如pgbench）来进一步探索数据库的各项指标。



### 参考链接

[openGauss开源社区正式上线 - 华为](https://www.huawei.com/en/news/2020/7/opengauss-open-source-community)

[欢迎来到 openGauss |openGauss 官方网站](https://opengauss.org/en/)

[PostgreSQL - Wikipedia](https://en.wikipedia.org/wiki/PostgreSQL)

[数据库性能压测之TPC-C基准测试_tpcc-CSDN博客](https://blog.csdn.net/TIME_1981/article/details/126114797)

[Home - BenchmarkSQL](https://benchmarksql.readthedocs.io/en/latest/)

[学习使用benchmarksql压测数据库 - 高&玉 - 博客园](https://www.cnblogs.com/haha029/p/17371480.html)

[AbdallahCoptan/PostGreSQL-Bench: PostGreSQL Database Server installation, configurationa and remote connection with the postgresql client](https://github.com/AbdallahCoptan/PostGreSQL-Bench)

