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

部署完在DataGrip根据设定好的参数添加数据源即可连接到数据库

#### 测试工具BenchmarkSQL：

##### 简介：

BenchmarkSQL 是一个用于测试关系型数据库性能的开源工具，特别适用于OLTP（在线事务处理）场景。它通过模拟真实的业务工作负载来评估数据库系统的性能，包括事务吞吐量、响应时间和并发支持能力等。它的主要功能有：

多线程执行: 支持多线程并发执行，可以模拟大量用户同时访问数据库的情况。

多种工作负载模式: 提供了多种预定义的工作负载模式，可以根据具体需求选择合适的模式;可以自定义新的工作负载模式。

详细的统计报告: 生成详细的性能报告，包括事务成功率、平均响应时间、最大和最小响应时间等。

灵活的配置选项: 支持丰富的配置选项，允许用户调整各种参数以适应不同的测试环境和需求。

支持多种数据库: 最初是为 PostgreSQL设计，但可以通过配置支持其他兼容 JDBC 的数据库系统，如 MySQL、Oracle 等。

##### 工具部署:

先从https://sourceforge.net/projects/benchmarksql 下载benchmarksql压缩包，我放在目录~/DBProj3下

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
