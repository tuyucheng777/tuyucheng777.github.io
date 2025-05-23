## LLVM

* [JLang](https://github.com/polyglot-compiler/JLang)：JLang向Polyglot编译器添加了LLVM后端，将Java转换为LLVM IR，由康奈尔大学开源。
* [LLFI](https://github.com/DependableSystemsLab/LLFI)：LLFI是一个基于LLVM的故障注入工具，它将故障注入到应用程序源代码的LLVM IR中，由不列颠哥伦比亚大学开源。
* [Maple-IR](https://github.com/LLVM-but-worse/maple-ir)：Maple-IR是一个基于工业IR的Java字节码静态分析框架。
* [JDA](https://github.com/LLVM-but-worse/java-disassembler)：JDA提供强大的静态分析工具，例如控制和数据流分析，以及使用自定义IL构建的代码简化。

## WebAssembly

* [GraalWasm](https://www.graalvm.org/webassembly/)：GraalWasm是一个适用于Java的高性能嵌入式WebAssembly运行时。
* [Bytecoder](https://github.com/mirkosertic/Bytecoder)：Bytecoder是Java字节码和框架的富域模型，用于将其解释并转换为其他语言，例如JavaScript、OpenCL或WebAssembly。
* [JWebAssembly](https://github.com/i-net-software/JWebAssembly)：JWebAssembly是Java字节码到WebAssembly的编译器。
* [TeaVM](https://github.com/konsoletyper/teavm)：TeaVM是Java字节码的提前编译器，可生成在浏览器中运行的JavaScript和WebAssembly。
* [DoppioJVM](https://github.com/plasma-umass/doppio)：Doppio是一个兼容POSIX的运行时系统以及一个用TypeScript编写的JVM，也是马萨诸塞大学PLASMA小组的一个活跃的研究项目。
* [Asmble](https://github.com/cretz/asmble)：Asmble是一个将WebAssembly代码编译为JVM字节码的编译器，它还包含一个解释器和实用程序，用于从命令行和JVM语言处理WASM代码。
* [Wasmtime Java](https://github.com/kawamuray/wasmtime-java)：Wasmtime的Java语言绑定。
* [CheerpJ](https://github.com/leaningtech/cheerpj-meta)：CheerpJ是一个基于WebAssembly的浏览器JVM，它与Java 8兼容，并提供完整的运行时环境，无需插件即可在浏览器中运行Java应用程序、Applet、库和Java Web Start/JNLP应用程序。
* [Chicory](https://github.com/dylibso/chicory)：Chicory是JVM原生WebAssembly运行时，它允许你以零本机依赖或JNI运行WebAssembly程序。
* [Wasmer](https://github.com/wasmerio/wasmer-java)：基于Wasmer的完整且成熟的Java WebAssembly运行时。

## JavaScript

* [Google Closure Compiler](https://github.com/google/closure-compiler)：Closure Compiler是一个使JavaScript下载和运行速度更快的工具，由Google开源。
* [DSBridge Android](https://github.com/wendux/DSBridge-Android)：现代跨平台JavaScript桥接器，通过它在JavaScript和原生应用程序之间可以同步或者异步调用彼此的函数。
* [GraalJS](https://github.com/oracle/graaljs)：JavaScript编程语言的高性能实现，由Oracle实验室基于GraalVM构建。
* [J2CL](https://github.com/google/j2cl)：J2CL是一个功能强大、简单且轻量级的从Java到Closure风格JavaScript的转译器，由Google开源。
* [Nashorn](https://github.com/openjdk/nashorn)：Nashorn的目标是使用原生JVM在Java中实现轻量级高性能JavaScript运行时。
* [Grakkit](https://github.com/grakkit/grakkit)：Minecraft的现代JavaScript开发环境。
* [Javet](https://github.com/caoccao/Javet)：在Java中嵌入Node.js和V8的绝佳方式。
* [Rhino](https://github.com/mozilla/rhino)：Rhino是完全用Java编写的JavaScript的开源实现，Mozilla开源。
* [PurpleJS](https://github.com/purplejs/purplejs)：PurpleJS是一个简单而强大的框架，用于在不离开JavaScript的情况下创建高性能的Web应用程序。
* [RingoJS](https://github.com/ringo/ringojs)：Ringo是一个基于JVM构建的JavaScript平台，并针对服务器端应用程序进行了优化。
* [Dynjs](https://github.com/dynjs/dynjs)：JVM的ECMAScript运行时。
* [Nodyn](https://github.com/nodyn/nodyn)：Nodyn是JVM上的Node.js兼容框架。
* [J2V8](https://github.com/eclipsesource/j2v8)：J2V8是V8的一组Java绑定，注重性能以及与V8的紧密集成。
* [ES4X](https://github.com/reactiverse/es4x)：ES4X是一款支持EcmaScript >= 5应用的小型运行时。
* [LebJS](https://github.com/LebsterFace/LebJS)：LebJS是用Java编写的JavaScript引擎。
* [JScript](https://github.com/TopchetoEU/jscript)：JScript是一个引擎，能够运行EcmaScript 5，完全用Java编写。
* [Karate JS](https://github.com/karatelabs/karate-js)：适用于JVM的轻量级JavaScript引擎。

## 编译器&插件

* [VirtualAPK](https://github.com/didi/VirtualAPK)：VirtualAPK是一个强大而轻量级的Android插件框架，由滴滴开源。
* [Janin](https://github.com/janino-compiler/janino)：Janino是一个超小、超快的Java编译器。
* [BugVM](https://github.com/ibinti/bugvm)：BugVM使用独立的JVM将字节码编译为二进制代码。
* [RoboVM](https://github.com/robovm/robovm)：RoboVM编译器将Java字节码转换为本机ARM或x86代码，应用程序直接在CPU上运行，不涉及解释器或虚拟机。
* [Jarslink](https://github.com/sofastack/sofa-jarslink)：Jarslink是SOFABoot官方基于SOFAArk开发的功能插件，负责管理多应用在SOFAArk容器之上的合并部署，由蚂蚁开源。
* [Chronicle Runtime Compiler](https://github.com/OpenHFT/Java-Runtime-Compiler)：Java运行时编译器。
* [Java-OO](https://github.com/amelentev/java-oo)：Java-OO是Java编译器和IDE的模块化扩展，用于支持运算符重载(类似Scala)。
* [Manifold](https://github.com/manifold-systems/manifold)：Manifold是一个Java编译器插件，其功能包括元编程、属性、扩展方法、运算符重载、模板、预处理器等。
* [DroidPlugin](https://github.com/DroidPluginTeam/DroidPlugin)：DroidPlugin是一个新的插件框架，它使宿主应用程序无需安装、修改和重新打包即可运行任何第三方APK。
* [PF4J](https://github.com/pf4j/pf4j)：PF4J是一个开源的轻量级Java插件框架。
* [JTransc](https://github.com/jtransc/jtransc)：JTransc是一个AOT，它将.class和.jar文件编译为目标编程语言/可执行文件，将所有必需的依赖项捆绑在单个文件或文件夹中，无需抖动或外部运行时。
* [Apache Commons JCI](https://github.com/apache/commons-jci)：Commons JCI是一个Java编译器接口，它可用于编译Java本身，或任何其他可编译为Java类的语言(例如Groovy或JavaScript)。
* [Deptitive](https://github.com/moditect/deptective)：Deptitive是javac的一个插件，它根据允许的依赖的描述来验证项目包之间的依赖关系，并在检测到任何无意的依赖关系时使编译失败。
* [Polyglot](https://github.com/polyglot-compiler/polyglot)：Polyglot是Java编程语言的高度可扩展的编译器前端，由康奈尔大学开源。
* [Jikes](https://jikes.sourceforge.net/)：Jikes是一个编译器，它将Java语言规范中定义的Java源文件转换为Java虚拟机规范中定义的字节码指令集和二进制格式，由IBM开源。
* [Qbicc](https://github.com/qbicc/qbicc)：Qbicc是一个实验性的Java原生镜像编译器。
* [OpenLDK](https://github.com/atgreen/openldk)：OpenLDK是Java的JIT编译器和运行时环境，完全以Common Lisp实现。
* [Java Comment Preprocessor](https://github.com/raydac/java-comment-preprocessor)：具有类似C注释格式的计算机语言预处理器。
* [Apache Royale Compiler](https://github.com/apache/royale-compiler)：Royale编译器将ActionScript 3.0和MXML代码编译为SWF或JavaScript。

## 语言服务器

* [JDT Language Server](https://github.com/eclipse-jdtls/eclipse.jdt.ls)：JDT语言服务器是语言服务器协议的Java语言特定实现，可以与支持该协议的任何编辑器一起使用，为Java语言提供良好的支持。
* [Eclipse LSP4J](https://github.com/eclipse-lsp4j/lsp4j)：语言服务器协议的Java实现，旨在由用Java实现的工具和语言服务器使用。
* [Language Server](https://github.com/georgewfraser/java-language-server)：基于协议v3.0并使用Java编译器API实现的Java语言服务器。
* [Eclipse LemMinX](https://github.com/eclipse/lemminx)：语言服务器协议的XML语言特定实现，可以与支持该协议的任何编辑器一起使用，为XML语言提供良好的支持。
* [Groovy Language Server](https://github.com/GroovyLanguageServer/groovy-language-server)：Groovy的语言服务器。
* [Lsp4IntelliJ](https://github.com/ballerina-platform/lsp4intellij)：Lsp4IntelliJ是一个客户端库，为IntelliJ IDEA和其他Jetbrains IDE提供语言服务器支持。
* [Build Server Protocol](https://github.com/build-server-protocol/build-server-protocol)：该项目致力于改进语言服务器/编辑器和构建工具之间的集成，由Scala中心和JetBrains领导。
* [BSL Language Server](https://github.com/1c-syntax/bsl-language-server)：BSL语言服务器协议的实现。
* [Meghanada Server](https://github.com/mopemope/meghanada-server)：适用于你的编辑器的Java IDE服务器，为你最喜欢的文本编辑器提供类似Java IDE的功能。

## 数据库工具库

这里列出了常用数据库的依赖、工具等。

#### 数据库驱动

* [Postgresql](https://github.com/pgjdbc/pgjdbc)：Postgresql JDBC驱动程序。
* [PGJDBC NG](https://github.com/impossibl/pgjdbc-ng)：PostgreSQL的新JDBC驱动程序，旨在支持JDBC和Postgres的高级功能。
* [Postgresql R2DBC](https://github.com/pgjdbc/r2dbc-postgresql)：Postgresql R2DBC驱动程序。
* [PostgreSQL Async](https://github.com/mauricio/postgresql-async)：用Scala编写的异步、基于Netty的PostgreSQL和MySQL数据库驱动程序。
* [MySQL](https://github.com/mysql/mysql-connector-j)：MySQL JDBC驱动程序。
* [AWS MySQL JDBC](https://github.com/awslabs/aws-mysql-jdbc)：AWS MySQL Driver是一个使应用程序能够充分利用集群MySQL数据库功能的驱动程序。
* [AWS JDBC Driver](https://github.com/aws/aws-advanced-jdbc-wrapper)：该包装器是对现有JDBC驱动程序的补充，旨在扩展驱动程序的功能，使应用程序能够充分利用Amazon Aurora等集群数据库的功能。
* [Oracle](https://www.oracle.com/database/technologies/maven-central-guide.html)：Oracle JDBC驱动程序。
* [Oracle R2DBC](https://github.com/oracle/oracle-r2dbc)：Oracle数据库的R2DBC驱动程序。
* [SqlServer](https://github.com/microsoft/mssql-jdbc)：SqlServer JDBC驱动程序。
* [R2DBC MySQL](https://github.com/mirromutth/r2dbc-mysql)：该项目包含R2DBC SPI的MySQL实现。
* [R2DBC SPI](https://github.com/r2dbc/r2dbc-spi)：R2DBC实现的SPI。
* [R2DBC MySQL](https://github.com/asyncer-io/r2dbc-mysql)：该项目包含R2DBC SPI的MySQL实现。
* [R2DBC MSSQL](https://github.com/r2dbc/r2dbc-mssql)：使用TDS协议的SQLServer的R2DBC驱动程序。
* [MariaDB R2DBC](https://github.com/mariadb-corporation/mariadb-connector-r2dbc)：非阻塞MariaDB和MySQL客户端。
* [Influx4j](https://github.com/brettwooldridge/influx4j)：适用于InfluxDB的高性能、零垃圾Java客户端/驱动程序。
* [OceanBase Client](https://github.com/oceanbase/obconnector-j)：兼容JDBC 4.2的OceanBase Java驱动程序。
* [SQLite JDBC](https://github.com/xerial/sqlite-jdbc)：用于在Java中访问和创建SQLite数据库文件的库。
* [Couchbase](https://github.com/couchbase/couchbase-java-client)：Couchbase Server的官方Java客户端。
* [InfluxDB Java](https://github.com/influxdata/influxdb-java)：InfluxDB的官方Java客户端库。
* [InfluxDB2 Java](https://github.com/influxdata/influxdb-client-java)：适用于JVM的InfluxDB 2客户端。
* [Neo4j Java Driver](https://github.com/neo4j/neo4j-java-driver)：Neo4j的官方Java驱动程序。
* [RxJava JDBC](https://github.com/davidmoten/rxjava2-jdbc)：RxJava 2与JDBC集成，包括非阻塞连接池。
* [Reactive SQL Client](https://github.com/eclipse-vertx/vertx-sql-client)：用Java编写的高性能响应式SQL客户端。
* [Hibernate Reactive](https://github.com/hibernate/hibernate-reactive)：Hibernate ORM的响应式API，支持非阻塞数据库驱动程序以及与数据库的响应式交互。
* [Snowflake JDBC Driver](https://github.com/snowflakedb/snowflake-jdbc)：Snowflake JDBC驱动程序。
* [JAsync-SQL](https://github.com/jasync-sql/jasync-sql)：JAsync-SQL是一个使用Kotlin编写的简单、基于Netty、异步、高性能且可靠的PostgreSQL和MySQL数据库驱动程序。
* [TiKV Java](https://github.com/tikv/client-java)：TiKV的Java客户端库。
* [VtDriver](https://github.com/jd-opensource/vtdriver)：VtDriver是一套基于分布式数据库Vitess而开发的Vitess Java客户端解决方案，由京东开源。
* [Aerospike Java Client](https://github.com/aerospike/aerospike-client-java)：Aerospike数据库的Java客户端库。
* [OpenGauss JDBC](https://gitee.com/opengauss/openGauss-connector-jdbc)：OpenGauss JDBC驱动程序。
* [Jackcess](https://github.com/jahlborn/jackcess)：Jackcess是一个纯Java库，用于读取和写入MS Access数据库。
* [Dgraph4j](https://github.com/dgraph-io/dgraph4j)：Java 1.8及更高版本的Dgraph客户端的最小实现。
* [Nebula Java](https://github.com/vesoft-inc/nebula-java)：Nebula Graph的Java客户端和数据导入器。

#### 数据库迁移

* [Liquibase](https://github.com/liquibase/liquibase)：Liquibase是一种数据库模式变更管理解决方案，使你能够更快、更安全地修改和发布从开发到生产的数据库变更。
* [Flyway](https://github.com/flyway/flyway)：Flyway是一款开源的数据库版本管理工具，它更倾向于规约优于配置的方式。
* [Obevo](https://github.com/goldmansachs/obevo)：Obevo是一种数据库部署工具，可处理企业规模的架构和复杂性，由高盛银行开源。
* [Mybatis Migrations](https://github.com/mybatis/migrations)：命令行数据库迁移工具。
* [Mongolastic](https://github.com/ozlerhakan/mongolastic)：Mongolastic使你能够将数据集从MongoDB节点迁移到ElasticSearch节点，反之亦然。
* [Datafall](https://github.com/forcedotcom/Data-Migration-Tool)：Datafall是一种将数据从一个Salesforce组织迁移到另一个Salesforce组织的工具。
* [Neo4j-Migrations](https://github.com/michael-simons/neo4j-migrations)：Neo4j-Migrations是一种数据库迁移和重构工具，允许以受控且可重复的方式针对一个或多个Neo4j数据库运行Cypher脚本和编程重构。
* [COS Migration](https://github.com/tencentyun/cos_migrate_tool_v5)：COS Migration是一个集成了COS数据迁移功能的一体化工具，由腾讯开源。
* [OSSImport](https://help.aliyun.com/zh/data-online-migration/user-guide/ossimport-overview)：OSSImport是阿里的一款将数据迁移至OSS的商业工具。
* [Celesta](https://github.com/CourseOrchestra/celesta)：Java的数据库迁移、SQL和测试工具。
* [Elasticsearch Evolution](https://github.com/senacor/elasticsearch-evolution)：用于迁移Elasticsearch映射的库。
* [Migrate2Postgres](https://github.com/isapir/Migrate2Postgres)：该工具允许你轻松地将数据库从其他兼容JDBC的DBMS迁移到Postgres。
* [Couchmove](https://github.com/tchlyah/couchmove)：Couchmove是Couchbase的开源Java迁移工具。
* [Mongock](https://github.com/mongock/mongock)：Mongock是一个基于Java的迁移工具，作为应用程序代码的一部分。
* [Solidbase](https://github.com/gitbucket/solidbase)：基于Liquibase的RDBMS和其他资源的通用迁移工具。
* [R2DBC Migration](https://github.com/nkonev/r2dbc-migrate)：R2DBC数据库迁移库。

#### 数据源增强

* [Dynamic DataSource](https://github.com/baomidou/dynamic-datasource)：Dynamic DataSource是一个基于Spring Boot的快速集成多数据源的Starter。
* [DataSource Proxy](https://github.com/jdbc-observations/datasource-proxy)：DataSource Proxy提供简单的API来拦截JDBC交互，并允许用户在查询或方法执行之前/之后执行自己的逻辑。
* [Spring Boot Dynamic DataSource](https://github.com/helloworlde/SpringBoot-DynamicDataSource)：Spring Boot多数据源、动态数据源配置。

#### 数据库工具

* [Screw](https://gitee.com/leshalv/screw)：Screw是一个简洁好用的数据库表结构文档生成器。
* [APGDiff](https://github.com/fordfrog/apgdiff)：APGDiff是免费的PostgreSQL diff工具，可用于比较/差异数据库模式。
* [Databasir](https://github.com/vran-dev/databasir)：Databasir是面向团队的关系型数据库模型文档管理平台。
* [Databench-T](https://gitee.com/caict-bigdata/databench-t)：Databench-T是面向金融核心业务系统场景的事务型数据库性能测试工具，由中国信通院云计算与大数据研究所联合北京银行、建设银行等企业共同设计开发。
* [SQL Formatter](https://github.com/vertical-blank/sql-formatter)：仅依赖Java标准库的SQL格式化程序。
* [SQL Relay](https://sqlrelay.sourceforge.net/)：SQL Relay是一个数据库代理和数据库连接管理解决方案。
* [ConceptBase](https://conceptbase.sourceforge.net/)：ConceptBase是一个多用户演绎数据库系统，具有面向对象的数据模型和无限的分类级别，使其成为元建模和定制建模语言工程的强大工具，由斯科夫德大学和亚琛大学开发。
* [Hypernomicon](https://github.com/jasonwinning/hypernomicon)：Hypernomicon是一款面向研究人员的个人生产力/数据库应用程序，它将结构化笔记记录、思维导图、文件(如PDF)和文件夹管理以及参考文献管理整合到一个集成环境中。

#### 存储过程

* [PL/Java](https://github.com/tada/pljava)：PL/Java是一个免费的附加模块，它将Java存储过程、触发器和函数引入PostgreSQL后端。
* [SPAN](https://github.com/americanexpress/SPAN)：SPAN是一个Java框架，它可以帮助开发人员通过提供配置和POJO详细信息来连接存储过程，由美国运通开源。

#### Redis库/工具

* [Redisson](https://github.com/redisson/redisson)：Redisson是一个具有内存数据网格功能的Redis Java客户端。
* [Jedis](https://github.com/redis/jedis)：Jedis是Redis的Java客户端，旨在提高性能和易用性。
* [Tedis](https://github.com/justified/tedis)：Tedis是另一个Redis的Java客户端，阿里开源。
* [Lettuce](https://github.com/lettuce-io/lettuce-core)：Lettuce是一个可扩展的线程安全Redis客户端，适用于同步、异步和响应式使用。
* [Spring Data Redis](https://github.com/spring-projects/spring-data-redis)：Spring Data Redis提供从Spring应用程序轻松配置和访问Redis的功能。
* [RedisCache](https://gitee.com/darkidiot/RedisCache)：RedisCache是基于Jedis的SDK。
* [JRedis](https://github.com/alphazero/jredis)：JRedis是Redis的Java客户端和连接器。
* [Redis Protocol](https://github.com/spullara/redis-protocol)：RedisClient是Redis的Java客户端和服务端实现。
* [Carmine](https://github.com/taoensso/carmine)：Carmine是Clojure的成熟Redis客户端，提供惯用的Clojure API，速度快、功能强大且易于使用。
* [Jesque](https://github.com/gresrun/jesque)：Jesque是Resque在Java中的实现。
* [JOhm](https://github.com/xetorthio/johm)：JOhm是一个速度超快的Java对象哈希映射库。
* [RedisScala](https://github.com/etaty/rediscala)：RedisScala是具有非阻塞和异步I/O操作的Scala Redis客户端。
* [Valkey GLIDE](https://github.com/valkey-io/valkey-glide)：Valkey GLIDE是一个开源Valkey客户端库，Valkey是Redis的开源fork版本，由AWS开源。
* [JRedisJSON](https://github.com/RedisJSON/JRedisJSON)：Redis RedisJSON的Java客户端。
* [Redis OM Spring](https://github.com/redis/redis-om-spring)：Redis OM Spring扩展了Spring Data Redis，以充分利用Redis和Redis Stack。
* [RedisClient](https://github.com/caoxinyu/RedisClient)：RedisClient是一个基于Java SWT和Jedis编写的Redis客户端GUI工具。
* [Redis Admin](https://github.com/mauersu/redis-admin)：Redis Admin是一个基于Java EE和Jedis编写的Redis客户端Web工具。
* [Redis Desktop Client](https://gitee.com/RedisDesktopClient/redis-desktop-client)：Redis Desktop Client是一款颜值较高、使用方便的redis客户端工具。
* [RedisPlus](https://gitee.com/MaxBill/RedisPlus)：RedisPlus是为Redis可视化管理开发的一款开源免费的桌面客户端软件。
* [Redis Admin](https://gitee.com/xuebusi/redis-admin)：Redis Admin是一个简单好用的Redis缓存图形化管理工具，包含Redis的5种数据类型的CRUD操作。
* [RedisFront](https://gitee.com/dromara/RedisFront)：RedisFront是一款开源跨平台Redis桌面客户端工具，支持单机模式、集群模式、哨兵模式以及SSH隧道连接，由dromara社区开源。
* [Redis Manager](https://github.com/ngbdf/redis-manager)：Redis Manager是Redis一站式管理平台，支持集群的监控、安装、管理、告警以及基本的数据操作功能。
* [XPipe](https://github.com/ctripcorp/x-pipe)：X-Pipe是由携程框架部门研发的Redis多数据中心复制管理系统。
* [CacheCloud](https://github.com/sohutv/cachecloud)：CacheCloud是一个Redis云管理平台，支持Redis多种架构高效管理、有效降低大规模Redis运维成本，提升资源管控能力和利用率，由搜狐开源。
* [Cymbal](https://github.com/dangdangdotcom/cymbal)：Cymbal是当当开源的Redis PaaS平台，目标是帮助技术团队以简单、低成本的方式管理大规模Redis集群。
* [RCT](https://github.com/xaecbd/RCT)：RCT是通过解析RDB文件进行Redis内存结构分析的一站式平台。
* [Redis RDB CLI](https://github.com/leonchen83/redis-rdb-cli)：Redis RDB CLI是一个可以解析、过滤、拆分、合并RDB以及离线分析内存使用情况的工具。
* [RIOT](https://github.com/redis/riot)：RIOT是一个命令行实用程序，旨在帮助你将数据输入和输出Redis。

#### Kafka库/工具

* [CMAK](https://github.com/yahoo/CMAK)：CMAK是用于管理Kafka集群的工具，由Yahoo开源。
* [Kafka-UI](https://github.com/provectus/kafka-ui)：用于管理Kafka集群的多功能、快速且轻量级的Web UI。
* [KafkaUI Lite](https://gitee.com/freakchicken/kafka-ui-lite)：非常好用的Kafka UI客户端工具，同时支持Zookeeper、Redis。
* [uReplicator](https://github.com/uber/uReplicator)：uReplicator提供了高性能、可扩展、稳定的Kafka复制解决方案，由Uber开源。
* [AKHQ](https://github.com/tchiotludo/akhq)：用于Kafka的Kafka GUI，可以管理主题、主题数据、消费者组、模式注册表、连接等等。
* [Kouncil](https://github.com/Consdata/kouncil)：Kouncil让你可以使用现代Web界面监控和管理Apache Kafka集群。
* [Offset Explorer](https://www.kafkatool.com/index.html)：Offset Explorer是一个用于管理和使用Apache Kafka集群的GUI应用程序。
* [Confluent Platform](https://www.confluent.io/en-gb/product/confluent-platform/gui-driven-management-and-monitoring/)：Confluent Platform提供直观的GUI来管理和监控Apache Kafka，由Confluent开发。
* [Kpow](https://factorhouse.io/kpow/)：Kpow是Apache Kafka的监控和管理工具。
* [Lenses](https://lenses.io/)：Lenses是开发人员体验工具，可帮助企业在一个地方使用每个Apache Kafka。
* [KafkaEsque](https://github.com/patschuh/KafkaEsque)：KafkaEsque是一个用JavaFX开发的Kafka GUI工具。
* [KnowStreaming](https://github.com/didi/KnowStreaming)：Know Streaming是一套云原生的Kafka管控平台，脱胎于众多互联网内部多年的Kafka运营实践经验，专注于Kafka运维管控、监控告警、资源治理、多活容灾等核心场景，由滴滴开源。
* [EFAK](https://github.com/smartloli/EFAK)：EAFK是一个开源的Kafka集群管理和监控工具，旨在帮助用户更好地管理和监控其Kafka集群。
* [KafkIO](https://kafkio.com/)：面向工程师和管理员的快速、简便的Apache Kafka GUI，适用于macOS、Windows和Linux，由Certak公司维护。
* [Kafka WebView](https://github.com/SourceLabOrg/kafka-webview)：Kafka WebView提供了一个易于使用的基于Web的界面，用于从Kafka主题中读取数据并提供基本的过滤和搜索功能。
* [Cruise Control](https://github.com/linkedin/cruise-control)：Cruise Control是一款帮助大规模运行Kafka集群的产品，由LinkedIn开源。
* [KCenter](https://github.com/xaecbd/KCenter)：KCenter是一个统一的Kafka集群管理维护、生产者/消费者监控、生态组件使用的一站式平台。
* [Kafka Map](https://github.com/dushixiang/kafka-map)：Kafka Map是使用Java 17和React开发的Kafka可视化工具。
* [Kstreamplify](https://github.com/michelin/kstreamplify)：Kstreamplify是一个Java库，使你能够快速创建基于Kafka Stream的应用程序，并提供许多附加高级功能，由米其林开源。
* [Kafka-Sprout](https://github.com/oslabs-beta/Kafka-Sprout)：Kafka Sprout是一个Web GUI，可以帮助你在本地计算机上快速启动Zookeeper和Kafka服务器，无需任何代码配置。
* [Xinfra Monitor](https://github.com/linkedin/kafka-monitor)：Xinfra Monitor是一个在真实集群中实现和执行长时间运行的kafka系统测试的框架，由LinkedIn开源。
* [Confluent Schema Registry](https://github.com/confluentinc/schema-registry)：Kafka的Confluence模式注册中心。
* [Kafdrop](https://github.com/HomeAdvisor/Kafdrop)：Kafdrop是一个用于监控Kafka集群的UI。
* [Mirus](https://github.com/salesforce/mirus)：Mirus是Kafka的跨数据中心数据复制工具，由Salesforce开源。
* [Kafdrop](https://github.com/obsidiandynamics/kafdrop)：Kafdrop是一个用于查看Kafka主题和浏览消费者组的Web UI。
* [Jikkou](https://github.com/streamthoughts/jikkou)：Jikkou是一款开源工具，旨在提供一种高效且简单的方法来管理、自动化和配置事件流平台上的资源。
* [Strimzi](https://github.com/strimzi/strimzi-kafka-operator)：Strimzi提供了一种在Kubernetes或OpenShift上以各种部署配置运行Kafka集群的方法，由RedHat开源。
* [Julie](https://github.com/kafka-ops/julie)：JulieOps帮助你自动化管理Kafka中的事物，从主题、配置到元数据，以及访问控制、模式。
* [Decaton](https://github.com/line/decaton)：Decaton是一个构建在Kafka之上的流式任务处理框架，由Line开源。
* [Kafka REST Proxy](https://github.com/confluentinc/kafka-rest)：Kafka REST Proxy为Kafka集群提供RESTful接口，它可以轻松地生成和消费数据、查看集群状态以及执行管理操作，而无需使用本机Kafka协议或客户端，由Confluent开源。
* [Reactor Kafka](https://github.com/reactor/reactor-kafka)：Reactor响应式Kafka驱动程序。
* [DoctorK](https://github.com/pinterest/DoctorK)：DoctorK是一个用于Kafka集群自动修复和工作负载均衡的服务，由Pinterest开源。
* [Kroxylicious](https://github.com/kroxylicious/kroxylicious)：Kroxylicious是Kafka协议代理，可解决加密、多租户和模式验证等用例。
* [Zilla](https://github.com/aklivity/zilla)：Zilla将Kafka抽象为Web应用程序、物联网客户端和微服务。
* [Chaperone](https://github.com/uber-archive/chaperone)：Chaperone作为Kafka审计系统，监控数据流的完整性和延迟，由Uber开源。
* [Azkarra Streams](https://github.com/streamthoughts/azkarra-streams)：Azkarra Streams是一个轻量级Java框架，可以轻松开发和操作Kafka Streams应用程序。
* [Kafka-Helmsman](https://github.com/teslamotors/kafka-helmsman)：Kafka-Helmsman是一个专注于自动化Kafka部署的工具，由特斯拉开源。
* [Kafbat UI](https://github.com/kafbat/kafka-ui)：Kafbat UI是一个免费的开源Web UI，用于监控和管理Kafka集群。
* [Klaw](https://github.com/Aiven-Open/klaw)：Klaw是一个自助式Kafka主题管理/治理工具/门户，由Aiven开源。
* [Astraea](https://github.com/opensource4you/astraea)：Astraea提供各式工具来降低Kafka使用门槛以及提高Kafka效能和资源使用率。
* [Kafka Web Console](https://github.com/cjmamo/kafka-web-console)：Kafka Web Console是一个用于监控Apache Kafka的Java Web应用程序。
* [Kafka Offset Monitor](https://github.com/Morningstar/kafka-offset-monitor)：这是一个用于监控Kafka消费者及其在日志中的位置(偏移量)的应用程序。

#### MongoDB库/工具

* [MongoDB](https://github.com/mongodb/mongo-java-driver)：适用于Java、Kotlin和Scala的官方MongoDB驱动程序。
* [Spring Data MongoDB](https://github.com/spring-projects/spring-data-mongodb)：Spring Data MongoDB项目旨在为新数据存储提供熟悉且一致的基于Spring的编程模型，同时保留特定于存储的特性和功能。
* [ReactiveMongo](https://github.com/ReactiveMongo/ReactiveMongo)：ReactiveMongo是一个Scala驱动程序，提供完全非阻塞和异步I/O操作。
* [Mars](https://github.com/whaleal/mars)：Mars是用于Java的MongoDB ORM/ODM框架，由上海锦木信息技术有限公司与中国东方航空公司共同开发。
* [Jongo](https://github.com/bguerout/jongo)：Jongo是Mongo查询语言可在Java中使用。
* [Morphia](https://github.com/MorphiaOrg/morphia)：Morphia是基于Java的MongoDB对象-文档映射器。
* [Mongojack](https://github.com/mongojack/mongojack)：Mongojack将Java对象映射到MongoDB文档。
* [MongoPlus](https://gitee.com/aizuda/mongo-plus)：MongoPlus可以使用MyBatisPlus的方式优雅的操作MongoDB，由爱组搭开源。
* [Variety](https://github.com/variety/variety)：Variety是MongoDB的模式分析器。
* [UMongo](https://github.com/agirbal/umongo)：UMongo是用于浏览和管理MongoDB集群的桌面应用程序。
* [Studio 3T](https://studio3t.com/)：Studio 3T是MongoDB的专业图形用户界面。
* [MongoBee](https://github.com/mongobee/mongobee)：MongoBee是一个Java工具，可帮助你管理MongoDB中的更改并将其与你的应用程序同步。
* [Mongo Lambda Query](https://github.com/DarMi7/mongo-lambda-query)：基于Lambda表达式，且面向对象的Mongo数据库查询插件。
* [MongoHelper](https://gitee.com/cym1102/mongoHelper)：Spring Data MongoDB增强工具包，简化CRUD操作，提供类Mybatis Plus的数据库操作体验。

#### Cassandra库/工具

* [Cassandra Java Driver](https://github.com/apache/cassandra-java-driver)：Cassandra的Java驱动程序。
* [DataStax Java Driver](https://github.com/datastax/java-driver)：适用于Cassandra的DataStax Java驱动程序。
* [Phantom](https://github.com/outworkers/phantom)：Phantom是适用于Cassandra/Datastax Enterprise的响应式类型安全Scala驱动程序。
* [Astyanax](https://github.com/Netflix/astyanax)：Astyanax是Netflix开源的Cassandra Java客户端库。
* [Spring Data Cassandra](https://github.com/spring-projects/spring-data-cassandra)：Spring Data Cassandra为Cassandra提供Spring Data模块熟悉的接口。
* [Hector](https://github.com/hector-client/hector)：Hector是Cassandra的高级客户端库。
* [Cassie](https://github.com/twitter-archive/cassie)：Cassie是一个小型、轻量级的Cassandra客户端，基于Finagle构建，由Twitter开源。
* [Priam](https://github.com/Netflix/Priam)：Priam是一个与Cassandra一起运行的流程/工具，由Netflix开源。
* [Stargate](https://github.com/stargate/stargate)：Stargate是部署在客户端应用程序和Cassandra数据库之间的数据网关，DataStax开源。
* [Cassandra Reaper](https://github.com/thelastpickle/cassandra-reaper)：Reaper是一种集中式、有状态且高度可配置的工具，用于针对单站点或多站点集群运行Cassandra修复，由DataStax开源。
* [Aegisthus](https://github.com/Netflix/aegisthus)：Aegisthus是Cassandra的批量数据管道，由Netflix开源。
* [Cassandra Schema Migration](https://github.com/patka/cassandra-migration)：该库可用于在Java应用程序内实现Cassandra数据库模式的迁移。
* [Cassandra Migration](https://github.com/smartcat-labs/cassandra-migration-tool-java)：适用于Java的Cassandra模式迁移工具。
* [Cassandra Migration](https://github.com/Contrast-Security-OSS/cassandra-migration)：Cassandra Migration是一个简单且轻量级的Cassandra数据库迁移工具，基于Flyway项目，由Contrast开源。
* [CQLMigrate](https://github.com/sky-uk/cqlmigrate)：CQLMigrate是一个用于在Cassandra集群上执行模式迁移的库，由英国天空公司开源。

#### Memcached库/工具

* [Folsom](https://github.com/spotify/folsom)：Folsom是Java的异步Memcached客户端库，由Spotify开源。
* [XMemcached](https://github.com/killme2008/xmemcached)：XMemcached是一个高性能、易于使用的Java阻塞多线程Memcached客户端。
* [Spymemcached](https://github.com/dustin/java-memcached-client)：Spymemcached是一个用Java编写的简单、异步、单线程Memcached客户端。

#### Zookeeper库/工具

* [Apache Curator](https://github.com/apache/curator)：Curator是ZooKeeper的Java/JVM客户端库，由Netflix开源。
* [ZkClient](https://github.com/sgroschupf/zkclient)：ZkClient是Zookeeper的客户端库，由Datameer开源。
* [ZKClient](https://github.com/adyliu/zkclient)：一个简单有效的Zookeeper Java客户端，由搜狐维护。
* [PrettyZoo](https://github.com/vran-dev/PrettyZoo)：PrettyZoo是由JavaFX和Apache Curator创建的Zookeeper GUI。
* [ZkUI](https://github.com/DeemOpen/zkui)：ZkUI是允许在Zookeeper上进行CRUD操作的UI仪表板。
* [Exhibitor](https://github.com/soabase/exhibitor)：ZooKeeper协同处理实例，例如监控、备份/恢复、清理和可视化，由Netflix开源。
* [Taokeeper](https://github.com/alibaba/taokeeper)：Taokeeper是Zookeeper的监视器，由阿里开源。
* [Shepher](https://github.com/XiaoMi/shepher)：Shepher是ZooKeeper的管理工具，在小米作为配置管理中心使用。
* [KafkaUI Lite](https://gitee.com/freakchicken/kafka-ui-lite)：非常好用的Kafka UI客户端工具，同时支持Zookeeper、Redis。
* [Zookeeper Visualizer](https://github.com/xin497668869/zookeeper-visualizer)：Zookeeper的可视化管理工具。

#### ClickHouse库/工具

* [ClickHouse Java](https://github.com/ClickHouse/clickhouse-java)：用于连接ClickHouse并处理各种格式数据的Java库。
* [Clickhouse4j](https://github.com/Blynk-Technologies/clickhouse4j)：Clickhouse4j是官方ClickHouse JDBC驱动程序的更轻更快的替代品。
* [ClickHouse Native JDBC](https://github.com/housepower/ClickHouse-Native-JDBC)：用于在Java 中访问ClickHouse的原生JDBC库，还提供用于与Spark集成的库。
* [Clickhouse Scala](https://github.com/crobox/clickhouse-scala-client)：Clickhouse Scala客户端，以响应式方式访问Clickhouse数据库。
* [CKibana](https://github.com/TongchengOpenSource/ckibana)：CKibana是一项使用原生Kibana方便分析ClickHouse数据的服务，由同程旅行开源。
* [Graphhouse](https://github.com/ClickHouse/graphouse)：Graphhouse允许你使用ClickHouse作为Graphite存储。
* [ClickHouse Client](https://github.com/Ecwid/clickhouse-client)：ClickHouse的Java/Kotlin客户端。

#### ElasticSearch库/工具

* [ElasticSearch Java](https://github.com/elastic/elasticsearch-java)：ElasticSearch官方Java客户端。
* [Jest](https://github.com/searchbox-io/Jest)：Jest是ElasticSearch的Java HTTP REST客户端。
* [Bboss](https://github.com/bbossgroups/bboss-elasticsearch)：Bboss是一个很好的ElasticSearch Java REST客户端，它操作和访问ElasticSearch的方式与MyBatis类似。
* [Flummi](https://github.com/otto-de/flummi)：Flummi是ElasticSearch的客户端库，提供了全面的Java查询DSL API，并通过HTTP/JSON与ElasticSearch集群进行通信。
* [Pallas](https://github.com/vipshop/pallas)：Pallas是唯品会的统一搜索平台，建立在ElasticSearch之上，旨在解决各种搜索问题。
* [Easy ES](https://gitee.com/dromara/easy-es)：Easy ES是一款简化ElasticSearch搜索引擎操作的开源框架，全自动智能索引托管，由dromara社区开源。
* [Spring Data Jest](https://github.com/VanRoy/spring-data-jest)：基于Jest Rest客户端的ElasticSearch的Spring Data实现。
* [Spring Data Elasticsearch](https://github.com/spring-projects/spring-data-elasticsearch)：Spring Data Elasticsearch项目提供与Elasticsearch搜索引擎的集成。
* [ESClientRHL](https://gitee.com/zxporz/ESClientRHL)：EsClientRHL是一个可基于Spring Boot的ElasticSearch客户端调用封装工具。
* [EsearchX](https://gitee.com/noear/esearchx)：EsearchX基于OkHttp、Snack3开发，是一个代码直白和简单的Elasticsearch ORM框架。
* [Elastic4s](https://github.com/Philippus/elastic4s)：Elastic4s是一个简洁、惯用、响应式、类型安全的Elasticsearch Scala客户端。
* [Ebatis](https://github.com/ymm-tech/ebatis)：Ebatis是一个声明式ElasticSearch ORM框架。
* [Querqy](https://github.com/querqy/querqy)：Querqy是一个在基于Java的搜索引擎中进行查询预处理的框架。
* [Anserini](https://github.com/castorini/anserini)：Anserini是一个用于可重复信息检索研究的Lucene工具包，由滑铁卢大学开源。
* [ES Fastloader](https://github.com/didi/ES-Fastloader)：ES Fastloader利用Hadoop的容错性和并行性，在多个reducer节点中构建单独的ElasticSearch分片，然后将分片传输到ElasticSearch集群进行服务，由滴滴开源。
* [Elasticsearch JDBC](https://github.com/jprante/elasticsearch-jdbc)：JDBC导入器允许从JDBC源获取数据以索引到Elasticsearch中。
* [Elasticsearch SQL](https://github.com/NLPchina/elasticsearch-sql)：使用此插件，你可以使用熟悉的SQL语法查询Elasticsearch。
* [Raigad](https://github.com/Netflix/Raigad)：Raigad是一个用于ElasticSearch备份/恢复、自动部署和集中配置管理的协同进程，由Netflix开源。
* [KnowSearch](https://github.com/didi/KnowSearch)：KnowSearch是面向Elasticsearch研发与运维人员，围绕集群、索引构建的零侵入、多租户的Elasticsearch GUI管控平台，由滴滴开源。

#### DynamoDB库/工具

* [Jcabi DynamoDB](https://github.com/jcabi/jcabi-dynamo)：AWS DynamoDB SDK的面向对象包装器。
* [Tempest](https://github.com/cashapp/tempest)：适用于Kotlin和Java的类型安全DynamoDB。
* [DynamoDB Transactions](https://github.com/awslabs/dynamodb-transactions)：DynamoDB Transactions在DynamoDB中提供原子性、一致性、隔离性和持久性(ACID)，使你能够更轻松地维护应用程序中的数据正确性。
* [DynamoDB Geo](https://github.com/amazon-archives/dynamodb-geo)：Amazon DynamoDB的地理库使Java开发人员能够轻松创建和查询地理空间数据。
* [Spring Data DynamoDB](https://github.com/michaellavelle/spring-data-dynamodb)：该模块提供对基于AWS DynamoDB构建的数据访问层的增强支持。
* [Amazon DynamoDB Encryption Java](https://github.com/aws/aws-dynamodb-encryption-java)：适用于Java的Amazon DynamoDB加密客户端。
* [Dynamoit](https://github.com/bykka/dynamoit)：这是一个用JavaFX编写的简单的DynamoDB图形客户端。

#### Milvus库/工具

* [Milvus Java SDK](https://github.com/milvus-io/milvus-sdk-java)：Milvus的Java SDK。
* [MilvusPlus](https://gitee.com/dromara/MilvusPlus)：MilvusPlus是一个功能强大的Java库，旨在简化与Milvus向量数据库的交互，为开发者提供类似MyBatis Plus注解和方法调用风格的直观API，由dromara社区开源。

## Pub/Sub库

* [Java Pub/Sub](https://github.com/googleapis/java-pubsub)：Google Cloud Pub/Sub的Java惯用客户端。
* [Google Pub/Sub](https://github.com/GoogleCloudPlatform/pubsub)：Google Cloud Pub/Sub开源项目。
* [PSC](https://github.com/pinterest/psc)：PSC是一个通用且可扩展的客户端库，允许应用程序通过统一的接口与不同的后端PubSub系统进行交互，由Pinterest开发。
* [Async Google Pub/Sub Client](https://github.com/spotify/async-google-pubsub-client)：一个低级别的Pub/Sub客户端和一个并发的每主题批处理发布者，由Spotify开发。

## 对象存储

* [Aliyun OSS](https://github.com/aliyun/aliyun-oss-java-sdk)：Aliyun OSS的Java SDK。
* [Aliyun OSS Android](https://github.com/aliyun/aliyun-oss-android-sdk)：阿里云对象存储服务Android SDK。
* [Ambry](https://github.com/linkedin/ambry)：Ambry是一个分布式对象存储，支持存储数万亿个小型不可变对象(50K-100K)以及数十亿个大型对象，由LinkedIn开发。
* [MinIO](https://github.com/minio/minio-java)：用于Java的MinIO客户端SDK。
* [X File Storage](https://gitee.com/dromara/x-file-storage)：将文件存储到各种云平台的Spring Boot库。
* [OSS Spring Boot](https://github.com/pig-mesh/oss-spring-boot-starter)：兼容S3协议的通用文件存储工具类。
* [Syncany](https://github.com/syncany/syncany)：Syncany是一款云存储和文件共享应用程序，重点关注存储的安全性和抽象性。
* [Qiniu Resource Storage SDK](https://github.com/qiniu/java-sdk)：七牛资源存储Java SDK。
* [KSAN](https://github.com/infinistor/ksan)：KSAN是一个软件定义的对象存储系统，旨在可靠、高效地提供大规模对象存储服务。
* [Huawei OBS](https://github.com/huaweicloud/huaweicloud-sdk-java-obs)：用于访问对象存储服务的OBS Java SDK。
* [BlobIt](https://github.com/diennea/blobit)：BlobIt是基于Apache BookKeeper构建的分布式二进制大对象(BLOB)存储。
* [ByteBin](https://github.com/lucko/bytebin)：ByteBin是一种快速、轻量级的内容存储网络服务。
* [Dante OSS](https://gitee.com/herodotus/dante-oss)：Dante OSS是一款简化对象存储操作的开源框架。

## 音视频处理

* [Jitsi](https://github.com/jitsi/jitsi)：Jitsi Desktop是一款免费的开源音频/视频和聊天通信器，支持SIP、XMPP/Jabber、IRC等协议和许多其他有用的功能。
* [Metadata Extractor](https://github.com/drewnoakes/metadata-extractor)：Metadata Extractor是一个用于从媒体文件中读取元数据的Java库。
* [Horizon](https://github.com/Yalantis/Horizon)：Horizon是适用于Android的简单视觉均衡器。
* [RxAndroidAudio](https://github.com/Piasy/RxAndroidAudio)：Android音频封装库，部分Rx支持。
* [Airsonic](https://github.com/airsonic/airsonic)：Airsonic是一款免费的基于Web的媒体服务器，可让你随时随地访问音乐。
* [TarsosDSP](https://github.com/JorenSix/TarsosDSP)：TarsosDSP是一个用于音频处理的Java库，其目的是为实用的音乐处理算法提供一个易于使用的接口。
* [FFmpeg CLI Wrapper](https://github.com/bramp/ffmpeg-cli-wrapper)：用于从Java运行FFmpeg的流式接口。
* [FFmpeg Android Java](https://github.com/cropsly/ffmpeg-android-java)：FFmpeg Android Java是一个Java库，简化了在Android项目中使用Ffmpeg的任务。
* [WaveInApp](https://github.com/Cleveroad/WaveInApp)：可以从任何来源(音频播放器、流、语音输入)获取音频，并以高帧速率为其制作动画。
* [Alexa Skills Kit Java SDK](https://github.com/alexa/alexa-skills-kit-sdk-for-java)：Alexa是Amazon基于云的语音服务，可在亚马逊和第三方设备制造商的数亿台设备上使用。
* [OpenAudible](https://github.com/openaudible/openaudible)：用于下载和管理Audible有声读物的跨平台桌面应用程序。
* [LiveBox](https://github.com/parzulpan/livebox)：LiveBox是一个跨平台的网络媒体聚合应用，支持直播视频、高清电视和广播电台的在线观看或收听。
* [Phon](https://github.com/phon-ca/phon)：Phon是一个软件程序，可以极大地促进与基于转录和声学测量的语音数据分析相关的许多任务，由纽芬兰纪念大学开源。
* [JAVE2](https://github.com/a-schild/jave2)：JAVE库是ffmpeg项目的Java包装器。
* [JCodec](https://github.com/jcodec/jcodec)：JCodec是视频/音频编解码器的纯Java实现。
* [Opencast](https://github.com/opencast/opencast)：Opencast是一个灵活、可靠、可扩展的开源视频管理系统，适用于学术机构，由来自全球领先大学和组织的开发人员社区构建。
* [LavaPlayer](https://github.com/sedmelluq/lavaplayer)：LavaPlayer是一个用Java编写的音频播放器库，它可以从各种源加载音轨并将其转换为Opus帧流，专为Discord机器人使用而设计。
* [Quick Media](https://github.com/liuyueyi/quick-media)：多媒体处理Web服务。
* [OmRecorder](https://github.com/kailash09dabhi/OmRecorder)：一个简单的Pcm/Wav录音机。
* [Sphinx-4](https://github.com/cmusphinx/sphinx4)：纯Java语音识别库，由美国卡内基梅隆大学开发。
* [Photon](https://github.com/Netflix/photon)：Photon是IMF标准的Java实现，由Netflix开源。
* [Concentus](https://github.com/lostromb/concentus)：Opus音频编解码器的纯可移植C#和Java实现。
* [WVP-GB28181](https://github.com/648540858/wvp-GB28181-pro)：Web Video Platform是一个基于GB28181-2016标准实现的开箱即用的网络视频平台，负责实现核心信令与设备管理后台部分，支持NAT穿透，支持海康、大华、宇视等品牌的IPC、NVR接入。
* [Minim](https://github.com/ddf/Minim)：一个Java音频库，设计用于与Processing一起使用。
* [Audiveris](https://github.com/Audiveris/audiveris)：Audiveris是一款开源光学音乐识别(OMR)软件，它用于扫描乐谱并将其转换为机器可读的格式，例如MusicXML或MIDI。
* [libjitsi](https://github.com/jitsi/libjitsi)：用于安全实时音频/视频通信的高级Java媒体库。
* [MP4 Parser](https://github.com/sannies/mp4parser)：用于读取、写入和创建MP4容器的Java API，操作容器与编码和解码视频和音频不同。
* [LiTr](https://github.com/linkedin/LiTr)：适用于Android的轻量级硬件加速视频/音频转码器，由LinkedIn开源。
* [AudioBookConverter](https://github.com/yermak/AudioBookConverter)：基于freeipod软件版本改进的AudioBookConverter(mp3到m4b转换器)。
* [Echoprint Server](https://github.com/spotify/echoprint-server)：Echoprint音频指纹系统服务器，由Spotify开源。
* [Monte Media Library](http://www.randelshofer.ch/monte/)：Monte Media Library是一个用于处理媒体数据的Java库，支持的媒体格式包括静态图像、视频、音频和元数据。
* [jPSXdec](https://github.com/m35/jpsxdec)：一款现代的跨平台PlayStation 1音频/视频转换器。
* [Beat-Link](https://github.com/Deep-Symmetry/beat-link)：一个Java库，用于与Pioneer DJ Link设备的节拍同步，并查找有关正在播放的曲目的详细信息。
* [JJazzLab](https://github.com/jjazzboss/JJazzLab)：JJazzLab是一个完整而开放的应用程序，专用于背景音乐生成。
* [MidiBus](https://github.com/sparks/themidibus)：MidiBus是一个用于处理的MIDI库，它提供了一种快速简便的方法来发送和接收MIDI数据。
* [Android MIDI Library](https://github.com/LeffelMania/android-midi-lib)：该项目主要适用于无法访问Java的javax.sound.midi库的Android应用程序。
* [Midica](https://github.com/truj/midica)：Midica是一种音乐编程语言的解释器，它将源代码翻译成MIDI。
* [Humble Video](https://github.com/artclarke/humble-video)：HumbleVideo允许JVM语言对音频和视频数据进行解码、分析/修改和编码为数百种不同格式(例如H264、AAC、MP3、FLV等)。
* [Supersonic](https://github.com/Mach5/supersonic)：基于网络的开源媒体流媒体和点唱机分支Subsonic，支持MP3、OGG、AAC等流媒体音视频格式。
* [Jave](https://github.com/dadiyang/jave)：音频转码工具，主要用于将微信语音amr格式转换为mp3格式以便在H5的audio标签中进行播放。
* [JSyn](https://github.com/philburk/jsyn)：Java模块化音频合成器。
* [JLayer](https://github.com/umjammer/jlayer)：为Java平台实时解码/播放/转换MPEG 1/2/2.5 Layer 1/2/3(即MP3)的库。
* [Xuggler](https://www.xuggle.com/xuggler/)：Xuggler是一个功能强大的工具，旨在简化直接从Java解压缩、修改和重新压缩任何媒体文件或流的过程。
* [Jaudiotagger](https://bitbucket.org/ijabz/jaudiotagger)：Jaudiotagger是音频标记库，用于标记音频文件中的数据。
* [MaryTTS](https://github.com/marytts/marytts)：MaryTTS是一个用纯Java编写的开源、多语言文本到语音合成系统。
* [Universal Media Server](https://github.com/UniversalMediaServer/UniversalMediaServer)：Universal Media Server是兼容DLNA的UPnP媒体服务器，它能够在大多数现代设备之间共享视频、音频和图像。
* [Smallville](https://github.com/nickm980/smallville)：生成代理是虚拟角色，可以存储记忆并对环境做出动态反应。
* [Rebound](https://github.com/facebookarchive/rebound)：Rebound是一个模拟弹簧动力学的Java库，由Facebook开源。
* [Vlcj](https://github.com/caprica/vlcj)：vlc媒体播放器的Java框架。
* [MP3agic](https://github.com/mpatric/mp3agic)：用于读取MP3文件和读取/操作ID3标签(ID3v1和ID3v2.2到ID3v2.4)的Java库。
* [Jaffree](https://github.com/kokorin/Jaffree)：Jaffree代表Java FFmpeg和FFprobe FREE命令行包装器。
* [Processing Video](https://github.com/processing/processing-video)：基于GStreamer的视频处理库。
* [M3U8-Parser](https://github.com/carlanton/m3u8-parser)：适用于Java的简单HLS播放列表解析器。
* [Panako](https://github.com/JorenSix/Panako)：Panako是一种声学指纹识别系统，该系统能够从音频流中提取指纹，并将这些指纹存储在数据库中，或者在提取的指纹和存储的指纹之间找到匹配。
* [JNAJack](https://github.com/jaudiolibs/jnajack)：JACK音频连接套件的Java绑定。
* [FFmpeg4j](https://github.com/Manevolent/ffmpeg4j)：FFmpeg4j是一个Java库，它封装了FFmpeg库的功能。
* [WhisperJNI](https://github.com/GiviMAD/whisper-jni)：whisper.cpp的JNI包装器，允许将语音转录为Java中的文本。
* [XYScope](https://github.com/ffd8/xyscope)：XYScope是一个处理库，用于通过将图形转换为音频来在矢量显示器(示波器、激光)上渲染图形。
* [SoundLibs](https://github.com/pdudits/soundlibs)：Java声音库的Maven工件。
* [RootEncoder](https://github.com/pedroSG94/RootEncoder)：RootEncoder是一个流编码器，使用RTMP、RTSP和SRT协议将视频/音频推送到媒体服务器。
* [FreeTTS](https://freetts.sourceforge.io/)：FreeTTS是一个完全用Java编程语言编写的语音合成系统。
* [LibSDL4J](https://github.com/libsdl4j/libsdl4j)：LibSDL4J是SDL2 API到Java的映射。
* [GStreamer 1.x Java Core](https://github.com/gstreamer-java/gst1-java-core)：这是GStreamer 1.x的一组Java绑定，GStreamer是一个用C语言编写的开源、基于管道的多媒体框架。
* [Video4j](https://github.com/metaloom/video4j)：Video4j是org.openpnp:opencv之上的高级库，它提供API在Java中处理视频媒体。
* [ZLM4J](https://gitee.com/aizuda/zlm4j)：本项目是对ZLMediaKit提供的C API的Java封装，由爱组搭开源。
* [V4L4j](https://code.google.com/archive/p/v4l4j/)：V4L4j是一个Java包，它提供了从Java简单访问Video4Linux(V4L) API捕获接口的功能。
* [FMJ](http://fmj-sf.net/)：FMJ是一个开源项目，其目标是提供Java媒体框架(JMF)的替代方案，同时保持与JMF的API兼容。
* [Tracker](https://github.com/OpenSourcePhysics/tracker)：基于开源物理(OSP)框架构建的视频分析和建模工具。
* [MWEngine](https://github.com/igorski/MWEngine)：适用于Android的音频引擎和DSP库，以C++编写，在音乐环境中提供低延迟性能，同时提供Java/Kotlin API。
* [JNA AAC Encoder](https://github.com/sheinbergon/jna-aac-encoder)：此库为JVM提供AAC编码功能。
* [JavaForce](https://github.com/pquiring/javaforce)：JavaForce库，用于构建强大的应用程序和服务(相机、OpenGL、OpenCL、ffmpeg的原生绑定)，包含VoIP协议栈、PLC I/O和众多应用程序。

## 数据结构

* [T-Digest](https://github.com/tdunning/t-digest)：一种新的数据结构，用于准确在线累积基于排名的统计数据，例如分位数和修剪平均值。
* [Bifurcan](https://github.com/lacuna/bifurcan)：该库提供了可变和不可变数据结构的高质量Java实现，每个实现都共享一个通用API。
* [Sux4J](https://github.com/vigna/Sux4J)：Sux4J提供了许多相关数据结构的实现，涵盖位数组、压缩列表和最小完美哈希函数的排名/选择，由米兰大学开发。
* [NetworkAnalysis](https://github.com/CWTSLeiden/networkanalysis)：该库提供了用于网络分析的算法和数据结构，专注于网络的聚类(或社区检测)和布局(或映射)，由莱顿大学开源。
* [Time-Utilities](https://github.com/Breinify/brein-time-utilities)：包含多个时间相关数据和索引结构(例如IntervalTree、BucketTimeSeries)以及算法的库。
* [Funcj](https://github.com/typemeta/funcj)：用于Java的面向函数的数据结构、算法和库的集合。
* [Athena](https://github.com/sanity/Athena)：支持任意布尔查询的高效内存数据结构。
* [HyperMinHash Java](https://github.com/LiveRamp/HyperMinHash-java)：用于计算对数空间中的并集、交集和集合基数的概率数据结构。
* [Dictomaton](https://github.com/danieldk/dictomaton)：该Java库实现存储在有限状态自动机中的字典。
* [ObjectLayout](https://github.com/ObjectLayout/ObjectLayout)：ObjectLayout提供了一组数据结构类，这些类的设计考虑了优化的内存布局。
* [Zero-Allocation Hashing](https://github.com/OpenHFT/Zero-Allocation-Hashing)：用于对Java中的任何字节序列进行哈希处理，包括各种原始数组、缓冲区、CharSequence等。

#### 树

* [SnapTree](https://github.com/nbronson/snaptree)：SnapTree是一种具有快速克隆、快照和一致迭代功能的并发AVL树，由斯坦福开源。
* [Darts Java](https://github.com/komiya-atsushi/darts-java)：Darts Java是Darts(双数组Trie系统)的Java移植。
* [RTree](https://github.com/davidmoten/rtree)：使用响应式API在Java中实现不可变的内存中R树和R*树。
* [Tree](https://github.com/Scalified/tree)：该库包含树数据结构的不同实现，例如K进制、二叉树、表达式树等。
* [RTree2](https://github.com/davidmoten/rtree2)：内存中不可变的二维R树实现。
* [BTree4j](https://github.com/myui/btree4j)：Btree4j是一个用纯Java编写的基于磁盘的B+树。
* [BPlusTree](https://github.com/andylamp/BPlusTree)：一种高效、简洁、简单的纯磁盘B+Tree数据结构实现。
* [Trie4J](https://github.com/takawitter/trie4j)：Trie4J是各种trie实现的排序集合。
* [PH-Tree](https://github.com/tzaeschke/phtree)：PH-Tree是一种多维索引和存储结构，默认情况下，它存储由k个64位整数组成的k维键(点)。
* [Suffix Tree](https://github.com/abahgat/suffixtree)：使用Ukkonen算法的广义后缀树的Java实现。
* [LSM-Tree](https://github.com/tomfran/LSM-Tree)：Java中日志结构合并树(LSM Tree)数据结构的实现。
* [TinSpin Indexes](https://github.com/tzaeschke/tinspin-indexes)：TinSpin Indexes是一个内存索引库。
* [PART](https://github.com/ankurdave/part)：PART是一种基于ART的具有高分支因子和自适应大小节点的字典树。

#### 堆

* [JHeaps](https://github.com/d-michail/jheaps)：JHeaps是一个免费库，提供各种用Java编写的堆实现。
* [Chronicle-Values](https://github.com/OpenHFT/Chronicle-Values)：通过接口生成Bean的堆上实现。

#### 图

* [Cassovary](https://github.com/twitter/cassovary)：Cassovary是一个简单的JVM大图处理库，由Twitter开源。
* [NetflixGraph](https://github.com/Netflix/netflix-graph)：NetflixGraph是一种紧凑的内存数据结构，用于表示有向图数据，由Netflix开源。
* [JGraLab](https://github.com/jgralab/jgralab)：JGraLab是一个Java图形库，实现了所谓的TGraphs：类型化、属性化、有序和有向图，由科布伦茨兰道大学开源。
* [Traverser](https://github.com/intuit/Traverser)：Traverser是一个Java库，可帮助软件工程师实现数据结构的高级迭代，由Intuit开发。

#### BitSet

* [SparseBitSet](https://github.com/brettwooldridge/SparseBitSet)：Java的高效稀疏位集实现。
* [RoaringBitmap](https://github.com/RoaringBitmap/RoaringBitmap)：Java中更好的压缩位集。
* [JavaEWAH](https://github.com/lemire/javaewah)：Java BitSet类的压缩替代方案。
* [Bit-Lib4j](https://github.com/devnied/Bit-lib4j)：Bit-Lib4j是一个用于在Java中处理字节或位的库。

#### 队列

* [Conversant](https://github.com/conversant/disruptor)：Conversant Disruptor是环形缓冲区中性能最高的实现，它几乎没有开销，并且采用了特别简单的设计。
* [CoralRing](https://github.com/coralblocks/CoralRing)：CoralRing是堆外共享内存中的超低延迟、无锁、无垃圾、批处理和并发循环队列(环)，用于使用内存映射文件跨不同JVM进行Java进程间通信。
* [CoralQueue](https://github.com/coralblocks/CoralQueue)：CoralQueue是一种超低延迟、高性能、无锁、无垃圾的并发队列、多路分解器、多路复用器、mpmc队列和拆分器。
* [Tape](https://github.com/square/tape)：Android和Java中与队列相关的类的集合，由Square开源。
* [Big Queue](https://github.com/bulldog2011/bigqueue)：基于内存映射文件的大、快速且持久的队列。
* [Low GC MemBuffers](https://github.com/cowtowncoder/low-gc-membuffers)：用于创建内存循环缓冲区的库，该缓冲区使用直接ByteBuffer来最大限度地减少GC开销。

#### Map

* [TinyMap](https://github.com/intelie/tinymap)：内存高效的不可变HashMap/HashSet。
* [CompactHashMap](https://github.com/vlsi/compactmap)：这是HashMap的内存高效替代方案。
* [PauselessHashMap](https://github.com/giltene/PauselessHashMap)：java.util.HashMap兼容的Map，在调整大小时不会停止put或get。
* [SmoothieMap](https://github.com/TimeAndSpaceIO/SmoothieMap)：SmoothieMap是Java的Map实现，具有最低的内存使用率并且不存在重哈希延迟峰值。
* [BigMap](https://github.com/fizzed/bigmap)：轻量级Map、SortedMap、LinkedMap、Set和SortedSet实现，可通过将任务卸载到磁盘来缓解内存压力。

#### List

* [GlueList](https://github.com/ertugrulcetin/GlueList)：GlueList是一个全新的List实现，它比ArrayList和LinkedList快得多。
* [Glazed Lists](https://github.com/glazedlists/glazedlists)：Java的开源List转换。

#### CRDT

* [Wurmloch CRDT](https://github.com/netopyr/wurmloch-crdt)：JVM无冲突复制数据类型(CRDT)的实验实现。
* [Java CRDT](https://github.com/ajantis/java-crdt)：Java中常见的无冲突复制数据类型集合。

#### 布隆过滤器

* [Orestes Bloomfilter](https://github.com/Baqend/Orestes-Bloomfilter)：Java中不同布隆过滤器的库，具有可选的Redis支持、计数和许多哈希选项。
* [InBloom](https://github.com/EverythingMe/inbloom)：跨语言布隆过滤器实现。
* [JRedisBloom](https://github.com/RedisBloom/JRedisBloom)：RedisBloom概率模块的Java客户端。
* [Bloom Filter Scala](https://github.com/alexandrnikitin/bloom-filter-scala)：Scala的布隆过滤器。
* [Greplin Bloom Filter](https://github.com/Cue/greplin-bloom-filter)：概率集合数据结构的Java实现。
* [PDD](https://github.com/jparkie/PDD)：基于高级布隆过滤器的算法，可在流中实现高效的近似数据去重复。
* [Minperf](https://github.com/thomasmueller/minperf)：极小的完美哈希函数库。
* [Bloofi](https://github.com/lemire/bloofi)：多维布隆过滤器的Java实现。

#### 布谷鸟过滤器

* [CuckooFilter4J](https://github.com/MGunlogson/CuckooFilter4J)：Cuckoo过滤器的高性能Java实现。
* [Setfilters](https://github.com/google/setfilters)：该仓库包含集合过滤器数据结构的集合的实现，通常也称为近似成员资格查询数据结构，由Google开源。
* [FastFilter](https://github.com/FastFilter/fastfilter_java)：Java中的快速近似成员资格过滤器。
## 基本类型

* [jOOU](https://github.com/jOOQ/jOOU)：jOOU为四种Java整数类型byte、short、int和long提供无符号整数版本。
* [Prim](https://github.com/mgormley/prim)：Prim是一个类似于Trove的Java原始类型库，重点关注向量和矩阵的稀疏表示，由约翰霍普金斯大学开发。
* [Primitive](https://github.com/mintern-java/primitive)：Primitive提供与基本类型相关的功能的实用方法，包括基于自定义比较器的排序和搜索。

## 随机数生成器

* [JNanoId](https://github.com/aventrix/jnanoid)：Java的唯一字符串ID生成器。
* [UUID-Creator](https://github.com/f4b6a3/uuid-creator)：这是一个用于生成通用唯一标识符的Java库。
* [ULID Creator](https://github.com/f4b6a3/ulid-creator)：这是一个用于生成ULID的Java库。
* [Apache Commons RNG](https://github.com/apache/commons-rng)：Commons RNG项目提供伪随机生成器的纯Java实现。
* [Java UUID Generator](https://github.com/cowtowncoder/java-uuid-generator)：JUG是一组用于处理UUID的Java类：使用任何标准方法生成UUID、高效输出、排序等。
* [FastUUID](https://github.com/jchambers/fast-uuid)：FastUUID是一个用于快速有效地解析和写入UUID的Java库。
* [FriendlyID](https://github.com/Devskiller/friendly-id)：FriendlyID库将给定的UUID(36个字符)转换为基于Base62(最多22个字符)的URL友好ID。
* [Juniper](https://github.com/tommyettinger/juniper)：Juniper提供了java.util.Random功能的超集，其中包含EnhancedRandom抽象类和各种具体实现。
* [Sqids Java](https://github.com/sqids/sqids-java)：Sqids是一个小型库，可让你从数字生成唯一的ID。
* [ULIDJ](https://github.com/azam/ulidj)：Java的ULID生成器和解析器。

## 堆外内存管理

* [Unsafe Tool](https://github.com/alexkasko/unsafe-tools)：使用sun.misc.Unsafe处理堆外内存的工具。
* [LLPL](https://github.com/pmem/llpl)：LLPL是一个Java库，提供对堆外持久性内存的访问，由Intel开源。
* [Slice](https://github.com/airlift/slice)：用于高效处理堆内存和堆外内存的Java库。
* [Externalsortinginjava](https://github.com/lemire/externalsortinginjava)：Java中的外部内存排序。
* [FastTuple](https://github.com/boundary/fasttuple)：FastTuple生成原始值的异构集合，并尽可能确保它们在内存中相邻排列。
* [Chronicle Core](https://github.com/OpenHFT/Chronicle-Core)：Chronicle Core是一个先进的低级库，为开发人员提供了与操作系统交互、管理内存、处理资源等功能强大的工具。
* [DataSketches Java Memory Component](https://github.com/apache/datasketches-memory)：Java的高性能本机内存访问库。
* [LArray](https://github.com/xerial/larray)：一个用于管理大型堆外数组的库，可以在Java和Scala中容纳超过2G(2^31)的条目。
* [JNVM](https://github.com/jnvm-project/jnvm)：J-NVM是一个Java框架，用于高效、本地地访问Java中的非易失性主存储器(NVMM)作为堆外内存，由南巴黎电信学院开源。

## Struct

* [JUnion](https://github.com/TehLeo/junion)：为Java编程语言提供结构类型。
* [Javastruct](https://github.com/dubrousky/javastruct)：Javastruct是一个将Java对象用作C或C++结构的库。

## 算法库

* [WikiSort](https://github.com/BonzaiThePenguin/WikiSort)：WikiSort是块合并排序的一种实现。
* [Trilateration](https://github.com/lemmingapex/trilateration)：使用非线性最小二乘优化器解决N维空间三边测量问题的公式。
* [Hashids.java](https://github.com/yomorun/hashids-java)：Hashids算法Java实现。
* [AhoCorasickDoubleArrayTrie](https://github.com/hankcs/AhoCorasickDoubleArrayTrie)：基于双数组Trie结构的Aho Corasick算法的极快实现。
* [JSI](https://github.com/aled/jsi)：JSI项目旨在维护高性能Java版本的RTree空间索引算法。
* [3D Bin Container Packing](https://github.com/skjolber/3d-bin-container-packing)：最大区域拟合优先算法+暴力算法的变体。
* [Min2phase](https://github.com/cs0x7f/min2phase)：Kociemba两阶段算法的优化实现。
* [JWave](https://github.com/graetz23/JWave)：离散傅里叶变换、快速小波变换和小波包变换算法的Java实现。
* [RendezvousHash](https://github.com/clohfink/RendezvousHash)：基于环的一致哈希的替代方案，这是Rendezvous(最高随机权重，HRW)哈希的快速线程安全实现。
* [Patricia Trie](https://github.com/rkapsi/patricia-trie)：检索以字母数字编码的信息的实用算法。
* [Jbsdiff](https://github.com/malensek/jbsdiff)：bsdiff算法的Java实现。
* [SZZ Unleashed](https://github.com/wogscpar/SZZUnleashed)：SZZ Unleashed是SZZ算法的实现，这是一种识别引入错误的提交的方法。
* [Hilbert Curve](https://github.com/davidmoten/hilbert-curve)：用于将沿N维希尔伯特曲线的距离转换为点并返回的Java实用程序。
* [Chronicle Algorithms](https://github.com/OpenHFT/Chronicle-Algorithms)：用于哈希、BitSet操作、访问数据类型的原始字节、堆外锁定的零分配，高效算法。
* [Compression](https://github.com/lichess-org/compression)：lichess.org的国际象棋时钟和着法压缩算法。
* [Timeseries Forecast](https://github.com/Workday/timeseries-forecast)：这是一个Java开源库，提供时序预测功能。
* [Delaunay Triangulation](https://github.com/jdiemke/delaunay-triangulator)：增量2D Delaunay三角剖分算法的简单Java实现。
* [JavaReedSolomon](https://github.com/Backblaze/JavaReedSolomon)：这是一个简单而高效的Java Reed-Solomon实现。
* [Fuzzy Matcher](https://github.com/intuit/fuzzy-matcher)：一个基于Java的库，用于对文档集合中的相似元素进行匹配和分组，由Intuit开发。
* [LattiCG](https://github.com/mjtb49/LattiCG)：反转Java的java.util.Random类的可能内部种子，并以各种Random调用的不等式系统的形式给出其输出信息。
* [ByteSeek](https://github.com/nishihatapalmer/byteseek)：ByteSeek是一个Java库，用于有效匹配字节模式并搜索这些模式。
* [LightGBM4j](https://github.com/metarank/lightgbm4j)：LightGBM4j是LightGBM项目的零依赖Java包装器。
* [TreeLayout](https://github.com/abego/treelayout)：TreeLayout可以为任意树创建树布局。
* [STL4j](https://github.com/ServiceNow/stl-decomp-4j)：Seasonal-Trend-Loess时序分解算法的Java实现。
* [Sudoku](https://github.com/sfuhrm/sudoku)：一个用于创建数独谜题的超快速算法的Java实现，同时还具有解答数独谜题的功能。
* [Dancing Links Java](https://github.com/rafalio/dancing-links-java)：这是Knuth的Dancing Links算法的Java实现，用于有效解决精确覆盖问题。
* [APTED](https://github.com/DatabaseGroup/apted)：这是APTED算法的实现，该算法是计算树编辑距离的最先进的解决方案，由奥地利萨尔茨堡大学开源。
* [Java Reinforcement Learning](https://github.com/chen0040/java-reinforcement-learning)：该软件包提供了强化学习算法的Java实现，例如Q-Learn、R-Learn、SARSA、Actor-Critic。

#### 聚类算法

* [Carrot2](https://github.com/carrot2/carrot2)：Carrot2是一个用于文本聚类的编程库。
* [T SNE Java](https://github.com/lejon/T-SNE-Java)：Van Der Maaten和Hinton的t-SNE聚类算法的纯Java实现。
* [Clust4j](https://github.com/tgsmith61591/clust4j)：Clust4j是一组基于Java的分类聚类算法。
* [Hierarchical Clustering Java](https://github.com/lbehnke/hierarchical-clustering-java)：用Java实现凝聚层次聚类算法，支持不同的链接方法。

#### 图算法

* [Dijkstras Algorithm](https://github.com/mburst/dijkstras-algorithm)：Dijkstra最短路径算法的不同语言实现。
* [PathFinder](https://github.com/kevinwang1975/PathFinder)：该项目包含广泛应用于路径查找和图遍历的A*搜索算法和用于Cisco路由器最短路径查找的Dijkstra算法的Java实现。
* [Hnswlib](https://github.com/jelmerk/hnswlib)：用于执行近似最近邻搜索的分层可导航小世界图(HNSW)算法的Java实现。
* [Graph Neo4j](https://github.com/neo4j-contrib/neo4j-graph-algorithms)：Neo4j的高效图算法。
* [Viterbi](https://github.com/hankcs/Viterbi)：通用的维特比算法实现。
* [Hipster4j](https://github.com/citiususc/hipster)：Hipster4j是一个轻量级且功能强大的Java和Android启发式搜索库，它包含常见的、完全可定制的算法，例如Dijkstra、A*、DFS、BFS、Bellman-Ford等，由圣地亚哥德孔波斯特拉大学开源。
* [Jaicore Search](https://starlibs.github.io/AILibs/projects/jaicore-search/)：Jaicore Search是一个启发式搜索库，由帕德博恩大学开发。
* [Maze](https://github.com/armin-reichert/mazes)：该项目提供了35多种算法的Java实现，用于生成所谓的“完美迷宫”(只是无向图的生成树)。

#### 随机流算法

* [Apache DataSketches](https://github.com/apache/datasketches-java)：Yahoo开源的随机流算法软件库。
* [Sketches Java](https://github.com/DataDog/sketches-java)：分布式分位数草图算法DDSketch的Java实现，由DataDog开源。
* [ZetaSketch](https://github.com/google/zetasketch)：用于单通道、分布式、近似聚合和草图绘制算法的库集合，由Google开源。

#### HyperLogLog算法

* [Stream Lib](https://github.com/addthis/stream-lib)：Stream Lib是一个用于汇总无法存储所有事件的流中数据的Java库，由AddThis开源。
* [Java HyperLogLog](https://github.com/aggregateknowledge/java-hll)：HyperLogLog算法的Java库。
* [Streaminer](https://github.com/mayconbordin/streaminer)：用于挖掘数据流的算法集合，包括频繁项集、分位数、采样、移动平均、集合成员资格和基数。
* [Hash4j](https://github.com/dynatrace-oss/hash4j)：Hash4j是Dynatrace的一个Java库，其中包括基于高质量哈希函数的各种非加密哈希算法和数据结构。

#### Simhash算法

* [Simhash Java](https://github.com/sing1ee/simhash-java)：Simhash算法的Java简单实现。
* [Simhash4J](https://github.com/xlturing/Simhash4J)：Simhash Java单机实现。

#### 协同过滤算法

* [LensKit](https://github.com/lenskit/lenskit)：LensKit是协作过滤算法的实现以及一组用于对其进行基准测试的工具，由明尼苏达大学开源。
* [CF4j](https://github.com/ferortega/cf4j)：Java的协同过滤库，用于开展基于协同过滤的推荐系统研究实验。
* [Sifarish](https://github.com/pranab/sifarish)：Sifarish是一套基于Hadoop和Storm实现的个性化推荐解决方案。

#### LSH算法

* [Java LSH](https://github.com/tdebatty/java-LSH)：局部敏感哈希(LSH)的Java实现。
* [TarsosLSH](https://github.com/JorenSix/TarsosLSH)：TarsosLSH是一个实现次线性最近邻搜索算法的Java库。
* [ScANNS](https://github.com/LinkedInAttic/scanns)：ScANNS是Apache Spark的一个最近邻搜索库，最初由LinkedIn开发。

#### LDA算法

* [LDA4j](https://github.com/hankcs/LDA4j)：LDA(潜在狄利克雷分配)的Java实现。
* [JGibbLDA](https://jgibblda.sourceforge.net/)：JGibbLDA是隐性狄利克雷分配(LDA)的Java实现，使用吉布斯抽样技术进行参数估计和推理，由越南国立大学开源。

#### 实体链接

* [FEL](https://github.com/yahoo/FEL)：快速实体链接器工具包，用于训练模型将实体链接到文档和查询中的知识库，由Yahoo开源。
* [Dexter](https://github.com/dexter/dexter)：Dexter是一个框架，它实现了一些流行的算法，并提供了开发任何实体链接技术所需的所有工具，由HPC实验室开源。

## 噪声库

* [Noise](https://github.com/SpongePowered/noise)：Java噪声生成库，基于Jason Bevins的libnoise C++库。
* [JNoise](https://github.com/Articdive/JNoise)：JNoise是一个简单易用的Java库，用于在Java中生成噪声(包括梯度噪声)。
* [FastNoise Lite](https://github.com/Auburn/FastNoiseLite)：FastNoise Lite是一个极其便携的开源噪声生成库，具有大量噪声算法可供选择。
* [OpenSimplex2](https://github.com/KdotJPG/OpenSimplex2)：OpenSimplex Noise的后继者，以及更新的OpenSimplex。

## 下采样

* [Downsampling Java](https://github.com/ggalmazor/lt_downsampling_java8)：Java 8的最大三角形下采样算法实现。
* [Downsample](https://github.com/drcrane/downsample)：这是flot charts插件使用的下采样实现。

## 原生开发

* [JNI](https://docs.oracle.com/javase/8/docs/technotes/guides/jni/)：JNI是一个标准编程接口，用于编写Java本机方法并将Java虚拟机嵌入到本机应用程序中。
* [Project Panama](https://github.com/openjdk/panama-foreign)：Project Panama旨在提高Java编程语言和本机库之间的互操作性的更改。
* [JNA](https://github.com/java-native-access/jna)：JNA使Java程序可以轻松访问原生共享库，而无需编写Java代码之外的任何内容-不需要JNI或原生代码。
* [JavaCPP](https://github.com/bytedeco/javacpp)：JavaCPP提供了对Java内部原生C++的高效访问。
* [JNR-FFI](https://github.com/jnr/jnr-ffi)：JNR-FFI是一个Java库，用于加载本机库，无需手动编写JNI代码或使用SWIG等工具。
* [ReLinker](https://github.com/KeepSafe/ReLinker)：适用于Android的强大原生库加载器。
* [SoLoader](https://github.com/facebook/SoLoader)：SoLoader是Android的本机代码加载器，Facebook开源。
* [JNIWrapper](https://teamdev.com/jniwrapper/)：该库可以在没有JNI的情况下在Java中使用本机代码。
* [JniHelpers](https://github.com/spotify/JniHelpers)：JniHelpers是一个旨在方便使用C++编写JNI代码的库，Spotify开源。
* [Spring Native](https://github.com/spring-attic/spring-native)：Spring Native提供了使用GraalVM本机镜像编译器将Spring应用程序编译为本机可执行文件的beta支持。
* [JavaCPP-Presets](https://github.com/bytedeco/javacpp-presets)：JavaCPP Presets包含广泛使用的C/C++库的Java配置和接口类。
* [JNAerator](https://github.com/nativelibs4java/JNAerator)：JNAerator为C、C++和Objective-C库生成完整的本机绑定，针对BridJ、JNA或Node.js运行时。
* [Nalim](https://github.com/apangin/nalim)：Nalim是一个使用JVMCI(JVM编译器接口)将Java方法链接到本机函数的库。
* [Native-Utils](https://github.com/adamheinrich/native-utils)：一个简单的工具库，用于加载存储在JAR存档中的动态库。
* [Jextract](https://github.com/openjdk/jextract)：Jextract是一个从本机库头自动生成Java绑定的工具，Oracle开发。
* [BridJ](https://github.com/nativelibs4java/BridJ)：BridJ是一个Java/原生互操作性库，专注于速度和易用性。
* [HawtJNI](https://github.com/fusesource/hawtjni)：基于Eclipse SWT中使用的JNI生成器的JNI代码生成器。
* [FastFFI](https://github.com/alibaba/fastFFI)：适用于Java和C++的现代高效FFI，由阿里开源。
* [Native-Library-Loader](https://github.com/scijava/native-lib-loader)：用于从Java中提取和加载本机库的本机库加载器。
* [Jssembly](https://github.com/dvx/jssembly)：Jssembly是一个库，允许你通过JNI桥从Java执行本机汇编。
* [Facebook JNI](https://github.com/facebookincubator/fbjni)：Facebook JNI工具库旨在简化Java JNI的使用。
* [JFFI](https://github.com/jnr/jffi)：libffi的Java绑定。
* [JNIPP](https://github.com/mitchdowd/jnipp)：JNIPP是标准JNI的C++包装器，它旨在简化Java和C++代码集成过程中的一些繁琐步骤。
* [GlueGen](https://github.com/sgothel/gluegen)：GlueGen是一个用于函数和数据结构声明的编译器，可在编译时离线生成Java和JNI C代码，并允许在Java应用程序中使用本机库。
* [Native Platform](https://github.com/gradle/native-platform)：用于各种本机API的跨平台Java API的集合，由Gradle团队开源。
* [Libcore Syscall](https://github.com/cinit/LibcoreSyscall)：Libcore Syscall是一个适用于Android的Java库，允许你直接从Java代码进行任何Linux系统调用。
* [JNI HPP](https://github.com/mapbox/jni.hpp)：JNI HPP是一个现代的、类型安全的、仅标头的C++14 JNI包装器，其目的是使从C++调用Java或从Java调用C++变得方便而安全。
* [JNI4Android](https://github.com/bilibili/jni4android)：从伪Java生成C包装器，由B站开源。
* [JFA](https://github.com/0x4a616e/jfa)：JFA是一个用于访问Apple Foundation框架的纯Java库。

## 操作系统信息

* [Sigar](https://github.com/hyperic/sigar)：Sigar提供跨平台的系统信息收集的API。
* [OSHI](https://github.com/oshi/oshi)：OSHI是一个免费的基于JNA的Java操作系统和硬件信息库，提供跨平台实现来检索系统信息，例如操作系统版本、进程、内存和CPU使用情况、磁盘和分区、设备、传感器等。
* [JavaSysMon](https://github.com/jezhumble/javasysmon)：JavaSysMon旨在提供一种独立于操作系统的方式来管理操作系统进程并获取实时系统性能信息(例如CPU和内存使用情况)，并作为单个jar文件分发。

## COM桥

* [ComfyJ](https://teamdev.com/comfyj/)：ComfyJ是一款双向Java-COM桥接工具，用于提供COM到Java和Java到COM的交互连接。
* [Jacob](https://github.com/freemansoft/jacob-project)：Jacob是一个Java库，允许Java应用程序与Microsoft Windows DLL或COM库进行通信。
* [Com4j](https://github.com/kohsuke/com4j)：类型安全的Java/COM绑定。
* [J-Integra](https://j-integra.intrinsyc.com/)：J-Integra是一个高性能中间件软件桥，可实现Java COM互操作性。
* [JacoZoom](https://jacozoom.software.informer.com/)：JacoZoom是一个Java类库，它允许你通过Java使用ActiveX控件和ActiveX服务器(COM/DCOM/自动化)。

## GPU编程

* [TornadoVM](https://github.com/beehive-lab/TornadoVM)：TornadoVM是OpenJDK和GraalVM的插件，允许程序员在异构硬件上自动运行Java程序，这是曼彻斯特大学高级处理器技术小组的研究项目。
* [JCuda](https://github.com/jcuda/jcuda)：CUDA的Java绑定。
* [Aparapi](https://github.com/Syncleus/aparapi)：Aparapi允许开发人员通过在运行时动态地将Java字节代码转换为OpenCL内核来编写能够直接在显卡GPU上执行的本机Java代码。
* [JOML](https://github.com/JOML-CI/JOML)：用于OpenGL渲染计算的Java数学库。
* [JOGL](https://github.com/sgothel/jogl)：OpenGL API的Java绑定。
* [PixelFlow](https://github.com/diwi/PixelFlow)：用于高性能GPU计算处理的Java库。
* [JavaCL](https://github.com/nativelibs4java/JavaCL)：Java的OpenCL绑定。
* [ArrayFire Java](https://github.com/arrayfire/arrayfire-java)：ArrayFire的Java包装器。
* [JOCL](https://github.com/gpu/JOCL)：OpenCL的Java绑定。
* [JOAL](https://github.com/sgothel/joal)：JOAL项目托管OpenAL API的Java绑定的参考实现，旨在为用Java编写的应用程序提供硬件支持的3D空间化音频。
* [grCUDA](https://github.com/NVIDIA/grcuda)：GraalVM的多语言CUDA集成，由英伟达开源。
* [CLIJ2](https://github.com/clij/clij2)：CLIJ2是一个适用于ImageJ/Fiji、Icy、Matlab和Java的GPU加速图像处理库，由萨塞克斯大学、荷兰癌症研究所、剑桥大学、牛津大学、悉尼大学等组织共同开发。
* [PanamaGL](https://gitlab.com/jzy3d/panama-gl)：PanamaGL旨在使用适用于JDK 22及更高版本的FFM API提供与OpenGL的多平台绑定。
* [Beehive LevelZero JNI](https://github.com/beehive-lab/levelzero-jni)：该项目是针对Intel LevelZero的Java本机接口(JNI)绑定，由曼彻斯特大学开源。

## 硬件操作

* [JNativeHook](https://github.com/kwhat/jnativehook)：JNativeHook是一个为Java提供全局键盘和鼠标监听器的库。
* [JCgroup](https://github.com/haosdent/jcgroup)：JCgroup是JVM上的Cgroup包装器，你可以使用该库来限制线程的CPU份额、磁盘I/O速度、网络带宽等。
* [Repeat](https://github.com/repeats/Repeat)：跨平台鼠标/键盘记录/重播和自动化热键/宏创建，以及更高级的自动化功能。
* [System Hook](https://github.com/kristian/system-hook)：Java System Hook为Java提供了一个非常轻量级的全局键盘和鼠标监听器。
* [NaturalMouseMotion](https://github.com/JoonasVali/NaturalMouseMotion)：该库提供了一种将光标可靠地移动到屏幕上指定坐标的方法，同时随机形成弧线，看起来就像真手使用鼠标将其移动到那里。
* [Webcam Capture](https://github.com/sarxos/webcam-capture)：该库允许你直接从Java使用内置或外部网络摄像头。
* [Picam](https://github.com/caprica/picam)：Picam是一个易于使用的开源Java库，用于访问Raspberry Pi相机模块。
* [PixelController](https://github.com/neophob/PixelController)：该应用程序的主要目标是创建一个易于使用的矩阵控制器软件。
* [Stream-Pi Client](https://github.com/stream-pi/client)：免费、开源、模块化、跨平台和可编程宏垫。
* [JIntellitype](https://github.com/melloware/jintellitype)：JIntellitype是一个Java API，用于与Microsoft Intellitype命令交互以及在Java应用程序中注册全局热键。
* [Cups4j](https://github.com/harwey/cups4j)：CUPS的Java打印库。
* [EDSDK4J](https://github.com/kritzikratzi/edsdk4j)：这是Canon EOS数字软件开发套件EDSDK的Java包装器，可让你在Windows上完全访问Canon SLR相机。
* [ProviewR](http://www.proview.se/v3/)：ProviewR可能是世界上第一个用于过程控制和自动化的开源系统。
* [KWSwitch](https://gitee.com/kerwincui/kwswitch)：智能开关平台，包含服务端、硬件端、安卓端和前端。
* [Mixly](https://gitee.com/mixlyplus/Mixly)：Mixly是一款面向初学者、硬件编程爱好者的图形化编程工具。
* [Attach](https://github.com/gluonhq/attach)：Gluon Attach是一个解决端到端Java Mobile解决方案中与低级平台API集成的组件。
* [JKeyMaster](https://github.com/tulskiy/jkeymaster)：用于使用JNA在Java中注册全局热键的库，目标是支持基于X11的平台、Windows和MacOSX。
* [XBee Java](https://github.com/digidotcom/xbee-java)：这是一个用Java开发的易于使用的API，允许你与Digi International的XBee射频(RF)模块进行交互。
* [ZSmartSystems](https://github.com/zsmartsystems/com.zsmartsystems.zigbee)：该项目旨在提供一个用Java编写并与Android兼容的ZigBee兼容框架。
* [VisiCut](https://github.com/t-oster/VisiCut)：VisiCut是一个用户友好、独立于平台的工具，用于准备、保存作业并将其发送到激光切割机。
* [jSensors](https://github.com/profesorfalken/jSensors)：jSensors是一个监控电脑所有硬件传感器的Java库。

## 运动规划

* [OWL](https://github.com/idsc-frazzoli/owl)：Java中的运动规划库，由苏黎世联邦理工学院开源。
* [RVO2-Java](https://github.com/snape/RVO2-Java)：最佳相互避免碰撞(ORCA)算法的Java实现，由北卡罗来纳大学开源。

## 自动规划

* [PDDL4J](https://github.com/pellierd/pddl4j)：PDDL4J的目的是促进基于PDDL语言(规划域描述语言)的自动化规划Java工具的开发，格勒诺布尔计算机科学实验室开源。
* [Neptus](https://github.com/LSTS/neptus)：Neptus是用于操作所有类型无人驾驶车辆的分布式指挥和控制基础设施，由波尔图大学开源。

## 操作系统

* [JOS](https://sourceforge.net/projects/jos/)：一个免费且开源的基于Java的操作系统。
* [JNode](https://github.com/jnode/jnode)：JNode是一个开源项目以创建一个Java平台的操作系统。
* [JX](https://github.com/mczero80/jx)：JX是一个Java操作系统，专注于灵活和健壮的操作系统架构。
* [JavaOS](https://zh.wikipedia.org/zh-cn/JavaOS)：JavaOS是一套操作系统，以JVM与一些基础软件组件所构成，由SUN公司开发。
* [AugmentOS](https://github.com/AugmentOS-Community/AugmentOS)：适用于智能眼镜的开源操作系统。

## 电力系统

* [PowSyBl](https://github.com/powsybl/powsybl-core)：PowSyBl是一个用Java编写的开源框架，可以轻松编写用于电力系统仿真和分析的复杂软件。
* [SIMONA](https://github.com/ie3-institute/simona)：SIMONA提供了一个仿真工具箱，用于运行和实施大规模基于代理的电网仿真，重点关注配电网，由多特蒙德工业大学开源。
* [j60870](https://github.com/gythialy/j60870)：j60870是一个实现IEC 60870-5-104通信标准的库。
* [OperatorFabric](https://github.com/opfab/operatorfabric-core)：OperatorFabric是一个模块化、可扩展、工业强度的平台，用于电力、水和其他公用事业运营。
* [JoularJX](https://github.com/joular/joularjx)：JoularJX是一个基于Java的源代码级别电源监控代理，支持现代Java版本和多操作系统，以监控硬件和软件的功耗，由波城大学开源。

## 量子计算

* [Strange](https://github.com/redfx-quantum/strange)：该项目定义了一个可用于创建量子程序的Java API。
* [iQuantum](https://github.com/Cloudslab/iQuantum)：iQuantum是用于量子计算环境建模和仿真的工具包，由墨尔本大学开源。
* [JQuantLib](https://github.com/frgomes/jquantlib)：JQuantLib是一个免费、开源、全面的量化金融框架，100%用Java编写。

## QA系统

* [TDuck](https://gitee.com/TDuckApp/tduck-platform)：TDuck是国内首批基于Vue开源的问卷/表单收集工具。
* [XIAOJUSURVEY](https://github.com/didi/xiaoju-survey)：XIAOJUSURVEY是一套轻量、安全的问卷系统，滴滴开源。
* [QuestionAnsweringSystem](https://github.com/ysc/QuestionAnsweringSystem)：QuestionAnsweringSystem是一个Java实现的人机问答系统，能够自动分析问题并给出候选答案，这是IBM Watson的Java开源实现。
* [Qanary](https://github.com/WDAqua/Qanary)：Qanary是一种创建问答系统的方法。
* [Scoold](https://github.com/Erudika/scoold)：Scoold是一个面向团队的问答和知识共享平台。
* [YodaQA](https://github.com/brmson/yodaqa)：YodaQA是一个开源Factoid问答系统，可以使用即时信息提取从数据库和文本语料库中生成答案。
* [OpenQA](https://bitbucket.org/emarx/openqa/src/master/)：OpenQA是一个用于问答开发和发布的框架和平台。
* [Mamute](https://github.com/caelum/mamute)：Mamute是一个基于CDI和VRaptor 4使用Java开发的问答引擎。
* [Openauth](https://gitee.com/yubaolee/openauth.qa)：Openauth是一个简洁实用的问答网站。

## CMS系统

* [Halo](https://github.com/halo-dev/halo)：Halo是一个强大易用的开源建站工具，由飞致云开源。
* [Novel](https://github.com/201206030/novel)：Novel是一套基于Java技术栈Spring Boot 3、Vue 3开发的前后端分离学习型小说项目。
* [MCMS](https://gitee.com/mingSoft/MCMS)：MCMS是免费可商用的开源Java CMS内容管理系统，由铭软科技公司开源。
* [JPress](https://gitee.com/JPressProjects/jpress)：JPress是一个使用Java开发、类似WordPress的产品，支持多站点、多语种自动切换等，由小码科技开源。
* [师说CMS](https://gitee.com/shishuo/CMS_old)：师说CMS是一款使用Java语言开发的CMS，使用了Spring MVC、Spring、MyBatis等流行框架，提供首页大图管理、目录管理、文章管理和管理员管理等功能。
* [VBlog](https://github.com/lenve/VBlog)：V部落是一个多用户博客管理平台，采用Vue、Spring Boot开发。
* [JFinal CMS](https://gitee.com/jflyfox/jfinal_cms)：JFinal CMS是一个Java开发的功能强大的信息咨询网站，采用JFinal作为web框架。
* [FastCMS](https://gitee.com/dianbuapp_admin/fastcms)：FastCMS是基于Spring Boot前后端分离技术，且具有插件化架构的CMS系统。
* [MoguBlog](https://github.com/moxi624/mogu_blog_v2)：MoguBlog是一个基于微服务架构的前后端分离博客系统。
* [DotCMS](https://github.com/dotCMS/core)：DotCMS是一种开源无头/混合内容管理系统，旨在跨多个渠道管理和提供基于权限的个性化内容体验。
* [Novel Plus](https://github.com/201206030/novel-plus)：Novel Plus是一个多端(PC、WAP)阅读，功能完善的原创文学CMS系统。
* [White Jotter](https://github.com/Antabot/White-Jotter)：White Jotter是一个Spring Boot和Vue.js开发的简单CMS。
* [PublicCMS](https://github.com/sanluan/PublicCMS)：PublicCMS是2024年采用主流技术开发的开源Java CMS系统。
* [iTranswarp](https://github.com/michaelliao/itranswarp)：功能齐全的CMS，包括博客、Wiki、讨论等，由Spring Boot提供支持的云原生应用程序。
* [Apache Roller](https://github.com/apache/roller)：Roller是一个基于Java、功能齐全的多用户和群组博客服务器，适用于大大小小的博客网站。
* [XWiki Platform](https://github.com/xwiki/xwiki-platform)：XWiki Platform是一个通用的Wiki平台，为构建在其之上的应用程序提供运行时服务。
* [Tianti](https://github.com/xujeff/tianti)：天梯是一款使用Java编写的免费的轻量级CMS系统，目前提供了从后台管理到前端展现的整体解决方案。
* [巡云轻论坛系统](https://gitee.com/diyhi/bbs)：巡云轻论坛系统包含论坛、问答模块，采用Java、MySQL架构。
* [LinCMS](https://github.com/TaleLin/lin-cms-spring-boot)：LinCMS是林间有风团队经过大量项目实践所提炼出的一套内容管理系统框架。
* [Ametys](https://www.ametys.org/community/en/index.html)：Ametys是一个用Java编写的免费开源内容管理系统。
* [网市场CMS](https://gitee.com/mail_osc/wangmarket)：私有化部署自己的SAAS云建站系统，跟可通过后台任意开通多个网站，每个网站使用自己的账号进行独立管理。
* [FlyCMS](https://github.com/sunkaifei/FlyCms)：FlyCMS是一个类似知乎以问答为基础的完全开源的Java语言开发的社交网络建站程序。
* [ThriveX](https://github.com/LiuYuYang01/ThriveX-Server)：ThriveX是一个高颜值、全开源、永不收费的现代化博客管理系统。
* [Gentics Mesh](https://github.com/gentics/mesh)：Gentics Mesh是为开发人员提供的开源无头CMS。
* [Tale Blog](https://github.com/otale/tale)：Tale是使用轻量级的MVC框架Blade进行开发的简洁美观的Java博客系统。
* [ThinkItCMS](https://gitee.com/slfj/ThinkItCMS)：ThinkItCMS是一款面向模板开发，支持静态生成的CMS系统。
* [Nuxeo Platform](https://github.com/nuxeo/nuxeo)：Nuxeo是一个开源的可定制和可扩展的内容管理平台，用于构建业务应用程序。
* [IceCMS](https://github.com/Thecosy/IceCMS)：IceCMS是基于Spring Boot、Vue前后端分离的内容管理系统。
* [Bolo Solo](https://github.com/adlered/bolo-solo)：菠萝博客是专为程序员设计的精致Java博客系统。
* [Brix](https://github.com/brix-cms/brix-cms)：Brix基于Wicket和JCR，是当今最好的基于Wicket的CMS框架。
* [ForestBlog](https://github.com/saysky/ForestBlog)：ForestBlog是一个简单漂亮的SSM博客系统。
* [Apache Stanbol](https://stanbol.apache.org/)：Stanbol提供了一组用于语义内容管理的可重用组件。
* [Solo](https://github.com/88250/solo)：Solo是一款小而美的开源博客系统，专为程序员设计。
* [Jease](https://jease.org/)：Jease是一个由Java驱动的开源CMS。
* [Quanta](https://github.com/Clay-Ferguson/quantizr)：Quanta是一种新型内容管理平台，具有强大的功能。
* [WallRide](https://github.com/tagbangers/wallride)：WallRide是一个多语言、易于定制的开源CMS。
* [MyBlog](https://github.com/ZHENFENG13/My-Blog)：MyBlog是由Spring Boot、Mybatis、Thymeleaf等技术实现的Java博客系统。
* [Magnolia CMS](https://www.magnolia-cms.com/)：Magnolia是一个开源CMS，由总部位于瑞士巴塞尔的Magnolia开发。
* [瀑布CMS](https://gitee.com/LinZhaoguan/pb-cms)：瀑布CMS采用Spring Boot、Shiro、MybatisPlus、Thymeleaf实现。
* [UJCMS](https://gitee.com/ujcms/ujcms)：Java开源内容管理系统，使用Spring Boot、MyBatis、Spring Security、Lucene、FreeMarker、TypeScript、Vue3、ElementPlus等技术开发。
* [OpenCMS](https://github.com/alkacon/opencms-core)：OpenCMS是Alkacon公司开发的Java内容管理系统。
* [DreamerCMS](https://gitee.com/iteachyou/dreamer_cms)：DreamerCMS采用流行的Spring Boot搭建，支持静态化、标签化建站。
* [Blossom](https://github.com/blossom-editor/blossom)：Blossom是一个支持私有部署的云端双链笔记软件，你可以将你的笔记，图片，个人计划安排保存在自己的服务器中，并在任意设备之间实时同步。
* [OneBlog](https://gitee.com/yadong.zhang/DBlog)：OneBlog是一个简洁美观、功能强大并且自适应的Java博客。
* [Symphony](https://gitee.com/dl88250/symphony)：Symphony是一款用Java实现的现代化社区(论坛/问答/BBS/社交网络/博客)系统平台。
* [MBlog](https://gitee.com/mtons/mblog)：MBlog开源Java博客系统，支持多用户、支持切换主题。
* [NemakiWare](https://github.com/aegif/NemakiWare)：NemakiWare是一个开源企业内容管理系统。
* [WebSight](https://www.websight.io/)：WebSight是一个容器化内容管理系统，与StreamX数字体验服务网格进行原生集成。
* [Lutece](https://github.com/lutece-platform/lutece-core)：Lutece是一个开放平台，使城市政府能够共享、重复使用和改编其他城市创建的数字服务。
* [Antville](https://github.com/antville/antville)：Antville是一个开源项目，提供高性能、功能丰富的博客托管软件。
* [JTopCMS](https://gitee.com/mjtop/JTopCMSV3)：JTopCMS基于Java EE标准研发，用于管理站群内容的内容管理软件，由合肥明靖信息科技公司开源。
* [Plumemo](https://github.com/open-snail/plumemo)：Plumemo是一个轻量、易用、前后端分离的博客系统。
* [PerfreeBlog](https://gitee.com/PerfreeBlog/PerfreeBlog)：PerfreeBlog是一款基于Java开发的博客/CMS建站平台。

## ERP系统

* [华夏ERP](https://gitee.com/jishenghua/JSH_ERP)：华夏ERP是基于Spring Boot框架和SaaS模式开源的ERP软件，目前专注进销存、财务、生产功能。
* [赤龙ERP](https://gitee.com/redragon/redragon-erp)：赤龙ERP是一款免费开源、业务闭环、灵活稳定的企业级ERP系统。
* [Metasfresh](https://github.com/metasfresh/metasfresh)：Metasfresh是一个响应迅速、免费且开源的ERP系统。
* [REBUILD](https://gitee.com/getrebuild/rebuild)：REBUILD通过创新的业务流程引擎帮助你快速搭建各类企业管理系统，全图形化配置无需了解技术，由上海锐昉科技公司开源。
* [OMS](https://github.com/FJ-OMS/oms-erp)：一站式全渠道业务中台系统，包括订单管理系统OMS/电商ERP、库存WMS统一管理系统和SAP财务管理系统等，由厦门飞骥公司开源。
* [ADempiere](https://github.com/adempiere/adempiere)：ADempiere商业套件ERP/CRM/MFG/SCM/POS以开放且不减的方式实现了Bazaar方式。
* [Apache OFBiz](https://github.com/apache/ofbiz-framework)：OFBiz是一个用Java编写的ERP系统，包含大量库、实体、服务和功能来运行你业务的各个方面。
* [IDempiere](https://github.com/idempiere/idempiere)：IDempiere是完全开源的商务套件ERP/CRM/MFG/SCM/POS。
* [Skyeye](https://gitee.com/doc_wei01/erp-pro)：Skyeye基于Spring Boot框架，为中小企业打造的开源好用ERP软件。
* [MyCompany](https://github.com/lsfusion-solutions/mycompany)：MyCompany是一个适用于小型企业免费的开源ERP构建器。
* [悟空CRM](https://github.com/WuKongOpenSource/WukongCRM-11.0-JAVA)：悟空CRM是基于Spring Cloud Alibaba微服务架构、Vue ElementUI的前后端分离CRM系统。
* [iBizEHR](https://gitee.com/ibizlab/iBizEHR)：iBizEHR是一套可满足万人应用的高性能人力资源管理软件，埃毕致开源。
* [Saas ERP](https://gitee.com/hy417393356/saas-java)：简云Saas平台是基于Spring Boot 2.2.0、Mybatis、JWT、Redis、Vue、ElementUI的前后端分离的Saas平台后台管理系统。
* [星云ERP](https://gitee.com/lframework/xingyun)：星云ERP基于Spring Boot框架，为中小企业提供完全开源、永久免费、用户体验好的进销存ERP系统。
* [Qcadoo MES](https://github.com/qcadoo/mes)：Qcadoo MES是一款针对中小企业的生产管理互联网应用，它结合了大型ERP系统的功能，适应中小企业的具体特点。
* [Wimoor ERP](https://github.com/wimoor-erp/wimoor)：Wimoor ERP是国内首款百分百开源、支持商用的亚马逊ERP系统。
* [CalLite CRM](https://www.callite.it/)：CalLite是市场上功能最丰富、速度最快的呼叫中心软件，它可以让你消除用户时间的浪费(操作员、代理、主管、协调员等)，从而实现收益最大化。
* [Compiere](https://www.aptean.com/en-US/solutions/erp/products/aptean-compiere-erp)：Compiere是一款开源ERP和CRM业务解决方案，适用于分销、零售、服务和制造领域的中小型企业。
* [Libertya](https://github.com/Disytel-Consulting-SA/libertya)：Libertya是一款综合管理管理软件，无需许可费用且完全免费使用，专为在任何类型的公司中快速实施和启动而设计。
* [BlueSeer ERP](https://github.com/BlueSeerERP/blueseer)：BlueSeer ERP是一个免费的开源多语言ERP软件包。
* [EAIRP](https://github.com/eairps/eairp)：开源Sass AI ERP系统。

## DMS系统

* [Teedy](https://github.com/sismics/docs)：Teedy是一个面向个人和企业的开源、轻量级文档管理系统。
* [OpenKM Community Edition](https://github.com/openkm/document-management-system)：OpenKM文档管理系统允许企业控制电子文档的制作、存储、管理和分发，从而提高效率以及重用信息和控制文档流的能力。
* [LogicalDOC Community Edition](https://github.com/logicaldoc/community)：LogicalDOC社区版是一款开源文档管理软件。
* [FormKiQ](https://github.com/formkiq/formkiq-core)：灵活的文档管理系统。
* [ECMS](https://github.com/exoplatform/ecms)：eXo的组合ECM(文档)和CMS管理系统。
* [瀚为云文档协作平台](https://github.com/honvay/hdms-community)：瀚为云文档协作平台是一款面向企业的文档协作平台，着力于满足企业文档管理中统一管理、安全存储、共享协作、权限控制等需求。
* [CrafterCMS](https://github.com/craftercms/craftercms)：CrafterCMS是一个现代内容管理平台，用于构建数字体验应用程序。

## SCRM系统

* [LinkWeChat](https://gitee.com/LinkWeChat/link-wechat)：LinkWeChat是基于企业微信的开源SCRM系统，是企业私域流量管理与营销的综合解决方案。
* [源雀SCRM](https://gitee.com/iyque/iYqueCode)：源雀SCRM是完全开源的私域数智化营销解决方案。
* [MarketGo](https://github.com/marketgo-scrm/MarketGo-SCRM)：MarketGo是一个营销引擎，通过提供的标准化功能和基础能力，让开发者能快速搭建适合自己企业的营销系统。
* [EasyLink](https://github.com/lianluoyi/easyink_System)：EasyLink是基于企业微信生态的一站式私域流量运营平台。
* [Pumplabs SCRM](https://github.com/Pumplabs/scrm)：Pumplabs SCRM是一款开源、基于企业微信、简洁便捷、高效协作的CRM应用，主要面向一定售周期的业务，如家装、汽车服务、保险、IT、美容、教培等。
* [MoChat](https://gitee.com/mochat/mochat-java)：MoChat是基于企业微信的开源应用开发框架&引擎，也是一套通用的企业私域流量管理系统。

## 门户框架

* [Apache Protals](http://portals.apache.org/)：Portals项目提供各种软件产品，包括Jetspeed-2、Pluto和Portals Applications。
* [Liferay Portal](https://github.com/liferay/liferay-portal)：Liferay是一个现成的，即开即用的，功能完备的门户网站。
* [Apereo uPortal](https://github.com/uPortal-Project/uPortal)：uPortal是由高等教育界构建并为其服务的领先开源企业门户框架，由威斯康星大学开源。

## 教育软件

* [TEAMMATES](https://github.com/TEAMMATES/teammates)：TEAMMATES是一个免费的在线工具，用于管理学生的同行评估和其他反馈路径，由新加坡国立大学开源。
* [BigBlueButton](https://github.com/bigbluebutton/bigbluebutton)：BigBlueButton是一个开源虚拟教室，旨在帮助教师教学和学习者学习。
* [Sakai](https://github.com/sakaiproject/sakai)：Sakai是一个免费提供的、功能丰富的技术解决方案，用于学习、教学、研究和协作，由美国印第安纳大学、密西根大学、斯坦福大学和麻省理工学院于2004年发起。
* [OpenOLAT](https://github.com/OpenOLAT/OpenOLAT)：OpenOlat是一个基于Web的电子学习平台，用于教学、学习、评估和交流，是一个LMS、一个学习管理系统。
* [SkillTree](https://github.com/NationalSecurityAgency/skills-service)：SkillTree是一个微型学习游戏化平台，提供开箱即用的UI可视化、方便的客户端集成库以及用于管理游戏化培训档案创建和管理的仪表板，由美国国家安全局开源。
* [学之思](https://gitee.com/mindskip/xzs-mysql)：学之思开源考试系统是一款Java、Vue前后端分离的考试系统，由武汉思维跳跃公司开源。
* [Roncoo Education](https://gitee.com/roncoocom/roncoo-education)：领课教育系统是一套基于点播、直播、班级、考试、营销等功能完善的在线教育系统，由广州领课网络公司开源。
* [SurveyKing](https://gitee.com/surveyking/surveyking)：SurveyKing是功能强大、搭建简单、界面更美观的在线考试/调查问卷/公开查询/题库刷题/360度评估/投票系统，由北京橙宇言信科技公司开源。
* [Exam++](https://gitee.com/ocelot/examxx)：Exam++是基于Java与MySQL开发的网络考试系统。
* [TamGuo](https://gitee.com/smiletocandy/tamguo)：TamGuo是基于Java开发的在线题库系统。
* [PlayEdu](https://gitee.com/playeduxyz/playedu)：PlayEdu是由白书科技团队打造出的一款业内领先的线上培训解决方案。

## 静态站点生成器

* [JBake](https://github.com/jbake-org/jbake)：JBake是一个为开发人员提供的基于Java的开源静态站点/博客生成器。
* [Orchid](https://github.com/orchidhq/Orchid)：Orchid是一个用于生成具有所有功能的项目文档网站的框架。
* [Znai](https://github.com/testingisdocumenting/znai)：Znai将人类书写的文本与代码、图形、REST API、Java文档、Doxygen等工件相结合，以创建最新、可维护、精美的用户指南和教程。
* [OpooPress](https://github.com/opoo/opoopress)：OpooPress框架是一个完全灵活、完全可扩展的基于Java的静态站点生成器。
* [Grain](https://github.com/sysgears/grain)：Grain是一个轻量级框架和一个非常强大的静态网站生成器，用Groovy编写，可帮助使网站创建直观且愉快。

## 网络开发

这里列出了网络相关的库、软件、工具集合。

#### 网络库

* [Netty](https://github.com/netty/netty)：Netty是一个异步事件驱动的网络应用程序框架，用于快速开发可维护的高性能协议服务器和客户端。
* [Apache MINA](https://github.com/apache/mina)：MINA是一个网络应用框架，可以帮助用户开发高性能和高可扩展性的网络应用程序。
* [T-IO](https://gitee.com/tywo45/t-io)：T-IO是基于Java开发的一款高性能网络编程框架，由钛特云开源。
* [Eclipse Grizzly](https://github.com/eclipse-ee4j/grizzly)：Grizzly的目标是帮助开发人员使用NIO构建可扩展且强大的服务器，Oracle开源。
* [Fast Android Networking](https://github.com/amitshekhariitbhu/Fast-Android-Networking)：Fast Android Networking是一个功能强大的库，用于在Android应用程序中执行任何类型的网络。
* [SOFABolt](https://github.com/sofastack/sofa-bolt)：SOFABolt是蚂蚁金融开发的一套基于Netty实现的网络通信框架。
* [Network Connection](https://github.com/facebookarchive/network-connection-class)：Network Connection是一个Android库，可让你了解当前用户的互联网连接质量，由Facebook开源。
* [SNF4J](https://github.com/snf4j/snf4j)：SNF4J是一个异步事件驱动的网络应用程序框架，用于快速轻松地开发网络应用程序。
* [CoralReactor](https://www.coralblocks.com/index.php/category/coralreactor/)：CoralReactor是一个功能强大、超低延迟、异步、非阻塞的网络I/O库，可通过简单的API提供高性能。
* [Smart Socket](https://gitee.com/smartboot/smart-socket)：Smart Socket是一款极简、易用、高性能的AIO通信框架。
* [One NIO](https://github.com/odnoklassniki/one-nio)：One NIO是一个用于构建高性能Java服务器的库，由Odnoklassniki开源。
* [Apache Commons Net](https://github.com/apache/commons-net)：Commons Net库包含网络实用程序和协议实现的集合。
* [Envoy Mobile](https://github.com/envoyproxy/envoy-mobile)：构建在Envoy项目的核心网络层上的多平台客户端HTTP/网络库。
* [Android Network Tools](https://github.com/stealthcopter/AndroidNetworkTools)：该项目是一组有用的Android网络工具库，例如端口扫描、ping等。
* [QuickServer](https://github.com/QuickServerLab/QuickServer-Main)：QuickServer是一个开源Java库/框架，用于快速创建强大的多客户端TCP服务器应用程序。
* [AdbLib](https://github.com/cgutman/AdbLib)：ADB网络协议的Java库实现。
* [Jpcap](https://github.com/jpcap/jpcap)：Jpcap是一组Java类，提供用于网络数据包捕获的接口和系统，由加州大学欧文分校开源。
* [Pcap](https://github.com/ardikars/pcap)：Pcap提供了一个数据包处理库，可以在JVM语言之上进行快速开发。
* [Gecko](https://github.com/killme2008/gecko)：Gecko是一个Java NIO的通讯组件，它在一个轻量级的NIO框架的基础上提供了更高层次的封装和功能，由淘宝开发。
* [Pkts](https://github.com/aboutsip/pkts)：Pkts是一个用于读取和写入pcap的纯Java库。
* [jNetPcap](https://github.com/slytechs-repos/jnetpcap-wrapper)：jNetPcap是一个用Java编写的软件库，提供与流行的libpcap本机库类似的API。
* [Pcap4J](https://github.com/kaitoy/pcap4j)：Pcap4J是一个用于捕获、制作和发送数据包的Java库。
* [JmDNS](https://github.com/jmdns/jmdns)：JmDNS是多播DNS的Java实现，支持服务发现和注册，并与Apple的Bonjour完全兼容。
* [Jocket](https://github.com/pcdv/jocket)：Jocket是使用共享内存的低延迟Java套接字实现。
* [Chronicle Network](https://github.com/OpenHFT/Chronicle-Network)：Chronicle Network是一个高性能网络(TCP/IP)库。
* [KryoNet](https://github.com/EsotericSoftware/kryonet)：KryoNet是一个Java库，它提供了一个干净、简单的API，用于使用NIO进行高效的TCP和UDP客户端/服务器网络通信。
* [COMSAT](https://github.com/puniverse/comsat)：COMSAT是一组开源库，将Quasar与各种Web或企业技术集成。
* [Infinileap](https://github.com/hhu-bsinfo/infinileap)：Infinileap是适用于Java 19+的基于ucx的现代网络框架，由杜塞尔多夫海因里希海涅大学计算机科学系操作系统小组开发。
* [TLS Channel](https://github.com/marianobarrios/tls-channel)：TLS Channel是一个通过TLS连接实现ByteChannel接口的库。
* [Voovan](https://gitee.com/helyho/Voovan)：Voovan是一个高性能异步网络框架和Web服务器框架。
* [JUnixSocket](https://github.com/kohlschutter/junixsocket)：JUnixSocket是一个Java/JNI库，允许在Java中使用Unix域套接字(AF_UNIX套接字)和其他地址/协议系列(AF_TIPC、AF_VSOCK和AF_SYSTEM)。
* [XNIO](https://github.com/xnio/xnio)：XNIO是一个基于Java NIO的框架，支持阻塞和非阻塞IO，由RedHat开源。
* [Acteur](https://github.com/timboudreau/acteur)：Acteur是一个使用Netty编写Web服务器应用程序的框架。
* [JXIO](https://github.com/accelio/JXIO)：JXIO是基于AccelIO(C库)的Java API。
* [SNMP4J](https://www.snmp4j.org/)：SNMP4J是一种企业级、免费开源且最先进的Java SNMP v1/2c/v3实现。
* [Commons Networking](https://github.com/CiscoSE/commons-networking)：由Cisco开源的公共网络实用程序库。
* [SimpleNet](https://github.com/jhg023/SimpleNet)：SimpleNet是一个用Java编写的简单的客户端/服务器框架。
* [DiSNI](https://github.com/zrlio/disni)：DiSNI是一个Java库，用于从用户空间直接存储和网络访问，它提供了一个RDMA接口来访问远程内存，由IBM开源。
* [Barchart UDT](https://github.com/barchart/barchart-udt)：Barchart UDT是原生C++ UDT协议的Java包装器。
* [Dragonite](https://github.com/dragonite-network/dragonite-java)：Dragonite是一种基于UDP的可靠应用级数据传输协议，针对有损和不稳定的网络进行了高度优化。
* [OkSocket](https://github.com/xuuhaoo/OkSocket)：OkSocket是一个Java库，旨在解决轻量级的Socket通信。
* [Java KCP](https://gitee.com/344453111/java-Kcp)：基于Netty实现的可靠UDP网络库(kcp算法)，包含fec实现。
* [Socket.D](https://gitee.com/noear/socket.d)：Socket.D是基于事件和语义消息流的网络应用协议。
* [PraxisCORE](https://github.com/praxis-live/praxiscore)：PraxisCORE是一个模块化JVM运行时，用于网络物理编程，支持实时系统的实时编码。
* [URNLib](https://github.com/slub/urnlib)：用于表示、解析和编码RFC 2141和RFC 8141中指定的URN的Java库，由德累斯顿工业大学开源。
* [Jcabi URN](https://github.com/jcabi/jcabi-urn)：Jcabi URN是根据RFC 2141的URN的不可变实现。
* [JSTUN](https://github.com/tking/JSTUN)：JSTUN是基于Java的STUN(通过网络地址转换(NAT)简单遍历UDP)实现。
* [Proxy Vole](https://github.com/MarkusBernhardt/proxy-vole)：Proxy Vole是一个用于自动检测平台网络代理设置的Java库。
* [JNet](https://gitee.com/eric_ds/jnet)：JNet框架是Java AIO接口的一层薄封装，仅进一步降低其编程复杂性，不提供额外的抽象。
* [JBoss Remoting](https://github.com/jboss-remoting/jboss-remoting)：JBoss Remoting的目的是提供一个用于通过网络进行对称和非对称通信的通用框架，由RedHat开发。

#### SSH工具

* [Bastillion](https://github.com/bastillion-io/Bastillion)：基于Web的SSH控制台，可集中管理对系统的管理访问。
* [ConnectBot](https://github.com/connectbot/connectbot)：适用于Android的安全Shell客户端，可让你通过加密安全链接连接到远程服务器。
* [Snowflake](https://github.com/subhra74/snowflake)：图形化SFTP客户端和终端仿真器以及有用的实用程序。
* [Termora](https://github.com/TermoraDev/termora)：Termora是一个终端模拟器和SSH客户端，支持Windows，macOS和Linux。
* [Apache MINA SSHD](https://github.com/apache/mina-sshd)：用于客户端和服务器端SSH的综合Java库。
* [Trilead SSH](https://github.com/jenkinsci/trilead-ssh2)：Trilead SSH-2是一个用纯Java实现SSH-2协议的库。
* [Pty4J](https://github.com/JetBrains/pty4j)：Java中的伪终端实现，由JetBrains开源。
* [JediTerm](https://github.com/JetBrains/jediterm)：纯Java终端模拟器，适用于SSH和PTY，由JetBrains开源。
* [JSch](https://github.com/mwiede/jsch)：实现SSH功能的Java库，可用于连接SFTP服务器。
* [Jcabi-SSH](https://github.com/jcabi/jcabi-ssh)：Java SSH客户端。
* [JSch](https://github.com/is/jsch)：JSch是SSH2的纯Java实现。
* [Maverick Synergy](https://github.com/sshtools/maverick-synergy)：下一代Java SSH API。
* [SSHJ](https://github.com/hierynomus/sshj)：以编程方式使用SSH、SCP或SFTP。
* [WebSSH](https://github.com/NoCortY/WebSSH)：纯Java实现的WebSSH。
* [T-Shell](https://github.com/TheBlindM/T-Shell)：T-Shell是一个可配置命令提示的终端模拟器和SSH客户端，目前只支持Windows。
* [SSH Proxy](http://github.com/cronn/ssh-proxy)：SSH端口隧道的纯Java实现，能够理解涉及多跳才能到达目标主机的OpenSSH配置。

#### DNS、内网穿透和代理

* [DNS66](https://github.com/julian-klode/dns66)：这是一款适用于Android的基于DNS的主机拦截器。
* [Decodo](https://github.com/Decodo/Decodo)：Decodo是一个轮换式住宅代理网络，允许用户使用超过1.15亿个IP地址池从网络收集数据。
* [DNSJava](https://github.com/dnsjava/dnsjava)：DNSJava是DNS协议的Java实现。
* [Neo-reGeorg](https://github.com/L-codes/Neo-reGeorg)：reGeorg是新一代内网穿透工具，这是该项目的重构版本。
* [Suo5](https://github.com/zema1/suo5)：Suo5是一个高性能HTTP隧道代理工具，它基于双向的Chunked-Encoding构建。
* [Neutrino Proxy](https://gitee.com/dromara/neutrino-proxy)：Neutrino-Proxy是一个基于Netty的开源Java内网穿透项目，由dromara社区开源。
* [BrowserUp Proxy](https://github.com/lightbody/browsermob-proxy)：BrowserMob Proxy允许你操作HTTP请求和响应、捕获HTTP内容以及将性能数据导出为HAR文件。
* [DNS Proxy](https://github.com/mageddo/dns-proxy-server)：DPS是一种轻量级最终用户DNS服务器工具，可以轻松地在一个主机名可以根据配置的环境解析为不同IP的系统中进行开发。
* [DNS Proxy](https://github.com/mageddo/dns-proxy-server)：DPS是一种轻量级最终用户DNS服务器工具，可以轻松地在一个主机名可以根据配置的环境解析为不同IP的系统中进行开发。
* [Apache Guacamole](https://github.com/apache/guacamole-client)：Guacamole是一个无客户端远程桌面网关，它支持VNC、RDP和SSH等标准协议。
* [Dns Cache Manipulator](https://github.com/alibaba/java-dns-cache-manipulator)：一个微小的0依赖线程安全Java库，用于以编程方式设置/查看DNS，无需接触host文件，使单元/集成测试可移植，由阿里开源。
* [Denominator](https://github.com/Netflix/denominator)：Denominator是一个用于操作DNS云的可移植Java库，由Netflix开源。
* [Happy DNS](https://github.com/qiniu/happy-dns-android)：用于Android的DNS库，由七牛云开源。
* [DNS Java](https://github.com/spotify/dns-java)：这个小型DNS包装器库提供了一些与SRV查找相关的有用功能，由Spotify开源。
* [DNS Cheater](https://gitee.com/matrixy/dns-cheater)：Java实现的DNS服务器，可通过Web管理界面随意设置灵活的解析规则。
* [Vert.x Http Proxy](https://github.com/eclipse-vertx/vertx-http-proxy)：Vert.x Http Proxy是基于Vert.x的反向代理，旨在实现可重用的反向代理逻辑以专注于更高的关注点。
* [MagpieBridge](https://gitee.com/jiucheng_org/magpiebridge)：使用Java基于AIO/NIO实现的内网穿透工具。
* [PacketProxy](https://github.com/DeNA/PacketProxy)：PacketProxy是一个开源代理工具，可以拦截和检查TCP/UDP上的任何协议，而不仅限于HTTP/1.x、HTTP2或HTTPS，由DeNA开源。
* [Proxyee](https://github.com/monkeyWie/proxyee)：Proxyee是一个Java编写的HTTP代理服务器库，支持HTTP、HTTPS、WebSocket协议，并支持MITM，可以捕获和篡改HTTP、HTTPS数据包。
* [OpenIG](https://github.com/OpenIdentityPlatform/OpenIG)：OpenIG是一种高性能反向代理服务器，具有专门的会话管理和凭证重播功能。
* [NoPE Proxy](https://github.com/summitt/Burp-Non-HTTP-Extension)：Burp Suite的非HTTP协议扩展代理和DNS。
* [HTTP Proxy Servlet](https://github.com/mitre/HTTP-Proxy-Servlet)：这是Java Servlet形式的HTTP代理。
* [Lanproxy](https://github.com/ffay/lanproxy)：Lanproxy是一个将局域网个人电脑、服务器代理到公网的内网穿透工具，支持TCP流量转发，可支持任何TCP上层协议。
* [S3Proxy](https://github.com/gaul/s3proxy)：S3Proxy实现S3 API和代理请求，支持多种用例。
* [PowerTunnel](https://github.com/krlvm/PowerTunnel)：PowerTunnel是一个构建在LittleProxy之上的可扩展代理服务器。
* [Styx](https://github.com/ExpediaGroup/styx)：Styx是用于JVM的可编程、异步、基于事件的反向代理，由Expedia开源。
* [LittleProxy](https://github.com/adamfisk/LittleProxy)：LittleProxy是一个用Java编写的高性能HTTP代理。
* [Joggle](https://github.com/joggle-cn/joggle)：Joggle是基于Ngrok二开的开源内网穿透项目，多节点、私有部署、云服务。
* [Carapaceproxy](https://github.com/diennea/carapaceproxy)：Carapac是一个分布式Java反向代理。

#### IO_Uring

* [Jasyncfio](https://github.com/ikorennoy/jasyncfio)：Jasyncfio提供了基于Linux io_uring接口的异步文件I/O API。
* [NIO_Uring](https://github.com/bbeaupain/nio_uring)：NIO_Uring是一个Java I/O库，它在底层使用io_uring。
* [Netty io_uring](https://github.com/netty/netty-incubator-transport-io_uring)：Netty为io_uring提供的实验性支持。
* [PanamaUring](https://github.com/dreamlike-ocean/PanamaUring)：这是一个探索性质的项目，使用Java的新FFI为Java引入io_uring。
* [JUring](https://github.com/davidtos/JUring)：JUring是一个高性能Java库，它使用Java的FFM API提供与Linux的io_uring异步I/O接口的绑定。

#### 网络工具

* [Tsunami](https://github.com/google/tsunami-security-scanner)：Tsunami是一款通用网络安全扫描器，具有可扩展的插件系统，可高置信度地检测高严重性漏洞，由Google开源。
* [ONOS](https://github.com/opennetworkinglab/onos)：ONOS是一个开源SDN网络操作系统，主要面向服务提供商和企业骨干网，由Linux基金会开源。
* [OpenVirteX](https://github.com/os-libera/OpenVirteX)：OVX是一个网络管理程序，可以在单个物理基础设施之上创建多个虚拟和可编程网络。
* [Angry IP Scanner](https://github.com/angryip/ipscan)：Angry IP Scanner是适用于Windows、Linux和Mac的快速且友好的网络扫描器。
* [jNetMap](https://rakudave.ch/jnetmap/)：jNetMap是一个图形网络监控和文档工具，它会每x分钟ping所有注册的设备，并根据ping的结果更新状态。
* [Discourse Network Analyzer](https://github.com/leifeld/dna)：Discourse Network Analyzer是一种具有网络导出功能的定性内容分析工具。
* [Batfish](https://github.com/batfish/batfish)：Batfish是一种网络验证工具，通过分析网络设备的配置，为安全性、可靠性和合规性提供正确性保证。
* [FDT](https://github.com/fast-data-transfer/fdt)：FDT是一种高效数据传输应用程序，能够通过广域网(使用标准TCP)以磁盘速度读写。
* [ANX](https://github.com/cisco-ie/anx)：适用于Java的高级NETCONF浏览器和NETCONF客户端库，Cisco开源。

#### 端口转发

* [Cling](https://github.com/4thline/cling)：Cling致力于用Java创建一个兼容UPnP的软件栈。
* [UPnP PortMapper](https://github.com/kaklakariada/portmapper)：UPnP PortMapper是一个易于使用的程序，用于管理本地网络中启用UPnP的互联网网关设备的端口映射。
* [Port Mapper](https://github.com/offbynull/portmapper)：Port Mapper项目是一个Java库，可用于在启用NAT的路由器上转发端口。
* [WeUPnP](https://github.com/bitletorg/weupnp)：WeUPnP是用Java编写的小型UPnP客户端库。
* [Cybergarage UPnP](https://github.com/cybergarage/cybergarage-upnp)：Cybergarage UPnP是面向Java开发人员的UPnP开发包。
* [WaifUPnP](https://github.com/adolfintel/WaifUPnP)：WaifUPnP是UPnP的一个非常基本的实现。

#### SDN

* [OpenDaylight](https://github.com/opendaylight)：OpenDaylight由Linux基金会支持，其目标在于开发支援软件定义网络(SDN)的各种软件工具，建立网络功能虚拟化的基础。
* [Floodlight](https://github.com/floodlight/floodlight)：Floodlight是领先的开源OpenFlow控制器，由Cisco维护。
* [Lighty](https://github.com/PANTHEONtech/lighty)：Lighty是一个由OpenDaylight提供支持的SDK，用于支持、简化和加速Java中软件定义网络(SDN)解决方案的开发，由PANTHEON开源。

#### SMB

* [jCIFS  NG](https://github.com/AgNO3/jcifs-ng)：jCIFS库的清理和改进版本。
* [SMBJ](https://github.com/hierynomus/smbj)：Java中的服务器消息块(SMB2、SMB3)实现。
* [JCIFS](https://github.com/codelibs/jcifs)：JCIFS是一个开源客户端库，以纯Java实现CIFS/SMB网络协议。

#### QUIC

* [Kwik](https://github.com/ptrd/kwik)：Kwik是QUIC协议Java的实现，Kwik最初仅作为客户端，但自2021年5月起它支持客户端和服务器。
* [Quiche4j](https://github.com/kachayev/quiche4j)：QUIC传输协议和HTTP/3的Java实现。
* [Netty QUIC Codec](https://github.com/netty/netty-incubator-codec-quic)：这是利用quiche的Netty的新型实验性QUIC编解码器。

#### SOCKS

* [JSocks](https://github.com/ravn/jsocks)：JSocks是一个完全用Java编写的SOCKS服务器，同时支持SOCKS4和SOCKS5协议。
* [SocksLib](https://github.com/fengyouchao/sockslib)：SocksLib是一个针对SOCKS5协议的Java库。
* [Java SOCKS Proxy Server](https://github.com/bbottema/java-socks-proxy-server)：Java SOCKS Proxy Server是Java的SOCKS 4/5服务器。

#### 以太网

* [Nzyme](https://github.com/nzymedefense/nzyme)：借助Nzyme，你可以监控所有以太网和WiFi网络流量是否存在威胁、确认预期行为并有选择地将数据转发到你的SIEM或日志管理系统。
* [EtherNet/IP](https://github.com/digitalpetri/ethernet-ip)：适用于Java的异步、非阻塞EtherNet/IP客户端实现。
* [CICFlowMeter](https://github.com/ahlashkari/CICFlowMeter)：CICFlowMeter是一种用于异常检测的以太网流量双向流生成器和分析器，由约克大学开发。

#### IP操作库

* [Ip2region](https://github.com/lionsoul2014/ip2region)：Ip2region是一个离线IP地址定位库和IP定位数据管理框架，提供了众多主流编程语言的xdb数据生成和查询客户端实现。
* [IPAddress](https://github.com/seancfoley/IPAddress)：IPAddress是用于处理IP地址和子网(IPv4和IPv6)的Java库。
* [Qqwry Java](https://github.com/jarod/qqwry-java)：纯真IP地址数据库。
* [Commons IP Math](https://github.com/jgonian/commons-ip-math)：Commons IP Math提供了丰富、类型安全的API，用于处理对IP资源执行的最常见操作，例如解析、以多种表示法打印、检查范围是否重叠或可以合并等。
* [Java IPv6](https://github.com/janvanbesien/java-ipv6)：Java IPv6是一个用于IPv6相关概念的Java库，例如IPv6地址、网络掩码、地址池等。
* [IpDB Java](https://github.com/ipipdotnet/ipdb-java)：IPIP.net官方支持的IP数据库ipdb格式解析库。
* [IP Info](https://gitee.com/jthinking/ip-info)：IP地理位置获取，支持获取IPv4、IPv6地址信息。

## 状态机

* [Squirrel](https://github.com/hekailiang/squirrel)：Squirrel为Java提供了一个易于使用、类型安全且高度可扩展的状态机实现。
* [Spring Statemachine](https://github.com/spring-projects/spring-statemachine)：Spring Statemachine项目提供了一个通用的基础设施来在Spring应用程序中使用状态机概念。
* [Stateless4j](https://github.com/stateless4j/stateless4j)：轻量级Java状态机。
* [EasyFlow](https://github.com/Beh01der/EasyFlow)：EasyFlow是一个简单且轻量级的Java有限状态机。
* [KStateMachine](https://github.com/KStateMachine/kstatemachine)：KStateMachine是一个强大的Kotlin多平台库，具有清晰的DSL语法，用于创建由Kotlin Coroutines驱动的复杂状态机和状态图。
* [Easy States](https://github.com/j-easy/easy-states)：Easy States是Java中事件驱动的确定性有限自动机实现。
* [StatefulJ](https://github.com/statefulj/statefulj)：StatefulJ是一个轻量级、开源的Java事件驱动有限状态机(FSM)和一个完整的基于Spring Data的框架，可让你轻松定义状态机并将其集成到你的应用程序中。
* [nFlow](https://github.com/NitorCreations/nflow)：nFlow是一种经过验证的用于编排业务流程的解决方案。
* [Winder](https://github.com/eBay/Winder)：Winder是一个基于Quartz的简单状态机，由eBay开源。
* [State Machine](https://github.com/davidmoten/state-machine)：Java的有限状态机类生成器。
* [Makina](https://github.com/clnhlzmn/makina)：Makina是一个分层状态机源到源转换器，它将状态机描述作为输入并生成这些状态机的C语言实现。
* [JState](https://github.com/UnquietCode/JState)：Java中的高级状态机。
* [State Machine Compiler](https://smc.sourceforge.net/)：SMC最大限度地利用了状态模式，允许你的对象处理意外事件、恢复并继续提供服务(而不是崩溃)的转换。
* [Morfologik Stemming](https://github.com/morfologik/morfologik-stemming)：用于有限状态自动机构建和基于字典的形态词典的工具。
* [TSM4j](https://github.com/weilueluo/tsm4j)：Java的类型化状态机。
* [Dk.Brics.Automaton](https://github.com/cs-au-dk/dk.brics.automaton)：Java的有限状态自动机和正则表达式，由奥胡斯大学开源。

## 二维码生成器

* [ZXing](https://github.com/zxing/zxing)：适用于Java、Android的ZXing条码扫描库。
* [QR-Code-Generator](https://github.com/nayuki/QR-Code-generator)：Java、TypeScript/JavaScript、Python、Rust、C++、C语言的高质量QR码生成器库。
* [ZXingLite](https://github.com/jenly1314/ZXingLite)：ZXing的精简极速版，优化扫码和生成二维码/条形码，内置闪光灯等功能。
* [Aspose.BarCode](https://products.aspose.com/barcode/java/)：Aspose.BarCode是一个强大且可靠的API，为Java应用程序提供条形码生成和识别功能。
* [Spire.Barcode](https://www.e-iceblue.com/Introduce/barcode-for-java.html)：Spire.Barcode是一款专业的条形码组件，专为开发人员在Java应用程序上生成、读取和扫描1D和2D条形码而设计。
* [QArt4J](https://github.com/dieforfree/qart4j)：一个QR码生成器，可提供ASCII Art输出图像。
* [Barbecue](https://barbecue.sourceforge.net/)：一个开源Java库，支持广泛的一维条形码格式。
* [Barcode4J](https://barcode4j.sourceforge.net/)：提供二维条形码格式(例如DataMatrix和PDF417)以及更多输出格式。
* [QRGen](https://github.com/kenglxn/QRGen)：一个基于ZXING构建的简单的Java二维码生成API。
* [Java-OCR-API](https://github.com/Asprise/java-ocr-api)：Java OCR允许你对图像(JPEG、PNG、TIFF、PDF等)执行OCR和条形码识别，并输出为纯文本、具有完整坐标的XML以及可搜索的PDF。
* [ZXingGenerator](https://github.com/vivian8725118/ZXingGenerator)：花式二维码生成库，提供了6种样式。
* [React-QR-Code](https://github.com/rosskhanas/react-qr-code)：用于React和React Native的QR代码生成器。
* [Visual-QR-Code](https://gitee.com/boat824109722/visual-qr-code)：可以创建出设置了虚拟背景图片的二维码。
* [QRext4j](https://gitee.com/BYSRepo/qrext4j)：一个简单易用的二维码生成工具，可自定义二维码颜色和码眼样式。
* [FiwanQRCode](https://gitee.com/frogchou/FiwanQRCode)：飞网开发的二维码生成工具。
* [WeChatQRCode](https://github.com/jenly1314/WeChatQRCode)：WeChatQRCode是一个基于OpenCV开源的微信二维码引擎移植封装的二维码识别库。
* [QRCode Generator](https://github.com/kazuhikoarase/qrcode-generator)：以JavaScript、Java等语言实现的QR码生成器。
* [QRGenerator](https://github.com/androidmads/QRGenerator)：二维码生成器库。
* [QRCode-Utils](https://github.com/binarywang/qrcode-utils)：二维码生成工具。
* [Okapi Barcode](https://github.com/woo-j/OkapiBarcode)：Okapi Barcode是一款完全用Java编写的开源条形码生成器，支持50多种编码标准，包括所有ISO标准。
* [EMV QRCode](https://github.com/mvallim/emv-qrcode)：基于Java的EMV二维码生成器和解析器(MPM、CPM)。
* [Barcode Java](https://github.com/barnhill/barcode-java)：该库旨在为开发人员提供一个简单的类，供开发人员在需要从一串数据生成条形码图像时使用。
* [SwissQRBill](https://github.com/manuelbl/SwissQRBill)：用于生成和解码瑞士二维码钞票的开源Java库。

## 文件系统

* [HDFS](https://github.com/apache/hadoop)：Hadoop软件库是一个框架，允许使用简单的编程模型跨计算机集群分布式处理大型数据集，由Yahoo开源。
* [Jimfs](https://github.com/google/jimfs)：Jimfs是Java 8及更高版本的内存文件系统，实现了java.nio.file抽象文件系统API，由Google开源。
* [DCache](https://github.com/dCache/dcache)：DCache是一个用于存储和检索分布在大量异构服务器节点中数据的系统，由费米实验室、德国电子加速器、北欧数据网格设施共同开源。
* [XtreemFS](https://github.com/xtreemfs/xtreemfs)：XtreemFS是一个用于联合IT基础设施的分布式、可复制和容错的文件系统，由柏林自由大学开源。
* [Memory File System](https://github.com/marschall/memoryfilesystem)：用于测试目的的JSR-203文件系统的内存实现。
* [RubiX](https://github.com/qubole/rubix)：RubiX是一个可供大数据引擎使用的轻量级数据缓存框架。
* [ADFS](https://github.com/taobao/ADFS)：ADFS是Hadoop的演进版本，提供高可用性、自动重启等特性，由阿里开源。
* [TngouFS](https://gitee.com/397713572/tngouFS)：TngouFS是天狗网用于该网站的图片存储。
* [JFileServer](https://github.com/FileSysOrg/jfileserver)：JFileServer是一个基于Java的文件服务器，目前支持SMB/CIFS、FTP/FTPS和NFS协议。
* [Apache Commons VFS](https://github.com/apache/commons-vfs)：Commons VFS是一个虚拟文件系统库。
* [FastDFS Client](https://github.com/happyfish100/fastdfs-client-java)：FastDFS Java客户端SDK。
* [FastDFS Client](https://github.com/tobato/FastDFS_Client)：FastDFS的Java客户端。
* [JNR FUSE](https://github.com/SerCeMan/jnr-fuse)：JNR FUSE是使用Java Native Runtime的Java中的FUSE实现。
* [NFS4J](https://github.com/dCache/nfs4j)：NFS服务器版本3、4.0和4.1的纯Java实现，包括带有nfs4.1-files和flex-files布局类型的pNFS扩展。
* [NFS Java Client](https://github.com/EMCECS/nfs-client-java)：该项目是一个NFS Java客户端，具有一些额外的抽象，允许扩展处理其他NFS版本(目前仅处理NFS v3)，由EMC开源。
* [JavaFS](https://github.com/puniverse/javafs)：Java文件系统，由FUSE提供支持。
* [ParallelGit](https://github.com/freelunchcap/ParallelGit)：适用于Git的高性能Java 7 NIO内存文件系统。
* [Amazon S3 FileSystem NIO2](https://github.com/Upplication/Amazon-S3-FileSystem-NIO2)：适用于Java 7(NIO2)的Amazon AWS S3文件系统提供程序。
* [Google NIO Filesystem](https://github.com/googleapis/java-storage-nio)：用于Google Cloud Storage的NIO文件系统提供程序的Java客户端。
* [FUSE-Java](https://github.com/EtiennePerot/fuse-jna)：使用JNA的Java FUSE绑定
* [JSR-203 Hadoop](https://github.com/damiencarol/jsr203-hadoop)：Hadoop分布式文件系统的JSR 203实现。
* [S3FS NIO](https://github.com/carlspring/s3fs-nio)：这是使用Java 8的JSR-203的Amazon AWS S3文件系统提供程序的实现。
* [Hadoop-COS](https://github.com/tencentyun/hadoop-cos)：Hadoop-COS实现了以腾讯云COS作为底层文件系统运行上层计算任务的功能，支持使用Hadoop、Spark以及Tez等处理存储在腾讯云COS对象存储系统上的数据。
* [Hadoop-20](https://github.com/facebookarchive/hadoop-20)：Facebook基于Hadoop 0.20-append的实时分布式FS。
* [SDFS](https://github.com/opendedup/sdfs)：一种去重文件系统，可以将数据存储在对象存储或块存储中。
* [N5](https://github.com/saalfeldlab/n5)：N5 API指定了存储大块n维张量以及类似于HDF5的组层次中的任意元数据所需的原始操作。
* [Sfs](https://github.com/pitchpoint-solutions/sfs)：PitchPoint开发的传统少云对象存储服务器能够以极小的资源存储存储亿个大小的文件。

## IPFS

* [Java IPFS HTTP Client](https://github.com/ipfs-shipyard/java-ipfs-http-client)：HTTP IPFS API的Java实现。
* [Mahuta](https://github.com/Consensys/Mahuta)：Mahuta是一个用于聚合和整合应用程序在IPFS网络上存储的文件或文档的库。
* [Nabu](https://github.com/Peergos/nabu)：IPFS的最小Java实现。

## 物流系统

* [RinSim](https://github.com/rinde/RinSim)：RinSim是一个用Java编写的物流模拟器，支持动态取货和送货问题的(去)中心化算法，由比利时鲁汶大学计算机科学系部门的imec-DistriNet小组开发。
* [OpenWMS](https://github.com/openwms/org.openwms)：OpenWMS是一个免费使用且可扩展的仓库管理系统(WMS)，带有适用于自动和手动仓库的物料流控制(MFC)系统。
* [Deer WMS](https://gitee.com/deerwms/deer-wms-2)：Deer WMS是基于自动化输送线、机械臂、点数机、提升机、堆垛机等自动化设备和现代化仓储精益管理思想开发出来的仓库管理系统，由南京大鹿智造开发。
* [myWMS](https://github.com/wms2/mywms)：myWMS LOS是开源仓库管理系统WMS，它在工业24/7环境中运行并支持所有基本流程。
* [OpenBoxes](https://github.com/openboxes/openboxes)：OpenBoxes是一个开源供应链管理系统，用于管理医疗机构和救灾工作的物资和药物。

## 打包部署运行

* [OneinStack](https://github.com/oneinstack/oneinstack)：OneinStack是一个PHP/Java部署工具。
* [Capsule](https://github.com/puniverse/capsule)：Capsule是JVM应用程序的打包和部署工具。
* [Rultor](https://github.com/yegor256/rultor)：Rultor是一个DevOps团队助理，它通过易于使用的直观聊天机器人界面帮助你自动执行日常操作(合并、部署和发布)。
* [jDeploy](https://github.com/shannah/jdeploy)：jDeploy Github Action允许你在Github工作流中为Java项目生成本机桌面安装程序。
* [JReleaser](https://github.com/jreleaser/jreleaser)：JReleaser是一个用于Java和非Java项目的自动化发布工具。
* [Stork](https://github.com/fizzed/stork)：Stork是一个轻量级实用程序的集合，用于通过填补Java构建系统和执行之间的空白来优化“构建后”工作流程。
* [WinRun4j](https://github.com/poidasmith/winrun4j)：WinRun4j是适用于Windows的Java启动器。
* [ShrinkWrap](https://github.com/shrinkwrap/shrinkwrap)：ShrinkWrap是一个Java库，用于创建Java存档，例如JAR、WAR、EAR和RAR，由JBoss开源。
* [Linux Deploy](https://github.com/meefik/linuxdeploy)：可在Android设备上快速轻松地安装操作系统GNU/Linux。
* [CloudCaptain](https://cloudcaptain.sh/)：使用不可变基础设施的原则将JVM应用程序部署到AWS。
* [Getdown](https://github.com/threerings/getdown)：Getdown是一个用于将Java应用程序部署到最终用户计算机并保持这些应用程序最新的系统。
* [JavaPackager](https://github.com/fvarrui/JavaPackager)：JavaPackager是Maven和Gradle的混合插件，它提供了一种在本机Windows、MacOS或GNU/Linux可执行文件中打包Java应用程序并为其生成安装程序的简单方法。
* [JPackage](https://github.com/Akman/jpackage-maven-plugin)：JPackage插件允许你使用Java 14中引入的jpackage工具创建自定义运行时镜像/安装程序。
* [JVMKill](https://github.com/airlift/jvmkill)：JVMKill是一个简单的JVMTI代理，当JVM无法分配内存或创建线程时，它会强制终止JVM。
* [Drip](https://github.com/ninjudd/drip)：Drip是Java虚拟机的启动器，它提供比java命令更快的启动时间。
* [Teletraan](https://github.com/pinterest/teletraan)：Teletraan是Pinterest的部署系统。
* [Trampoline](https://github.com/ErnestOrt/Trampoline)：Trampoline是一个开源项目，可帮助你在开发阶段启动和停止基于Spring Boot的服务。
* [Kayenta](https://github.com/spinnaker/kayenta)：Kayenta是一个自动金丝雀分析(ACA)平台。
* [Spring Boot Thin Launcher](https://github.com/spring-projects-experimental/spring-boot-thin-launcher)：用于Java应用程序的瘦JAR启动器。
* [JApp](https://github.com/Glavo/japp)：Java程序的新打包格式。
* [JLink.Online](https://github.com/AdoptOpenJDK/jlink.online)：JLink.Online是一个HTTP微服务，可动态构建优化/最小化的Java运行时。
* [Badass](https://github.com/beryx/badass-runtime-plugin)：创建非模块化应用程序的自定义运行时镜像。
* [Layrry](https://github.com/moditect/layrry)：Layrry是一个启动器和Java API，用于执行模块化Java应用程序。
* [Nailgun](https://github.com/facebookarchive/nailgun)：Nailgun是一个客户端、协议和服务器，用于从命令行运行Java程序，而不会产生JVM启动开销，由Facebook开源。
* [SlimFast](https://github.com/HubSpot/SlimFast)：SlimFast是Java应用程序的一个工具，可帮助它们停止构建用于部署的fat jar，由HubSpot开源。
* [Jar Jar Links](https://github.com/google/jarjar)：Jar Jar Links是一个实用程序，可以轻松地重新打包Java库并将它们嵌入到你自己的发行版中。
* [ExeBuilder](https://gitee.com/qsyan/ExeBuilder)：ExeBuilder是一款利用JDK模块化的特性帮你把jar打包成独立exe的工具，它支持GUI和控制台应用程序的创建。
* [IzPack](https://github.com/izpack/izpack)：IzPack是一种广泛使用的工具，用于将Java平台上的应用程序打包为跨平台安装程序。
* [Packr](https://github.com/libgdx/packr)：用于打包JAR、资源和JVM，以便在Windows、Linux和Mac OS X上分发。
* [Update4j](https://github.com/update4j/update4j)：Update4j是第一个专为Java 9+设计的自动更新和启动器库。
* [Install4j](https://www.ej-technologies.com/products/install4j/overview.html)：Install4j是一个功能强大的多平台Java安装程序生成器，可生成Java应用程序的本机安装程序和应用程序启动器。
* [JWrapper](https://www.jwrapper.com/)：JWrapper是一款高端现代Java安装程序，可用于部署Java应用程序。
* [InstallAnywhere](https://www.revenera.com/install/products/installanywhere)：InstallAnywhere使开发人员可以轻松创建具有相同功能的专业安装软件-无论平台是什么。
* [JNE](https://github.com/fizzed/jne)：用于在jar文件中打包本机库、可执行文件和运行时资源的Java库。

## 地理空间

* [GeoTools](https://github.com/geotools/geotools)：GeoTools是一个开源Java库，它提供符合标准的方法来操作地理空间数据，例如实现地理信息系统(GIS)，由英国利兹大学开源。
* [Open Location Code](https://github.com/google/open-location-code)：Open Location Code是一种将位置编码为比纬度和经度更易于使用的形式的技术，由Google开源。
* [Gisgraphy](https://github.com/gisgraphy/gisgraphy)：Gisgraphy提供正向和反向地理编码、地理定位和车辆跟踪Web服务。
* [MeteoInfo](https://github.com/meteoinfo/MeteoInfo)：MeteoInfo是GIS应用(MeteoInfoMap)、科学计算和可视化环境(MeteoInfoLab)的集成框架，特别适合气象界，由中国气象科学研究院开源。
* [Apache SIS](https://github.com/apache/sis)：SIS是一个用于开发地理空间应用程序的Java语言库。
* [Geo Assist](https://github.com/thegeekyasian/geo-assist)：Geo Assist是一个开源Java库，旨在简化空间数据的处理过程。
* [Proj4J](https://github.com/locationtech/proj4j)：Proj4J是一个用于在不同地理空间坐标参考系之间转换坐标的Java库，这是一个Eclipse基金会项目。
* [GeoMesa](https://github.com/locationtech/geomesa)：GeoMesa是一套开源工具，可在分布式计算系统上进行大规模地理空间查询和分析。
* [Barefoot](https://github.com/bmwcarit/barefoot)：Barefoot是一个宝马开源的Java库，用于与OpenStreetMap进行在线和离线地图匹配。
* [Deegree](https://github.com/deegree/deegree3)：Deegree是用于空间数据基础设施和地理空间网络的开源软件，Deegree包含地理空间数据管理组件，包括数据访问、可视化、发现和安全性，由德国波恩大学地理系开发。
* [Mapsforge](https://github.com/mapsforge/mapsforge)：Mapsforge是一个Android、Java平台可用的地图库，支持OpenStreetMap地图数据的离线呈现。
* [SeaRoute](https://github.com/eurostat/searoute)：SeaRoute可以计算两个地点之间的最短海上路线，由欧盟统计局开源。
* [GAMA](https://github.com/gama-platform/gama.old)：GAMA是一个易于使用的开源建模和仿真环境，用于创建基于代理的空间显式仿真。
* [MrGeo](https://github.com/ngageoint/mrgeo)：MrGeo是一个地理空间工具包，旨在提供可大规模执行的基于栅格的地理空间功能，由美国国家地理空间情报局与DigitalGlobe合作开发。
* [Tinfour](https://github.com/gwlucastrig/Tinfour)：Tinfour是一个用Java编写的软件库，提供了用于构建和应用符合Delaunay准则的不规则三角网络(TIN)的工具。
* [NoiseModelling](https://github.com/Universite-Gustave-Eiffel/NoiseModelling)：NoiseModelling是一个能够生成噪声图的库，由古斯塔夫埃菲尔大学开源。
* [Spatial4j](https://github.com/locationtech/spatial4j)：Spatial4j是一个通用空间/地理空间开源Java库，其核心功能有三重：提供常见的地理空间感知形状，提供距离计算和其他数学运算，以及读取形状并将其写入字符串。
* [geOrchestra](https://github.com/georchestra/georchestra)：geOrchestra是一个完整的空间数据基础设施解决方案。
* [OrbisGIS](https://github.com/orbisgis/orbisgis)：OrbisGIS是一个跨平台开源地理信息系统(GIS)，由法国Lab-STICC实验室内的CNRS领导。
* [GeoServer](https://github.com/geoserver/geoserver)：GeoServer是一个用Java编写的开源软件服务器，允许用户共享和编辑地理空间数据。
* [Geohash Java](https://github.com/kungfoo/geohash-java)：Geohashes的纯Java实现。
* [GeoIP2 Java](https://github.com/maxmind/GeoIP2-java)：用于GeoIP2 Web服务客户端和数据库读取器的Java API。
* [GeoFire Java](https://github.com/firebase/geofire-java)：GeoFire是一个Java开源库，允许你根据地理位置存储和查询一组密钥，由Google开源。
* [GeoWave](https://github.com/locationtech/geowave)：GeoWave在Accumulo、HBase、BigTable、Cassandra、Kudu、Redis、RocksDB和DynamoDB之上提供地理空间和时间索引。
* [Geo Platform](https://github.com/geosdi/geo-platform)：Geo-Plaform是一个开发富Web GIS应用程序的框架，由意大利国家研究委员会开源。
* [Geotoolkit](https://github.com/Geomatys/geotoolkit)：Geotoolkit是一个开源库，提供了操作制图数据的工具。
* [Photon](https://github.com/komoot/photon)：Photon是一个为OpenStreetMap数据构建的开源地理编码器。
* [Geo](https://github.com/davidmoten/geo)：用于地理哈希的Java实用方法。
* [Beast](https://bitbucket.org/bdlabucr/beast)：Beast是一个针对时空数据进行大型探索性分析的系统，支持矢量和栅格数据，具有多维数据类型和索引结构，由加州大学河滨分校开源。
* [SEPAL](https://github.com/openforis/sepal)：SEPAL是一个用于地理数据处理的云计算平台。
* [GeoPackage Java](https://github.com/ngageoint/geopackage-java)：GeoPackage是开放地理空间联盟GeoPackage规范的Java实现，由美国国家地理空间情报局开源。
* [GeoGig](https://github.com/locationtech/geogig)：GeoGig是一个地理空间分布式版本控制系统。
* [GeoWebCache](https://github.com/GeoWebCache/geowebcache)：GeoWebCache是一个用Java实现的图块缓存服务器，提供各种图块缓存服务，如WMS-C、TMS、WMTS、谷歌Maps、MS Bing等。
* [CLAVIN](https://github.com/Novetta/CLAVIN)：CLAVIN是一个开源软件包，用于文档地理解析和地理分辨率，采用基于上下文的地理实体分辨率。
* [GeoNetwork](https://github.com/geonetwork/core-geonetwork)：GeoNetwork是一个用于管理空间参考资源的目录应用程序，它提供强大的元数据编辑和搜索功能以及交互式Web地图查看器，由OSGeo开源。
* [DHIS 2](https://github.com/dhis2/dhis2-core)：DHIS 2是一个灵活、基于Web的开源信息系统，具有出色的可视化功能，包括GIS、图表和数据透视表，由奥斯陆大学HISP中心开发。
* [地图瓦片图下载器](https://gitee.com/CrimsonHu/java_map_download)：使用Java开发的地图瓦片图下载工具，支持OpenStreetMap、天地图、谷歌地图、高德地图、腾讯地图、必应地图的XYZ瓦片图下载与合并。
* [GAF](https://gitee.com/supermapgaf/GAF)：SuperMap GAF基于SuperMap GIS基础软件进行研发，是连接GIS基础软件与行业应用的重要纽带。
* [JPX](https://github.com/jenetics/jpx)：JPX是一个Java库，用于创建、读取和写入GPX格式的GPS数据。
* [GeoDesk](https://github.com/clarisma/geodesk)：GeoDesk是一个用于OpenStreetMap数据的快速且存储高效的地理空间数据库。
* [Timeshape](https://github.com/RomanIakovlev/timeshape)：Timeshape是一个Java库，可用于确定给定地理坐标属于哪个时区。
* [Traccar](https://github.com/traccar/traccar)：Traccar是一个开源GPS跟踪系统，支持200多种GPS协议和2000多种GPS跟踪设备型号。
* [goGPS](https://github.com/goGPS-Project/goGPS_Java)：goGPS是一个软件包，旨在通过相对定位提高低成本GPS设备的定位精度，由大阪市立大学维护。
* [Apache Sedona](https://github.com/apache/sedona)：Sedona是一种空间计算引擎，使开发人员能够在Spark和Flink等现代集群计算系统中轻松处理任何规模的空间数据，由亚利桑那州立大学开源。
* [H3-Java](https://github.com/uber/h3-java)：该库为H3核心库提供Java绑定，由Uber开源。
* [Planetiler](https://github.com/onthegomap/planetiler)：Planetiler是一种从OpenStreetMap等地理数据源生成矢量切片的工具。
* [ElasticGeo](https://github.com/ngageoint/elasticgeo)：ElasticGeo提供了一个GeoTools数据存储，允许使用GeoServer通过OGC服务发布Elasticsearch索引中的地理空间特征，由美国国家地理空间情报局开源。
* [Unfolding](https://github.com/tillnagel/unfolding)：Unfolding是一个用Processing和Java创建交互式地图和地理可视化的库。
* [H2GIS](https://github.com/orbisgis/h2gis)：H2数据库的空间扩展，由法国Lab-STICC实验室内CNRS的GIS和信息科学领域的科学家和工程师领导。
* [Jgeohash](https://github.com/astrapi69/jgeohash)：一个易于实现的库，可以帮助Java开发人员使用GeoHash算法来创建基于自定义纬度和经度值的地理编码。
* [OSHDB](https://github.com/GIScience/oshdb)：OpenStreetMap全历史数据的高性能时空数据分析平台，由海德堡大学开源。
* [Proj4J](https://github.com/Proj4J/proj4j)：Proj4J是一个Java库，用于将点坐标从一个地理坐标系转换到另一个地理坐标系，包括基准面转换。
* [OpenJUMP](https://github.com/openjump-gis/openjump)：OpenJUMP诞生于JUMP，JUMP是一个开源GIS，最初由Vividsolutions用Java开发，并由不列颠哥伦比亚省(加拿大)自然资源部资助。
* [Nunaliit](https://github.com/GCRC/nunaliit)：Nunaliit是一个用于创建交互式、数据驱动的网络地图集的系统，支持用户编辑文档和几何图形、集成多媒体、文档关系、灵活的数据模式、自我复制、动态推送更新到浏览器以获取新对象，以及基于平板电脑的在线/离线编辑和同步；由卡尔顿大学地理信息学和制图研究中心(GCRC)的一个团队开发。
* [Apache Baremaps](https://github.com/apache/incubator-baremaps)：Baremaps是一个工具包和一组用于创建、发布和操作在线地图的基础设施组件。
* [Wilayah Indonesia](https://github.com/yusufsyaifudin/wilayah-indonesia)：印度尼西亚行政地图。
* [SimpleLatLng](https://github.com/JavadocMD/simplelatlng)：SimpleLatLng提供了一个简单、轻量级的库，可满足Java中常见的纬度和经度计算需求。
* [CityGML4j](https://github.com/citygml4j/citygml4j)：CityGML4j是OGC CityGML的开源Java库和API。
* [GeoAPI](https://github.com/opengeospatial/geoapi)：GeoAPI为地理空间应用程序提供了一组Java和Python语言编程接口。
* [THREDDS Data Server](https://github.com/Unidata/tds)：TDS提供对科学数据集的元数据和数据访问，由美国国家科学基金会开源。
* [JGiscoTools](https://github.com/eurostat/JGiscoTools)：JGiscoTools是一个用于操作地理空间和统计数据的Java库，重点关注Eurostat和Eurostat-GISCO生成的欧洲数据，由欧盟统计局开源。
* [Time Zone Map](https://github.com/dustin-johnson/timezonemap)：用于将位置或地区映射到时区的Java/Kotlin库。
* [OpenMap](https://github.com/OpenMap-java/openmap)：OpenMap是一个基于Java Beans的工具包，用于构建需要地理信息的应用程序和Applet，由雷神公司开源。
* [MapLibre Tiles](https://github.com/maplibre/maplibre-tile-spec)：下一代矢量切片格式。
* [Mago 3DTiler](https://github.com/Gaia3D/mago-3d-tiler)：Mago 3DTiler是一款OGC 3D Tiles工具，由Gaia3D开源。
* [Maxent](https://github.com/mrmaxent/Maxent)：Maxent是一个独立的Java应用程序，用于对物种地理分布进行建模，由美国自然历史博物馆开源。
* [JavAPRSlib](https://github.com/ab0oo/javAPRSlib)：JavAPRSlib是一个用于解析和创建APRS数据包的Java库。
* [GeoGeometry](https://github.com/jillesvangurp/geogeometry)：GeoGeometry是一组用于处理地理哈希和具有地理坐标的几何形状的算法和函数。
* [Mapbox Java SDK](https://github.com/mapbox/mapbox-java)：Mapbox Java SDK包含路线、地理编码以及更多可在Android或Java应用程序中使用的API。
* [Location4j](https://github.com/tomaytotomato/location4j)：Location4j是一个简单的Java库，旨在高效、准确地查找国家、州和城市的地理数据。

## 大地测量

* [JAG3D](https://github.com/applied-geodesy/jag3d)：JAG3D是用于大地测量科学的最流行的开源最小二乘软件包之一，由斯坦拜斯转运中心开源。
* [Geodesy](https://github.com/mgavaghan/geodesy)：这是实现Thaddeus Vincenty算法的Java源代码，用于解决正向和逆向大地测量问题。
* [CTS](https://github.com/orbisgis/cts)：CTS是一个为使用众所周知的大地测量算法和参数集执行坐标变换而开发的库，由法国Lab-STICC实验室开源。

## 路由引擎

* [GraphHopper](https://github.com/graphhopper/graphhopper)：OpenStreetMap的开源路由引擎，可以将其用作Java库或独立的Web服务器。
* [Openrouteservice](https://github.com/GIScience/openrouteservice)：具有大量功能的开源路线规划器API，由海德堡大学开源。
* [OpenTripPlanner](https://github.com/opentripplanner/OpenTripPlanner)：OpenTripPlanner是一款开源多模式旅行规划器，专注于通过定期公共交通结合自行车、步行和移动服务(包括自行车共享和叫车)出行，由俄勒冈州波特兰市交通机构TriMet开源。
* [R5](https://github.com/conveyal/r5)：R5是Conveyal的路线引擎，Conveyal是一个基于Web的系统，允许用户创建交通场景并根据累积机会可达性指标对其进行评估。
* [Public Transport Enabler](https://github.com/schildbach/public-transport-enabler)：这是一个Java库，允许你从公共交通提供商获取数据。
* [OneBusAway](https://github.com/OneBusAway/onebusaway-application-modules)：OneBusAway应用程序套件的主要功能是通过各种界面与乘客共享实时公共交通信息。
* [MovSim](https://github.com/movsim/movsim)：MovSim是一款基于微观车道的交通模拟器，具有基于XML的配置和CSV文本输出。
* [MATSim](https://github.com/matsim-org/matsim-libs)：MATSim提供了一个工具箱来运行和实施大规模基于代理的运输模拟，由柏林工业大学开源。
* [EqaSim](https://github.com/eqasim-org/eqasim-java)：EqaSim包是用于基于代理和基于活动的传输模拟框架MATSim的准备运行的高质量场景的集合。
* [OpenTrafficSim](https://github.com/averbraeck/opentrafficsim)：OpenTrafficSim是一个多级交通模拟器，由代尔夫特理工大学开源。
* [RouteConverter](https://github.com/cpesch/RouteConverter)：RouteConverter是一款流行的开源工具，用于显示、编辑、丰富和转换路线、轨迹和航点。
* [MobiTopp](https://github.com/kit-ifv/mobitopp)：MobiTopp是卡尔斯鲁厄理工学院交通研究所开发的基于代理的出行需求模型。

## 几何学

* [Apache Commons Geometry](https://github.com/apache/commons-geometry)：Commons Geometry项目提供几何类型和实用程序。
* [S2 Geometry Library](https://github.com/google/s2-geometry-library-java)：这是一个Google开源的用于操作几何形状的包，S2主要设计用于处理球面几何，即在球体上而不是在平面2D地图上绘制的形状。
* [Geolatte-geom](https://github.com/GeoLatte/geolatte-geom)：符合OGC SQL简单功能规范的几何模型。
* [JGEX](https://github.com/yezheng1981/Java-Geometry-Expert)：JGEX是一款结合了动态几何软件、自动几何定理证明器(GTP)和视觉动态证明呈现方法的软件，由威奇托州立大学开源。
* [JTS](https://github.com/locationtech/jts)：JTS Topology是一个用于创建和操作向量几何的Java库。
* [PGS](https://github.com/micycle1/PGS)：Processing Geometry Suite是一个软件项目，它以Processing库的形式提供对2D几何算法的轻松访问。
* [Geometry API](https://github.com/Esri/geometry-api-java)：Esri几何API可用于在第三方数据处理解决方案中启用空间数据处理。
* [JCSG](https://github.com/miho/JCSG)：基于BSP的CSG(构造实体几何)的Java实现。
* [Campskeleton](https://github.com/twak/campskeleton)：Java中的加权直骨架实现。
* [GeoRegression](https://github.com/lessthanoptimal/GeoRegression)：GeoRegression是一个基于Java的免费几何库，用于机器人和计算机视觉等领域的科学计算，重点关注2D/3D空间。
* [Computational Geometry](https://github.com/mikhaildubov/computational-geometry)：该项目包含二维空间中基本计算几何算法的实现和可视化工具。

## 天文学

* [Sentinel Application Platform](https://github.com/senbox-org/snap-engine)：SNAP是欧洲航天局为Sentinel-1、Sentinel-2和Sentinel-3任务的科学开发而开发的三个Sentinel工具箱的通用软件平台。
* [Sentinel-1](https://github.com/senbox-org/s1tbx)：Sentinel-1 Toolbox是由欧空局开发的一款界面友好的开源SAR图像处理软件，它能够处理1级以及更高级的SAR数据。
* [Sentinel-2](https://github.com/senbox-org/s2tbx)：ESA Sentinel-2卫星上的MSI仪器的工具箱。
* [Sentinel-3](https://github.com/senbox-org/s3tbx)：用于ESA Sentinel-3卫星上的OLCI和SLSTR仪器的工具箱。
* [Trick](https://github.com/nasa/Trick)：Trick模拟环境由NASA约翰逊航天中心开发，是一个功能强大的模拟开发框架，使用户能够为航天器开发的所有阶段构建应用程序。
* [NanoSat MO Framework](https://github.com/esa/nanosat-mo-framework)：NanoSat MO是基于CCSDS任务运营服务的纳米卫星软件框架，由格拉茨科技大学与欧洲航天局合作开发。
* [GUSTO](https://github.com/esa/GUSTO)：GUSTO包含赫歇尔空间天文台科学任务规划软件的组件，该软件用于天文观测的交互式调度，由欧洲航天局开源。
* [SBMT](https://sbmt.jhuapl.edu/)：SBMT是一种搜索、访问和分析小天体航天器数据的简单方法，由约翰霍普金斯大学应用物理实验室开发。
* [DERT](https://github.com/nasa/DERT)：DERT是一款开源软件工具，用于探索NASA的3D数字地形模型。
* [CCDD](https://github.com/nasa/CCDD)：CCDD是一款用于管理CFS和CFS应用程序的命令和遥测数据的软件工具，NASA开发。
* [WorldWind Java](https://github.com/NASAWorldWind/WorldWindJava)：美国国家航空航天局发布的一个开源的地理科普软件，由NASA Research开发。它是一个可视化地球仪，将NASA、USGS以及其它WMS服务商提供的图像通过一个三维的地球模型展现。
* [SNAP Desktop](https://github.com/senbox-org/snap-desktop)：SNAP是欧洲航天局为光学和微波任务的科学开发而开发的三个工具箱的通用软件平台。
* [Orekit](https://github.com/CS-SI/Orekit)：Orekit是一个用Java编写的免费低级空间动力学库，它提供了基本元素(轨道、日期、姿态、框架...)以及处理它们的各种算法，由CS GROUP开发。
* [JMARS](https://jmars.mars.asu.edu/)：JMARS是由亚利桑那州立大学火星太空飞行设施开发的地理空间信息系统，旨在为NASA科学家、仪器团队成员、各个年龄段的学生和公众提供任务规划和数据分析工具。
* [EarthSci](https://github.com/GeoscienceAustralia/earthsci)：用于地球科学可视化的Eclipse RCP平台，基于NASA WorldWind Java SDK构建。
* [SolarPositioning](https://github.com/klausbrunner/solarpositioning)：SolarPositioning是一个用于查找地心太阳坐标的Java库。
* [Nom.Tam.Fits](https://github.com/nom-tam-fits/nom-tam-fits)：用于读写FITS文件的全功能100% Java库，最初起源于NASA，目前由哈佛大学维护。
* [Astro4j](https://github.com/melix/astro4j)：用于Java中天文图像处理的库和应用程序的集合。
* [AstroImageJ](https://github.com/AstroImageJ/astroimagej)：提供了一个天文特定的图像显示环境和工具，用于天文特定的图像校准和数据缩减。
* [Predict4Java](https://github.com/davidmoten/predict4java)：Predict4Java提供实时卫星跟踪和轨道预测信息。
* [Gaia Sky](https://codeberg.org/gaiasky/gaiasky)：适用于桌面和VR的开源3D宇宙模拟器，支持超过10亿个对象，由海德堡大学开源。
* [JSOFA](https://github.com/Javastro/jsofa)：JSOFA是国际天文学联合会的C SOFA软件库的纯Java版本。
* [AstroLib](https://mhuss.com/AstroLib/docs/Overview.html)：一个Java天文算法库。
* [JPARSEC](https://arxiv.org/abs/1806.03088)：经过十二年开发和使用的天文学Java包。
* [DSTE](https://ai-solutions.com/dste/)：DSTE是一个交互式软件包，具有创新可视化功能的多体轨迹设计技术，可显著减少轨迹设计所花费的时间，被约翰逊航天中心用作设计工具。
* [JHelioviewer](https://github.com/Helioviewer-Project/JHelioviewer-SWHV)：JHelioviewer是基于JPEG 2000图像压缩标准的太阳物理数据可视化工具，由ESA/NASA开源。
* [Commons-Suncalc](https://github.com/shred/commons-suncalc)：用于计算太阳和月亮位置和相位的Java库。
* [Debian Astro Java](https://blends.debian.org/astro/tasks/java)：用于天文学的Java包集合。
* [Starlink Java](https://github.com/Starlink/starjava)：星链项目是一个长期运行的英国项目，支持天文数据处理。它于2005年关闭，但该软件继续在联合天文中心开发，直到2015年3月，现在由东亚天文台维护。
* [Java Astrodynamics Toolkit](https://sourceforge.net/projects/jat/)：JAT是一个开源软件组件库，用于用Java或Matlab编写的航天应用程序，该软件由NASA使用，是德克萨斯大学计算机科学系的主要合作项目。
* [JSky](https://jsky.sourceforge.net/)：JSky项目的目标是构建一系列可重用的Java组件以用于天文学，最初源于ESO的Skycat应用程序的开发。
* [IDV](https://github.com/Unidata/IDV)：IDV是一个用于分析和显示地球科学数据的框架，由美国国家科学基金会Unidata开源。
* [Mars-SIM](https://github.com/mars-sim/mars-sim)：Mars-SIM被设计为一个通用模拟器，描绘火星上人类住区的早期发展。
* [Simple-Astronomy-Lib](https://github.com/SimpleAstronomy/simple-astronomy-lib)：一个简单的天文学库，用于计算月相、日食等。
* [Aerie](https://github.com/NASA-AMMOS/aerie)：Aerie是一个用于航天器建模的软件框架，NASA开源。
* [Geocalc](https://github.com/grumlimited/geocalc)：Geocalc是一个简单的Java库，旨在使用地球坐标进行算术运算。
* [Orbdetpy](https://github.com/ut-astria/orbdetpy)：Orbdetpy是一个Python轨道确定库，航天先进科学技术研究(ASTRIA)开源。
* [Yamcs](https://github.com/yamcs/yamcs)：Yamcs是一个用Java开发的任务控制框架。
* [CCSDS](https://github.com/dariol83/ccsds)：基于Java 11的CCSDS协议和格式的开源实现。
* [ProsEO](https://github.com/dlr-eoc/prosEO)：ProsEO软件系统是一个开源处理控制系统，旨在执行处理地球观测卫星数据(例如Sentinel数据)所需的所有活动，根据配置的任务生成用户级数据、工程数据和/或内部遥测数据，由德国航空航天中心开发。
* [Aladin](http://aladin.cds.unistra.fr/aladin.gml)：Aladin是一款广泛使用的Java工具，能够解决诸如定位目标数据、访问和探索分布式数据集以及可视化多波长数据等挑战，由法国斯特拉斯堡数据中心开发。

## 水文学

* [SOS](https://github.com/52North/SOS)：SOS提供了一个可互操作的基于Web的界面，用于插入和查询传感器数据和传感器描述。
* [OpenDCS](https://github.com/opendcs/opendcs)：水文/气象数据开放数据采集系统。
* [Vortex](https://github.com/HydrologicEngineeringCenter/Vortex)：Vortex是针对水文工程中心应用程序的数据处理实用程序的集合，例如HEC-HMS、HEC-RAS。
* [SeaDAS](https://github.com/seadas/seadas)：SeaDAS软件由NASA开发，专门用于海洋水色数据的处理、可视化、分析和质量保证。

## 物理库

* [FreeHEP](https://java.freehep.org/)：FreeHEP库的目标是鼓励高能物理领域Java代码的共享和重用，由SLAC国家加速器实验室开源。

## 无人机

* [RosettaDrone](https://github.com/RosettaDrone/rosettadrone)：RosettaDrone是一个用于开发和测试DJI无人机软件的框架。
* [MAVGCL](https://github.com/ecmnet/MAVGCL)：这个基于JavaFX的工具使PX4用户能够根据PX4Logs或ULogs记录和分析飞行期间或离线时通过UDP发布的数据。
* [MAVLink](https://github.com/dronefleet/mavlink)：用于MAVLink通信的Java API。
* [OpenAMASE](https://github.com/afrl-rq/OpenAMASE)：OpenAMASE是用于模拟多无人机任务的项目，由空军研究实验室、航空航天系统局、动力和控制部门开发。
* [MAVLinkJava](https://github.com/ghelle/MAVLinkJava)：用于MAVLink的Java代码生成器和Java库。
* [MAVLink Kotlin](https://github.com/divyanshupundir/mavlink-kotlin)：适用于Kotlin多平台的现代MAVLink库。

## AIS库

* [AISmessages](https://github.com/tbsalling/aismessages)：AISmessages是一种基于Java的轻量级、零依赖、超高效消息解码器，用于符合ITU 1371(NMEA装甲AIS消息)的海上导航和安全消息。
* [AisLib](https://github.com/dma-ais/AisLib)：AisLib是一个用于处理AIS消息的Java库，由丹麦海事局开源。
* [Java Marine API](https://github.com/ktuukkan/marine-api)：Java Marine API是一个NMEA 0183解析器库，用于对各种电子海洋设备(例如GPS、回声测深仪和气象仪器)提供的数据进行解码和编码。
* [Risky](https://github.com/amsa-code/risky)：用于分析带有时间戳的位置数据的工具，例如来自AIS的船舶位置报告。

## 跨语言

* [Swift Java](https://github.com/swiftlang/swift-java)：Swift Java互操作性工具和库，Apple开源。
* [SwiftJava](https://github.com/SwiftJava/SwiftJava)：SwiftJava是一个Swift代码生成器，以及一个支持用Swift 3.0的Xcode beta6版本编写的代码的小框架。
* [Py4J](https://github.com/py4j/py4j)：Py4J使Python程序能够动态访问任意Java对象。
* [Jep](https://github.com/ninia/jep)：Jep通过JNI将CPython嵌入到Java中。
* [RCaller](https://github.com/jbytecode/rcaller)：RCaller是一个软件库，旨在简化从Java调用R的过程，由伊斯坦布尔大学数值方法教授开源。
* [FastR](https://github.com/oracle/fastr)：FastR是基于GraalVM构建的R编程语言的高性能实现，由Oracle开发。
* [RJava](https://github.com/s-u/rJava)：R/Java接口允许在R中使用Java以及将R嵌入到Java中(通过JRI)。
* [Renjin](https://github.com/bedatadriven/renjin)：Renjin是基于JVM的R语言解释器。
* [Rserve](https://github.com/s-u/REngine)：Rserve是一个TCP/IP服务器，它允许其他程序使用各种语言的R功能，而无需初始化R或链接到R库。
* [RSession](https://github.com/yannrichet/rsession)：Rsession提供了一个易于使用的Java类，可以访问远程或本地R会话。
* [J4RS](https://github.com/astonbitecode/j4rs)：J4RS允许从Rust轻松调用Java代码，反之亦然。
* [Robusta](https://github.com/giovanniberti/robusta)：该库提供了一个过程宏，使得在Rust中编写与JNI兼容的代码变得更容易。
* [Duchess](https://github.com/duchess-rs/duchess)：Duchess是一个Rust包，可以简单、符合人体工程学且高效地与Java代码进行互操作。
* [LuaJava](https://github.com/jasonsantos/luajava)：LuaJava是一个Java脚本编写工具，该工具的目标是允许用Lua编写的脚本操作用Java开发的组件。
* [Rococoa](https://github.com/iterate-ch/rococoa)：Rococoa是绑定到Mac Objective-C对象系统的通用Java，它允许在Java中创建和使用Objective-C对象，以及在Java中实现Objective-C接口。
* [DWR](https://github.com/directwebremoting/dwr)：DWR是一个Java库，它使服务器上的Java和浏览器中的JavaScript能够尽可能简单地交互和调用。
* [J2C](https://github.com/arnetheduck/j2c)：J2C可以将Java代码转换为可编译的C++(11)代码。
* [JTcl](https://github.com/jtcl-project/jtcl)：JTcl是用Java编写的Tcl(工具命令语言)的实现。
* [PemJa](https://github.com/alibaba/pemja)：PemJa是一个基于FFI的开源跨语言调用框架，由阿里开源。
* [JNI4NET](https://github.com/jni4net/jni4net)：JVM和CLR之间的快速、面向对象、进程内桥梁。
* [Trireme](https://github.com/apigee/trireme)：Trireme在JVM内运行Node.js脚本。
* [Javonet](https://www.javonet.com/)：Javonet是一个高级库，可以在任何编程语言和模块之间进行直接方法调用，从而无需集成层。
* [DynamicObject](https://github.com/rschmitt/dynamic-object)：DynamicObject使Java开发人员能够以惯用的方式和最少的样板使用Clojure强大的数据建模功能。

## 转换库

* [Apache Commons Convert](https://commons.apache.org/sandbox/commons-convert)：Commons Convert是一个专用于将一种类型的对象转换为另一种类型的库。
* [TypeMap](https://github.com/YunaBraska/type-map)：TypeMap和TypeConverter是一个动态、可扩展、高性能的类型转换库，原生支持GraalVM。
* [LibNumberText](https://github.com/Numbertext/libnumbertext)：C++、Java、JavaScript和Python中的数字到数字名称和金钱文本转换库。

## IO操作

* [Apache Commons IO](https://github.com/apache/commons-io)：Commons IO库包含实用程序类、流实现、文件过滤器、文件比较器、字节序转换类等等。
* [Okio](https://github.com/square/okio)：Okio是一个补充java.io和java.nio的库，使你可以更轻松地访问、存储和处理数据，由Square开源。
* [UberFire I/O](https://github.com/kiegroup/appformer/tree/main/uberfire-io)：NIO.2的实用程序/门面集。
* [JNR-UnixSocket](https://github.com/jnr/jnr-unixsocket)：Java的本机I/O访问。
* [Plexus IO](https://github.com/codehaus-plexus/plexus-io)：Plexus IO是一组Plexus组件，设计用于I/O操作。
* [Jaydio](https://github.com/smacke/jaydio)：Jaydio是一个Java库，可让程序员更好地控制文件I/O，部分方法是绕过操作系统缓冲区高速缓存。
* [JTar](https://github.com/kamranzafar/jtar)：JTar是一个简单的Java Tar库，它提供了一种使用IO流创建和读取tar文件的简单方法。
* [PerfIO](https://github.com/szeiger/perfio)：PerfIO为二进制和文本数据提供缓冲流式I/O抽象。
* [jMimeMagic](https://github.com/arimus/jmimemagic)：jMimeMagic是一个用于确定文件或流的MIME类型的Java库。
* [SimpleMagic](https://github.com/j256/simplemagic)：简单的文件幻数和内容类型库，提供文件和字节数组的MIME类型确定。
* [MimeCraft](https://github.com/square/mimecraft)：用于创建符合RFC要求的Multipart和表单编码HTTP请求主体的实用程序，由Square开源。
* [Ballerina MIME](https://github.com/ballerina-platform/module-ballerina-mime)：该库提供了一组用于处理消息的API，这些API遵循RFC 2045标准中指定的多用途Internet邮件扩展规范。
* [Apache MIME4J](https://github.com/apache/james-mime4j)：Mime4j可用于解析纯rfc822和MIME格式的电子邮件消息流，并构建电子邮件消息的树表示形式。
* [MIME Type](https://github.com/overview/mime-types)：用于检测文件MIME类型的Java库。
* [Chronicle Bytes](https://github.com/OpenHFT/Chronicle-Bytes)：Chronicle Bytes的用途与Java的NIO ByteBuffer类似，但具有一些附加功能。
* [Kdio](https://github.com/lexburner/kdio)：一个使用非常简单的Java Direct IO框架。
* [FastJavaIO](https://github.com/williamfiset/FastJavaIO)：非常快的Java输入读取器。
* [Rsync4j](https://github.com/fracpete/rsync4j)：适用于Linux、OSX和Windows的rsync的简单Java包装器。
* [Jayo](https://github.com/jayo-projects/jayo)：Jayo是一个基于java.io的JVM同步I/O库，这会产生简单、可读和可调试的代码，就像标准的阻塞程序一样，但它在幕后执行非阻塞I/O。

## 文件操作

* [AppDirs](https://github.com/harawata/appdirs)：AppDirs是一个小型Java库，它提供了平台相关的特殊文件夹/目录的路径。
* [FSWatch](https://github.com/vorburger/ch.vorburger.fswatch)：用于基于java.nio.file.WatchService监视目录或单个文件的Java库。
* [Directory Watcher](https://github.com/gmethvin/directory-watcher)：适用于JDK 8+的目录监视实用程序，旨在为Linux、macOS和Windows提供准确且高效的递归监视。
* [Play File Watch Library](https://github.com/playframework/play-file-watch)：这是Play文件监视库，它可用于以独立于平台的方式监视文件。
* [Giraffe](https://github.com/palantir/giraffe)：Giraffe是一个Java库，可让你轻松访问本地和远程计算机上的文件并执行命令，由Palantir开源。
* [Wildcard](https://github.com/EsotericSoftware/wildcard)：Wildcard是一个小型Java库，用于执行文件和目录的高效模式匹配。
* [Directories](https://github.com/dirs-dev/directories-jvm)：Directories是一个提供配置/缓存/数据路径的小型库，遵循Linux、MacOS、BSD和Windows上的相应约定。

## 文件上传

* [Apache Commons FileUpload](https://github.com/apache/commons-fileupload)：Commons FileUpload组件提供了一种简单而灵活的方法来向Servlet和Web应用程序添加对分段文件上传功能的支持。
* [FastUpload](https://sourceforge.net/projects/fastupload/)：该组件基于RFC1867，它使用高性能的字节搜索算法来解析提交的请求，然后将数据保存到文件系统中；此外，它还提供了一个智能解决方案来解决上传文本文件的编码问题。
* [MinIO Plus](https://gitee.com/lxp135/minio-plus)：MinIO-Plus是一个MinIO的二次封装与增强工具。
* [Upload Parser](https://github.com/Elopteryx/upload-parser)：Upload Parser是一个用于Servlet和Web应用程序的文件上传库。
* [Phloc FileUpload](https://mvnrepository.com/artifact/com.phloc/phloc-fileupload/1.0.2)：用于在Web应用程序中扩展文件上传处理的库。
* [Fulcrum Upload](https://turbine.apache.org/fulcrum/fulcrum-upload/)：处理来自Servlet和Portlet的POST请求的multi-part/form-data解析，使multi-part文件可从内存或文件系统上的指定位置获取。
* [AWS S3 OutputStream](https://github.com/CI-CMG/aws-s3-outputstream)：AWS S3 OutputStream项目允许通过java.io.OutputStream分段上传到AWS S3存储桶。
* [NIO-Multipart](https://github.com/synchronoss/nio-multipart)：NIO Multipart项目包含一个轻量级通用Java库，用于以非阻塞方式处理Multipart请求和响应，并具有可配置但恒定的内存占用。

## 文件比较

* [Diff Match Patch](https://github.com/google/diff-match-patch)：Diff Match Patch库提供了强大的算法来执行同步纯文本所需的操作，Google开源。
* [GumTree](https://github.com/GumTreeDiff/gumtree)：GumTree是一个代码区分工具。
* [DiffPlug](https://www.diffplug.com/)：DiffPlug是免费的文本编辑器/差异和图像查看器/差异。
* [Diff Utils](https://github.com/java-diff-utils/java-diff-utils)：Diff Utils库是一个开源库，用于执行文本之间的比较操作：计算差异、应用补丁、生成统一差异或解析它们、生成差异输出以方便将来显示(如并排视图)等。
* [Cafecompare](https://github.com/GraxCode/cafecompare)：Cafecompare是一个用于分析和比较Java档案和class文件的GUI应用程序。

## 文件预览

* [KKFileView](https://gitee.com/kekingcn/file-online-preview)：KKFileView是基于Spring Boot的通用文件在线预览项目，由凯京科技开源。
* [DocPreview](https://gitee.com/hcwdc/docpreview)：文件在线预览模块，支持多格式转PDF文件，由华创数字云开源。
* [WDA](https://gitee.com/macplus/WDA)：配合OpenOffice实现文档的在线预览、本地文档添加、文档转换为HTML，文档HTML方式预览，由太原扁舟科技开源。
* [WPS View](https://gitee.com/mose-x/wps-view-java)：基于WPS在线编辑、在线预览后台服务。
* [File Preview Spring Boot Starter](https://github.com/wb04307201/file-preview-spring-boot-starter)：一个文档在线预览的中间件，可通过简单的配置即可集成到Spring Boot中。

## 邮件操作

* [Simple Java Mail](https://github.com/bbottema/simple-java-mail)：最简单的Java轻量级邮件库，同时能够发送复杂的电子邮件，包括CLI支持、附件、嵌入图像、自定义标头和属性、强大的地址验证、构建模式甚至DKIM签名、S/MIME支持和具有属性覆盖的外部配置文件、Spring支持和电子邮件转换工具。
* [FakeSMTP](https://github.com/Nilhcem/FakeSMTP)：FakeSMTP是一个带有GUI的免费虚拟SMTP服务器，可轻松测试应用程序中的电子邮件。
* [Apache James](https://github.com/apache/james-project)：具有基于丰富的现代高效组件的模块化架构，最终提供在JVM上运行的完整、稳定、安全和可扩展的邮件服务器。
* [Mail Utils](https://github.com/hellokaton/oh-my-email)：非常轻量的Java邮件发送类库，支持抄送、附件、模板等功能。
* [SendGrid](https://github.com/sendgrid/sendgrid-java)：该库允许你通过Java快速轻松地使用Twilio SendGrid Web API v3。
* [Fake SMTP Server](https://github.com/gessnerfl/fake-smtp-server)：Fake SMTP Server是一个简单的SMTP服务器，专为开发目的而设计。
* [Mailgun](https://github.com/sargue/mailgun)：这是一个小型Java库，可以使用出色的Mailgun服务轻松发送电子邮件。
* [NioImapClient](https://github.com/HubSpot/NioImapClient)：基于Netty的Java高性能IMAP客户端。
* [Apache Commons Email](https://github.com/apache/commons-email)：Commons Email提供用于发送电子邮件的API，它构建在JavaMail API之上，旨在简化JavaMail API。
* [JMail](https://github.com/RohanNagar/jmail)：一个现代、快速、零依赖的库，用于在Java中处理电子邮件地址并执行电子邮件地址验证。
* [SubEtha SMTP](https://github.com/voodoodyne/subethasmtp)：SubEtha SMTP是一个Java库，它允许你的应用程序通过简单、易于理解的API接收SMTP邮件。
* [Jakarta Mail](https://github.com/jakartaee/mail-api)：Jakarta Mail定义了一个独立于平台和协议的框架来构建邮件和消息传递应用程序。
* [Eclipse Angus Mail](https://github.com/eclipse-ee4j/angus-mail)：该项目提供了Jakarta Mail规范2.1+的实现。
* [TrashEmail](https://github.com/rosehgal/TrashEmail)：TrashEmail是托管的Telegram机器人，它可以通过提供一次性电子邮件地址来保存你的私人电子邮件地址，它可以创建、管理一次性电子邮件地址并将其与你的Telegram机器人聊天链接。
* [ExJello](https://code.google.com/archive/p/exjello/)：ExJello是一个连接到Microsoft Exchange服务器的JavaMail提供程序，它被设计为标准POP3和SMTP提供商的直接替代品。
* [DKIM](https://www.agitos.de/dkim-for-javamail/)：允许你使用DKIM对邮件进行签名的开源库。
* [Jack Mail](https://sourceforge.net/projects/jackmailclient/)：一个简单的邮件客户端，可以以最少的配置使用任何邮件服务器。
* [Aspirin](https://github.com/masukomi/aspirin)：Aspirin是一个供Java开发人员使用的嵌入式仅发送SMTP服务器。
* [Yawebmail](https://yawebmail.sourceforge.net/)：Yawebmail是一个用Java编写的Web邮件客户端，它支持SMTP(包括SMTP身份验证)、POP3和IMAP。
* [JMBox](https://sourceforge.net/projects/jmbox/)：JMBox是JavaMail的本地存储提供程序，使开发人员能够使用JavaMail API来管理存储在本地仓库(如Outlook Express、Outlook、Mozilla、Netscape等)中的邮件。
* [ImapNIO](https://github.com/yahoo/imapnio)：ImapNIO是一个支持基于NIO的IMAP客户端的Java库，由Yahoo开源。
* [JavaMail Crypto](http://javamail-crypto.sourceforge.net/)：这是JavaMail API的一个补充，它使用S/MIME和/或OpenPGP提供简单的电子邮件加密和解密。
* [Jcabi-Email](https://github.com/jcabi/jcabi-email)：面向对象的电子邮件Java SDK。
* [Spring Boot Email Tools](https://github.com/ozimov/spring-boot-email-tools)：一组使用模板引擎在Spring Boot 1.5.x应用程序中发送电子邮件的服务和工具。
* [Email-RFC2822-Validator](https://github.com/bbottema/email-rfc2822-validator)：基于Java且符合RFC2822标准的电子邮件地址验证器和解析器。
* [Mailjet Java Wrapper](https://github.com/mailjet/mailjet-apiv3-java)：Mailjet Java API包装器，Mailjet是法国的电子邮件营销平台。
* [Email4J](https://github.com/juandesi/email4j)：Email4J是一个构建在javax.mail API之上的高级Java库，用于管理和发送电子邮件，无需了解底层传输的任何规范。
* [ErmesMail](https://github.com/SoftInstigate/ermes-mail)：ErmesMail是一个用于异步发送电子邮件的Java库和命令行接口。
* [ePADD](https://github.com/ePADD/epadd)：ePADD是由斯坦福大学特殊馆藏和大学档案馆开发的软件包，支持围绕电子邮件档案的评估、摄取、处理、发现和交付的档案流程。
* [MsgViewer](https://github.com/lolo101/MsgViewer)：MsgViewer是用于.msg电子邮件消息的电子邮件查看器实用程序，以纯Java实现。
* [Jodd Mail](https://github.com/oblac/jodd-mail)：Jodd Mail提供了一些工具类，用于以更简单、实用的方式发送和接收电子邮件。
* [Email Template Builder](https://github.com/rocketbase-io/email-template-builder)：该库可以以流式的方式构建HTML/文本电子邮件。
* [NioSmtpClient](https://github.com/HubSpot/NioSmtpClient)：基于Netty的Java高性能SMTP客户端。

## 电子签名

* [XAdES4j](https://github.com/luisgoncalves/xades4j)：XAdES4j是XML高级电子签名(XAdES 1.3.2和1.4.1)的高级、可配置和可扩展的Java实现。
* [DigiDoc4j](https://github.com/open-eid/digidoc4j)：DigiDoc4j是一个Java库，用于对文档进行数字签名并创建签名文档的数字签名容器。
* [El Cliente @firma](https://github.com/ctt-gob-es/clienteafirma)：El Cliente @firma是自由软件(EUPLv1.1 + GPLv2)电子签名应用程序的集合，它允许在不同的操作环境中创建不同格式的电子签名。
* [DSS](https://github.com/esig/dss)：DSS是一个用于电子签名创建和验证的开源软件库，由欧盟委员会开源。
* [JHOVE](https://github.com/openpreserve/jhove)：JHOVE是一个可扩展的软件框架，用于执行数字对象的格式识别、验证和表征，由哈佛大学开发。
* [SignServer Community](https://github.com/Keyfactor/signserver-ce)：SignServer是基于PKI的开源签名软件，用于签署代码、文档、时间戳等。
* [Apache Santuario](https://github.com/apache/santuario-xml-security-java)：Santuario是实现XML数字签名规范和XML加密规范的库。

## 安全培训

* [CRAPI](https://github.com/OWASP/crAPI)：CRAPI在设计上很容易受到攻击，但你将能够安全地运行它来教育/培训自己，由OWASP开源。
* [WebGoat](https://github.com/WebGoat/WebGoat)：WebGoat是由OWASP维护的故意不安全的Web应用程序，旨在教授Web应用程序安全课程。
* [BodgeIt](https://github.com/psiinon/bodgeit)：BodgeIt Store是一个易受攻击的Web应用程序，目前针对渗透测试的新手。
* [OWASP Benchmark](https://github.com/OWASP-Benchmark/BenchmarkJava)：OWASP Benchmark项目是一个Java测试套件，旨在验证漏洞检测工具的速度和准确性。
* [OWASP Security Shepherd](https://github.com/OWASP/SecurityShepherd)：OWASP Security Shepherd项目是一个Web和移动应用程序安全培训平台，旨在培养和提高不同技能人群的安全意识。
* [MASTG Hacking Playground](https://github.com/OWASP/MASTG-Hacking-Playground)：MASTG Hacking Playground是一系列教育性iOS和Android移动应用程序，这些应用程序故意构建为不安全的，以便为开发人员、安全研究人员和渗透测试人员提供实用指导。

## RSS

* [ROME](https://github.com/rometools/rome)：ROME是一个用于RSS和Atom提要的Java框架。
* [Huntly](https://github.com/lcomplete/huntly)：Huntly是一个信息管理工具，它不仅可以自托管，也可以通过客户端在本地运行。
* [CommaFeed](https://github.com/Athou/commafeed)：受Google Reader启发而开发的自托管RSS阅读器，基于Dropwizard和React/TypeScript。
* [Android-RSS](https://github.com/ahorn/android-rss)：用于解析RSS 2.0提要的轻量级Android库。
* [Sismics Reader](https://github.com/sismics/reader)：一个开源、基于Web的内容聚合器，由Web Feeds(RSS、Atom)提供服务。
* [RSSOwl](https://github.com/rssowl/RSSOwl)：一个功能强大的应用程序，可以以舒适的方式组织、搜索和阅读RSS、RDF和Atom新闻源。
* [RSS Reader](https://github.com/w3stling/rssreader)：一个简单的Java库，用于读取RSS和Atom提要。
* [Makagiga](https://github.com/kdt/makagiga)：Makagiga是一款开源、易于使用的便携式应用程序，用于执行各种任务，例如待办事项列表、文本编辑或RSS阅读。
* [RSS Parser](https://github.com/prof18/RSS-Parser)：RSS Parser是一个用于解析RSS和Atom提要的Kotlin多平台库，支持Android、iOS和JVM。

## SSE

* [Okhttp-EventSource](https://github.com/launchdarkly/okhttp-eventsource)：基于OkHttp的Java SSE客户端实现。
* [OkSse](https://github.com/heremaps/oksse)：OkSse是OkHttp的扩展库，用于创建SSE客户端。
* [Turbine](https://github.com/Netflix/Turbine)：Turbine是一种用于将SSE JSON数据流聚合到单个流中的工具，由Netflix开源。
* [SSE-EventBus](https://github.com/ralscha/sse-eventbus)：EventBus库，用于使用SSE将事件从Spring应用程序发送到Web浏览器。

## RPM

* [Redline](https://github.com/craigwblake/redline)：Redline是一个纯Java库，用于操作RPM包。
* [Eclipse Packager](https://github.com/eclipse/packager)：Eclipse Packager项目提供了一组核心功能，可在纯Java中使用RPM和Debian包文件。

## EPC

* [EPCIS](https://github.com/JaewookByun/epcis)：Oliot EPCIS 2.2.0是电子产品代码信息服务(EPCIS)
  v2.0的原型实现，能够捕获和共享GS1批准的标准化事件/主数据，由韩国世宗大学开源。
* [OpenEPCIS](https://github.com/openepcis)：OpenEPCIS是GS1 EPCIS标准的开源完全兼容实现，它还提供了许多开源项目、工具和工件。
* [EPCtagCoder](https://github.com/jlcout/epctagcoder)：EPCtagCoder是一个极其直观、小型且超快的Java EPC编码和解码库。

## FMI

* [FMI4j](https://github.com/NTNU-IHB/FMI4j)：FMI4j是一个用Kotlin编写的软件包，用于处理功能模型单元(FMU)，由挪威科技大学开源。
* [JavaFMI](https://bitbucket.org/siani/javafmi)：JavaFMI是一组与功能模型接口FMI配合使用的组件。

## AS4

* [Phase4](https://github.com/phax/phase4)：Phase4是一个可嵌入的轻量级Java库，用于发送和接收不同配置文件的AS4消息。
* [Oxalis](https://github.com/OxalisCommunity/oxalis)：Oxalis是AS4规范的领先开源软件实现。

## OSGI

* [OSGi](https://github.com/osgi/osgi)：OSGi是Java动态化模块化系统的一系列规范。
* [Apache Karaf](https://github.com/apache/karaf)：Karaf提供了一个轻量级的OSGi容器，可以用于部署各种组件。
* [OPS4j Pax Web](https://github.com/ops4j/org.ops4j.pax.web)：Pax Web通过更好的Servlet支持、过滤器、监听器、错误页面和JSP等扩展了OSGi HTTP服务，以满足最新版本的Servlet规范。
* [Bnd](https://github.com/bndtools/bnd)：用于构建OSGi包的工具，包括Eclipse、Maven和Gradle插件。
* [OSGi enRoute](https://github.com/osgi/osgi.enroute)：OSGi enRoute项目提供了OSGi应用程序的编程模型，该项目包含为OSGi enRoute基本配置文件提供API的捆绑包和用于OSGi enRoute项目的捆绑包。
* [Apache Aries](https://github.com/apache/aries)：Aries项目由一组可插拔Java组件组成，支持企业OSGi应用程序编程模型。
* [Apache Felix](https://github.com/apache/felix-dev)：Felix项目是一些半相关的OSGi子项目的集合，这些子项目可以单独构建和发布。
* [Eclipse Equinox](https://eclipse.dev/equinox/)：Equinox是OSGi核心框架规范的实现，这是一组实现各种可选OSGi服务和其他用于运行基于OSGi的系统的基础设施的捆绑包。
* [Eclipse Virgo](https://projects.eclipse.org/projects/rt.virgo)：Virgo Web Server是一个完全基于模块的Java应用程序服务器，旨在以高度的灵活性和可靠性运行企业Java应用程序和Spring驱动的应用程序，由VMware开发。
* [OSGi Testing](https://github.com/osgi/osgi-test)：该项目提供了一组捆绑包，其中包含用于测试OSGi API的有用类。

## RAML

* [RAML Java Parser](https://github.com/raml-org/raml-java-parser)：这是RAML解析器的官方Java实现，适用于1.0和0.8版本。
* [RAML JAX-RS](https://github.com/mulesoft-labs/raml-for-jax-rs)：RAML JAX-RS的目标是提供一组工具来与这些技术协同工作，以便能够基于现有的RAML API定义(代码生成)构建Java + JAX-RS应用程序，或者基于现有的Java + JAX-RS应用程序(文档)生成RAML API定义。
* [Raml Tester](https://github.com/nidi3/raml-tester)：测试请求/响应是否与给定的RAML定义匹配。

## OData

* [OData](https://github.com/RWS/odata)：这是基于OData标准的Tridion开放数据框架，完全用Java实现。
* [Apache Olingo](https://github.com/apache/olingo-odata4)：Apache Olingo是一个围绕OData规范的Java库及扩展。
* [OData Java](https://github.com/SAP-archive/cloud-odata-java)：OData Java是一个使开发人员能够实现OData生产者和OData消费者的库，由SAP开源。

## 数控

* [Universal G-Code Sender](https://github.com/winder/Universal-G-Code-Sender)：Universal G-Code Sender是一个基于Java的跨平台G-Code发送器，与GRBL、TinyG、g2core和Smoothieware兼容。
* [CNC-GCode-Controller](https://github.com/im-pro-at/cncgcodecontroller)：在CNC机器上使用rerap控制器。
* [GCode Sender](https://github.com/SourceRabbit/gcode-sender)：跨平台3轴数控机床控制软件。

## 数电

* [Workcraft](https://github.com/workcraft/workcraft)：Workcraft是一个跨平台工具集，用于捕获、模拟、合成和验证图形模型，由伦敦大学学院开源。
* [Logisim Evolution](https://github.com/logisim-evolution/logisim-evolution)：Logisim-evolution是用于设计和模拟数字逻辑电路的教育软件。
* [Digital](https://github.com/hneemann/Digital)：Digital是一款易于使用的数字逻辑设计器和电路模拟器，专为教育目的而设计。
* [Cello](https://github.com/CIDARLAB/cello)：遗传电路设计自动化，由CIDAR实验室开发。
* [Logisim](https://github.com/Logisim-Ita/Logisim)：Logisim是一个数字电路模拟器。
* [CircuitJS1](https://github.com/sharpie7/circuitjs1)：CircuitJS1是一个在浏览器中运行的电子电路模拟器。

## 海关

* [GTAS](https://github.com/US-CBP/GTAS)：GTAS是用于提高边境安全的网络应用程序，它使政府机构能够在高风险航空旅客计划旅行之前自动识别他们，由美国海关和边境保护局开源。
* [China E-Port Data Signature](https://github.com/Weasley-J/chinaport-data-signature)：该项目为中国电子口岸海关总署XML报文和海关179数据上报加签服务，提供一站式的免费解决方案，开箱即用。

## 蓝牙

* [Bluetooth Manager](https://github.com/sputnikdev/bluetooth-manager)：用于管理蓝牙适配器、蓝牙设备、GATT服务和特性的库/框架。
* [BlueCove](https://sourceforge.net/projects/bluecove/)：Bluecove是Java的跨平台蓝牙库，最初由英特尔研究院开发。
* [SmartGattLib](https://github.com/movisens/SmartGattLib)：SmartGattLib是一个Java库，可简化与蓝牙SMAR设备的协作。
* [Bluez DBus](https://github.com/hypfvieh/bluez-dbus)：使用DBus和Bluez的Linux操作系统蓝牙库。

## 校验

* [Jakarta Validation](https://github.com/jakartaee/validation)：Jakarta Validation为JavaBean和方法验证定义了元数据模型和API。
* [Hibernate Validator](https://github.com/hibernate/hibernate-validator)：Hibernate Validator是Jakarta Bean Validation的参考实现。
* [Apache BVal](https://github.com/apache/bval)：这是Jakarta EE和Java SE的Java Bean Validation(JSR 303、349、380)规范的实现。
* [Fluent Validator](https://github.com/neoremind/fluent-validator)：Fluent Validator通过利用流式接口风格和JSR 303 Bean Validation规范，提供了轻松支持验证的功能。
* [YAVI](https://github.com/making/yavi)：YAVI是一种基于Lambda的Java类型安全验证框架。
* [RAVE](https://github.com/uber-archive/rave)：RAVE使用Java注解处理来利用模型类中已有的注解(Nullness、Value Constraint、Typedef)来提高运行时的安全性，Uber开源。
* [Apache Commons Validator](https://github.com/apache/commons-validator)：Commons Validator为客户端验证和服务器端数据验证提供构建块。
* [Java Fluent Validator](https://github.com/mvallim/java-fluent-validator)：Java Fluent Validator在Java语言中定义了一个内部DSL供程序员使用。
* [Coody Verification](https://gitee.com/coodyer/coody-verification)：Coody Verification是一款参数自动化校验工具。
* [JBVE](https://github.com/nomemory/java-bean-validation-extension)：JBVE是一个小型工具库，它通过额外注解扩展了Java Bean Validation规范。
* [Collection Validator](https://github.com/jirutka/validator-collection)：该库可以轻松地为任何验证约束创建“伪约束”来标注简单类型的集合，而无需为每个集合编写额外的验证器或不必要的包装类。
* [dOOv](https://github.com/doov-org/doov)：dOOv是一个用于类型安全域模型验证和映射的流式API。
* [Avaje Validator](https://github.com/avaje/avaje-validator)：通过APT源代码生成进行无反射POJO验证的Java库。
* [OVal](https://github.com/sebthom/oval)：OVal是一个实用且可扩展的验证框架，适用于任何类型的Java对象，可以使用注解(@NotNull、@MaxLength)、POJO或XML来声明约束。
* [Functional Validation](https://github.com/MAIF/functional-validation)：该库提供了工具来验证Bean并组合验证堆栈错误。
* [Vador](https://github.com/salesforce-misc/Vador)：Vador是一个现代验证框架，旨在简化和提高REST API验证，由Salesforce开源。
* [SpEL Validator](https://github.com/stick-i/spel-validator)：SpEL Validator是基于Spring
  EL的参数校验包，也是javax.validation的扩展增强包，用于简化参数校验。

## IPP

* [JIPP](https://github.com/HPInc/jipp)：IPP的Java兼容实现，由惠普开源。
* [Cups4j](https://github.com/harwey/cups4j)：CUPS的Java打印库。
* [IPP Client Kotlin](https://github.com/gmuth/ipp-client-kotlin)：适用于Java和Kotlin的IPP协议客户端实现。

## OSC

* [JavaOSC](https://github.com/hoijui/JavaOSC)：JavaOSC赋予JVM语言程序以OSC内容格式序列化、解析、发送和接收数据的能力。
* [OSCP5](https://github.com/sojamo/oscp5)：Java和Processing的OSC实现。

## FinTS

* [HBCI4Java](https://github.com/hbci4j/hbci4java)：基于Java的FinTS协议实现，支持所有功能(chipTAN、pushTAN、HHD、SEPA、PSD2)。
* [Hibiscus](https://github.com/willuhn/hibiscus)：适用于Linux、Windows和MacOS的免费家庭银行业务分析。

## CalDAV

* [CalDAV4j](https://github.com/caldav4j/caldav4j)：CalDAV4j是一个实现CalDAV协议的Java库。
* [Cosmo](https://github.com/mam-dev/cosmo)：Cosmo日历服务器实现了CalDAV协议的服务器端。

## WebDav

* [Sardine](https://github.com/lookfirst/sardine)：Sardine可用于与WebDAV服务器交互，并且与FTP相比，以编程方式管理远程文件要容易得多。
* [Jackrabbit WebDAV](https://jackrabbit.apache.org/jcr/components/jackrabbit-webdav-library.html)：该组件提供用于构建WebDAV服务器或客户端的接口和常用实用程序类。
* [Dav4JVM](https://github.com/bitfireAT/dav4jvm)：Dav4JVM是用于JVM(Java/Kotlin)的WebDAV/CalDAV/CardDAV库，它最初是为DAVx⁵开发的。
* [Milton](https://github.com/miltonio/milton2)：可在Windows、Mac、Linux、Android和iOS上运行的Milton Java WebDAV/CalDAV/CardDAV服务器库。

## AirPlay

* [Open AirPlay](https://github.com/openairplay/open-airplay)：Apple的AirPlay协议的库集合。
* [Java AirPlay Lib](https://github.com/serezhka/java-airplay-lib)：此库旨在轻松创建类似Apple TV的AirPlay2服务器。

## 充电桩

* [SteVe](https://github.com/steve-community/steve)：SteVe于2013年在亚琛工业大学成立，提供了管理充电点、用户数据和用于用户身份验证的RFID卡的基本功能，并已在运行中进行了成功测试。
* [Java OCA OCPP](https://github.com/ChargeTimeEU/Java-OCA-OCPP)：OCA定义的开放充电点协议(OCPP)的开源客户端和服务器库。
* [OCPP](https://github.com/ShellRechargeSolutionsEU/ocpp)：该库是欧洲最大的电动汽车充电站运营商之一NewMotion开发和使用的OCPP的实现。

## 元编程

* [JParsec](https://github.com/jparsec/jparsec)：Jparsec是一个为Java编写的递归下降解析器组合器框架。
* [Parboiled](https://github.com/sirthias/parboiled)：Parboiled是一个混合Java/Scala库，提供基于解析表达式语法(PEG)的轻量级且易于使用但功能强大且优雅的任意输入文本解析。
* [Eclipse Xtext](https://github.com/eclipse/xtext)：Xtext是一个用于开发编程语言和特定领域语言的框架。
* [MPS](https://github.com/JetBrains/MPS)：JetBrains元编程系统。
* [GenSym](https://github.com/Generative-Program-Analysis/GenSym)：GenSym是LLVM IR的高性能并行符号执行引擎。
* [Rascal](https://github.com/usethesource/rascal)：Rascal是一个开源、基于Java的元编程库，由荷兰阿姆斯特丹自由大学的UseTheSource团队开发。
* [Procyon](https://github.com/mstrobel/procyon)：Procyon是一套专注于代码生成和分析的Java元编程工具。
* [JNI Bind](https://github.com/google/jni-bind)：JNI Bind是一个新的元编程库，为C++ => Java/Kotlin提供语法糖，Google开源。
* [JBSE](https://github.com/pietrobraione/jbse)：JBSE是一个用于自动程序分析、验证和测试生成的符号Java虚拟机。
* [Symbolic Java PathFinder](https://github.com/SymbolicPathFinder/jpf-symbc)：此JPF扩展为Java字节码提供符号执行，由NASA开源。

## 分词器

* [NLPIR](https://github.com/NLPIR-team/NLPIR)：NLPIR是由中科院计算所开发的分词工具。
* [Jieba Analysis](https://github.com/huaban/jieba-analysis)：结巴分词Java版。
* [Stanford Segmenter](https://nlp.stanford.edu/software/segmenter.shtml)：该软件用于对中文或阿拉伯语文本中的单词进行“标记化”或“分段”，由斯坦福开源。
* [Elasticsearch IK Analysis](https://github.com/medcl/elasticsearch-analysis-ik)：IK分词插件可将Lucene IK分词器集成到ElasticSearch中，支持自定义字典。
* [Sudachi](https://github.com/WorksApplications/Sudachi)：Sudachi是日本形态分析仪。
* [JTokkit](https://github.com/knuddelsgmbh/jtokkit)：JTokkit是一个专为与OpenAI模型一起使用而设计的Java分词器库。
* [Kuromoji](https://github.com/atilika/kuromoji)：Kuromoji是一个独立且非常易于使用的日语形态分析器，专为搜索而设计。
* [Word](https://github.com/ysc/word)：Word分词是一个Java实现的分布式的中文分词组件，提供了多种基于词典的分词算法。
* [Segment](https://github.com/houbb/segment)：Segment是基于结巴分词词库实现的更加灵活，高性能的Java分词实现。
* [IK Analyzer Solr](https://github.com/magese/ik-analyzer-solr)：Solr 7.X-8.X的IK分词器。
* [Twitter Korean Text](https://github.com/twitter/twitter-korean-text)：Twitter创建的开源韩语处理器。
* [STConvert](https://github.com/infinilabs/analysis-stconvert)：STConvert是一款将中文字符在繁体和简体之间转换的分析器，由INFINI Labs维护。
* [ElasticSearch Analysis Vietnamese](https://github.com/duydo/elasticsearch-analysis-vietnamese)：用于ElasticSearch的越南语分词插件。
* [ElasticSearch Analysis Ansj](https://github.com/NLPchina/elasticsearch-analysis-ansj)：ElasticSearch是一个基于ansj分词算法的ElasticSearch的中文分词插件。
* [IdeaSeg](https://gitee.com/indexea/ideaseg)：IdeaSeg是Indexea推出的一个基于最新的HanLP自然语言处理工具包实现的中文分词器。
* [THULAC](https://github.com/thunlp/THULAC-Java)：THULAC是由清华大学自然语言处理与社会人文计算实验室研制推出的一套中文词法分析工具包，具有中文分词和词性标注功能。
* [THUCTC](https://github.com/thunlp/THUCTC)：THUCTC是由清华大学自然语言处理实验室推出的中文文本分类工具包，能够自动高效地实现用户自定义的文本分类语料的训练、评测、分类功能。
* [ElasticSearch BosonNLP Analysis](https://github.com/bosondata/elasticsearch-analysis-bosonnlp)：玻森数据开发的一款基于玻森中文分词的ElasticSearch插件。
* [Ansj中文分词](https://github.com/NLPchina/ansj_seg)：这是一个基于n-Gram+CRF+HMM的中文分词的Java实现。
* [Ik Analyzer](https://github.com/blueshen/ik-analyzer)：支持Lucene 5/6/7/8/9+版本的分词器。
* [Elasticsearch Analysis Morfologik](https://github.com/allegro/elasticsearch-analysis-morfologik)：适用于ElasticSearch 8.x、7.x、6.x、5.x和2.x的Morfologik(波兰语)插件，由Allegro开源。
* [MMSeg4j](https://github.com/chenlb/mmseg4j-core)：MMSeg4j是使用Chih-Hao Tsai的MMSeg算法实现的中文分词器。

## 文本表

* [Picnic Tables](https://github.com/JakeWharton/picnic)：Kotlin DSL和Java/Kotlin构建器API，用于构建可呈现为文本的类似HTML的表格。
* [Flip Tables](https://github.com/JakeWharton/flip-tables)：用于在Java中打印漂亮的文本表。
* [ASCII Table](https://github.com/vdmeer/asciitable)：ASCII Table是一个简单的工具，用于格式化表格，具有缩进、缩进字符、对齐、填充、填充字符和行内空白字符的各种行/列选项。
* [ASCII Data](https://github.com/MitchTalmadge/ASCII-Data)：一个小型Java库，用于生成漂亮的基于文本的线图和表格。
* [Ascii Art Table](https://github.com/klaus31/ascii-art-table)：通过Java将数据打印到Ascii表的简单库。
* [ASCII Tables](https://github.com/freva/ascii-table)：使用Java轻松创建和自定义简单的ASCII表。

## 字体库

* [Aspose.Font](https://products.aspose.com/font/java/)：Aspose.Font使你能够轻松加载、保存、绘制、转换和渲染字体文件。
* [SFNTly](https://github.com/googlefonts/sfntly)：使用、编辑和创建基于SFNT的字体的库。
* [FontBox](https://github.com/apache/pdfbox/tree/trunk/fontbox)：FontBox库是一个用于处理PDF字体的开源Java工具。

## 语言库

* [TinyPinyin](https://github.com/promeG/TinyPinyin)：适用于Java和Android的快速、低内存占用的汉字转拼音库。
* [Elasticsearch Pinyin Analysis](https://github.com/infinilabs/analysis-pinyin)：该拼音分析插件用于进行汉字与拼音之间的转换。
* [Pinyin4j](https://github.com/belerweb/pinyin4j)：支持汉字(简体和繁体)到最流行的拼音系统，包括汉语拼音、通用拼音、Wade-Giles、MPS2、Yale和Gwoyeu Romatzyh。
* [Opencc4j](https://github.com/houbb/opencc4j)：Opencc4j支持中文繁简体转换，考虑到词组级别。
* [Pinyin](https://github.com/houbb/pinyin)：Java高性能中文转拼音工具，支持同音字。
* [Pinyin Plus](https://github.com/taptap/pinyin-plus)：汉字转拼音库，支持多音字，由Taptap开源。
* [Bopomofo4j](https://gitee.com/rnkrsoft/Bopomofo4j)：零依赖，纯Java开发的汉字转拼音库。
* [JPinyin](https://github.com/qzw1210/jpinyin)：JPinyin是一个汉字转拼音的Java开源类库，在PinYin4j的功能基础上做了一些改进。
* [Moji4J](https://github.com/andree-surya/moji4j)：Moji4J是一个开源Java库，用于在日语平假名、片假名和罗马字脚本之间进行转换。
* [Myanmar Tools](https://github.com/google/myanmar-tools)：该项目包括用于处理缅甸使用的字体编码的工具，目前支持广泛的Zawgyi-One字体编码，由Google开源。
* [BadWordFiltering](https://github.com/VaneProject/bad-word-filtering)：这是一个检查和处理脏话的库。
* [Jakaroma](https://github.com/nicolas-raoul/jakaroma)：Java汉字/等到罗马字转换器。

## 泛型库

* [TypeTools](https://github.com/jhalterman/typetools)：一个用于处理类型的简单、零依赖库，支持Java 1.6+和Android。
* [ClassMate](https://github.com/FasterXML/java-classmate)：ClassMate是一个零依赖Java库，用于准确内省类型信息，包括可靠解析类(“类型”)和成员(字段、方法和构造函数)的泛型类型声明。
* [Generics-Resolver](https://github.com/xvik/generics-resolver)：Java泛型运行时解析器。
* [GeantyRef](https://github.com/leangen/geantyref)：用于Java的泛型类型反射库。

## 国际化

* [CLDR](https://github.com/unicode-org/cldr)：CLDR为支持世界语言的软件提供了关键构建块，拥有最大、最广泛的可用区域设置数据标准存储库，由Unicode Consortium开源。
* [Java Locales](https://github.com/spotify/java-locales)：集中并标准化Java组件中Unicode区域设置的使用的库，由Spotify开发。
* [L10nMessages](https://github.com/pinterest/l10nmessages)：L10nMessages是一个使Java应用程序的国际化(i18n)和本地化(l10n)变得简单且安全的库，由Pinterest开源。
* [NV-1i8n](https://github.com/TakahikoKawasaki/nv-i18n)：支持国际化的包，包含ISO 3166-1国家代码枚举、ISO 639-1语言代码枚举、ISO 15924脚本代码枚举等。
* [Mojito](https://github.com/box/mojito)：Mojito是一个持续本地化平台，依靠持续集成将所有软件字符串收集到一处，实时查看哪些产品需要本地化。
* [Tolgee](https://github.com/tolgee/tolgee-platform)：Tolgee是一个本地化平台，允许你将应用程序翻译成任何语言，而无需修改代码。
* [Humanize](https://github.com/mfornos/humanize)：Humanize是一种Java工具，用于为数据添加国际化。
* [Tradukisto](https://github.com/allegro/tradukisto)：用于将数字转换为其单词表示形式的Java库，由Allegro开源。
* [Kilt](https://github.com/hupfdule/kilt)：Kilt是一组小工具，用于简化Java i18n资源包的处理。
* [ICU4j](https://github.com/unicode-org/icu)：为软件应用提供Unicode和国际化支持，由IBM开源。
* [Gettext](https://github.com/jgettext/gettext-commons)：Gettext Commons项目为国际化(i18n)通过GNU gettext和Java实现资源包。
* [Cosmopolitan](https://github.com/rodionmoiseev/c10n)：一个Java库，专注于使国际化更加模块化、更易于发展和维护、易于更改且IDE友好，无需过多的外部工具。
* [Easy I18N](https://github.com/awkay/easy-i18n)：这是一个Java库，旨在使创建国际化程序变得更加容易。
* [Resource4j](https://github.com/resource4j/resource4j)：Resource4j库是Java ResourceBundle机制的替代品，支持大型和遗留应用程序的复杂i18n场景，并提供对键/值应用程序配置和任意资源文件的安全访问。
* [Loc4J](https://loc4j.sourceforge.net/)：Loc4J是一个Java库，可帮助本地化应用程序。
* [I18n](https://github.com/vidageek/i18n)：在Java Web应用程序上使用i18n的简单方法。
* [Lokalized](https://github.com/lokalized/lokalized-java)：Lokalized有助于在JVM上进行听起来自然的软件翻译。
* [OmegaT](https://github.com/omegat-org/omegat)：OmegaT是一款免费开源多平台计算机辅助翻译工具，具有模糊匹配、翻译记忆库、关键字搜索、术语表以及翻译到更新项目中的功能。
* [Singleton](https://github.com/vmware/singleton)：Singleton是一个用于简化软件全球化的开源应用程序，由VMWare开源。
* [Kilt](https://github.com/poiu-de/kilt)：Kilt是一组小工具，用于简化Java i18n资源包的处理。

## 翻译库

* [MTrans](https://github.com/hujingshuang/MTrans)：MTrans提供了集多种主流的在线翻译及TTS功能于一身的轻量级服务。
* [Zanata](https://github.com/zanata/zanata-platform)：Zanata是一个基于Web的系统，供翻译人员使用Web浏览器在线翻译文档和软件。
* [Translator](https://github.com/therealbush/translator)：适用于Kotlin/JVM和Java的简单且免费的Google翻译库。
* [DeepL Java](https://github.com/DeepLcom/deepl-java)：DeepL API是一种语言翻译API，它允许其他计算机程序将文本和文档发送到DeepL的服务器并接收高质量的翻译。

## 字典库

* [Mdict Java](https://github.com/KnIfER/mdict-java)：用于Java的Mdict字典文件格式的查询库。
* [ExtJWNL](https://github.com/extjwnl/extjwnl)：ExtJWNL是一个用于创建、读取和更新WordNet格式词典的Java API。
* [Stardict4j](https://codeberg.org/miurahr/stardict4j)：Stardict4j是Java版StarDict词典文件的访问库。

## 颜色库

* [Color Java](https://github.com/ngageoint/color-java)：Color是一个Java库，提供颜色表示，支持十六进制、RBG、算术RBG、HSL和整数颜色，由美国国家地理空间情报局开发。
* [Color](https://github.com/bowbahdoe/color)：JVM的颜色库。

## 短链接

* [ShortLink](https://github.com/Enndfp/short-link)：SaaS短链接系统，为企业和个人用户提供了一个高效、安全和可靠的短链接管理平台。
* [Octopus](https://github.com/zjcscut/octopus)：Octopus是一个长链接压缩为短链接的服务。
* [URLShorter](https://gitee.com/tinyframework/urlshorter)：满足多种场景下的短链接生成需求。
* [URL Shortener](https://github.com/cami-la/url-shortener-preview)：URL Shortener是一个提供URL缩短服务的项目。
* [UrlShortener API](https://github.com/marinsborg/UrlShortener-API)：Url Shorter是一项在消息、Twitter、演示文稿等中共享URL时将长URL转换为短别名以节省空间的服务。

## 单位库

* [UnitOf](https://github.com/Digidemic/UnitOf)：适用于Java、JavaScript和C#的测量和数据类型转换库。
* [Unit API](https://github.com/unitsofmeasurement/unit-api)：Unit API提供了一组用于处理单位和数量的Java语言编程接口。
* [Measured](https://github.com/nacular/measured)：Measured提供了一种安全且简单的使用测量单位的方法。
* [QUDTLib](https://github.com/qudtlib/qudtlib-java)：QUDTLib为Java提供单位转换及相关功能。

## 调用图

* [Java CallGraph](https://github.com/Adrninistrator/java-all-call-graph)：Java CallGraph能够通过静态分析的方式批量生成指定Java方法向下的完整调用链。
* [Java CallGraph](https://github.com/gousiosg/java-callgraph)：一套用于在Java中生成静态和动态调用图的程序。

## 语言检测

* [Language Detection](https://github.com/shuyo/language-detection)：这是一个用纯Java实现的语言检测库。
* [Language Detector](https://github.com/optimaize/language-detector)：Java语言检测库。
* [JLangDetect](https://github.com/melix/jlangdetect)：JLangDetect是JVM的语言检测库。

## 词法解析

* [ANTLR](https://github.com/antlr/antlr4)：ANTLR是一个强大的解析器生成器，用于读取、处理、执行或翻译结构化文本或二进制文件。
* [SableCC](https://github.com/SableCC/sablecc)：SableCC是一个解析器生成器，它能够生成功能齐全的面向对象框架，用于构建编译器、解释器和其他文本解析器，由麦吉尔大学开发。
* [JavaParser](https://github.com/javaparser/javaparser)：该项目包含一组实现具有高级分析功能的Java 1.0 - Java 17解析器的库。
* [Flexmark Java](https://github.com/vsch/flexmark-java)：Flexmark Java是CommonMark(规范0.28)解析器的Java实现，使用块优先、内联后Markdown解析架构。
* [Kotlinx.AST](https://github.com/kotlinx/ast)：Kotlinx.AST是一个通用的AST解析库，Kotlin是目前唯一支持的语言。
* [PartiQL Lang Kotlin](https://github.com/partiql/partiql-lang-kotlin)：PartiQL是一种富有表现力的、与SQL兼容的查询语言，可以访问关系型、半结构化和嵌套数据，由Amazon开源。
* [Gumtree Spoon AST Diff](https://github.com/SpoonLabs/gumtree-spoon-ast-diff)：使用Gumtree算法计算两个Spoon抽象语法树之间的AST差异。
* [JavaCC](https://github.com/javacc/javacc)：JavaCC是用于Java应用程序的最流行的解析器生成器，最初由Sun开发。
* [JFlex](https://github.com/jflex-de/jflex)：JFlex是Java的词法分析器生成器(也称为扫描器生成器)。
* [JCPP](https://github.com/shevek/jcpp)：JCPP是C预处理器的完整、兼容、独立、纯Java实现。
* [JLex](https://www.cs.princeton.edu/~appel/modern/java/JLex/)：JLex是一个用Java编写的词法分析器生成器，由普林斯顿大学开发。
* [RSQL Parser](https://github.com/jirutka/rsql-parser)：RSQL是一种用于对RESTful API中的条目进行参数化过滤的查询语言。
* [Parboiled](https://github.com/sirthias/parboiled)：Parboiled是一个混合Java/Scala库，提供基于解析表达式语法(PEG)的轻量级且易于使用但功能强大且优雅的任意输入文本解析。
* [CUP](http://www2.cs.tum.edu/projects/cup/)：CUP是Java的一个LALR解析器生成器，慕尼黑工业大学开发。
* [Parrot](https://github.com/daniellansun/groovy-parser)：Parrot可以解析Groovy源代码并构造相关的AST，与旧解析器生成的AST几乎相同。
* [Canopy](https://github.com/jcoglan/canopy)：Canopy是一个针对Java、JavaScript、Python和Ruby的解析器编译器。

## Tree Sitter

* [Tree Sitter Java](https://github.com/tree-sitter/tree-sitter-java)：Tree-Sitter的Java语法。
* [Java Tree Sitter](https://github.com/serenadeai/java-tree-sitter)：Tree-Sitter是一个解析器生成工具和增量解析库。
* [Java Tree Sitter](https://github.com/seart-group/java-tree-sitter)：Tree-Sitter的Java绑定，由瑞士卢加诺的意大利大学软件研究所开源。

## 形式验证

* [CATG](https://github.com/ksen007/janala2)：Concolic单元测试引擎，使用形式化方法自动生成单元测试。
* [JKind](https://github.com/loonwerks/jkind)：JKind是基于SMT的Lustre安全属性无限状态模型检查器，由柯林斯航空航天公司开源。
* [Asmeta](https://github.com/asmeta/asmeta)：Asmeta是抽象状态机(ASM)形式化方法的框架，它由用于执行不同验证和确认活动的不同工具组成，米兰大学开源。
* [PRISM](https://github.com/prismmodelchecker/prism)：PRISM是一个概率模型检查器，用于对表现出随机或概率行为的系统进行形式化建模和分析，由牛津大学开源。
* [Checker Framework](https://checkerframework.org/)：可插拔类型系统，包括空类型、物理单位、不变性类型等等。
* [Daikon](https://github.com/codespecs/daikon)：检测可能的程序不变量并根据这些不变量生成JML规范，由华盛顿大学开源。
* [Java PathFinder](https://github.com/javapathfinder/jpf-core)：JVM形式验证工具，包含模型检查器等，由NASA开源。
* [JmlOk2](https://massoni.computacao.ufcg.edu.br/home/jmlok)：通过反馈引导的随机测试生成来检测代码和JML规范之间的不一致，并建议检测到的每个不符合项的可能原因，由大坎皮纳联邦大学开发。
* [jCUTE](https://github.com/osl/jcute)：jCUTE自动生成Java程序的单元测试，Concolic执行将随机具体执行与符号执行和自动约束求解相结合。
* [Event-B](https://www.event-b.org/index.html)：Event-B是一种用于系统级建模和分析的形式化方法，由苏黎世联邦理工学院、杜塞尔多夫海因里希海涅大学、纽卡斯尔大学和南安普顿大学开发和维护。
* [KeY](https://github.com/KeYProject/key)：形式化软件开发工具，旨在尽可能无缝地集成面向对象软件的设计、实现、形式化规范和形式化验证，由卡尔斯鲁厄理工学院开源。
* [OpenJML](https://github.com/OpenJML/OpenJML)：Java程序的程序验证工具，可让你检查以Java建模语言注释的程序规范。
* [Java Modeling Language](https://www.cs.ucf.edu/~leavens/JML/index.shtml)：JML是一种行为接口规范语言，可用于指定Java模块的行为，佛罗里达大学开发。
* [Theta](https://github.com/ftsrg/theta)：Theta是布达佩斯技术经济大学关键系统研究组开发的通用、模块化和可配置的模型检查框架，旨在支持基于抽象细化的算法的设计和评估，以进行各种形式主义的可达性分析。

## 项目模板

* [JBoss Forge](https://github.com/forge/core)：JBoss Forge是一款软件开发工具，可扩展你的Java IDE，为不同的技术和解决方案提供向导和扩展。
* [PLMCodeTemplate](https://github.com/xwjie/PLMCodeTemplate)：Spring开发代码模板。
* [Spring Boot Microservices](https://github.com/rohitghatol/spring-boot-microservices)：用于微服务架构的Spring Boot模板。
* [AWS CloudFormation Template](https://github.com/widdix/aws-cf-templates)：AWS CloudFormation的免费模板。
* [Selenium Maven Template](https://github.com/Ardesco/Selenium-Maven-Template)：快速搭建Selenium项目的Maven模板。
* [Spring Boot Template](https://github.com/hmcts/spring-boot-template)：该模板的目的是加快新Spring应用程序的创建速度，并帮助在多个团队之间保持相同的标准，由英国法院及审裁处事务局开源。
* [SpringBoot Java Template](https://github.com/team-dodn/spring-boot-java-template)：基于Java的Spring Boot基本结构模板。
* [AEM Project Archetype](https://github.com/adobe/aem-project-archetype)：Maven模板，创建一个最小的、基于最佳实践的Adobe Experience Manager(AEM)项目作为你网站的起点。
* [Spring MVC Quickstart Maven Archetype](https://github.com/kolorobot/spring-mvc-quickstart-archetype)：该项目是Spring MVC Web应用程序的Maven原型。
* [Spring Boot Starter](https://github.com/ericus20/spring-boot-starter)：一个高度固执且完整的Spring Boot生产就绪项目的Starter。
* [Spring Boot Starter-kit](https://github.com/khandelwal-arpit/springboot-starterkit)：适用于Spring Boot应用程序的生产就绪入门套件。
* [Spring Boot Boilerplate](https://github.com/Genc/spring-boot-boilerplate)：Spring Boot
  Boilerplate是一个Starter套件，该项目包括Spring Boot(v 2.7.10)、Spring Data JPA、Spring Validation、Spring Security + JWT Token、PostgreSQL、Mapstruct、Lombok、Swagger。
* [Spring Boot Supabase](https://github.com/ChangeNode/spring-boot-supabase)：现代Java Web应用程序入门模板。

## 印章生成

* [SealKit](https://gitee.com/liuzy1988/SealKit)：印章生成工具。
* [SealUtil](https://github.com/localhost02/SealUtil)：印章生成工具，使用Java Graphics2D生成各类圆形/椭圆公章、私章图片。
* [开放签](https://gitee.com/kaifangqian/kaifangqian-base)：开放签提供企业印章制作、证书签发、文件签署API接口服务。

## 数据脱敏

* [eJMask](https://github.com/eBay/ejmask)：eJMask是一个基于JVM的屏蔽库，它提供了一个易于使用的API来屏蔽Java应用程序中的敏感数据，由eBay开源。
* [DeSensitization](https://github.com/allurx/desensitization)：Blur是一个Java库，用于屏蔽和混淆任何数据结构中的敏感数据。

## N+1问题

* [DBUtil](https://github.com/vladmihalcea/db-util)：该工具可以在测试期间自动检测N+1查询问题。
* [JPlusOne](https://github.com/adgadev/jplusone)：JPlusOne是用于自动检测和断言基于JPA的Spring Boot Java应用程序中发生的“N+1问题”并查找JPA发出的SQL语句的一般来源的工具。
* [Spring Hibernate Query Utils](https://github.com/yannbriancon/spring-hibernate-query-utils)：该库提供了检测N+1查询并对Spring和Hibernate生成的查询进行计数的工具。

## 敏感词过滤

* [Sensitive-Word](https://github.com/houbb/sensitive-word)：基于DFA算法实现的高性能敏感词工具。
* [Sensitive-Word-Filter](https://github.com/hailin0/sensitive-word-filter)：简易敏感词处理器，支持返回敏感词、高亮敏感词、替换敏感词等操作。
* [Sensitive-Words-Filter](https://github.com/hooj0/sensitive-words-filter)：敏感词过滤项目，提供TTMP、DFA、DAT、Hash Bucket、Tire算法支持过滤。

## 正则表达式

* [RegexGenerator](https://github.com/MaLeLabTs/RegexGenerator)：该项目包含用于生成文本提取正则表达式的工具的源代码。
* [JavaVerbalExpressions](https://github.com/VerbalExpressions/JavaVerbalExpressions)：VerbalExpressions是一个Java库，可帮助构建困难的正则表达式。
* [Generex](https://github.com/mifmif/Generex)：用于生成与给定正则表达式匹配的字符串的Java库。
* [RE2/J](https://github.com/google/re2j)：一个正则表达式引擎，其运行时间与输入大小成线性关系，由Google开源。
* [Joni](https://github.com/jruby/joni)：Oniguruma正则表达式库的Java端口。
* [Hyperscan-Java](https://github.com/gliwka/hyperscan-java)：Hyperscan是一个高性能的多正则表达式匹配库。
* [DRegex](https://github.com/marianobarrios/dregex)：DRegex是一个Java库，它使用确定性有限自动机(DFA)实现正则表达式引擎。
* [Readable Regex](https://github.com/ricoapon/readable-regex)：使用这个库，可以以可读的方式创建正则表达式。
* [Readable Regex](https://github.com/codebox/readable-regex)：该库提供了一种使Java代码中的复杂正则表达式更具可读性的方法。
* [RgxGen](https://github.com/curious-odd-man/RgxGen)：根据正则表达式模式生成匹配和不匹配的字符串。
* [JRegex](https://sourceforge.net/projects/jregex/)：Java正则表达式库。
* [RegExodus](https://github.com/tommyettinger/RegExodus)：正则表达式库，可跨Java变体移植。

## 代码生成器

* [Auto](https://github.com/google/auto)：一系列用于Java的源代码生成器，由Google开发。
* [Spring Initializr](https://github.com/spring-io/initializr)：Spring项目的快速生成器。
* [Joda-Beans](https://github.com/JodaOrg/joda-beans)：Joda-Beans提供了一个向Java添加属性的小型框架，极大地增强了Java Bean。
* [Burningwave](https://github.com/burningwave/core)：一个先进且高度优化的Java库，用于构建框架：它对于扫描类路径、在运行时生成类、促进反射的使用、扫描文件系统、执行字符串化源代码等等很有用。
* [JavaPoet](https://github.com/square/javapoet)：用于生成.java源文件的Java API，由Square开源。
* [Java::Geci](https://github.com/verhas/javageci)：Java::Geci是一个用于生成Java代码的库，可以使用Java::Geci执行代码生成程序来生成新的源代码或修改现有的Java源文件。
* [Fulib](https://github.com/fujaba/fulib)：Fulib是一个为UML类模型和一些模型管理功能提供代码生成的库，使用Java API提供的特定于域的语言，它允许你定义类、属性以及与元模型的关联。
* [Wsilk](https://github.com/wuba/wsilk)：Wsilk是一个辅助开发人员通过Java语言生成代码的一个工具框架，由58同城开源。
* [AutoRecord](https://github.com/pawellabaj/auto-record)：AutoRecord是一个代码生成器，可以帮助你轻松生成Java记录。
* [MAKU](https://gitee.com/makunet/maku-generator)：一款低代码生成器，可根据自定义模板内容，快速生成代码，可实现项目的快速开发、上线，减少重复的代码编写。
* [Sculptor](https://github.com/sculptor/sculptor)：Sculptor是一个代码生成器，应用了领域驱动设计和领域特定语言的概念。
* [Bootify](https://bootify.io/)：使用JPA模型和REST API生成基于浏览器的Spring Boot应用程序，商业项目。
* [Scrooge](https://github.com/twitter/scrooge)：Scrooge是一个用Scala编写的thrift代码生成器，目前可以为Scala、Java、Cocoa、Android和Lua生成代码，Twitter开源。
* [SourceBuddy](https://github.com/sourcebuddy/sourcebuddy)：SourceBuddy是一个Java库，可用于在程序中编译和加载动态生成的Java源代码。
* [Code Gen](https://gitee.com/durcframework/code-gen)：一款代码生成工具，可自定义模板生成不同的代码，支持MySQL、Oracle、SQL Server、PostgreSQL。
* [AiCode](https://gitee.com/lemur/aicode)：新一代代码生成器，根据模板配置生成代码。
* [Jeddict](https://github.com/jeddict/jeddict)：Jakarta EE 10和MicroProfile应用程序生成器和建模器。
* [Generator-SpringBoot](https://github.com/sivaprasadreddy/generator-springboot)：用于生成Spring Boot微服务的Yeoman生成器。
* [Enunciate](https://github.com/stoicflame/enunciate)：Enunciate是一个构建时Web服务增强工具，可应用于基于Java的项目，以便从Web服务端点的源代码生成大量很酷的工件。
* [Telosys](https://github.com/telosys-tools-bricks/telosys-cli)：简单而轻量的代码生成器，可作为Eclipse插件和CLI使用。
* [Magic Bean](https://github.com/bowbahdoe/magic-bean)：一个生成Getter和Setter的非常基本的库。
* [Parceler](https://github.com/johncarl81/parceler)：Parceler是一个代码生成库，可生成Android Parcelable样板源代码。
* [JCodeModel](https://github.com/phax/jcodemodel)：Java代码生成库。
* [Spring Boot Code Generator](https://github.com/moshowgame/SpringBootCodeGenerator)：基于Spring Boot 2 + Freemarker的Java代码生成器。
* [Spring Roo](https://github.com/spring-attic/spring-roo)：Spring Roo是面向Java开发人员的快速应用程序开发(RAD)工具，它允许我们使用简单易用的命令为Spring应用程序生成样板代码和项目结构。
* [Acceleo](https://eclipse.dev/acceleo/)：Acceleo是Eclipse的开源代码生成器，可从任何元模型(UML、SysML等)定义的EMF模型生成代码。

## 类路径扫描

* [ClassGraph](https://github.com/classgraph/classgraph)：ClassGraph是一个超快速的并行类路径扫描器和模块扫描器，适用于Java、Scala、Kotlin和其他JVM语言。
* [Scannotation](https://scannotation.sourceforge.net/)：Scannotation是一个Java库，它从一组.class文件创建注解数据库。
* [Annovention](https://github.com/ngocdaothanh/annovention)：Annovention是一个Java注解发现库。
* [ClassIndex](https://github.com/atteo/classindex)：ClassIndex通过提供标准注解处理器的实现来在编译时对你的类进行索引。
* [Jandex](https://github.com/smallrye/jandex)：Jandex是一个节省空间的Java类文件索引器和离线反射库。
* [Extcos](https://sourceforge.net/projects/extcos/)：Extcos是一个Java组件扫描库。
* [QDox](https://github.com/paul-hammant/qdox)：QDox是一个高速、占用空间小的解析器，用于完全提取类/接口/方法定义(包括注解、参数、参数名称)，由Throughworks开发。
* [INFOMAS ASL](https://github.com/rmuller/infomas-asl)：INFOMAS ASL可用于扫描类路径以查找带注解的类、方法或实例变量。

## 目录服务

* [PWM](https://github.com/pwm-project/pwm)：PWM是一个用于LDAP目录的开源密码自助服务应用程序。
* [DavMail](https://github.com/mguessan/davmail)：POP/IMAP/SMTP/Caldav/Carddav/LDAP Exchange和Office 365网关。
* [Spring LDAP](https://github.com/spring-projects/spring-ldap)：Spring LDAP是一个用于简化Java LDAP编程的库，其构建原理与Spring JDBC相同。
* [UnboundID LDAP SDK](https://github.com/pingidentity/ldapsdk)：UnboundID LDAP
  SDK是一个快速、功能强大、用户友好且完全免费的开源Java库，用于与LDAP目录服务器进行通信。
* [Apache DS](https://github.com/apache/directory-server)：DS是一个完全用Java编写的可扩展、可嵌入的目录服务器，已通过Open Group认证，兼容LDAPv3。
* [OpenDJ Community Edition](https://github.com/ForgeRock/opendj-community-edition)：OpenDJ是一个目录服务器，它实现了广泛的轻量级目录访问协议和相关标准，包括完全符合LDAPv3，而且还支持目录服务标记语言。
* [Apache Directory Kerby](https://github.com/apache/directory-kerby)：Directory子项目，是Java Kerberos绑定。它提供了丰富、直观且可互操作的实现、库、KDC和各种设施，根据云、Hadoop和移动等现代环境的需要集成了PKI、OTP和令牌。
* [Apache Directory Studio](https://github.com/apache/directory-studio)：Directory Studio是一个完整的目录工具平台，旨在与任何LDAP服务器一起使用，但它是专门为与ApacheDS一起使用而设计的。
* [Rogue JNDI](https://github.com/veracode-research/rogue-jndi)：用于JNDI注入攻击的恶意LDAP服务器。
* [Spring Data LDAP](https://github.com/spring-projects/spring-data-ldap)：Spring Data LDAP项目旨在为Spring LDAP提供熟悉且一致的存储库抽象。
* [OpenDJ](https://github.com/OpenIdentityPlatform/OpenDJ)：OpenDJ是一种兼容LDAPv3的目录服务，专为Java平台开发，可为组织管理的身份提供高性能、高可用性且安全的存储。
* [LDAP Synchronization Connector](https://github.com/lsc-project/lsc)：LDAP同步连接器从任何数据源(
  包括数据库、LDAP目录或文件)读取数据，并转换此数据并将其与LDAP目录进行比较。
* [JXplorer](https://github.com/pegacat/jxplorer)：JXplorer是一个跨平台的LDAP浏览器和编辑器。
* [Ldaptive](https://github.com/vt-middleware/ldaptive)：用于与LDAP服务器交互的简单、可扩展的Java API。

## 异常处理

* [SneakyThrow](https://github.com/rainerhahnekamp/sneakythrow)：SneakyThrow是一个用于忽略受检异常的Java库。
* [Faux Pas](https://github.com/zalando/faux-pas)：Faux Pas是一个简化Java函数式编程错误处理的库，由Zalando开源。
* [Either](https://github.com/jbock-java/either)：Java的函数式错误处理库。
* [Catch Exception](https://github.com/Codearte/catch-exception)：Catch Exception库在单行代码中捕获异常，并使它们可用于进一步分析。
* [Maybe](https://github.com/JoseLion/maybe)：Maybe是一个类似于java.util.Optional的单子包装器，但意图不同。

## 表情处理

* [Emoji](https://github.com/delight-im/Emoji)：对Java和Android的表情符号支持。
* [Emoji-Java](https://github.com/vdurmont/emoji-java)：Emoji-Java是一个轻量级的Java库，可帮助你在Java应用程序中使用表情符号。
* [Emoji4j](https://github.com/kcthota/emoji4j)：用于将短代码、HTML实体转换为表情符号的Java库，还支持解析表情符号、代理HTML实体。
* [Java Emoji Converter](https://github.com/binarywang/java-emoji-converter)：Emoji转换工具，便于各种规格客户端生成的Emoji字符串转换成另外一种格式。
* [Emoji](https://github.com/vanniktech/Emoji)：一个Kotlin多平台库，用于向Android应用程序/JVM后端添加表情符号支持。
* [Emoji-Java](https://github.com/coding/emoji-java)：一个轻量级的Java库，可帮助你在Java应用程序中使用表情符号，由Coding开源。
* [JEmoji](https://github.com/felldo/JEmoji)：JEmoji是一个轻量级且快速的Java表情符号库，包含来自unicode联盟的所有表情符号的完整列表。
* [EmojiReader](https://github.com/YvesCheung/EmojiReader)：EmojiReader是一个能在字符串中识别出Emoji的简单工具。

## 行为分析

* [Dawn](https://github.com/eventtracing/dawn)：曙光埋点集自动化埋点与全链路追踪等特点于一身，近乎完美地解决了传统埋点的所有痛点，兼顾了开发效率与埋点数据的高精度特点，网易云开源。
* [小象用户行为分析平台](https://gitee.com/xiaoxiangopen/analysis)：商用产品开源，包括用户埋点数据采集、用户标签分群和画像、智慧运营、营销等。
* [ClkLog](https://gitee.com/clklog/clklog)：ClkLog是一款记录用户行为分析和画像的免费可商用开源软件，技术人员可快速搭建私有的应用系统。

## ASCII艺术

* [Jansi](https://github.com/fusesource/jansi)：Jansi是一个小型Java库，允许使用ANSI转义序列来格式化控制台输出，甚至可以在Windows上运行。
* [Mordant](https://github.com/ajalt/mordant)：Mordant是一个多平台库，用于在终端中渲染样式文本。
* [Java ASCII Render](https://github.com/indvd00m/java-ascii-render)：纯Java的ASCII渲染器，没有外部依赖，支持图形基元/元素、图层、上下文、画布。
* [ConsoleUI](https://github.com/awegmann/consoleui)：微型Java库，可在基于ANSI控制台的终端上启用简单的UI元素。
* [Jfiglet](https://github.com/lalyos/jfiglet)：FIGfonts的Java实现，用于创建Ascii横幅。
* [Java-Ascii-Table](https://github.com/nedtwigg/asciitable)：Java中用于ASCII表的简单库。
* [JColor](https://github.com/dialex/JColor)：JColor提供了一种简单的语法，可以在终端上以彩色字体或背景打印消息。
* [Colorized Java](https://github.com/vieitesss/colorize-Java)：在Java控制台中打印彩色文本的小型库。
* [Asciimg](https://github.com/korhner/asciimg)：Asciimg是一个用Java编写的可扩展Ascii艺术生成器。

## Unicode

* [AnyAscii](https://github.com/anyascii/anyascii)：AnyAscii为几乎所有Unicode字符提供纯ASCII替换字符串。
* [JUnidecode](https://github.com/gcardone/junidecode)：JUnidecode是用于将Unicode转换到ASCII的Java库。
* [Unidecode](https://github.com/xuender/unidecode)：Java的Unicode字符串的ASCII音译库。

## URL操作

* [Url Detector](https://github.com/linkedin/URL-Detector)：Url Detector是由LinkedIn安全团队创建的一个库，用于检测和提取长文本中的URL。
* [Slugify](https://github.com/slugify/slugify)：用于生成语音URL的小型工具类库。
* [Unbescape](https://github.com/unbescape/unbescape)：Unbescape是一个Java库，旨在执行功能齐全且高性能的转义和取消转义操作。
* [Java URLBuilder](https://github.com/mikaelhg/urlbuilder)：无运行时依赖的URL构建器。
* [AutoLink Java](https://github.com/robinst/autolink-java)：用于从纯文本中提取URL和电子邮件地址等链接的Java库。
* [Rewrite](https://github.com/ocpsoft/rewrite)：适用于Java EE 6+和Servlet 2.5+应用程序的高度可配置的URL重写工具。
* [Galimatias](https://github.com/smola/galimatias)：Galimatias是一个用Java编写的URL解析和规范化库。
* [JURL](https://github.com/anthonynsimon/jurl)：快速简单的Java URL解析库，支持UTF-8和路径解析。
* [Handy URI Templates](https://github.com/damnhandy/Handy-URI-Templates)：实现RFC6570的Java URI模板处理器。
* [UrlRewriteFilter](https://github.com/paultuckey/urlrewritefilter)：具有类似于Apache的mod_rewrite功能的Java Web过滤器。
* [URLCanon](https://github.com/iipc/urlcanon)：适用于Python和Java的URL规范化库，由国际互联网保护联盟开源。
* [UrlEncoder](https://github.com/ethauvin/urlencoder)：UrlEncoder是一个简单的防御库，用于对URL组件进行编码/解码。

## WebRTC

* [OpenVidu](https://github.com/OpenVidu/openvidu)：OpenVidu是一个方便在Web或移动应用程序中添加视频通话的平台，它提供了完整的技术堆栈，非常容易集成到你的应用程序中。
* [WebRTC Java](https://github.com/devopvoid/webrtc-java)：基于免费、开源的WebRTC项目的Java原生接口实现。
* [RestComm SIP Servlet](https://github.com/RestComm/sip-servlets)：RestComm SIP Servlet是SIP、IMS和WebRTC应用服务器。
* [ICE4j](https://github.com/jitsi/ice4j)：ICE协议的Java实现。
* [NextRTC](https://github.com/mslosarz/nextrtc-signaling-server)：NextRTC是用Java编写的简单WebRTC信令服务器，它提供信号交换和易于集成的API。
* [Kurento](https://github.com/Kurento/kurento)：Kurento Media Server负责媒体传输、处理、加载和记录。
* [BulletJournal](https://github.com/singerdmx/BulletJournal)：BulletJournal是一个开源平台，用于笔记本保存、账本管理、任务/项目管理和协调，擅长个人组织、日程安排、提醒、待办事项列表、笔记共享、多人账本和团队项目协作。
* [OnChat](https://github.com/onch-at/onchat)：一个简单、美观、移动优先的即时消息渐进式Web应用程序。
* [Jitsi Videobridge](https://github.com/jitsi/jitsi-videobridge)：Jitsi Videobridge是一个兼容WebRTC的选择性转发单元(SFU)，即多媒体路由器。

## Expect库

* [ExpectIt](https://github.com/agavrilov76/ExpectIt)：ExpectIt是Expect工具的另一个纯Java 1.6+实现。
* [Expect4J](https://github.com/cverges/expect4j)：用Java重写Expect并提供与TclJava解释器的绑定。
* [ExpectJ](https://expectj.sourceforge.net/)：ExpectJ可用于自动与进程或telnet会话进行交互。
* [Enchanter](https://bitbucket.org/mrdon/enchanter)：Enchanter是一个小型库，可以帮助你以类似于Expect的方式编写SSH会话脚本。
* [Expect Java](https://github.com/ronniedong/Expect-for-Java)：Expect工具的纯Java实现。
* [Expect4Java](https://github.com/iTransformers/expect4java)：Java的Expect语言实现，使用Java 8闭包。

## JavaME

* [MicroEmu](https://code.google.com/archive/p/microemu/)：MicroEmu是Java ME的纯Java实现。
* [J2ME-Loader](https://github.com/nikita36078/J2ME-Loader)：J2ME-Loader是适用于Android的J2ME模拟器，它支持大多数2D和3D游戏(包括Mascot Capsule 3D游戏)。
* [SquirrelJME](https://github.com/SquirrelJME/SquirrelJME)：SquirrelJME是用于嵌入式和物联网设备的Java ME 8虚拟机，它的最终目标是与Java ME标准99.9%兼容。
* [FreeJ2ME](https://github.com/hex007/freej2me)：包含libretro、awt和sdl2前端的免费J2ME模拟器。
* [PluotSorbet](https://github.com/mozilla/pluotsorbet)：PluotSorbet是一个用JavaScript编写的兼容J2ME的虚拟机，由Mozilla开源。

## JavaCard

* [Sun/Oracle JavaCard SDK](https://github.com/martinpaljak/oracle_javacard_sdks)：Java Card开发工具包是一套工具，用于设计Java Card技术的实现并根据Java Card API规范开发小应用程序。
* [GPPro](https://github.com/martinpaljak/GlobalPlatformPro)：GPPro允许在兼容的JavaCard智能卡上加载和管理小程序。
* [Ant JavaCard](https://github.com/martinpaljak/ant-javacard)：Ant JavaCard是一个易于使用的Ant任务，用于构建JavaCard Classic小程序(2.1.1至3.1.0)。
* [JCardSim](https://github.com/licel/jcardsim)：JCardSim是Java Card的开源模拟器。
* [Gradle JavaCard](https://github.com/fidesmo/gradle-javacard)：该插件允许将编译的class文件转换为转换后的存档格式，这些文件可用于安装在支持SUN/Oracle JavaCard技术的智能卡和SIM卡上。
* [Keycard](https://github.com/status-im/status-keycard)：Keycard是在JavaCard 3.0.4+上运行的BIP-32 HD钱包的实现。
* [SmartPGP](https://github.com/github-af/SmartPGP)：SmartPGP是JavaCard中OpenPGP卡3.4规范的免费开源实现。
* [Java Card OpenPGP Card](https://github.com/jderuiter/javacard-openpgpcard)：这是OpenPGP智能卡规范的Java Card实现。
* [IsoApplet](https://github.com/philipWendland/IsoApplet)：IsoApplet是Java Card智能卡的开源小程序，其目的是与OpenSC一起在现代智能卡上执行公钥加密。
* [JCAlgTest](https://github.com/crocs-muni/JCAlgTest)：基于JavaCard平台的某类智能卡所支持的密码算法的自动化测试工具，由马萨里克大学开源。
* [PivApplet](https://github.com/arekinath/PivApplet)：适用于JavaCard 2.2.2和3.0.4+的PIV小程序，具有完整的ECDSA/ECDH支持。
* [JCMathLib](https://github.com/OpenCryptoProject/JCMathLib)：JCMathLib是JavaCard平台的开源库，旨在实现标准JavaCard API中不可用的低级加密计算。
* [OpenJavaCard Tools](https://github.com/OpenJavaCard/openjavacard-tools)：该项目是一个用于JavaCard开发和配置的工具包。
* [EMV Card Simulator](https://github.com/mrautio/emv-card-simulator)：用于支付终端功能和安全测试/模糊测试的EMV卡的JavaCard实现。
* [OpenJavaCard Libraries](https://github.com/OpenJavaCard/openjavacard-libraries)：这是一组用于JavaCard环境的可重用库。
* [OpenJavaCard NDEF](https://github.com/OpenJavaCard/openjavacard-ndef)：JavaCard的NDEF标签实现。

## Wikipedia

* [JWiki](https://github.com/fastily/jwiki)：用于轻松与Wikipedia/MediaWiki交互的库。
* [DBpedia](https://github.com/dbpedia/extraction-framework)：DBpedia旨在从Wikipedia中提取结构化信息并将这些信息发布到网络上。
* [JWPL](https://github.com/dkpro/dkpro-jwpl)：JWPL是一个基于Java的免费应用程序编程接口，允许访问维基百科中的所有信息，由达姆施塔特工业大学开源。

## WebService

* [Apache CXF](https://github.com/apache/cxf)：CXF提供了用于方便地构建和开发Web Service的可靠基础架构。
* [Spring WS](https://github.com/spring-projects/spring-ws)：Spring Web Services是Spring社区的一个产品，专注于创建文档驱动的Web服务。
* [Apache Axis2](https://axis.apache.org/axis2/java/core/index.html)：Axis2是一个Web Services JSON/SOAP/WSDL引擎，是广泛使用的Axis SOAP堆栈的继承者。
* [Apache Axiom](https://ws.apache.org/axiom/)：Axiom库提供了符合XML Infoset的对象模型实现，支持按需构建对象树。
* [Apache Woden](https://ws.apache.org/woden/)：Woden库是作为Apache Web Services项目的子项目开发的，提供用于读取、操作、创建和编写WSDL文档的Java API。
* [Libre-wsdl4j](https://github.com/librewsdl4j/libre-wsdl4j)：Libre-wsdl4j是WSDL4J 1.6.3的一个分支，WSDL4j是WSDL的Java存根生成器。
* [WSDL2REST](https://github.com/jboss-fuse/wsdl2rest)：一种允许从现有JAX-WS服务快速迁移到REST服务的工具。
* [SOA Model](https://github.com/membrane/soa-model)：SOA Model是一个开源工具包和用于WSDL、XML Schema的Java API。
* [SOAP WS](https://github.com/reficio/soap-ws)：SOAP WS是一个轻量级且易于使用的Java库，它包装了Spring-WS并支持在纯XML级别处理SOAP。
* [jPasskit](https://github.com/drallgood/jpasskit)：jPasskit是Apple PassKit Web Service的Java实现。
* [OpenMRS REST Web Services](https://github.com/openmrs/openmrs-module-webservices.rest)：为OpenMRS提供RESTful Web服务。
* [Apache WSS4J](https://github.com/apache/ws-wss4j)：WSS4J项目提供了Web Services主要安全标准的Java实现，即OASIS Web Services Security(WS-Security)规范。

## 银行账号操作

* [BankCardUtils](https://github.com/nanchen2251/BankCardUtils)：根据银行卡号获取银行卡类型、银行名称和银行编码，自动格式化银行卡号、手机号、身份证号输入的工具类。
* [Java-IBAN](https://github.com/barend/java-iban)：一个用于处理国际银行帐号(IBAN)的小型Java库。
* [IBAN4j](https://github.com/arturmkrtchyan/iban4j)：用于生成和验证国际银行帐号(IBAN ISO_13616)和企业标识符代码(BIC ISO_9362)的Java库。
* [Bank4j](https://github.com/inisos/bank4j)：生成ISO 20022 XML传输并提供IBAN和BIC验证。

## 用户代理解析

* [Uap-Java](https://github.com/ua-parser/uap-java)：这是ua-parser的Java实现。
* [User Agent Utils](https://github.com/HaraldWalker/user-agent-utils)：用于处理用户代理字符串的实用程序，可用于实时处理HTTP请求或分析日志文件。
* [HTTPRequest](https://github.com/Konloch/HTTPRequest)：HTTPRequest是一个易于使用的零依赖Java包装器，用于从URL读取Cookie、代理、UserAgent、发布数据等。
* [BrowsCap Java](https://github.com/blueconic/browscap-java)：一个基于BrowsCap CSV源文件的速度极快且内存高效的Java客户端。
* [Yauaa](https://github.com/nielsbasjes/yauaa)：这是一个Java库，可以解析和分析useragent字符串(
  以及可用的User-Agent客户端提示)提取尽可能多的相关属性。
* [Salvation](https://github.com/shapesecurity/salvation)：这是一个用于处理内容安全策略策略的通用库。

## 语义发布工具

* [JSemVer](https://github.com/zafarkhaja/jsemver)：Java SemVer是语义版本控制规范的Java实现。
* [Semver4j](https://github.com/vdurmont/semver4j)：一个处理版本的轻量级Java库，它遵循语义版本控制规范的规则，提供多种版本控制模式。
* [Semver4j](https://github.com/semver4j/semver4j)：Semver4j是一个轻量级Java库，可帮助你处理版本，它遵循语义版本控制规范的规则。
* [JGitVer](https://github.com/jgitver/jgitver)：基于JGit的库，用于从Git树计算SemVer兼容版本。
* [Version Compare](https://github.com/G00fY2/version-compare)：适用于Android、Java和Kotlin的轻量级库，用于比较版本字符串。
* [Semantic Version](https://github.com/skuzzle/semantic-version)：Java的快速单类语义版本实现。
* [Nyx](https://github.com/mooltiverse/nyx)：Nyx是一个强大、灵活且可配置性极高的语义发布工具。
* [Semantic Versioning](https://github.com/jeluard/semantic-versioning)：Semantic Versioning是一个Java库，允许验证(使用字节码检查)库版本号是否遵循语义版本控制定义的语义版本控制原则。
* [Reckon](https://github.com/ajoberstar/reckon)：用于从Git仓库推断下一个版本的API。

## 数字信号处理

* [JDSP](https://github.com/psambit9791/jdsp)：JDSP是一个信号处理工具库，旨在提供MATLAB或Python的scipy-signal包中可用的功能。
* [IIRJ](https://github.com/berndporr/iirj)：用Java编写的高效IIR滤波器库。
* [Mines JTK](https://github.com/MinesJTK/jtk)：Mines Java Toolkit是一组用于科学和工程的Java包和原生软件库，目前的应用包括数字信号处理、线性代数、优化、网格划分、插值以及2D和3D图形，由科罗拉多矿业学院开发。
* [JRadio](https://github.com/dernasherbrezon/jradio)：用Java编写的软件无线电解码，这个项目的想法是从gnuradio获取块并用Java实现它们。

## 企业集成模式

* [Mule](https://github.com/mulesoft/mule)：Mule是一个轻量级集成平台，可让你在任何地方连接任何东西。
* [Apache Camel](https://github.com/apache/camel)：Camel是一个开源集成框架，使你能够快速轻松地集成使用或生成数据的各种系统，由RedHat开源。
* [Spring Integration](https://github.com/spring-projects/spring-integration)：Spring Integration提供了Spring编程模型的扩展，以支持众所周知的企业集成模式。
* [WSO2 Enterprise Integrator](https://github.com/wso2/product-ei)：WSO2 Enterprise Integrator是一个开源、快速、云原生且可扩展的集成解决方案，是WSO2集成敏捷平台的核心。
* [Syndesis](https://github.com/syndesisio/syndesis)：Syndesis是一个灵活且可定制的开源平台，以服务形式提供核心集成功能，由Fuse Online开源。
* [Metl](https://github.com/JumpMind/metl)：Metl是一个简单、基于Web的集成平台，允许多种不同类型的数据集成，包括消息传递、基于文件的ETL以及通过Web Service的远程过程调用。
* [Frank!Framework](https://github.com/frankframework/frankframework)：Frank!
  Framework是一个易于使用的无状态集成框架，允许在不同系统之间修改和交换(事务)消息，由WeAreFrank开源。
* [RACE](https://github.com/aegisql/conveyor)：RACE是一个可扩展的异步企业集成和创建型Java框架。
* [Ikasan](https://github.com/ikasanEIP/ikasan)：开源企业集成平台。

## 数字资产管理

* [LOCKSS](https://github.com/lockss/lockss-daemon)：LOCKSS是LOCKSS计划开发和维护的成熟、垂直集成、开源、分布式数字保存系统，由斯坦福赞助。
* [DSpace](https://github.com/DSpace/DSpace)：DSpace是一个专门的数字资产管理系统，它管理和发布由数字文件或“位流”组成的数字条目，并且允许创建、索引和搜索相关的元数据以便定位和存取该条目，由麻省理工学院联合美国惠普公司实验室开源。
* [Cudami](https://github.com/dbmdz/cudami)：Cudami是一个编辑后台，用于管理网站、文章、数字化对象、数字原生对象和实体等文化数字资产，由巴伐利亚国立图书馆开源。
* [OCFL Java](https://github.com/OCFL/ocfl-java)：该项目是牛津大学OCFL规范的Java实现，OCFL规范描述了一种独立于应用程序的方法，以结构化、透明且可预测的方式存储数字信息。
* [DuraCloud](https://github.com/duracloud/duracloud)：DuraCloud是一种开源托管数字保存服务，它将灵活的存储选项与强大的工具相结合，以简化你的保存工作流程。
* [Goobi](https://github.com/intranda/goobi-workflow)：Goobi是一款用于数字化项目的开源软件应用程序，它允许你对可自由定义的生产流程进行建模、管理和监督，并且许多机构每天都使用它来处理创建数字图书馆或博物馆所涉及的所有步骤。
* [Kitodo](https://www.kitodo.org/)：Kitodo是一款开源软件套件，用于对大大小小的图书馆、档案馆、博物馆和文献中心的文化资产进行数字化。
* [MyCoRe](https://github.com/MyCoRe-Org/mycore)：MyCoRe是一个开源仓库软件框架，用于构建学科或机构存储库、数字档案、数字图书馆和科学期刊。
* [MARC4J](https://github.com/marc4j/marc4j)：MARC4J的目标是提供一个易于使用的API，以便在Java中使用MARC和MARCXML。
* [Kitodo.Production](https://github.com/kitodo/kitodo-production)：Kitodo.Production是一种用于大规模数字化的工作流程管理工具，是Kitodo数字图书馆套件的一部分。
* [Fedora](https://github.com/fcrepo/fcrepo)：Fedora是一个强大、模块化、开源存储库系统，用于管理和传播数字内容，它特别适合数字图书馆和档案馆的访问和保存。
* [DROID](https://github.com/digital-preservation/droid)：DROID是英国国家档案馆开发的一款软件工具，用于自动批量识别文件格式。

## 文档管理系统

* [Logseq](https://github.com/logseq/logseq)：Logseq是一个知识管理和协作平台。
* [JVS-Knowledge](https://gitee.com/software-minister/jvs-knowledge-ui)：企业级在线文档，解决企业内部文档编辑、知识沉淀、知识协同等痛点。
* [Zyplayer DOC](https://gitee.com/dromara/zyplayer-doc)：Zyplayer
  DOC是一款适合团队和个人使用的WIKI文档管理工具，同时还包含数据库文档、API接口文档，由dromara社区开源。
* [APIManager](https://gitee.com/zhoujingjie/apiManager)：小幺鸡文档管理工具，支持富文本、Markdown、HTTP、WebSocket及其在线测试。
* [WCP](https://gitee.com/macplus/WCP)：WCP是一套BS架构的开源知识管理系统、知识库系统，由太原扁舟科技开源。
* [Knowledge](https://github.com/support-project/knowledge)：Knowledge是一个开源知识库平台。
* [MxsDoc](https://gitee.com/RainyGao/DocSys)：MxsDoc是基于Web的文件管理系统，主要用于企业或个人的文件存储管理，方便随时查看和统一管理。
* [Apache JSPWiki](https://github.com/apache/jspwiki)：JSPWiki是一个简单的WikiWiki克隆，用Java和JSP编写。
* [Apache Jackrabbit Oak](https://github.com/apache/jackrabbit-oak)：Jackrabbit Oak是一个可扩展、高性能的分层内容仓库，旨在用作现代世界级网站和其他要求高的内容应用程序的基础。
* [科亿知识库KYKMS](https://gitee.com/kyxxjs/km_community)：基于Elasticsearch的文件管理系统/知识管理系统，由广州科亿信息开源。
* [DocPreview](https://gitee.com/hcwdc/docpreview)：文件在线预览模块，支持多格式转PDF文件，由华创数字云开源。
* [WDA](https://gitee.com/macplus/WDA)：配合OpenOffice实现文档的在线预览、本地文档添加、文档转换为HTML，文档HTML方式预览，由太原扁舟科技开源。
* [WPS View](https://gitee.com/mose-x/wps-view-java)：基于WPS在线编辑、在线预览后台服务。
* [Hackpad](https://github.com/dropbox/hackpad)：Hackpad是一个基于Web的实时Wiki，基于开源EtherPad协作文档编辑器。
* [InfoSphere](https://gitee.com/devlive-community/incubator-infosphere)：InfoSphere是一款面向企业和个人的开源Wiki系统，旨在提供简单而强大的知识管理解决方案，由Devlive社区开源。

## 数据匿名工具

* [ARX](https://github.com/arx-deidentifier/arx)：ARX是一款用于对敏感个人数据进行匿名化的综合开源软件，由德国慕尼黑工业大学开发。
* [DataDefender](https://github.com/armenak/DataDefender)：敏感数据管理：数据发现和匿名化工具包。
* [Anonymouth](https://github.com/psal/anonymouth)：Anonymouth是一个基于Java的应用程序，旨在为用户提供开始对其编写的文档进行匿名化所需的工具和知识，由德雷塞尔大学开源。
* [Anonymouse](https://github.com/CaravanaCloud/Anonymouse)：数据库匿名化工具。
* [Anonimatron](https://github.com/realrolfje/anonimatron)：Anonimatron是一个免费、可扩展、开源数据匿名化工具。
* [Rapiddweller Benerator](https://github.com/rapiddweller/rapiddweller-benerator-ce)：Rapiddweller Benerator是一个功能强大的软件解决方案，用于开发、测试和培训目的的数据生成、混淆和迁移。
* [Cinnamon](https://github.com/KI-AIM/Cinnamon)：Cinnamon是一个模块化应用程序，旨在为数据匿名化、合成和评估提供强大的功能，由德国联邦教育与研究部开发。
* [Data Privacy Toolkit](https://github.com/IBM/data-privacy-toolkit)：Data Privacy Toolkit是一个用于数据类型识别、隐私风险评估、数据屏蔽和数据匿名化的工具包，以Java/Scala库和REST API的形式公开，由IBM开源。

## 外部进程执行

* [NuProcess](https://github.com/brettwooldridge/NuProcess)：Java的低开销、非阻塞I/O、外部进程执行实现，它是java.lang.ProcessBuilder和java.lang.Process的替代品。
* [Apache Commons Exec](https://github.com/apache/commons-exec)：Commons Exec是一个从JVM内可靠地执行外部进程的库。
* [Ch.Vorburger.Exec](https://github.com/vorburger/ch.vorburger.exec)：这是一个小型库，允许在后台从Java代码启动外部进程。
* [JProc](https://github.com/fleipold/jproc)：用于运行外部进程的Java库。
* [Overthere](https://github.com/xebialabs/overthere)：Overthere是一个Java库，用于在远程主机上操作文件和执行进程。
* [ZT-EXEC](https://github.com/zeroturnaround/zt-exec)：Java进程执行库。
* [ZT-Process-Killer](https://github.com/zeroturnaround/zt-process-killer)：停止从Java启动的进程或通过PID的系统进程。
* [Winrm4j](https://github.com/cloudsoft/winrm4j)：Winrm4j是一个使Java应用程序能够使用WinRM在远程Windows服务器上执行批处理或PowerShell命令的项目。
* [JNR Process](https://github.com/jnr/jnr-process)：JNR Process库提供了JDK ProcessBuilder API的直接替代品，但它不是线程泵填充程序，而是围绕posix_spawn C API的直接抽象，并提供可选择的in、out和err通道。
* [WinP](https://github.com/jenkinsci/winp)：该项目可以让你更好地控制Windows进程，超越JDK中的可用功能。
* [Jash](https://github.com/jbangdev/jbang-jash)：Jash是一个Java库，提供流式、可预测且具有出色开发人员体验的Process接口。

## 苹果推送通知

* [Pushy](https://github.com/jchambers/pushy)：Pushy是一个用于发送APN(iOS、macOS和Safari)推送通知的Java库。
* [Java-Apns](https://github.com/notnoop/java-apns)：Java-Apns是Apple推送通知服务(APN)
  的Java客户端，该库旨在为Apple服务器提供高度可扩展的接口，同时保持简单和模块化。
* [DBay-APNS-Java](https://github.com/RamosLi/dbay-apns-for-java)：APNS的高性能Java客户端。
* [APNS-HTTP2](https://github.com/CleverTap/apns-http2)：用于使用Apple的新HTTP/2 API通过APNS发送通知的Java库。
* [Apns4j](https://github.com/teaey/apns4j)：Apple推送通知服务Java实现，阿里开源。
* [JavaAPNS-JDK16](https://github.com/fernandospr/javapns-jdk16)：适用于Java的Apple推送通知服务提供程序。

## 自动程序修复

* [Astor](https://github.com/SpoonLabs/astor)：Astor是Java的自动软件修复框架，由法国国立计算机及自动化研究院、里尔大学、法兰西理工大学和皇家理工学院共同开发。
* [SimFix](https://github.com/xgdsmileboy/SimFix)：SimFix是一种自动程序修复技术，它利用其他项目中现有的补丁和同一项目中的类似代码片段来生成补丁，由天津大学开源。
* [Defects4J](https://github.com/rjust/defects4j)：Defects4J是可重现错误和支持基础设施的集合，旨在推动软件工程研究。
* [Angelix](https://github.com/msv-lab/angelix)：Angelix是基于语义的C程序自动程序修复工具，由新加坡国立大学开源。
* [Sorald](https://github.com/ASSERT-KTH/sorald)：Sorald是一款自动修复使用SonarQube检查的静态分析规则违规的工具，由瑞典斯德哥尔摩皇家理工学院开源。
* [KGenProg](https://github.com/kusumotolab/kGenProg)：KGenProg是一个用Java编写的用于Java的自动程序修复工具，由大阪大学开源。

## Java服务包装器

* [YAJSW](https://github.com/yajsw/yajsw)：YAJSW是tanuki的Java服务包装器(JSW)的以Java为中心的实现。
* [Java Service Wrapper](https://wrapper.tanukisoftware.org/doc/english/home.html)：Java Service Wrapper是一种经过验证的企业级解决方案，已被数千家公司和开发人员使用，它极大地简化了Java应用程序在各种平台上的部署、启动和监控。
* [Apache Commons Daemon](https://github.com/apache/commons-daemon)：Commons Daemon是一组实用程序和Java支持类，用于将Java应用程序作为服务器进程运行。
* [Launch4j](https://launch4j.sourceforge.net/)：Launch4j是一个跨平台工具，用于将作为jar分发的Java应用程序包装在轻量级Windows本机可执行文件中。
* [JSmooth](https://github.com/BrunoReX/jsmooth)：JSmooth是一个Java可执行包装器，可构建启动Java应用程序的标准Windows可执行二进制文件(.exe)。
* [JSystemd](https://github.com/jpmsilva/jsystemd)： JSystemd旨在提供一个更好的平台来将Java应用程序与systemd集成，并将它们作为适当的操作系统服务运行。

## 守护进程

* [Termd](https://github.com/termd/termd)：一个开源终端守护程序库，提供Java终端处理。
* [Akuma](https://github.com/kohsuke/akuma)：这是一个Java库，你可以在应用程序中使用它来支持Unix守护进程。
* [SDNotify](https://github.com/faljse/SDNotify)：SDNotify在Java中实现了systemd通知协议。

## 协议实现

* [JSIP](https://github.com/usnistgov/jsip)：Java SIP规范参考实现，由美国国家标准技术研究院开源。
* [Open eCard](https://github.com/ecsec/open-ecard)：Open eCard旨在提供eCard-API-Framework(BSI TR-03112)和相关国际标准ISO/IEC 24727的开源和跨平台实现。
* [HAP Java](https://github.com/hap-java/HAP-Java)：HAP-Java是HomeKit附件协议的Java实现。
* [SECS4Java8](https://github.com/kenta-shimizu/secs4java8)：该库是Java 8上的SEMI-SECS通信实现。
* [DBus Java](https://github.com/hypfvieh/dbus-java)：该库是D-Bus协议的原生Java实现。
* [Calimero Core](https://github.com/calimero-project/calimero-core)：Calimero Core提供(安全)KNX通信协议、KNX数据点和属性访问以及管理功能。
* [Bacnet4J Wrapper](https://github.com/Code-House/bacnet4j-wrapper)：Bacnet4j是bacnet协议的Java实现，这是Bacnet4j API的简单门面。
* [RISE V2G](https://github.com/SwitchEV/RISE-V2G)：车辆到电网(V2G)通信接口ISO 15118的开源参考实现。

## BitTorrent

* [PeerBanHelper](https://github.com/PBH-BTN/PeerBanHelper)：PeerBanHelper是一个开源的个人网络防火墙安全软件。
* [Bt](https://github.com/atomashpolskiy/bt)：BitTorrent库和客户端，具有DHT、磁力链接、加密等功能。
* [BitLet](https://github.com/bitletorg/bitlet)：BitLet是BitTorrent协议的简单Java实现。
* [Ttorrent](https://github.com/mpetazzoni/ttorrent)：BitTorrent协议的Java实现。
* [FrostWire JLibTorrent](https://github.com/frostwire/frostwire-jlibtorrent)：FrostWire为libtorrent提供的swig Java接口。
* [Libtorrent4j](https://github.com/aldenml/libtorrent4j)：Libtorrent的swig Java接口。
* [FrostWire](https://github.com/frostwire/frostwire)：FrostWir是BitTorrent网络的媒体播放器和点对点(P2P)信息共享客户端。

## 编解码

* [Apache Commons Codec](https://github.com/apache/commons-codec)：Commons Codec包含各种格式(例如Base64和十六进制)的简单编码器和解码器。
* [OWASP Java Encoder](https://github.com/OWASP/owasp-java-encoder)：OWASP Java Encoder是一个简单易用的嵌入式高性能编码器类，没有依赖且包袱很少，由OWASP开源。
* [Simple Binary Encoding](https://github.com/real-logic/simple-binary-encoding)：SBE是OSI第6层表示，用于对低延迟金融应用程序的二进制应用程序消息进行编码和解码。
* [Juniversalchardet](https://github.com/albfernandez/juniversalchardet)：Juniversalchardet是universalchardet的Java端口，universalchardet是Mozilla的编码检测器库。
* [PETSCII BBS Builder](https://github.com/sblendorio/petscii-bbs)：一个Java框架，用于构建高度可定制的PETSCII(和ASCII)支持的BBS，可从8位Commodore计算机访问。
* [FastProto](https://github.com/indunet/fastproto)：FastProto是一个功能强大的二进制数据处理工具，旨在简化Java环境中的二进制数据编码和解码。

## 打印机

* [RepRap](https://sourceforge.net/projects/reprap/)：RepRap是一种三维打印机原型机，它具有一定程度的自我复制能力，能够打印出大部分其自身的塑料组件，由英国巴斯大学开发。
* [Zebra-zpl](https://github.com/w3blogfr/zebra-zpl)：用于生成通用ZPL命令以使用Java在Zebra打印机上打印标签的库。
* [Bambu Farm](https://github.com/TFyre/bambu-farm)：基于Web的应用程序，使用MQTT/FTP/RTSP监控多台Bambu打印机(无需自定义固件)。
* [ESC/POS Java](https://github.com/stefanosbou/esc-pos-java)：用于ESC/POS兼容热敏打印机的Java库，串行或网络连接。

## Web资源

* [YUI Compressor](https://github.com/yui/yuicompressor)：YUI Compressor是一个JavaScript压缩器，除了删除注释和空格之外，它还使用尽可能小的变量名称来混淆局部变量，该库由Yahoo开源。
* [GWT Bootstrap](https://github.com/gwtbootstrap/gwt-bootstrap)：提供了简单灵活的组件来表示Bootstrap组件、样式和插件，由Twitter开源。
* [CSSEmbed](https://github.com/nzakas/cssembed)：用于在CSS文件中嵌入数据URI的工具。
* [WebJars](https://github.com/webjars/webjars)：打包到JAR中的客户端Web库。
* [Caja](https://github.com/googlearchive/caja)：Caja是一个用于在你的网站中安全嵌入第三方HTML、CSS和JavaScript的工具，由Google开源。
* [Wro4j](https://github.com/wro4j/wro4j)：Wro4j是一个免费的开源Java项目，可以帮助缩短Web应用程序页面加载时间。
* [Closure Stylesheets](https://github.com/google/closure-stylesheets)：一个CSS+转译器，可进行Lints、优化和国际化，由Google开源。
* [CSS Validator](https://github.com/w3c/css-validator)：W3C CSS验证服务。
* [Ph-CSS](https://github.com/phax/ph-css)：Java CSS 2和CSS 3解析器和构建器。
* [Closure Templates](https://github.com/google/closure-templates)：客户端和服务器端模板系统，可帮助你动态构建可重用的HTML和UI元素，由Google开源。
* [Chart.java](https://github.com/mdewilde/chart)：Chart.java可以在Java应用程序中与优秀的Chart.js库集成。
* [LESS Engine](https://github.com/asual/lesscss-engine)：LESS引擎提供对核心LESS功能的基本访问。
* [LESS CSS Compiler](https://github.com/marceloverdijk/lesscss-java)：LESS CSS Compiler是一个将LESS源代码编译为CSS样式表的库。
* [JLessC](https://github.com/i-net-software/jlessc)：JLessC是一个完全用Java编写的Less CSS编译器。

## Web开发库

* [BaasBox](https://github.com/baasbox/baasbox)：BaasBox是一个开源项目，旨在为移动和Web应用程序提供后端。
* [Gargl](https://github.com/jodoglevy/gargl)：记录发生的Web请求，并将其转换为任何编程语言的可重用代码。
* [Elemento](https://github.com/hal/elemento)：Elemento可以简化GWT Elemental2的使用。
* [Elemental](https://github.com/google/elemental2)：Elemental2为Java代码提供对所有浏览器API的类型检查访问，由Google开源。
* [Displaytag](https://github.com/hazendaz/displaytag)：Displaytag库是一个开源的自定义标签套件，提供可在MVC模型中工作的高级Web表示模式。
* [Boilerplate](https://github.com/kohlschutter/boilerpipe)：Boilerplate库提供了算法来检测和删除网页主要文本内容周围多余的“混乱”(样板、模板)。
* [Stapler](https://github.com/jenkinsci/stapler)：Stapler是一个将应用程序对象“装订”到URL的库，使编写Web应用程序变得更加容易，由Jenkins组织开源。
* [Orbeon Forms](https://github.com/orbeon/orbeon-forms)：Orbeon Forms是一个开源Web表单解决方案，它包括XForms引擎、基于Web的表单编辑器Form Builder和Form Runner运行时。
* [Nu Html Checker](https://github.com/validator/validator)：Nu Html Checker可帮助你发现HTML、CSS和SVG中的意外错误。
* [Yoga](https://github.com/skyscreamer/yoga)：Yoga扩展了JAX-RS和Spring MVC RESTful服务器，以提供GData和LinkedIn风格的字段选择器。
* [AngularGWT](https://github.com/cromwellian/angulargwt)：这是一个能够用Java为AngularJS编写组件或完整的应用程序的库。
* [HAR Reader](https://github.com/sdstoehr/har-reader)：用于使用Java访问HTTP存档(HAR)的库。
* [WebMVC](https://github.com/beangle/webmvc)：Web模型-视图-控制器库。
* [AutoREST](https://github.com/intendia-oss/autorest)：GWT自动RESTful服务代理生成器。
* [Wicket Stuff](https://github.com/wicketstuff/core)：Wicket Stuff是由Wicket社区创建和维护的Apache Wicket Web框架的开源项目集合。
* [ORCID-Source](https://github.com/ORCID/ORCID-Source)：ORCID Source是一组使用Java构建的Web应用程序和库，使用Spring Web MVC和Postgres数据库提供的持久性。
* [JBossWS-CXF](https://github.com/jbossws/jbossws-cxf)：JBossWS-CXF堆栈，与Apache CXF集成。
* [Domino-UI](https://github.com/DominoKit/domino-ui)：类型安全且功能丰富的UI组件库，供Java开发人员使用流式API，并且不依赖于外部JavaScript。
* [J4TS](https://github.com/j4ts/j4ts)：J4TS基于GWT的JRE模拟库的一个分支，用Java编写，并使用JSweet转译器转译为TypeScript/JavaScript。
* [Mateu](https://github.com/miguelperezcolom/mateu)：Mateu是一个用于以光速从Java创建出色的响应式Web应用程序的框架。
* [Metawidget](https://github.com/metawidget/metawidget)：Metawidget是一个智能小部件，它可以静态或运行时填充自身，并使用UI组件来匹配域对象的属性。
* [PatternFly Java](https://github.com/patternfly-java/patternfly-java)：PatternFly Java是基于GWT/J2CL和Elemento的PatternFly 5的Java实现，由RedHat开源。
* [AngularBeans](https://github.com/bessemHmidi/AngularBeans)：AngularBeans是一个框架，其目的是将Java EE 7(更准确地说是CDI规范)与AngularJS结合使用。
* [React4j](https://github.com/react4j/react4j)：该项目的目标是能够从GWT无缝使用React的组件模型，并利用React开发支持工具(例如React的Devtools)生态系统。
* [DnComponents](https://dncomponents.com/index.html)：客户端Java UI框架，用于使用GWT编译器和Elemental2浏览器API纯粹使用Java语言构建丰富的Web应用程序，无需任何外部JS库。
* [GWT Material](https://github.com/GwtMaterialDesign/gwt-material)：GWT的Google Material Design包装器。
* [Vue GWT](https://github.com/VueGWT/vue-gwt)：Vue GWT使用JsInterop和Elemental2将Vue.js与GWT 2.9集成，它允许你用Java编写Vue组件。
* [GwtBootstrap3](https://github.com/gwtbootstrap3/gwtbootstrap3)：GWTBootstrap3是Twitter
  Bootstrap的包装器，可帮助你使用Java和GWT在Web上开发响应式、移动优先的HTML、CSS和JS项目。
* [SwellRT](https://github.com/SwellRT/swellrt)：SwellRT是一个开源后端即服务，它提供预构建的功能来加速协作Web应用程序的开发。
* [N2O Framework](https://github.com/i-novus-llc/n2o-framework)：N2O Framework是一个用Java和ReactJS编写的库，允许你创建具有复杂用户界面的Web应用程序，而无需深入了解Web技术和前端框架。
* [ApexCharts Flow](https://github.com/appreciated/apexcharts-flow)：Vaadin平台的ApexCharts.js包装器。
* [JWt](https://github.com/emweb/jwt)：JWt是一个用于开发Web应用程序的Java库，它提供了一种纯Java组件驱动的方法来构建Web应用程序，并使用Ajax或纯HTML进行呈现。
* [Charba](https://github.com/pepstock-org/Charba)：基于Chart.js的J2CL和GWT图表库。
* [Nalu](https://github.com/NaluKit/nalu)：Nalu是一个微型框架，可帮助你轻松创建基于GWT的应用程序。
* [LightAdmin](https://github.com/la-team/light-admin)：该项目的主要目标是通过为基于JPA的应用程序引入可插入的完全可操作的数据管理后端来加速应用程序开发。
* [Fastball](https://github.com/fastball-projects/fastball)：Fastball是一套面向后端、声明式界面开发框架。

## Web过滤器

* [CORS Filter](https://github.com/eBay/cors-filter)：CORS Filter是用于Java Web容器的服务器端CORS的Java Servlet Filter实现，由eBay开源。
* [Lucy XSS Filter](https://github.com/naver/lucy-xss-servlet-filter)：该库是一个基于Java Servlet过滤器的库，为了解决XSS攻击问题而开发，由Naver开源。

## Cron解析

* [Cron Utils](https://github.com/jmrozanec/cron-utils)：CronUtils是一个Java库，用于定义、解析、验证、迁移cron以及获取人类可读的描述。
* [Cron Parser](https://github.com/grahamar/cron-parser)：将cron表达式转换为人类可读字符串的Java库。

## 手机号解析

* [LibPhoneNumber](https://github.com/google/libphonenumber)：Google的通用Java、C++和JavaScript库，用于解析、格式化和验证国际电话号码。
* [PhoneNumber-Geo](https://github.com/fengjiajie/phone-number-geo)：手机号码归属地本地解析Java实现。
* [PhoneNumber-Geo](https://github.com/EeeMt/phone-number-geo)：根据手机号确定手机号运营商即归属地，支持包括虚拟运营商的中国大陆手机号查询。
* [PhoneNumber](https://github.com/xdtianyu/PhoneNumber)：一个获取号码归属地和其他信息(诈骗、骚扰等)的开源库。
* [PhoneNumber Normalizer](https://github.com/telekom/phonenumber-normalizer)：使用PhoneNumber Normalizer库，你可以将电话号码标准化为E164格式和国家格式，同时考虑到德国号码计划的特定复杂性，由德国电信开源。

## 表达式引擎

* [Aviator](https://code.google.com/archive/p/aviator/)：Aviator是一个高性能、轻量级的基于Java实现的表达式引擎，它动态地将String类型的表达式编译成Java字节码并交给JVM执行，Google开源。
* [Apache Commons JEXL](https://github.com/apache/commons-jexl)：Commons JEXL库是Java共生表达式语言的实现。
* [AviatorScript](https://github.com/killme2008/aviatorscript)：AviatorScript是一门高性能、轻量级寄宿于JVM(包括Android平台)之上的脚本语言。
* [JSEL](https://code.google.com/archive/p/lite/wikis/JSEL.wiki)：JSEL是一个兼容JavaScript运算规则的简单表达式解释引擎。
* [IK Expression](https://code.google.com/archive/p/ik-expression/)：IK Expression是一个开源、可扩展、基于Java语言开发的一个超轻量级的公式化语言解析执行工具包。
* [FastEL](https://github.com/dbcxy/fast-el)：轻量级的高效表达式计算引擎。
* [QLExpress](https://github.com/alibaba/QLExpress)：QLExpress是一种强大的、轻量级的、动态的Java平台语言，旨在提高开发人员在不同业务场景中的生产力，阿里开源。
* [MVEL](https://github.com/mvel/mvel)：MVEL是一种混合动态/静态类型、可嵌入的表达式语言和Java平台运行时。
* [Recognizers Text](https://github.com/microsoft/Recognizers-Text)：Microsoft.Recognizers.Text提供对数字、单位和日期/时间等实体的强大识别和解析。
* [OGNL](https://github.com/orphan-oss/ognl)：OGNL代表对象图导航语言，它是一种用于获取和设置Java对象属性的表达式语言。
* [Formula](https://github.com/salesforce/formula-engine)：可重用公式引擎的实现，具有JavaScript和SQL生成以及Salesforce批准的语法。
* [SpEL](https://github.com/spring-projects/spring-framework)：SpEL是一种功能强大的表达式语言，支持在运行时查询和操作对象图。
* [Janino](https://github.com/janino-compiler/janino)：Janino是一个超小、超快的Java编译器。
* [JUEL](https://github.com/beckchr/juel)：JUEL是统一表达语言(EL)的实现，指定为JSP 2.1标准(JSR-245)的一部分，已在JEE5中引入。
* [FEEL](https://github.com/camunda/feel-scala)：由Camunda用Scala编写的FEEL引擎。
* [Liquor](https://gitee.com/noear/liquor)：Java动态编译、表达式、脚本工具。
* [Cucumber Expressions](https://github.com/cucumber/cucumber-expressions)：Cucumber Expressions是正则表达式的替代品，具有更直观的语法。
* [CEL Java](https://github.com/google/cel-java)：CEL是一种非图灵完备语言，旨在简单、快速、安全和可移植，由Google开发。
* [Grammaticus](https://github.com/salesforce/grammaticus)：Grammaticus是一个语法引擎，允许用户重命名名词，同时保持内容的语法正确，由Salesforce开源。
* [CEL-Java](https://github.com/projectnessie/cel-java)：通用表达式语言的Java实现。
* [Eclipse Expressly](https://github.com/eclipse-ee4j/expressly)：Expressly实现了Jakarta Expression Language，这是一种用于Java应用程序的表达式语言。
* [JFireEL](https://gitee.com/eric_ds/jfireEL)：快速的EL表达式解析器，支持丰富的EL表达式。
* [Crunch](https://github.com/boxbeam/Crunch)：快速的Java表达式编译器/评估器。
* [SimpleEL](https://github.com/alibaba/simpleel)：SimpleEL是阿里开发的简单表达式语言。

## 数学表达式

* [Exp4j](https://github.com/fasseg/exp4j)：Exp4j是用于Java编程语言的小型数学表达式计算器。
* [Keval](https://github.com/notKamui/Keval)：用于数学表达式字符串求值的Kotlin迷你库。
* [ExprK](https://github.com/Keelar/ExprK)：ExprK是一个用于Kotlin和Java的简单数学表达式评估器。
* [EvalEx](https://github.com/ezylang/EvalEx)：EvalEx是一个方便的Java表达式计算器，它允许评估简单的数学和布尔表达式。
* [Javaluator](https://github.com/fathzer/javaluator)：Javaluator是一个简单、功能强大的Java中缀表达式计算器。
* [Parsington](https://github.com/scijava/parsington)：Parsington是一个中缀到后缀和中缀到语法树表达式解析器，用于用Java编写的数学表达式。
* [MathParser](https://github.com/mariuszgromada/MathParser.org-mXparser)：MathParser是一个超级简单、丰富且高度灵活的数学表达式解析器库，适用于Java、Android、.NET、TypeScript和JavaScript。
* [Parsii](https://github.com/scireum/parsii)：用Java编写的数学表达式的超级快速且简单的计算器。
* [ParserNG](https://github.com/gbenroscience/ParserNG)：ParserNG是一个功能强大的开源数学工具，可以解析和评估代数表达式。
* [Paralithic](https://github.com/PolyhedralDev/Paralithic)：Paralithic是一个用于解析和评估数学表达式的库。
* [Expression Parser](https://slovesnov.users.sourceforge.net/index.php?parser)：Expression Parser是sin(pi/4)等字符串的解析器，它使用C++、JavaScript、Java、PHP语言实现。
* [Mathematical Expression](https://github.com/BeardedManZhao/mathematical-expression)
  ：本框架是一种针对数学公式解析的有效工具，能够解析包含嵌套函数，包含函数，数列步长累加等数学公式。

## SQL解析器

* [SQLSolver](https://github.com/SJTU-IPADS/SQLSolver)：SQLSolver是一个查询等效性验证器，由上海交通大学、耶鲁大学、纽约大学和普林斯顿大学的研究人员开发。
* [JSqlParser](https://github.com/JSQLParser/JSqlParser)：JSqlParser是一个与RDBMS无关的SQL语句解析器，它将SQL语句转换为可遍历的Java类层次结构。
* [SQL Parser](https://www.sqlparser.com/sql-parser-java.php)：SQL Parser提供了对各种数据库的SQL脚本的深入和详细分析，这是一个付费产品。
* [ElasticSearch SQL](https://github.com/iamazy/elasticsearch-sql)：使用Antlr4将SQL解析为ElasticSearch DSL。

## 解析器组合器

* [Autumn](https://github.com/norswap/autumn)：Autumn是一个Java解析器组合器库。
* [Java PetitParser](https://github.com/petitparser/java-petitparser)：PetitParser结合了无扫描器解析、解析器组合器、解析表达式语法(PEG)和Packrat解析器的思想，将语法和解析器建模为可以动态重新配置的对象。

## 源代码解析

* [Roaster](https://github.com/forge/roaster)：Roaster是一个可以轻松解析和格式化Java源文件的库。
* [JRecordBind](https://github.com/ffissore/jrecordbind)：JRecordBind是小型且超快的定长文件读取器/解析器。

## 对象图导航

* [Apache Commons OGNL](https://github.com/apache/commons-ognl)：OGNL代表对象图导航语言；它是一种表达式语言，用于获取和设置Java对象的属性，以及其他附加功能，例如列表投影和选择以及Lambda表达式。
* [OGNL](https://github.com/orphan-oss/ognl)：对象图导航库。
* [OGNL Expression](https://mvnrepository.com/artifact/marmalade/marmalade-el-ognl)：OGNL表达式库。

## 超媒体类型

* [Spring HATEOAS](https://github.com/spring-projects/spring-hateoas)：该项目提供了一些API来简化创建在使用Spring时遵循HATEOAS原则的REST表示。
* [Hate](https://github.com/blackdoor/hate)：根据HAL规范构建超媒体友好的对象。
* [Siren4J](https://github.com/eserating-chwy/siren4j)：Siren规范库。
* [Spring HATEOAS JSON API](https://github.com/toedter/spring-hateoas-jsonapi)：这是与Spring
  HATEOAS集成的媒体类型application/vnd.api+json(JSON:API)的实现。
* [Edison HAL](https://github.com/otto-de/edison-hal)：使用Jackson生成和使用REST资源的application/hal+json表示的库。
* [REST-Shell](https://github.com/spring-attic/rest-shell)：用于与Spring HATEOAS兼容的REST资源交互的命令行shell。

## 术语服务器

* [Snow Owl](https://github.com/b2ihealthcare/snow-owl)：Snow Owl是一款高度可扩展的开源术语服务器，具有修订控制功能和协作创作平台功能。
* [Snowstorm](https://github.com/IHTSDO/snowstorm)：Snowstorm是一个开源术语服务器，特别支持SNOMED CT。

## Minecraft

* [Paper](https://github.com/PaperMC/Paper)：使用最广泛的高性能Minecraft服务器，旨在修复游戏玩法和机制的不一致问题。
* [MinecraftForge](https://github.com/MinecraftForge/MinecraftForge)：一个免费的开源模组API，所有常见模组都在使用。
* [HMCL](https://github.com/huanghongxun/HMCL)：一款多功能、跨平台、流行的Minecraft桌面。
* [PojavLauncher](https://github.com/PojavLauncherTeam/PojavLauncher)：基于Boardwalk的Minecraft：适用于Android和iOS的Java版启动器。
* [Sodium](https://github.com/CaffeineMC/sodium-fabric)：旨在提高帧速率并减少微卡顿的Fabric模组。
* [Geyser](https://github.com/GeyserMC/Geyser)：允许使用Minecraft基岩版连接到Minecraft Java版服务器的桥接器。
* [Brigadier](https://github.com/Mojang/brigadier)：一个命令解析器和调度器，专为Minecraft Java版设计和开发。
* [WorldEdit](https://github.com/EngineHub/WorldEdit)：Minecraft地图编辑器和模组。
* [Iris](https://github.com/IrisShaders/Iris)：Minecraft的现代着色器模组，旨在与现有的OptiFine着色器包兼容。
* [MCA Selector](https://github.com/Querz/mcaselector)：一种从Minecraft世界中选择块进行删除或导出的工具。
* [Bukkit](https://github.com/Bukkit/Bukkit)：Minecraft服务器API。
* [Create](https://github.com/Creators-of-Create/Create)：一个为建筑、装饰和美学自动化提供各种工具和模块的模组。
* [Amidst](https://github.com/toolbox4minecraft/amidst)：用于显示Minecraft世界概览的工具，而无需实际创建它。
* [Minestom](https://github.com/Minestom/Minestom)：1.19.3轻量级Minecraft服务器。
* [Dynmap](https://github.com/webbukkit/dynmap)：一组Minecraft模组，为各种Minecraft服务器实现提供基于Web的实时地图系统。
* [Fabric](https://github.com/FabricMC/fabric)：Fabric mods的基本钩子和互操作机制的库。
* [Glowstone](https://github.com/GlowstoneMC/Glowstone)：一个快速、可定制且兼容的Minecraft Java版开源服务器。
* [CatServer](https://github.com/Luohuayu/CatServer)：高性能和高兼容性的1.12.2/1.16.5/1.18.2版本Forge + Bukkit + Spigot服务端。
* [LuckPerms](https://github.com/LuckPerms/LuckPerms)：Minecraft服务器的权限插件。
* [Lithium](https://github.com/CaffeineMC/lithium-fabric)：免费开源的Minecraft模组，可优化游戏的许多领域，以提供更好的整体性能。
* [Essentials](https://github.com/EssentialsX/Essentials)：用于Spigot和Paper的现代Essentials套件。
* [Meteor](https://github.com/MeteorDevelopment/meteor-client)：基础Minecraft实用程序模组。
* [BungeeCord](https://github.com/SpigotMC/BungeeCord)：一个复杂的代理和API，主要设计用于在多个Minecraft服务器之间传送玩家。
* [BlueMap](https://github.com/BlueMap-Minecraft/BlueMap)：一款Minecraft地图工具，可创建Minecraft世界的3D模型并将其显示在Web查看器中。
* [Velocity](https://github.com/PaperMC/Velocity)：下一代Minecraft服务器代理。
* [FarPlaneTwo](https://github.com/PorkStudios/FarPlaneTwo)：Minecraft中的细节层次渲染器，允许渲染数百万个块的距离。
* [Applied Energistics 2](https://github.com/AppliedEnergistics/Applied-Energistics-2)：一个关于物质、能量并利用它们征服世界的模组。
* [Botania](https://github.com/VazkiiMods/Botania)：以自然和植物生命的魔力为主题的Minecraft技术模组。
* [SpongeAPI](https://github.com/SpongePowered/SpongeAPI)：成熟的Minecraft插件API，不包括实现。
* [Nukkit](https://github.com/CloudburstMC/Nukkit)：Minecraft基岩版的核动力服务器软件。
* [SpongeForge](https://github.com/SpongePowered/SpongeForge)：一个实现SpongeAPI的Forge模组。
* [Adventure](https://github.com/KyoriPowered/adventure)：Minecraft Java版的服务器端用户界面库。
* [BuildCraft](https://github.com/BuildCraft/BuildCraft)：Minecraft模组。
* [MultiPaper](https://github.com/MultiPaper/MultiPaper)：多服务器、单一世界papermc实现。

## Maven插件

* [Frontend Maven Plugin](https://github.com/eirslett/frontend-maven-plugin)：该插件会在你的项目本地下载/安装Node和NPM，运行npm install，然后运行Bower、Grunt、Gulp、Jspm、Karma或Webpack的任意组合。
* [Git Commit Id Maven Plugin](https://github.com/git-commit-id/git-commit-id-maven-plugin)：可以将构建时Git仓库信息包含到POJO/properties文件中的Maven插件。
* [Android Maven Plugin](https://github.com/simpligility/android-maven-plugin)：用于Android应用程序开发等的Maven插件。
* [JavaFX Maven Plugin](https://github.com/javafx-maven-plugin/javafx-maven-plugin)：JavaFX
  Maven插件提供了一种从Maven内组装JavaFX应用程序(8+)分发包的方法。
* [TypeScript Generator](https://github.com/vojtechhabarta/typescript-generator)：TypeScript Generator是一个用于从Java JSON类生成TypeScript定义文件(.d.ts)的工具。
* [SonarQube Maven Plugin](https://github.com/SonarSource/sonar-scanner-maven)：用于Maven的SonarQube扫描器。
* [Nexus Maven Plugin](https://github.com/sonatype/nexus-maven-plugins)：支持Nexus Suite的Apache Maven插件集合。
* [Scala Maven Plugin](https://github.com/davidB/scala-maven-plugin)：Scala Maven插件用于在Maven中编译/测试/运行/记录Scala代码。
* [JMeter Maven Plugin](https://github.com/jmeter-maven-plugin/jmeter-maven-plugin)：能够在构建过程中运行JMeter测试的Maven插件。
* [Polyglot Maven](https://github.com/takari/polyglot-maven)：Polyglot Maven是Maven 3.3.1+的一组扩展，允许使用XML以外的方言编写POM模型。
* [JAXB Tools](https://github.com/highsource/jaxb-tools)：用于XML模式编译的最先进的JAXB2 Maven插件。
* [Aadarchi](https://github.com/Riduidel/aadarchi)：Aadarchi是一个Maven原型，可轻松生成项目，允许使用C4、敏捷架构、Asciidoc和PlantUML的组合进行架构描述。
* [DepClean](https://github.com/ASSERT-KTH/depclean)：DepClean自动检测并删除Maven项目中未使用的依赖，由瑞士皇家理工学院开源。
* [Maven Javadoc Plugin](https://github.com/apache/maven-javadoc-plugin)：Javadoc插件使用Javadoc工具为指定项目生成javadoc。
* [GitFlow Helper Maven Plugin](https://github.com/egineering-llc/gitflow-helper-maven-plugin)：一个构建扩展和插件，可帮助Maven与gitflow项目、CI服务器和本地开发完美配合。
* [Modernizer Maven Plugin](https://github.com/gaul/modernizer-maven-plugin)：Modernizer Maven插件检测现代Java版本取代的遗留API的使用。
* [JavaFX Maven Plugin](https://github.com/openjfx/javafx-maven-plugin)：用于运行JavaFX 11+应用程序的Maven插件。
* [Versions Maven Plugin](https://github.com/mojohaus/versions)：当你想要管理项目POM中的工件版本时，可以使用Versions插件。
* [Asciidoctor Maven Plugin](https://github.com/asciidoctor/asciidoctor-maven-plugin)：通过JRuby使用Asciidoctor来处理项目内的AsciiDoc源文件的Maven插件。
* [Fmt Maven Plugin](https://github.com/spotify/fmt-maven-plugin)：格式化Java代码的固定Maven插件，由Spotify开源。
* [OS Maven Plugin](https://github.com/trustin/os-maven-plugin)：用于设置从${os.name}和${os.arch}属性检测到的各种有用属性的Maven插件。
* [Native Build Tools](https://github.com/graalvm/native-build-tools)：包含用于与GraalVM Native Image互操作的构建工具插件的仓库。
* [Azure Maven Plugin](https://github.com/microsoft/azure-maven-plugins)：该仓库包含Microsoft Azure服务的所有Maven插件。
* [Protoc-Jar Maven Plugin](https://github.com/os72/protoc-jar-maven-plugin)：简单的Maven插件，使用protoc-jar嵌入式protoc编译器编译.proto文件，提供跨主要平台的可移植性。
* [License Maven Plugin](https://github.com/mathieucarbou/license-maven-plugin)：用于管理源文件中许可证标头的Maven插件。
* [Appbundle Maven Plugin](https://github.com/federkasten/appbundle-maven-plugin)：可为OS X创建包含所有项目依赖项和必要元数据的应用程序包的Maven插件。
* [Duplicate-Finder Maven Plugin](https://github.com/basepom/duplicate-finder-maven-plugin)：用于查找并标记Java类路径上重复的类和资源的Maven插件。
* [GluonFX Maven Plugin](https://github.com/gluonhq/gluonfx-maven-plugin)：简化为Java/JavaFX Maven项目创建本机镜像的插件。
* [Tomcat Maven Plugin](https://github.com/apache/tomcat-maven-plugin)：在构建期间启动Tomcat服务器的Maven插件。
* [Exec Maven Plugin](https://github.com/mojohaus/exec-maven-plugin)：该插件提供了2个目标来帮助执行系统和Java程序。
* [GWT Maven Plugin](https://github.com/tbroyer/gwt-maven-plugin)：该插件旨在通过提供两个特定的包gwt-lib和gwt-app，使使用Maven构建GWT项目变得更容易。
* [JShell Maven Plugin](https://github.com/johnpoth/jshell-maven-plugin)：Java Shell工具(JShell)的Maven插件。
* [Cucable Maven Plugin](https://github.com/trivago/cucable-plugin)：简化并行运行Cucumber场景的Maven插件。
* [Web3j Maven Plugin](https://github.com/web3j/web3j-maven-plugin)：Web3j Maven插件用于根据Solidity合约文件创建Java类。
* [AspectJ Maven Plugin](https://github.com/mojohaus/aspectj-maven-plugin)：该插件使用AspectJ编译器ajc将AspectJ切面编织到类中。
* [Google App Engine Maven plugin](https://github.com/GoogleCloudPlatform/app-maven-plugin)：该Maven插件提供了构建和部署Google App Engine应用程序的目标。
* [Mosec Maven Plugin](https://github.com/momosecurity/mosec-maven-plugin)：用于检测Maven项目的第三方依赖组件是否存在安全漏洞。
* [Rewrite Maven Plugin](https://github.com/openrewrite/rewrite-maven-plugin)：OpenRewrite的Maven插件，将Rewrite检查和修复任务应用为构建任务。
* [Allure Maven Plugin](https://github.com/allure-framework/allure-maven)：该插件在Maven构建过程中通过现有XML文件生成Allure报告。
* [Heroku Maven Plugin](https://github.com/heroku/heroku-maven-plugin)：用于将Java应用程序直接部署到Heroku，而无需推送到Git仓库。
* [Cargo Maven Plugin](https://github.com/codehaus-cargo/cargo)：Cargo是一个瘦Java包装器，允许你以标准方式操作各种类型的应用程序容器(J2EE、Java EE、Jakarta EE等)。
* [Maven Build Scanner](http://github.com/intuit/maven-build-scanner)：Maven Build Scanner是一款挂载到Maven构建的工具，它可以生成报告和图表，详细分析Maven执行不同任务所花费的时间，由Intuit开发。
* [Maven IT Extension](https://github.com/khmarbaise/maven-it-extension)：实验性JUnit Jupiter扩展，用于为Maven插件/Maven扩展/Maven核心编写集成测试。
* [Maven PlantUML Plugin](https://github.com/arnaudroques/maven-plantuml-plugin)：一个使用PlantUML语法生成UML图的Maven插件。
* [Helm Maven Plugin](https://github.com/kokuwaio/helm-maven-plugin)：这是一个用于测试、打包和上传HELM图表的Maven插件。
* [Maven Download Plugin](https://github.com/maven-download-plugin/maven-download-plugin)：该插件可帮助Maven用户在Maven构建过程中下载不同协议上的不同文件。
* [Yeoman Maven Plugin](https://github.com/trecloux/yeoman-maven-plugin)：使用此插件可以将yeoman构建集成到你的Maven构建中。
* [Maven Dependency Plugin](https://github.com/apache/maven-dependency-plugin)：Dependency插件提供了操作工件的能力，它可以将工件从本地或远程仓库复制和/或解压到指定位置。
* [Spring MVC-RAML Plugin](https://github.com/phoenixnap/springmvc-raml-plugin)：Spring MVC-RAML项目旨在为使用Spring MVC框架的项目强制实施契约优先方法。
* [Git Build Hook Maven Plugin](https://github.com/rudikershaw/git-build-hook)：一个用于添加配置、安装git hooks以及初始化本地项目的git仓库的Maven插件。
* [Libsass Maven Plugin](https://github.com/warmuuh/libsass-maven-plugin)：Libsass Maven插件使用libsass编译sass文件。
* [Lombok Maven Plugin](https://github.com/awhitford/lombok.maven)：Lombok项目的Maven插件。
* [Prettier Maven Plugin](https://github.com/HubSpot/prettier-maven-plugin)：用于在构建期间运行prettier-java的Maven插件。
* [ArchUnit Maven plugin](https://github.com/societe-generale/arch-unit-maven-plugin)：ArchUnit Maven插件是ArchUnit的简单Maven包装器，使你能够轻松确保所有项目都遵循相同的架构规则。
* [Maven Release Plugin](https://github.com/apache/maven-release)：Maven Release提供了使用Maven发布项目的工具。
* [Build Helper Maven Plugin](https://github.com/mojohaus/build-helper-maven-plugin)：Build
  Helper包含多个目标来支持完成不同类型的任务，例如解析版本信息、向Maven项目添加补充源/测试文件夹或附加补充工件。
* [Rust Maven Plugin](https://github.com/questdb/rust-maven-plugin)：在Java Maven项目中构建Rust Cargo crates。
* [Gatling AWS Maven Plugin](https://github.com/electronicarts/gatling-aws-maven-plugin)：Gatling AWS Maven插件消除了扩展Gatling测试的痛苦，它在可配置数量的EC2实例上运行负载测试，聚合单个负载测试报告，并将结果上传到S3。
* [Maven Surefire JUnit5 TreeView Extension](https://github.com/fabriciorby/maven-surefire-junit5-tree-reporter)：Maven Surefire JUnit5插件的树视图控制台报告。
* [License Maven Plugin](https://github.com/mojohaus/license-maven-plugin)：用于从项目依赖项下载和收集许可证文件的Maven插件。
* [Jaxb2 Maven Plugin](https://github.com/mojohaus/jaxb2-maven-plugin)：从XML模式(以及可选的绑定文件)生成Java类以及从带注解的Java类创建XML模式的Maven插件。
* [Elasticsearch Maven Plugin](https://github.com/alexcojocaru/elasticsearch-maven-plugin)：一个用于在构建的集成测试阶段运行Elasticsearch版本5+实例的Maven插件。
* [Mojo Executor](https://github.com/mojo-executor/mojo-executor)：Mojo Executor提供了一种在Maven插件中执行其他Mojo(插件)的方法，允许你轻松创建由其他插件组成的Maven插件。
* [Maven Compiler Plugin](https://github.com/apache/maven-compiler-plugin)：Compiler插件用于编译Java源代码。
* [VisualEE](https://github.com/Thomas-S-B/visualee)：一个用于可视化Java EE项目的Maven插件。
* [Maven Golang](https://github.com/raydac/mvn-golang)：用于自动化GoSDK加载和构建项目的Maven插件。
* [Go Offline Maven Plugin](https://github.com/qaware/go-offline-maven-plugin)：用于下载Maven构建所需的所有依赖项和插件的Maven插件，这样构建之后可以在没有互联网连接的情况下运行。
* [Maven Shade Plugin](https://github.com/apache/maven-shade-plugin)：该插件提供了将工件打包在uber-jar中的功能，包括其依赖项，并遮蔽(即重命名)某些依赖项的包。
* [Maven Enforcer Plugin](https://github.com/apache/maven-enforcer)：Enforcer插件提供了控制某些环境约束的目标，例如Maven版本、JDK版本和操作系统系列，以及更多内置规则和用户创建的规则。
* [JasperReports Maven Plugin](https://github.com/alexnederlof/Jasper-report-maven-plugin)：这个Maven插件会将JasperReport报告文件编译到target目录。
* [Maven Archetype Plugin](https://github.com/apache/maven-archetype)：Archetype是一个Maven项目模板工具包。
* [Takari Maven Plugin](https://github.com/takari/takari-maven-plugin)：用于安装Maven Wrapper的Maven插件。
* [Really Executable Jars Maven Plugin](https://github.com/brianm/really-executable-jars-maven-plugin)：用于制作chmod +x jar文件的Maven插件。
* [Multi Module Maven Release Plugin](https://github.com/danielflower/multi-module-maven-release-plugin)：一个快速的Maven发布插件，不添加额外的提交，并且可以很好地与单个或多个模块配合使用。
* [Liberty Maven Plugin](https://github.com/OpenLiberty/ci.maven)：Liberty Maven插件支持Liberty运行时和服务器的安装和操作控制。
* [NAR Maven Plugin](https://github.com/maven-nar/nar-maven-plugin)：这个插件允许你在许多不同的架构上以及使用许多不同的编译器/链接器编译本机代码生成的输出包含在本机存档文件中。
* [JSass](https://github.com/bit3/jsass)：jsass是一个模块化的Java sass编译器。
* [P2 Maven Plugin](https://github.com/reficio/p2-maven-plugin)：这是一个易于使用的Maven插件，负责Eclipse RCP环境中第三方依赖管理的自动化。
* [Minify Maven Plugin](https://github.com/samaxes/minify-maven-plugin)：Minify Maven插件组合并最小化你的CSS和JavaScript文件，以加快页面加载速度。
* [Grunt Maven Plugin](https://github.com/allegro/grunt-maven-plugin)：Grunt Maven插件允许你将Grunt任务集成到Maven构建过程中。
* [Formatter Maven Plugin](https://github.com/revelc/formatter-maven-plugin)：该项目提供了一种在Maven构建期间自动重新格式化Maven项目或验证其格式的机制。
* [CycloneDX Maven Plugin](https://github.com/CycloneDX/cyclonedx-maven-plugin)：CycloneDX
  Maven插件生成CycloneDX软件BOM，其中包含项目的所有直接和传递依赖项的聚合。
* [Clojure Maven Plugin](https://github.com/talios/clojure-maven-plugin)：该插件旨在使在混合语言企业项目中工作时尽可能轻松地使用Clojure。
* [Confluence Publisher](https://github.com/confluence-publisher/confluence-publisher)：Confluence Publisher允许用AsciiDoc编写并直接使用要发布到Confluence空间的文档代码库进行版本控制的文档。
* [Flatten Maven Plugin](https://github.com/mojohaus/flatten-maven-plugin)：该插件会生成pom.xml的扁平化版本，并让Maven来安装和部署该版本，而不是原始的pom.xml。
* [Git Code Format Maven Plugin](https://github.com/Cosium/git-code-format-maven-plugin)：一个可自动将代码格式化程序部署为预提交git hook的Maven插件。
* [BuildNumber Maven Plugin](https://github.com/mojohaus/buildnumber-maven-plugin)：这个Mojo旨在为你每次构建项目时获取唯一的构建号。
* [GitHub Maven Plugins](https://github.com/github/maven-plugins)：与GitHub集成的Maven插件集合，这些插件通过GitHub Java库构建在API v3之上。
* [ProGuard Maven Plugin](https://github.com/wvengen/proguard-maven-plugin)：ProGuard Maven插件支持模块化ProGuard包。
* [Git-Flow Maven Plugin](https://github.com/aleksandr-m/gitflow-maven-plugin)：Git-Flow
  Maven插件支持各种Git工作流，包括GitFlow和GitHub Flow，该插件从命令行运行Git和Maven命令。
* [Java Debian Package](https://github.com/tcurdt/jdeb)：该库提供了一个Ant任务和一个Maven插件，可以以真正跨平台的方式从Java构建创建Debian软件包。
* [Launch4j Maven Plugin](https://github.com/orphan-oss/launch4j-maven-plugin)：一个包装Launch4j的Maven插件。
* [Coveralls Maven Plugin](https://github.com/trautonen/coveralls-maven-plugin)：用于向Coveralls Web服务提交Java代码覆盖率报告的Maven插件。
* [Sortpom Maven Plugin](https://github.com/Ekryd/sortpom)：通过格式化XML并按预定义的顺序组织XML部分来帮助用户对pom.xml进行排序的Maven插件。
* [Jasmine Maven Plugin](https://github.com/searls/jasmine-maven-plugin)：用于执行Jasmine Specs的Maven插件。
* [GMavenPlus](https://github.com/groovy/GMavenPlus)：GMavenPlus是GMaven的重写版本，GMaven是一个Maven插件，允许你将Groovy集成到Maven项目中。
* [Mvnpm](https://github.com/mvnpm/mvnpm)：Mvnpm允许直接从Maven或Gradle项目使用NPM Registry包作为依赖。

## Gradle插件

* [Gradle Retrolambda](https://github.com/evant/gradle-retrolambda)：用于在Java 6、7和Android中获取Java Lambda支持。
* [Dexcount Gradle Plugin](https://github.com/KeepSafe/dexcount-gradle-plugin)：用于报告每次构建时APK中方法引用的数量。
* [Hunter](https://github.com/Leaking/Hunter)：一个快速、增量、并发的框架，用于开发Android项目的编译插件来操作字节码。
* [Gradle Shadow](https://github.com/johnrengelman/shadow)：Gradle插件，用于创建fat/uber JAR，支持包重定位。
* [Gradle Docker Plugin](https://github.com/bmuschko/gradle-docker-plugin)：用于管理Docker镜像和容器的Gradle插件。
* [Dependency Management Plugin](https://github.com/spring-gradle-plugins/dependency-management-plugin)：提供类似Maven的依赖管理功能。
* [Gradle Test Logger Plugin](https://github.com/radarsh/gradle-test-logger-plugin)：用于在运行测试时在控制台上打印漂亮日志的Gradle插件。
* [JavaFX Gradle Plugin](https://github.com/FibreFoX/javafx-gradle-plugin)：用于JavaFX的Gradle插件。
* [Google Play Gradle Plugin](https://github.com/google/play-services-plugins)：帮助使用Google Play服务SDK的插件。
* [Clean Architecture Gradle Plugin](https://github.com/bancolombia/scaffold-clean-architecture)：用于按照最佳实践创建基于Clean Architecture的Java和Kotlin应用程序的Gradle插件，由哥伦比亚银行开源。
* [WSDL2Java Gradle Plugin](https://github.com/nilsmagnus/wsdl2java)：用于从WSDL文件生成Java源代码的Gradle插件。
* [Git-Version Gradle Plugin](https://github.com/palantir/gradle-git-version)：使用git describe生成版本字符串的Gradle插件。
* [Gradle Avro Plugin](https://github.com/davidmc24/gradle-avro-plugin)：允许轻松执行Avro的Java代码生成的Gradle插件。
* [Gradle Baseline Plugin](https://github.com/palantir/gradle-baseline)：为开发人员配置默认的代码质量工具。
* [Gradle AWS Plugin](https://github.com/classmethod/gradle-aws-plugin)：用于管理Amazon Web Services的Gradle插件。
* [Gradle Modules Plugin](https://github.com/java9-modularity/gradle-modules-plugin)：这个Gradle插件有助于使用Java 9平台模块系统。
* [Android SVG Drawable](https://github.com/avianey/androidsvgdrawable-plugin)：可在Android项目构建时从SVG文件生成合格的、特定于密度的PNG绘图。
* [OkBuck Gradle Plugin](https://github.com/uber/okbuck)：OkBuck是一个Gradle插件，允许开发人员在Gradle项目上使用Buck构建系统，由Uber开源。
* [Gradle Dependency Graph Generator Plugin](https://github.com/vanniktech/gradle-dependency-graph-generator-plugin)：可让你在图表中可视化依赖关系的Gradle插件。
* [Gradle Graal](https://github.com/palantir/gradle-graal)：一个Gradle插件，添加了下载、提取任务以及与GraalVM工具交互的任务。
* [Gradle Dependency Analyze](https://github.com/gradle-dependency-analyze/gradle-dependency-analyze)：Gradle的依赖分析插件。

## Intellij插件

* [Translation](https://github.com/YiiGuxing/TranslationPlugin)：基于IntelliJ的IDE/Android Studio的翻译插件。
* [IdeaVim](https://github.com/JetBrains/ideavim)：IdeaVim是适用于JetBrains IDE的Vim引擎。
* [Golang Plugin](https://github.com/go-lang-plugin-org/go-lang-idea-plugin)：用于IntelliJ的Go插件。
* [Rainbow Brackets](https://github.com/izhangzhihao/intellij-rainbow-brackets)：适用于基于IntelliJ的IDE/Android Studio/HUAWEI DevEco Studio的Rainbow Brackets插件。
* [EasyCode](https://gitee.com/makejava/EasyCode)：EasyCode是基于IntelliJ IDEA开发的一个代码生成插件，主要通过自定义模板(基于Velocity)来生成各种你想要的代码。
* [Leetcode Editor](https://github.com/shuzijun/leetcode-editor)：在JetBrains IDE中练习LeetCode的插件。
* [Lombok Intellij Plugin](https://github.com/mplushnikov/lombok-intellij-plugin)：提供对Lombok注解的支持。
* [Key Promoter X](https://github.com/halirutan/IntelliJ-Key-Promoter-X)：用于学习快捷方式的现代IntelliJ插件。
* [Flutter Plugin](https://github.com/flutter/flutter-intellij)：用于Flutter开发的IntelliJ插件。
* [EmmyLua](https://github.com/EmmyLua/IntelliJ-EmmyLua)：IntelliJ IDEA的Lua IDE/调试器插件。
* [EasyDoc](https://github.com/starcwang/easy_javadoc)：IntelliJ IDEA插件，自动生成JavaDoc文档注释。
* [AutoDev](https://github.com/unit-mesh/auto-dev)：人工智能驱动的编码向导，具有多语言支持、自动代码生成，以及有用的错误消除助手，由UnitMesh开源。
* [Elixir](https://github.com/KronicDeth/intellij-elixir)：适用于JetBrain IntelliJ平台(包括Rubymine)的Elixir插件。
* [Haskell](https://github.com/rikvdkleij/intellij-haskell)：用于Haskell的IntelliJ插件。
* [WakaTime](https://github.com/wakatime/jetbrains-wakatime)：WakaTime是一个开源Jetbrains插件，用于根据你的编程活动自动生成指标、见解和时间跟踪。
* [Mybatislog](https://github.com/Link-Kou/intellij-mybaitslog)：Mybatislog是基于Intellij开发的插件项目，用来格式化输出Mybatis的SQL。
* [GsonFormat](https://github.com/zzz40500/GsonFormat)：这是一个可以从JSON字符串生成JSON模型的插件。
* [MinecraftDev](https://github.com/minecraft-dev/MinecraftDev)：IntelliJ IDEA插件，为Minecraft模组项目提供特殊支持。
* [GitIgnore](https://github.com/JetBrains/idea-gitignore)：.ignore IntelliJ IDEA支持插件。
* [Intellij Swagger](https://github.com/zalando/intellij-swagger)：可帮助你在IntelliJ IDEA中轻松编辑Swagger和OpenAPI规范文件。
* [Scala Plugin](https://github.com/JetBrains/intellij-scala)：IntelliJ IDEA的Scala插件。
* [CheckStyle IDEA](https://github.com/jshiell/checkstyle-idea)：IntelliJ IDEA的CheckStyle插件。
* [Android Parcelable](https://github.com/mcharmas/android-parcelable-intellij-plugin)：用于Android Parcelable样板代码生成的IntelliJ插件。
* [IntelliJ Plugins](https://github.com/JetBrains/intellij-plugins)：IntelliJ IDEA Ultimate和其他基于IntelliJ平台的IDE发行版中包含的开源插件。
* [PHP Inspections](https://github.com/kalessil/phpinspectionsea)：PHP静态代码分析器。
* [Restful Fast Request](https://github.com/dromara/fast-request)：Restful Fast Request是Postman的Intellij IDEA版本，它是一个强大的RESTful API工具包插件，由dromara社区开源。
* [Devoxx Genie](https://github.com/devoxx/DevoxxGenieIDEAPlugin)：DevoxxGenie是IntelliJ IDEA的一个插件，它使用本地LLM(Ollama、LMStudio、GPT4All、Jan和Llama.cpp)和基于云的LLM来帮助审查、测试和解释你的项目代码。

## Spring库

* [Spring Brick](https://gitee.com/starblues/springboot-plugin-framework-parent)：为动态扩展系统而生的插件开发框架。
* [MyBatis Spring Boot](https://github.com/mybatis/spring-boot-starter)：MyBatis与Spring Boot集成。
* [Retrofit Spring Boot](https://github.com/LianjiaTech/retrofit-spring-boot-starter)：适用于Retrofit的Spring Boot Starter，支持快速集成和功能增强，由链家开源。
* [Spring Cloud Zuul RateLimit](https://github.com/marcosbarbero/spring-cloud-zuul-ratelimit)：用于在Netflix Zuul中启用每个服务的速率限制的模块。
* [Spring Boot DataSource Decorator](https://github.com/gavlyukovskiy/spring-boot-data-source-decorator)：Spring Boot与p6spy、datasource-proxy、flexy-pool和spring-cloud-sleuth集成。
* [ChatGPT Spring Boot](https://github.com/linux-china/chatgpt-spring-boot-starter)：Spring Boot ChatGPT Starter。
* [ChatGPT Spring Boot](https://github.com/flashvayne/chatgpt-spring-boot-starter)：基于OpenAI官方API的Spring Boot Starter。
* [Wavefront Spring Boot Starter](https://github.com/wavefrontHQ/wavefront-spring-boot)：该项目为Wavefront提供了Spring Boot 3 Starter。
* [Spring Boot Dubbo](https://github.com/apache/dubbo-spring-boot-project)：Dubbo Spring Boot项目可以轻松使用Dubbo作为RPC框架创建Spring Boot应用程序。
* [Spring Boot Jasypt](https://github.com/ulisesbocchio/jasypt-spring-boot)：Jasypt Spring Boot为Spring Boot应用程序中的属性源提供加密支持。
* [Okta Spring Boot](https://github.com/okta/okta-spring-boot)：Okta Spring Boot Starter。
* [Chaos Monkey Spring Boot](https://github.com/codecentric/chaos-monkey-spring-boot)：该项目为Spring Boot应用程序提供了Chaos Monkey，并将尝试攻击你正在运行的Spring Boot应用程序。
* [Spring Boot Logging](https://github.com/piomin/spring-boot-logging)：用于记录Spring Boot应用程序的HTTP请求/响应以及与Elastic Stack集成的库。
* [Spring Boot Starter Calma](https://github.com/marvinSpring/spring-boot-starter-calma)：异常通知框架。
* [Spring Boot HTMX](https://github.com/wimdeblauwe/htmx-spring-boot)：用于使用htmx的Spring Boot和Thymeleaf助手。
* [Spring Batch Plus](https://github.com/naver/spring-batch-plus)：Spring Batch Plus为Spring Batch提供了扩展功能，NAVER开源。
* [WireMock Spring Boot](https://github.com/maciejwalkowiak/wiremock-spring-boot)：WireMock Spring Boot极大地简化了基于Spring Boot和Junit 5的集成测试中的HTTP客户端测试。
* [DJL Spring Boot](https://github.com/deepjavalibrary/djl-spring-boot-starter)：DJL Spring Boot Starter。
* [Spring ViewComponent](https://github.com/tschuehly/spring-view-component)：使用Spring创建服务器端ViewComponent的库。
* [Narayana Spring Boot](https://github.com/snowdrop/narayana-spring-boot)：Narayana Spring Boot自动配置和Starter。
* [Disruptor Spring Boot Starter](https://github.com/hiwepy/disruptor-spring-boot-starter)：基于Disruptor的Spring Boot Starter实现。
* [Velocity Spring Boot Starter](https://github.com/alibaba/velocity-spring-boot-project)：一个针对Velocity的Spring Boot Starter，包括Spring官方和阿里巴巴的实现，例如Layout、Tools支持。
* [Spring JDBC Plus](https://github.com/naver/spring-jdbc-plus)：Spring JDBC Plus提供基于Spring Data JDBC的扩展，由Naver开源。
* [Alibaba Spring Boot](https://github.com/alibaba/aliyun-spring-boot)：阿里云服务Spring Boot Starter。
* [Camel Spring Boot](https://github.com/apache/camel-spring-boot)：Camel Spring Boot支持。
* [Wicket Spring Boot](https://github.com/MarcGiffing/wicket-spring-boot)：Wicket的Spring Boot Starter。
* [Bitcoin Spring Boot](https://github.com/theborakompanioni/bitcoin-spring-boot-starter)：使用Spring Boot编写企业比特币应用程序的工具。
* [Spring Boot Bucket4j](https://github.com/MarcGiffing/bucket4j-spring-boot-starter)：Bucket4j的Spring Boot Starter。
* [Camunda Spring Boot](https://github.com/camunda/camunda-bpm-spring-boot-starter)：Camunda的Spring Boot Starter。
* [Charon Spring Boot](https://github.com/mkopylec/charon-spring-boot-starter)：以Spring Boot Starter形式的反向代理实现。
* [reCAPTCHA Spring Boot](https://github.com/mkopylec/recaptcha-spring-boot-starter)：Google reCAPTCHA的Spring Boot Starter。
* [RocketMQ Spring](https://github.com/apache/rocketmq-spring)：该项目旨在帮助开发者快速将RocketMQ与Spring Boot集成。
* [PageHelper Spring Boot](https://github.com/pagehelper/pagehelper-spring-boot)：Mybatis分页插件与Spring Boot的集成。
* [Pug4j Spring Boot](https://github.com/domix/jade4j-spring-boot-starter)：Spring Boot Jade4j Starter。
* [RESTEasy Spring Boot](https://github.com/resteasy/resteasy-spring-boot)：RESTEasy Spring Boot Starter。
* [CXF Spring Boot](https://github.com/codecentric/cxf-spring-boot-starter)：由Spring Boot和CXF提供支持的企业和生产就绪SOAP Web Service。
* [Spring Boot Batch Web](https://github.com/codecentric/spring-boot-starter-batch-web)：由Spring Boot提供支持的企业就绪、生产就绪的批处理应用程序。
* [Spring-Dotenv](https://github.com/paulschwarz/spring-dotenv)：为Spring提供Dotenv属性源。
* [Spring Boot TestJars](https://github.com/spring-projects-experimental/spring-boot-testjars)：该项目允许用户通过将外部Spring
  Boot应用程序创建为Bean来轻松启动它。
* [Infobip Spring Data Querydsl](https://github.com/infobip/infobip-spring-data-querydsl)：Infobip Spring Data Querydsl使用户能够在Spring Data Repository之上利用Querydsl API的全部功能。
* [Spring Boot Graceful Shutdown](https://github.com/SchweizerischeBundesbahnen/springboot-graceful-shutdown)：Spring Boot Graceful Shutdown使你的Spring Boot应用程序能够在OpenShift上进行滚动部署，而无需任何停机，由瑞士联邦铁路开源。
* [Atlassian Connect Spring Boot](https://bitbucket.org/atlassian/atlassian-connect-spring-boot)：该仓库包含一个Spring
  Boot Starter，用于为JIRA(软件、服务台和核心)和Confluence构建Atlassian Connect附加组件。
* [Assistant](https://github.com/Geniusay/Assistant)：Assistant是一个基于Spring Boot框架的后端开发工具。
* [Spring Boot Dynamic Config](https://github.com/Code2Life/spring-boot-dynamic-config)：一个注解实现Spring Boot应用的动态配置，配置热重载最简洁的方案。
* [Spring Data JDBC Repository](https://github.com/nurkiewicz/spring-data-jdbc-repository)：该项目的目的是提供基于Spring框架的JdbcTemplate的关系数据库的通用、轻量级且易于使用的DAO实现，并与Spring Data项目范围兼容。

## Mybatis库

* [Mybatis Plus](https://github.com/baomidou/mybatis-plus)：MyBatis Plus是MyBatis的一个强大的增强工具包，用于简化开发。
* [Mybatis Flex](https://github.com/mybatis-flex/mybatis-flex)：Mybatis Flex是一个优雅的Mybatis增强框架。
* [Fluent Mybatis](https://github.com/atool/fluent-mybatis)：Fluent MyBatis是一个MyBatis增强工具。
* [MybatisPlus Ext](https://gitee.com/dromara/mybatis-plus-ext)：MybatisPlus Ext对MybatisPlus做了进一步的拓展封装，即保留原功能，又添加了更多有用便捷的功能。
* [Xbatis](https://gitee.com/xbatis/xbatis)：Xbatis是一款基于Mybatis的ORM框架。
* [Mybatis PageHelper](https://github.com/pagehelper/Mybatis-PageHelper)：Mybatis通用分页插件。
* [Mapper](https://github.com/abel533/Mapper)：易于使用的Mybatis通用Mapper。
* [Mapper](https://gitee.com/free/Mapper)：极其方便的使用Mybatis单表的增删改查工具。
* [MyBatis Mapper](https://github.com/mybatis-mapper/mapper)：这是一个不需要任何配置就可以直接使用的通用Mapper。
* [FastMybatis](https://gitee.com/durcframework/fastmybatis)：FastMybatis是一个Mybatis开发框架，其宗旨为简单、快速、有效。
* [MyBatis Plus Join](https://gitee.com/best_handsome/mybatis-plus-join)：对MyBatis-Plus多表查询的扩展。
* [MyBatis Dynamic SQL](https://github.com/mybatis/mybatis-dynamic-sql)：适用于Kotlin和Java的SQL DSL，支持MyBatis或Spring JdbcTemplate的渲染。
* [Mybatis Generator](https://github.com/mybatis/generator)：用于Mybatis的代码生成器。
* [MybatisPlus Generator](https://github.com/baomidou/generator)：用于MybatisPlus的代码生成器。
* [Mybatis Mate](https://gitee.com/baomidou/mybatis-mate-examples)：Mybatis-Mate为MP企业级模块，支持分库分表，数据审计、数据敏感词过滤(AC算法)，字段加密，字典回写(数据绑定)，数据权限，表结构自动生成SQL维护，支持国密SM2、SM3、SM4加密算法等。
* [SQL Analysis](https://github.com/jd-opensource/sql-analysis)：SQL Analysis是基于Mybatis插件设计的一款慢SQL分析组件，由京东开源。
* [MyBatis Generator UI](https://github.com/zouzg/mybatis-generator-gui)：MyBatis Generator UI是基于MyBatis Generator开发一款界面工具。
* [MyBatis Generator Plugin](https://github.com/itfsw/mybatis-generator-plugin)：Mybatis Generator代码生成插件拓展。
* [MyBatis Plus Generator UI](https://github.com/davidfantasy/mybatis-plus-generator-ui)：提供交互式的Web UI用于生成兼容Mybatis Plus框架的相关功能代码。
* [MybatisPlus Code Generator](https://github.com/fengwenyi/mybatis-plus-code-generator)：MyBatisPlus代码生成器。
* [PNDao](https://gitee.com/piaoniu/pndao)：一个非常简单的MyBatis辅助工具，可以基于DAO的命名约定帮你生成并维护SQL语句，由票牛开源。

## Hibernate库

* [Hypersistence Utils](https://github.com/vladmihalcea/hypersistence-utils)：Hypersistence Utils库提供Spring和Hibernate实用程序，可以帮助充分利用数据访问层。
* [HibernatePlus](https://gitee.com/baomidou/hibernate-plus)：Hibernate增强工具包，只做增强不做改变，更加精简持久层CRUD操作。
* [TorpedoQuery](https://github.com/xjodoin/torpedoquery)：类型安全的Hibernate查询生成器。
* [Hibernate Hydrate](https://github.com/arey/hibernate-hydrate)：Hibernate Hydrate项目的主要目标是填充持久实体图，从而避免著名的LazyInitializationException。
* [Hibernate Redis](https://github.com/debop/hibernate-redis)：使用Redis的Hibernate二级缓存权限。
* [SQLite Dialect](https://github.com/gwenn/sqlite-dialect)：受NHibernate启发的SQLite的Hibernate方言。

## JPA库

* [JPA Spec](https://github.com/wenhao/jpa-spec)：JPA按Specification查询框架。
* [RSQL JPA](https://github.com/tennaito/rsql-jpa)：该库提供了RSQL表达式到JPA Criteria Query(JPQL的对象表示)的转换器。
* [Kotlin JDSL](https://github.com/line/kotlin-jdsl)：Kotlin JDSL是一个Kotlin库，它简化了查询的构建和执行，由Line开源。
* [RSQL JPA Specification](https://github.com/perplexhub/rsql-jpa-specification)：将RSQL查询转换为org.springframework.data.jpa.domain.Specification或com.querydsl.core.types.Predicate并支持实体关联查询。
* [FluentJPA](https://github.com/streamx-co/FluentJPA)：FluentJPA是一种用于关系型数据库和JPA的语言集成查询(LINQ) 技术，它允许你通过直接集成到Java语言中来编写强类型查询。
* [Blaze Persistence](https://github.com/Blazebit/blaze-persistence)：Blaze Persistence是面向JPA提供程序的丰富Criteria API。
* [JPAStreamer](https://github.com/speedment/jpa-streamer)：JPAStreamer是一个轻量级库，用于将JPA查询表达为Java Stream。
* [ActiveJPA](https://github.com/ActiveJpa/activejpa)：ActiveJPA是一个试图在JPA之上实现ActiveRecord模式的Java库。
* [QueryStream](https://github.com/querystream/querystream)：QueryStream允许你使用类似Stream的API执行JPA查询。
* [Spring Filter](https://github.com/turkraft/springfilter)：使用用户友好的查询语法动态过滤JPA实体和Mongo集合。
* [Fenix](https://github.com/blinkfox/fenix)：Fenix是一个为了解决复杂动态SQL(JPQL)而生的Spring Data JPA扩展库。
* [Kundera](https://github.com/Impetus/kundera)：Kundera是一个带有JPA接口的多语言对象映射器。
* [Spring Data JPA EntityGraph](https://github.com/Cosium/spring-data-jpa-entity-graph)：Spring Data JPA扩展允许在Repository上完全动态使用EntityGraph。
* [Spring Data JPA DataTables](https://github.com/darrachequesne/spring-data-jpa-datatables)：该项目是Spring Data JPA项目的扩展，以便于与启用了服务器端处理的jQuery插件DataTables一起使用。
* [Spring Search](https://github.com/sipios/spring-search)：Spring Search提供了一种简单的查询语言来对JPA实体执行高级搜索。
* [Specification Arg Resolver](https://github.com/tkaczmarzyk/specification-arg-resolver)：用于使用Spring MVC和Spring Data JPA过滤数据的替代API。
* [Spring Data JPA MongoDB Expressions](https://github.com/mhewedy/spring-data-jpa-mongodb-expressions)：Spring Data JPA MongoDB Expressions是一个允许你使用MongoDB查询语言查询Spring Data JPA Repository的库。

## 其他

* [FizzBuzz Enterprise Edition](https://github.com/EnterpriseQualityCoding/FizzBuzzEnterpriseEdition)：FizzBuzz是一款作为编程作业而广受欢迎的游戏，用于在工作面试期间淘汰非程序员。
* [Jpostal](https://github.com/openvenues/jpostal)：Jpostal是libpostal的Java绑定，用于快速国际街道地址解析/规范化。
* [Gurux.DLMS](https://github.com/Gurux/gurux.dlms.java)：Gurux.DLMS库是一个高性能Java组件，可帮助你读取DLMS/COSEM兼容的电表、燃气表或水表。
* [FlowGate](https://github.com/vmware/flowgate)：FlowGate是一个与供应商无关的开源项目，可帮助企业集成设施系统数据和IT数据，形成其运营的单一整体视图，由VMWare开源。
* [ALARA](https://github.com/svalinn/ALARA)：ALARA的主要目的是计算整个核系统(包括裂变反应堆、聚变反应堆和加速器)中子辐射引起的诱导活化，由威斯康星大学麦迪逊分校开源。
* [OpenLCA](https://github.com/GreenDelta/olca-app)：OpenLCA是一款用于可持续性和生命周期评估的开源免费软件。
* [XraysLib](https://github.com/tschoonj/xraylib)：用于X射线荧光应用的X射线物质相互作用截面库。
* [Unique4j](https://github.com/prat-man/unique4j)：Unique4j是一个跨平台Java库，仅允许Java应用程序的单个实例运行，并支持第一个实例和后续实例之间的通信。
* [Portico](https://github.com/openlvc/portico)：Portico是一个开源HLA运行时基础设施(RTI)实现，它旨在确保开源和免费访问驱动HLA联盟所需的必要基础设施。
* [K Framework](https://github.com/runtimeverification/k)：K Framework是一种用于设计和建模编程语言和软件/硬件系统的工具。
* [OpenWayback](https://github.com/iipc/openwayback)：OpenWayback是全球网络档案馆用来在用户浏览器中“回放”存档网站的关键软件，由国际互联网保护联盟开源。
* [Alf.io](https://github.com/alfio-event/alf.io)：Alf.io是一款免费开源活动出席管理系统，专为关心客户隐私、安全和公平定价政策的活动组织者而开发。
* [Alchemist](https://github.com/AlchemistSimulator/Alchemist)：Alchemist是一个用于普适性、聚合性和受自然启发的计算的模拟器。
* [NeqSim](https://github.com/equinor/neqsim)：NeqSim是一个用于估计流体特性和流程设计的Java库，由挪威科技大学开源。
* [AsTeRICS](https://github.com/asterics/AsTeRICS)：AsTeRICS是一个用于辅助技术的免费开源图形构建集。
* [JIDT](https://github.com/jlizier/jidt)：JIDT提供了复杂系统中分布式计算的信息论测量的独立、开源代码Java实现。
* [Metis](https://github.com/europeana/metis-framework)：Metis是数据发布框架，包括客户端应用程序和许多数据处理服务，由欧洲数位图书馆开源。
* [Hello eBPF](https://github.com/parttimenerd/hello-ebpf)：允许直接用Java编写eBPF程序。
* [MetarParser](https://github.com/mivek/MetarParser)：这个Java库提供了Metar和TAF解码器。
* [XINCHECK](https://github.com/tianlian0/duplicate-check-sample)：文本查重SDK，可用于论文查重、标书查重、文档查重、作业查重、合同查重、防串标等场景，由芯锋科技公司开发。

## 教程系列

这里包含不同领域的教程项目。

#### Java教程

* [JavaGuide](https://github.com/Snailclimb/JavaGuide)：一份涵盖大部分Java程序员所需要掌握的核心知识。
* [Advanced Java](https://github.com/doocs/advanced-java)：本项目大部分内容来自中华石杉，内容涵盖高并发、分布式、高可用、微服务、海量数据处理等领域知识。
* [Java Family](https://github.com/AobingJava/JavaFamily)：一份涵盖大部分Java程序员所需要掌握的核心知识。
* [Tutorials](https://github.com/eugenp/tutorials)：该项目是小型且重点突出的教程的集合，每个教程都涵盖Java生态系统中一个明确定义的开发领域。
* [IntelliJ IDEA Tutorial](https://github.com/judasn/IntelliJ-IDEA-Tutorial)：IntelliJ IDEA简体中文专题教程。
* [Java 8 Tutorial](https://github.com/winterbe/java8-tutorial)：本教程将逐步指导你了解所有新的语言功能。
* [Java](https://github.com/DuGuQiuBai/Java)：学习Java的基础仓库。
* [ToBeBetterJavaer](https://github.com/itwanger/toBeBetterJavaer)：一份通俗易懂、风趣幽默的Java学习指南，内容涵盖Java基础、Java并发编程、Java虚拟机、Java企业级开发、Java面试等核心知识点。
* [JVM](https://github.com/doocs/jvm)：Java虚拟机底层原理知识总结。
* [JavaTutorial](https://github.com/h2pl/JavaTutorial)：本仓库涵盖大部分Java程序员所需要掌握的核心知识，整合了互联网上的很多优质Java技术文章。
* [Athena](https://github.com/ZhongFuCheng3y/athena)：Java后端知识图谱。
* [On Java 8](https://github.com/lingcoder/OnJava8)：《On Java 8》中文版。
* [JGrowing](https://github.com/javagrowing/JGrowing)：Java学习路线仓库。
* [Concurrent](https://github.com/RedSpider1/concurrent)：这是RedSpider社区成员原创与维护的Java多线程系列文章。
* [Blog Demo](https://github.com/zq2599/blog_demos)：这里有六百多篇原创文章的详细分类和汇总，以及对应的源码，内容涉及Java、Docker、Kubernetes、DevOps等方面。
* [Tech Weekly](https://github.com/mercyblitz/tech-weekly)：小马哥技术周报。
* [Code Guide](https://github.com/fuzhengwei/CodeGuide)：本代码库是Java开发的学习历程技术汇总，旨在为大家提供一个清晰详细的学习教程，侧重点更倾向编写Java核心内容。
* [Effective Java Third Edition](https://github.com/jbloch/effective-java-3e-source-code)：来自《Effective Java》第三版的源代码，根据需要进行了少量添加以使其可运行。
* [Technology Talk](https://github.com/aalansehaiyang/technology-talk)：一份Java程序员需要的技术指南，这里有面试题、系统架构、职场锦囊、主流中间件等。
* [Learning Notes](https://github.com/francistao/LearningNotes)：Java学习笔记。
* [Java Developer Roadmap](https://github.com/s4kibs4mi/java-developer-roadmap)：2024年成为Java开发人员的路线图。
* [JavaCore](https://github.com/dunwu/javacore)：JavaCore是对Java核心技术的经验总结。
* [BookSource](https://github.com/guolindev/booksource)：《第一行代码 第2版》全书源代码。
* [On Java 8 Examples](https://github.com/BruceEckel/OnJava8-Examples)：《On Java 8》一书的代码示例。
* [Java Practice](https://github.com/HelloWorld521/Java)：Java项目实战练习。
* [Code Examples](https://github.com/thombergs/code-examples)：该仓库包含示例项目，展示如何使用不同的Java技术。
* [Learn Java Bug](https://github.com/threedr3am/learnjavabug)：Java安全相关的漏洞和技术Demo。
* [30 Seconds Of Java 8](https://github.com/hellokaton/30-seconds-of-java8)：你可以在30秒或更短时间内收集有用的Java 8代码片段。
* [Note](https://github.com/scalad/Note)：常规Java工具、算法、加密、数据库、面试题、源代码分析、解决方案。
* [Java Lambda Internals](https://github.com/CarpenterLee/JavaLambdaInternals)：深入理解Java函数式编程和Streams API。
* [Java Learning](https://github.com/brianway/java-learning)：旨在打造在线最佳的Java学习笔记，含博客讲解和源码实例，包括Java SE和Java Web。
* [Java Tutorial](https://github.com/dunwu/java-tutorial)：Java Tutorial是一个Java教程。
* [Java Keeper](https://github.com/Jstarfish/JavaKeeper)：Java工程师必备架构体系知识总结：涵盖分布式、微服务、RPC等互联网公司常用架构，以及数据存储、缓存、搜索等必备技能。
* [Java 8 Lambdas Exercises](https://github.com/RichardWarburton/java-8-lambdas-exercises)：该仓库包含Java 8 Lambdas书籍的支持材料。
* [Six Finger](https://github.com/bin392328206/six-finger)：从Java基础、Java Web基础到常用的框架再到面试题、微服务、分布式、大数据都有完整的教程，几乎涵盖了Java必备的知识点。
* [WhatsMars](https://github.com/javahongxi/whatsmars)：Java生态研究(Spring Boot + Redis + Dubbo + RocketMQ + ElasticSearch)。

#### 大数据教程

* [BigData-Notes](https://github.com/heibaiying/BigData-Notes)：大数据入门指南。
* [Flink Learning](https://github.com/zhisheng17/flink-learning)：含Flink入门、概念、原理、实战、性能调优、源码解析等内容。
* [Flink Recommand System Demo](https://github.com/will-che/flink-recommandSystem-demo)：基于Flink实现的商品实时推荐系统。
* [DB Tutorial](https://github.com/dunwu/db-tutorial)：DB Tutorial是一个数据库教程。
* [Movie Recommend](https://github.com/LuckyZXL2016/Movie_Recommend)：基于Spark的电影推荐系统，包含爬虫项目、Web网站、后台管理系统以及Spark推荐系统。
* [BigData Guide](https://github.com/MoRan1607/BigDataGuide)：大数据学习指南，从零开始学习大数据开发，包含大数据学习各个阶段资汇总。
* [Learning Spark](https://github.com/databricks/learning-spark)：Learning Spark书中的示例。
* [Spark Doc Zh](https://github.com/apachecn/spark-doc-zh)：Spark官方文档中文版。
* [Coolplay Spark](https://github.com/lw-lin/CoolplaySpark)：Coolplay Spark包含Spark源代码解析、Spark类库、Spark代码等。

#### Spring教程

* [Spring Boot Demo](https://github.com/xkcoding/spring-boot-demo)：Spring Boot Demo是一个用来深度学习并实战Spring Boot的项目。
* [Spring Boot Examples](https://github.com/ityouknow/spring-boot-examples)：Spring Boot使用的各种示例，以最简单、最实用为标准，此开源项目中的每个示例都以最小依赖，最简单为标准，帮助初学者快速掌握Spring Boot各组件的使用。
* [SpringAll](https://github.com/wuyouzhuguli/SpringAll)：循序渐进，学习Spring Boot、Spring Batch、Spring Cloud、Spring Cloud Alibaba、Spring Security。
* [Spring Boot Reference Guide](https://github.com/qibaoguang/Spring-Boot-Reference-Guide)：Spring Boot Reference Guide中文翻译。
* [SpringBoot Labs](https://github.com/yudaocode/SpringBoot-Labs)：一个涵盖六个专栏：Spring Boot 2.X、Spring Cloud、Spring Cloud Alibaba、Dubbo、分布式消息队列、分布式事务的仓库。
* [Spring Cloud Learning](https://github.com/forezp/SpringCloudLearning)：史上最简单的Spring Cloud教程源码。
* [Spring Boot Learning Example](https://github.com/JeffLi1993/springboot-learning-example)：Spring Boot实践学习案例，是Spring Boot初学者及核心技术巩固的最佳实践。
* [SpringBoot-Learning](https://github.com/dyc87112/SpringBoot-Learning)：打造全网内容最全，比收费教程更好的Spring Boot免费教程。
* [Paas Cloud](https://github.com/paascloud/paascloud-master)：Spring Cloud + Vue +
  OAuth2.0全家桶实战，前后端分离模拟商城，完整的购物流程、后端运营平台，可以实现快速搭建企业级微服务项目。
* [SpringCloud-Learning](https://github.com/dyc87112/SpringCloud-Learning)：本项目内容为Spring Cloud教程的程序样例。
* [SpringBoot](https://github.com/527515025/springBoot)：Spring
  Boot框架与其它组件结合如JPA、MyBatis、WebSocket、Security、Shiro、Cache等的教程。
* [Spring Cloud Examples](https://github.com/ityouknow/spring-cloud-examples)：Spring Cloud使用的各种示例，以最简单、最实用为标准。
* [Spring Cloud Learning](https://github.com/macrozheng/springcloud-learning)：一套涵盖大部分核心组件使用的Spring Cloud教程，包括Spring Cloud Alibaba及分布式事务Seata，基于Spring Cloud Greenwich及Spring Boot 2.1.7。
* [JavaEE Test](https://github.com/lenve/JavaEETest)：Spring、Spring MVC、MyBatis、Spring Boot案例。
* [SSM](https://github.com/liyifeng1994/ssm)：手把手教你整合最优雅SSM框架：Spring MVC + Spring + MyBatis。
* [Spring Boot Vulnerability Exploit](https://github.com/LandGrey/SpringBootVulExploit)：Spring Boot相关漏洞学习资料，利用方法和技巧合集，黑盒安全评估check list。
* [Spring Boot Projects](https://github.com/ZHENFENG13/spring-boot-projects)：该仓库中主要是Spring Boot的入门学习教程以及一些常用的Spring Boot实战项目教程。
* [Spring Data Examples](https://github.com/spring-projects/spring-data-examples)：该仓库包含不同Spring Data模块的示例项目，以展示API以及如何使用模块提供的功能。
* [Spring Boot Guide](https://github.com/CodingDocs/springboot-guide)：Spring Boot 2.0+从入门到实战。
* [Spring MVC Showcase](https://github.com/spring-attic/spring-mvc-showcase)：通过小而简单的示例演示Spring MVC Web框架的功能。
* [Spring Boot In Action](https://github.com/hansonwang99/Spring-Boot-In-Action)：Spring Boot系列实战合集。
* [Small Spring](https://github.com/fuzhengwei/small-spring)：该项目以Spring源码学习为目的，通过手写简化版Spring框架，了解Spring核心原理。
* [Spring Boot Best Practice](https://github.com/javastacks/spring-boot-best-practice)：Spring Boot最佳实践，包括自动配置、核心原理、源码分析、国际化支持、调试、日志集成、热部署等。
* [SSM](https://github.com/crossoverJie/SSM)：从0开始构建SSM和分布式微服务。
* [MyBatis Spring Boot](https://github.com/abel533/MyBatis-Spring-Boot)：Spring Boot集成MyBatis的基础项目。
* [JWT Spring Security Demo](https://github.com/szerhusenBC/jwt-spring-security-demo)：将JWT与Spring Security和Spring Boot 2结合使用的演示。
* [SpringMVC MyBatis Learning](https://github.com/brianway/springmvc-mybatis-learning)：Spring MVC和MyBatis学习笔记，搭配示例，主要讲解一些基础的概念、用法和配置。
* [Spring Boot NoteBook](https://github.com/chengxy-nds/Springboot-Notebook)：Spring Boot整合各种技术的学习项目。
* [Spring Boot Socks](https://github.com/yizhiwazi/springboot-socks)：Spring Boot基础教程。
* [Spring Cloud REST TCC](https://github.com/prontera/spring-cloud-rest-tcc)：以Spring Cloud Netflix作为服务治理基础，展示基于TCC思想所实现的分布式事务解决方案。
* [Spring Boot Quick](https://github.com/vector4wang/spring-boot-quick)：基于Spring Boot的快速学习示例。
* [Spring Boot Learning](https://github.com/forezp/SpringBootLearning)：Spring Boot教程源码。
* [SpringBoot](https://github.com/lxy-go/SpringBoot)：这是一个SpringBoot的从入门程序集锦。
* [Spring Boot Cloud](https://github.com/zhangxd1989/spring-boot-cloud)：基于Spring Boot、Spring Cloud、Spring Oauth2和Spring Cloud Netflix等框架构建的微服务项目。
* [Spring Boot Learning](https://github.com/ityouknow/spring-boot-leaning)：Spring Boot 2.X最全课程代码。
* [Spring Guide](https://github.com/cheese10yun/spring-guide)：Spring实用指南。
* [SpringCloud](https://github.com/yinjihuan/spring-cloud)：《Spring Cloud微服务-全栈技术与案例解析》和《Spring Cloud微服务入门实战与进阶》配套源码。
* [Spring Cloud Code](https://github.com/SpringCloud/spring-cloud-code)：《重新定义Spring Cloud实战》实体书对应源码。
* [Spring Boot Vue](https://github.com/boylegu/SpringBoot-vue)：基于Spring Boot和Vue.js 2.x + Webpack 2.x作为Java全栈Web实践的示例演示。
* [Spring Security OAuth](https://github.com/Baeldung/spring-security-oauth)：致力于探索Spring Security 5中的新OAuth2堆栈。
* [Spring Boot Vue.js](https://github.com/jonashackt/spring-boot-vuejs)：展示如何构建一个使用Vue.js提供GUI的Spring Boot应用程序的示例项目。
* [Thinking In Spring Boot Samples](https://github.com/mercyblitz/thinking-in-spring-boot-samples)：《Spring Boot编程思想》示例工程。
* [Staffjoy](https://github.com/spring2go/staffjoy)：微服务和云原生架构教学案例项目，基于Spring Boot和Kubernetes技术栈。

#### 算法和数据结构教程

* [LeetCode Animation](https://github.com/MisterBooo/LeetCodeAnimation)：用动画的形式呈现解LeetCode题目的思路。
* [Hello Algo](https://github.com/krahets/hello-algo)：动画图解、一键运行的数据结构与算法教程。
* [The Algorithms](https://github.com/TheAlgorithms/Java)：所有算法的Java实现。
* [LeetCode Master](https://github.com/youngyangyang04/leetcode-master) ：《代码随想录》LeetCode刷题攻略：200道经典题目刷题顺序，共60w字的详细图解，视频难点剖析，50余张思维导图。
* [Hello Algorithm](https://github.com/geekxh/hello-algorithm)：一套针对小白的完整的算法训练流程。
* [LeetCode](https://github.com/doocs/leetcode)：本项目包含LeetCode、《剑指Offer(第2版)》、《剑指Offer(专项突击版)》、《程序员面试金典(第6版)》等题目的相关题解。
* [Algorithms](https://github.com/williamfiset/Algorithms)：该仓库的目标是演示如何以最简单、最优雅的方式正确实现常见的数据结构和算法。
* [CodeLibrary](https://github.com/indy256/codelibrary)：C++、Java、Kotlin、Python和Rust中的算法和数据结构集合。
* [Awesome Java LeetCode](https://github.com/Blankj/awesome-java-leetcode)：LeetCode算法与Java解决方案。
* [Algs4](https://github.com/kevin-wayne/algs4)：该仓库包含Robert Sedgewick和Kevin Wayne编写的教科书《算法》第4版中的算法和客户端的Java源代码。
* [LeetCode](https://github.com/yuanguangxin/LeetCode)：LeetCode题目分类与面试问题整理。
* [JS Sorting Algorithm](https://github.com/hustcc/JS-Sorting-Algorithm)：一本关于排序算法的GitBook在线书籍《十大经典排序算法》，多语言实现。
* [LeetCode](https://github.com/awangdev/leet-code)：Java LintCode/LeetCode问题解决方案。
* [Java Algorithms Implementation](https://github.com/phishman3579/java-algorithms-implementation)：算法和数据结构的Java实现。
* [Algorithm Base](https://github.com/chefyuan/algorithm-base)：用动画将算法说的通俗易懂。
* [LeetCode Book](https://github.com/krahets/LeetCode-Book)：《剑指Offer》 Python、Java、C++解题代码，LeetBook《图解算法数据结构》配套代码仓库。
* [LeetCode](https://github.com/fishercoder1534/Leetcode)：LeetCode问题的解决方案，每日更新。
* [Play With Algorithms](https://github.com/liuyubobobo/Play-with-Algorithms)：慕课网上的课程《算法与数据结构》示例代码，包括C++和Java版本。
* [AlgoDS](https://github.com/sherxon/AlgoDS)：这是算法、数据结构和面试问题及其解决方案的集合。
* [LeetCode](https://github.com/gouthampradhan/leetcode)：针对一些常见LeetCode面试问题的解决方案。
* [Data Structures](https://github.com/williamfiset/DEPRECATED-data-structures)：强大的数据结构集合。
* [LeetCode Solutions In Good Style](https://github.com/liweiwei1419/LeetCode-Solutions-in-Good-Style) ：这是一个《算法与数据结构》的入门级教程，适用于算法零基础的小白。
* [Algorithms](https://github.com/pedrovgs/Algorithms)：用Java编写的一些常见算法问题的解决方案。
* [Play With Data Structures](https://github.com/liuyubobobo/Play-with-Data-Structures)：慕课网上的课程《Java语言玩转数据结构》示例代码。
* [Algorithms Sedgewick](https://github.com/aistrate/AlgorithmsSedgewick)：《算法》(第四版)一书的代码。
* [Algorithms](https://github.com/jimmysuncpt/Algorithms)：这个项目包含《算法(第4版)》书中的代码和对部分课后练习的解答。
* [Algorithms](https://github.com/reneargento/algorithms-sedgewick-wayne)：《算法(第4版)》一书的练习题答案。
* [DataMining Algorithm](https://github.com/linyiqun/DataMiningAlgorithm)：数据挖掘18大算法实现以及其他相关经典DM算法。
* [HackerRank Solutions](https://github.com/RodneyShag/HackerRank_solutions)：317个HackerRank问题的有效解决方案。

#### 软件工程教程

* [Java Design Patterns](https://github.com/iluwatar/java-design-patterns)：用Java实现的设计模式教程。
* [ITStack Demo Design](https://github.com/fuzhengwei/itstack-demo-design)：《重学Java设计模式》是一本互联网真实案例实践书籍。
* [Awesome Architect](https://github.com/xingshaocheng/architect-awesome)：后端架构师技术图谱。
* [DesignPattern](https://github.com/youlookwhat/DesignPattern)：Java 23种设计模式全归纳。
* [IDDD Samples](https://github.com/VaughnVernon/IDDD_Samples)：这是Vaughn Vernon所著的《实现领域驱动设计》一书中的限界上下文示例。
* [Migration](https://github.com/phodal/migration)：《系统重构与迁移指南》手把手教你分析、评估现有系统、制定重构策略、探索可行重构方案、搭建测试防护网、进行系统架构重构、服务架构重构、模块重构、代码重构、数据库重构、重构后的架构守护。
* [FTGO Example Application](https://github.com/microservices-patterns/ftgo-application)：这是《微服务模式》一书的示例代码。
* [Head First Design Patterns](https://github.com/bethrobson/Head-First-Design-Patterns)：《Head First设计模式》一书代码。
* [Buckpal](https://github.com/thombergs/buckpal)：这个仓库以六角形架构风格实现了一个小型Web应用程序。
* [Three High Import](https://github.com/qiurunze123/threadandjuc)：高并发、高可靠、高性能导入系统-高并发多线程进阶。

#### 其他技术教程

* [RabbitMQ Tutorials](https://github.com/rabbitmq/rabbitmq-tutorials)：该项目包含RabbitMQ教程的代码及其对各种语言的移植。
* [API Samples](https://github.com/youtube/api-samples)：YouTube API的代码示例，包括YouTube Data API、YouTube Analytics API和YouTube Live Streaming API。
* [RxJava Samples](https://github.com/rengwuxian/RxJavaSamples)：RxJava 2和Retrofit结合使用的几个最常见使用方式举例。
* [Vert.x 4 Examples](https://github.com/vert-x3/vertx-examples)：该仓库包含一系列Vert.x 4示例。
* [RxJava2 Examples](https://github.com/nanchen2251/RxJava2Examples)：从RxJava 1跳到RxJava 2(学习RxJava2 )的例子Demo。
* [Intro to RxJava](https://github.com/Froussios/Intro-To-RxJava)：本指南旨在向初学者响应式程序员介绍RxJava实现JVM响应式编程的完整功能。
* [Java EE 7 Samples](https://github.com/javaee-samples/javaee7-samples)：该仓库由Java EE 7示例和单元测试组成。
* [AWS Lambda Developer Guide](https://github.com/awsdocs/aws-lambda-developer-guide)：该仓库包含AWS Lambda开发人员指南的其他资源。
* [Netty Learning Example](https://github.com/sanshengshui/netty-learning-example)：Netty实践学习案例。
* [Netty in Action](https://github.com/normanmaurer/netty-in-action)：该仓库包含《Netty in Action》一书所有章节的源代码。
* [Netty Demos](https://github.com/waylau/netty-4-user-guide-demos)：《Netty 4.x用户指南》/《Netty原理解析与开发实战》文中用到的例子源码。
* [MyBatis-Plus Samples](https://github.com/baomidou/mybatis-plus-samples)：MyBatis Plus示例代码。
* [Kafka Streams Examples](https://github.com/confluentinc/kafka-streams-examples)：该项目包含演示如何使用Apache
  Kafka的Streams API实现实时应用程序和事件驱动的微服务的代码示例。
* [MIT Deep Learning Book](https://github.com/janishar/mit-deep-learning-book-pdf)：麻省理工学院深度学习书籍PDF格式。
* [Deeplearning4J Examples](https://github.com/deeplearning4j/deeplearning4j-examples)：Deeplearning4j示例(DL4J、DL4J Spark、DataVec)。
* [DeepLearning](https://github.com/yusugomori/DeepLearning)：深度学习教程(Python、C、C++、Java、Scala、Go)。
* [Quarkus QuickStarts](https://github.com/quarkusio/quarkus-quickstarts)：该仓库包含一组Quarkus框架的快速入门。
* [LWJGL Basics](https://github.com/mattdesl/lwjgl-basics)：LibGDX/LWJGL教程和示例。
* [DevOps For Beginners](https://github.com/in28minutes/devops-master-class)：DevOps初学者教程-学习Docker、Kubernetes、Terraform、Ansible、Jenkins和Azure Devops。
* [Ninety-Nine Problems](https://github.com/shekhargulati/99-problems)：这是瑞士伯尔尼伯尔尼应用科学大学Werner Hett撰写的《九十九个Prolog问题》的改编版。

#### 秒杀系统

* [Miaosha](https://github.com/qiurunze123/miaosha)：秒杀系统设计与实现，互联网工程师进阶与分析。
* [Spring Boot Seckill](https://github.com/zaiyunduan123/springboot-seckill)：基于Spring Boot + MySQL + Redis + RabbitMQ + Guava开发的高并发商品限时秒杀系统。
* [Seckill](https://github.com/codingXiaxw/seckill)：Java高并发秒杀系统API。

#### 源码分析

* [Source Code Hunter](https://github.com/doocs/source-code-hunter)：从源码层面，剖析挖掘互联网行业主流技术的底层实现原理。
* [Spring Analysis](https://github.com/seaswalker/spring-analysis)：Spring相关组件阅读笔记。
* [Mybatis](https://github.com/tuguangquan/mybatis)：Mybatis源码中文注释。
* [Spring Reading](https://github.com/xuchengsheng/spring-reading)：涵盖了Spring框架的核心概念和关键功能，包括控制反转容器的使用，面向切面编程的原理与实践。
* [IoT Technical Guide](https://github.com/IoT-Technology/IoT-Technical-Guide)：从零搭建高性能物联网平台及物联网解决方案和Thingsboard源码分析。
* [Java Source Code Learning](https://github.com/coderbruis/JavaSourceCodeLearning)：Java流行框架源码分析，学习以及总结。
* [Netty Learning](https://github.com/code4craft/netty-learning)：本系列文章是Netty的源码导读。
* [JDK Source Code Analysis](https://github.com/seaswalker/jdk-sourcecode-analysis)：JDK源码阅读笔记。

#### 面试宝典

* [Interviews](https://github.com/kdn251/interviews)：你的软件工程技术面试个人指南。
* [ToBeTopJavaer](https://github.com/hollischuang/toBeTopJavaer)：一份Java面试宝典。
* [Tech Interview](https://github.com/gyoogle/tech-interview-for-developer)：新开发者主要知识技能面试百科。
* [DSA Bootcamp Java](https://github.com/kunal-kushwaha/DSA-Bootcamp-Java)
  ：该仓库包含WeMakeDevs的Java数据结构和算法+面试准备训练营的代码示例、作业和注释。
* [CtCI-6th-Edition](https://github.com/careercup/CtCI-6th-Edition)：破解编码面试第六版的解决方案。
* [Interview](https://github.com/mission-peace/interview)：面试问题。
* [FullStack Tutorial](https://github.com/frank-lam/fullstack-tutorial)：后台技术栈/架构师之路/全栈开发社区，春招/秋招/校招/面试。
* [Java Eight Part](https://github.com/CoderLeixiaoshuai/java-eight-part)：Java八股文仓库。
* [SDE-Interview-Questions](https://github.com/twowaits/SDE-Interview-Questions)
  ：从Geeksforgeeks、CareerCup和Glassdoor中抓取的最全面的技术面试问题列表。
* [Learning Note](https://github.com/rbmonster/learning-note)：Java开发及面试(个人面试、工作总结、资料收集站)。
* [Interview Guide](https://github.com/NotFound9/interviewGuide)：包括Java基础、JVM、数据库、MySQL、Redis、计算机网络、算法、数据结构、操作系统、设计模式、系统设计、框架原理。
* [Internet Architect](https://github.com/bjmashibing/InternetArchitect)：互联网架构师课程文档及源码。
* [Java Notes](https://github.com/DreamCats/java-notes)：秋招经历、牛客面经问题按照频率总结、Java系列知识、数据库、分布式、微服务、前端、技术面试、每日文章等。
* [System Design Interview](https://github.com/DreamOfTheRedChamber/system-design-interviews)：系统设计面试。