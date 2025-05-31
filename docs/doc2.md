## 测试

这里主要是一些测试框架和工具库，包括单元测试、集成测试、性能测试、断言库、Mock框架等。

#### 单元测试

* [JUnit 4](https://github.com/junit-team/junit4)：JUnit是一个用于编写可重复测试的简单框架。
* [JUnit 5](https://github.com/junit-team/junit5)：JUnit 5是Java单元测试框架的最新版本，相较于JUnit 4，它引入了许多新的特性和改进。
* [TestNG](https://github.com/testng-team/testng)：TestNG是一个受JUnit启发的测试框架，但引入了一些新功能，使其更强大且更易于使用。
* [Spock](https://github.com/spockframework/spock)：Spock是一个用于Java和Groovy应用程序的BDD风格的开发人员测试和规范框架。
* [Kotest](https://github.com/kotest/kotest)：Kotest是一个灵活且全面的Kotlin测试工具，具有多平台支持。
* [uTest](https://github.com/com-lihaoyi/utest)：uTest是一个简单、直观的Scala测试库。

#### 集成测试

* [Testcontainers](https://github.com/testcontainers/testcontainers-java)：Testcontainers是一个支持JUnit测试的Java库，提供通用数据库、Selenium Web浏览器或任何其他可以在Docker容器中运行的东西的轻量级一次性实例。
* [MicroShed](https://github.com/MicroShed/microshed-testing)：MicroShed Test提供了一种快速、简单的方法来为Java微服务应用程序编写和运行真正的生产集成测试。
* [Embedded Kafka](https://github.com/embeddedkafka/embedded-kafka)：提供内存中的Kafka实例来运行测试的库。
* [Embedded Redis](https://github.com/kstyrc/embedded-redis)：用于Java集成测试的Redis嵌入式服务器。
* [Redis Mock](https://github.com/microwww/redis-mock)：纯Java实现的Redis Server，单元测试时嵌入Redis服务。
* [Embedded PostgreSQL](https://github.com/opentable/otj-pg-embedded)：允许使用Docker容器将PostgreSQL嵌入到Java应用程序代码中。
* [Embedded LDAP JUnit](https://github.com/zapodot/embedded-ldap-junit)：用于在JUnit测试中运行嵌入式LDAP服务器的JUnit Rule。
* [Embedded MySQL](https://github.com/wix-incubator/wix-embedded-mysql)：用于测试的嵌入式MySQL。
* [Embedded MongoDB](https://github.com/flapdoodle-oss/de.flapdoodle.embed.mongo)：Embedded MongoDB提供一种平台中立的方式在单元测试中运行MongoDB。
* [Embedded Postgres Binaries](https://github.com/zonkyio/embedded-postgres-binaries)：该项目提供了PostgreSQL二进制文件的轻量级捆绑包，大小更小，旨在用于测试目的。
* [Embedded Database](https://github.com/zonkyio/embedded-database-spring-test)：用于为Spring支持的集成测试创建隔离的嵌入式数据库的库。
* [Embedded ElasticSearch](https://github.com/allegro/embedded-elasticsearch)：简化使用Elasticsearch创建集成测试的工具，由Allegro开源。
* [Embedded Consul](https://github.com/pszymczyk/embedded-consul)：Embedded Consul提供了在集成测试中运行Consul的简单方法。
* [DbFit](https://github.com/dbfit/dbfit)：DbFit是一个数据库测试框架，支持对数据库代码进行简单的测试驱动开发。
* [DbUnit](https://www.dbunit.org/)：DbUnit是一个JUnit扩展，针对数据库驱动的项目。
* [DbSetup](https://github.com/Ninja-Squad/DbSetup)：DbSetup允许在执行自动化集成测试(通常是DAO/Repository自动化测试)之前填充数据库。
* [Kafka JUnit](https://github.com/salesforce/kafka-junit)：该库包装了Kafka的嵌入式测试集群，使你可以更轻松地使用JUnit针对在测试上下文中运行的真实kafka服务器创建和运行集成测试，由Salesforce开源。
* [Database Rider](https://github.com/database-rider/database-rider)：Database Rider集成了DBUnit和JUnit，使数据库测试变得轻而易举。
* [ElasticSearch Test](https://github.com/tlrx/elasticsearch-test)：一个让ElasticSearch单元测试变得轻而易举的框架。
* [LDAP Server](https://github.com/intoolswetrust/ldap-server)：用于测试目的的简单内存LDAP服务器。
* [Fongo](https://github.com/fakemongo/fongo)：Fongo是MongoDB的内存中Java实现，主要用途是轻量级集成测试，由Foursquare开源。
* [MongoDB Java Server](https://github.com/bwaldvogel/mongo-java-server)：MongoDB Java Server是用Java伪造核心MongoDB服务器的实现，可用于集成测试。
* [Testcontainers Spring Boot](https://github.com/PlaytikaOSS/testcontainers-spring-boot)：基于Spring Boot的集成测试的容器自动配置。
* [Embedded Process Util](https://github.com/flapdoodle-oss/de.flapdoodle.embed.process)：Embedded Process Util为在单元测试中运行进程提供一种平台中立的方式。
* [Embedded PostgreSQL Server](https://github.com/yandex-qatools/postgresql-embedded)：嵌入式PostgreSQL服务器提供了一种平台中立的方式来在单元测试中运行Postgres二进制文件。
* [Embedded Cassandra](https://github.com/nosan/embedded-cassandra)：Embedded Cassandra提供了一种启动和停止Cassandra的简单方法。
* [CassandraUnit](https://github.com/jsevellec/cassandra-unit)：CassandraUnit是一个Java测试工具，它可以用于测试使用Cassandra数据库后端创建的Java应用程序。
* [Alternator](https://github.com/mboudreau/Alternator)：用于测试目的在本地运行的模拟DynamoDB。
* [Keycloak Testcontainer](https://github.com/dasniko/testcontainers-keycloak)：Keycloak SSO的Testcontainers实现。
* [MongoUnit](https://github.com/mongounit/mongounit)：MongoUnit是一个数据驱动的集成测试框架，适用于使用MongoDB进行持久化的基于Spring Boot的应用程序。
* [Arquillian](https://github.com/arquillian/arquillian-core)：Arquillian是一个创新且高度可扩展的JVM测试平台，使开发人员能够轻松地为Java中间件创建自动化集成、功能和验收测试，由RedHat开源。
* [Kindcontainer](https://github.com/dajudge/kindcontainer)：基于Java的Testcontainers容器实现，为集成测试提供临时Kubernetes集群。
* [Embedded DB JUnit](https://github.com/zapodot/embedded-db-junit)：提供内存数据库的JUnit Rule(支持H2和HyperSQL)。
* [Nats Server Embedded](https://github.com/YunaBraska/nats-server-embedded)：用于测试的Nats服务器。
* [PelicanDT](https://github.com/alibaba/PelicanDT)：PelicanDT是阿里云提供的一款Linux系统远程控制利器，是主要针对分布式应用提供的集成测试解决方案，用于帮助开发者简单、高效地测试分布式应用。

#### 接口测试

* [Rest Assured](https://github.com/rest-assured/rest-assured)：Rest Assured是用于轻松测试REST服务的Java DSL。
* [Wisdom](https://github.com/wisdom-projects/rest-client)：Wisdom可以自动化测试REST API并生成精美的测试报告，同时基于测试过的历史数据，可以生成精美的REST API文档。
* [Milkman](https://github.com/warmuuh/milkman)：Postman的可扩展替代方案，用于制作各种请求，不仅适用于gRPC，还适用于HTTP、SQL等。
* [CATS](https://github.com/Endava/cats)：CATS是一个REST API模糊器和OpenAPI端点的负面测试工具，由Endava开源。
* [Everest](https://github.com/RohitAwate/Everest)：Everest是一个用JavaFX编写的REST API测试客户端。
* [ACTS](https://github.com/sofastack/sofa-acts)：ACTS是一个基于数据模型驱动的白盒测试框架，由蚂蚁开源。
* [HybridTestFramework](https://github.com/dipjyotimetia/HybridTestFramework)：HybridTestFramework是一个全面而多功能的测试框架，旨在涵盖软件测试的各个方面。
* [RESTClient](https://github.com/wiztools/rest-client)：RESTClient是一个用于测试RESTful Web服务的Java应用程序，它可用于测试各种HTTP通信。
* [Rest Driver](https://github.com/rest-driver/rest-driver)：用于测试RESTful服务和客户端的工具。
* [Hikaku](https://github.com/codecentric/hikaku)：Hikaku可以测试REST-API实现是否满足其规范。
* [Cukes](https://github.com/ctco/cukes)：用于测试RESTful Web服务的Cucumber DSL。
* [Citrus](https://github.com/citrusframework/citrus)：Citrus是一个用Java编写的测试框架，能够为企业SOA应用程序创建完全自动化的端到端用例测试，由RedHat开发。
* [ChocoTea](https://github.com/cleopatra27/chocotea)：Chocotea是一个从Java代码生成Postman集合、环境和集成测试的库。
* [Heat](https://github.com/ExpediaGroup/heat)：Heat是一个基于REST Assured框架的简单解决方案，由Expedia开源。
* [iTest](https://gitee.com/itestwork/itest)：iTest包含任务管理、测试管理、缺陷管理、测试环境管理、接口测试、接口Mock、压力测试。
* [Hrun4j](https://github.com/lematechvip/hrun4j)：Hrun4j是由乐马技术推出的开源一站式接口测试解决方案。

#### 端到端测试

* [Maestro](https://github.com/mobile-dev-inc/Maestro)：Maestro是最简单、最强大、最值得信赖的移动和Web应用程序端到端测试平台。
* [Webtau](https://github.com/testingisdocumenting/webtau)：WebTau是一个测试API、命令行工具和一个用于编写单元、集成和端到端测试的框架。
* [Stove](https://github.com/Trendyol/stove)：Stove是一个端到端测试框架，可以将物理依赖项和你的应用程序一起启动，由Trendyol开源。
* [Testlum](https://github.com/TestlumFramework/Testlum)：无代码端到端测试框架，让你的测试更加轻松。
* [Sakuli](https://github.com/ConSol/sakuli)：Sakuli是一款端到端测试和监控工具，适用于具有多个监控集成的网站和常见UI。

#### 功能测试

* [SoapUI](https://github.com/SmartBear/soapui)：SoapUI是一个免费、开源的跨平台API和Web Service功能测试解决方案。
* [Galen](https://github.com/galenframework/galen)：Galen是一个开源工具，用于测试Web应用程序的布局和响应式设计，它也是一个强大的功能测试框架。
* [Markov](https://github.com/alibaba/intelligent-test-platform)：Markov是阿里开源的新一代功能测试平台，包含可视化用例编写管理、分布式的沙盒环境和测试数据构建、测试流程pipeline管理等优点。
* [Acai](https://github.com/google/acai)：Acai使你可以轻松地使用JUnit 4和Guice编写应用程序的功能测试，由Google开源。

#### 突变测试

* [Pitest](https://github.com/hcoles/pitest)：Pitest是最先进的Java和JVM突变测试系统。
* [Major](https://mutation-testing.org/)：Major是一个高效、灵活的突变分析框架。
* [Descartes](https://github.com/STAMP-project/pitest-descartes)：Descartes通过报告所覆盖代码中的弱点来支持开发人员改进他们的测试套件。
* [Stryker4s](https://github.com/stryker-mutator/stryker4s)：Stryker4s是一个Scala的突变测试框架，它允许你通过临时插入错误来测试你的测试。
* [Judy](http://mutationtesting.org/judy2/)：Judy是一个用Java编写的突变测试器，由弗罗茨瓦夫理工大学开源。
* [Code Defenders](https://github.com/CodeDefenders/CodeDefenders)：突变测试的游戏化，由帕绍大学和莱斯特大学开发。
* [PG Index Health](https://github.com/mfvanek/pg-index-health)：PG Index Health是一个用于分析和维护PostgreSQL数据库中索引和表健康状况的Java库。

#### 模糊测试

* [SQLancer](https://github.com/sqlancer/sqlancer)：SQLancer是一个自动测试DBMS以发现其实现中的逻辑错误的工具。
* [Javafuzz](https://github.com/fuzzitdev/javafuzz)：Javafuzz是用于测试Java包的覆盖率引导模糊器。
* [JQF](https://github.com/rohanpadhye/JQF)：JQF是一个针对Java的反馈导向模糊测试平台。
* [Mu2](https://github.com/cmu-pasta/mu2)：Mu2是一个用于突变引导模糊测试的模糊测试平台，构建在用于模糊Java程序的JQF平台之上，由CMU程序分析、软件测试和应用实验室开发。
* [EvoMaster](https://github.com/WebFuzzing/EvoMaster)：EvoMaster是第一个开源AI驱动工具，可为Web/企业应用程序自动生成系统级测试用例。
* [Jazzer](https://github.com/CodeIntelligenceTesting/jazzer)：由Code Intelligence开发的适用于JVM平台的覆盖率引导的进程内模糊器，它基于libFuzzer，并将许多由仪器驱动的突变功能引入JVM。
* [Snodge](https://github.com/npryce/snodge)：一个小型、可扩展的Kotlin库，用于随机变异JSON和XML文档、文本和二进制数据，适用于模糊测试。

#### 性能测试

* [Apache JMeter](https://github.com/apache/jmeter)：JMeter开源负载测试工具，用于分析和测量各种服务的性能。
* [NGrinder](https://github.com/naver/ngrinder)：NGrinder是一个压力测试平台，使你能够同时执行脚本创建、测试执行、监控和结果报告生成器，Naver开源。
* [Gatling](https://github.com/gatling/gatling)：Gatling是一个负载测试工具，它正式支持HTTP、WebSocket、Server-Sent-Events和JMS。
* [Grinder](https://grinder.sourceforge.net/)：Grinder是一个Java负载测试框架，可以使用许多负载注入器轻松运行分布式测试。
* [NeoLoad](https://www.tricentis.com/neoload-quick-links)：NeoLoad是一个自动化性能测试平台，适用于企业组织从API到应用程序的持续测试。
* [JMH](https://github.com/openjdk/jmh)：JMH是一个Java工具，用于构建、运行和分析用Java和其他针对JVM的语言编写的宏观基准测试，Oracle开源。
* [Lago](https://github.com/twitter-archive/iago)：Lago是一种负载生成工具，可针对给定目标重放生产或合成流量，由Twitter开源。
* [Perfidix](https://github.com/sebastiangraf/perfidix)：Perfidix是一个轻量级Java库，使用户能够对源代码进行基准测试，由康斯坦茨大学开源。
* [PerfCake](https://github.com/PerfCake/PerfCake)：轻量级性能测试框架和负载生成器。
* [Caliper](https://github.com/google/caliper)：Caliper是一个用于测量Java代码性能的工具，主要侧重于微基准测试，由Google开源。
* [Criterium](https://github.com/hugoduncan/criterium)：使用Clojure编写的用于JVM的基准测试库。
* [ScalaMeter](https://github.com/scalameter/scalameter)：适用于JVM平台的微基准测试和性能回归测试框架，ScalaMeter可以自动测量和收集程序的各种指标，然后生成漂亮的报告，或存储你的数据。
* [RabbitMQ Performance Testing Tool](https://github.com/rabbitmq/rabbitmq-perf-test)：RabbitMQ性能测试工具，由Broadcom的RabbitMQ团队维护。
* [JLBH](https://github.com/OpenHFT/JLBH)：JLBH是一个可让你对在上下文中运行的代码进行基准测试(而不是在微基准测试中)的工具。
* [KoPeMe](https://github.com/DaGeRe/KoPeMe)：KoPeMe是一个用于在Java中进行性能测试的框架。
* [Hyperfoil](https://github.com/Hyperfoil/Hyperfoil)：Hyperfoil是面向微服务的分布式基准测试框架，解决了协调遗漏谬误。
* [JUnitPerf](https://github.com/houbb/junitperf)：JUnitPerf是一款为Java开发者设计的性能测试框架。
* [JPerf](https://github.com/AgilData/jperf)：JPerf是一个简单的Java性能和可扩展性测试框架。
* [kraken](https://github.com/OctoPerf/kraken)：Kraken是一个基于Gatling的负载测试IDE。
* [XLT](https://github.com/Xceptance/XLT)：XLT是由Xceptance开发和维护的广泛负载和性能测试工具。
* [Intuit Tank](https://github.com/intuit/Tank)：Intuit Tank是一个在云环境中运行的负载测试平台，它目前支持Amazon Web界面并利用EC2、S3、CloudWatch(日志/指标)的服务。
* [JUnitPerf](https://github.com/noconnor/JUnitPerf)：使用JUnit构建的API性能测试框架。

#### 属性测试

* [JUnit Quickcheck](https://github.com/pholser/junit-quickcheck)：JUnit Quickcheck是一个支持在JUnit中编写和运行基于属性的测试的库。
* [Jqwik](https://github.com/jqwik-team/jqwik)：Jqwik的主要目的是将基于属性的测试引入JVM。
* [ScalaCheck](https://github.com/typelevel/scalacheck)：ScalaCheck是一个用Scala编写的库，用于对Scala或Java程序进行基于属性的自动化测试。
* [QuickTheories](https://github.com/quicktheories/QuickTheories)：Java 8基于属性的测试。
* [JetCheck](https://github.com/JetBrains/jetCheck)：JetCheck是一个基于属性的Java 8+测试库，由JetBrains开源。
* [QuickPerf](https://github.com/quick-perf/quickperf)：QuickPerf是Java的一个测试库，用于快速评估和改进一些与性能相关的属性。
* [ScalaProps](https://github.com/scalaprops/scalaprops)：Scala基于属性的测试库。
* [Hypothesis-Java](https://github.com/HypothesisWorks/hypothesis-java)：Hypothesis是一个专为主流语言设计的基于属性的现代测试系统。

#### A/B测试

* [Wasabi](https://github.com/intuit/wasabi)：Wasabi A/B测试服务是一个实时、企业级、100% API驱动的项目，由Intuit开发。
* [Proctor](https://github.com/indeedeng/proctor)：Proctor是一个用Java编写的A/B测试框架，由Indeed开发并大量使用。
* [Sixpack Java](https://github.com/sixpack/sixpack-java)：Sixpack A/B测试框架的Java客户端。
* [PlanOut4J](https://github.com/Glassdoor/planout4j)：PlanOut4J是Facebook PlanOut的基于Java的实现，PlanOut是一个A/B测试框架，旨在在网络上进行大规模实验。
* [Izanami](https://github.com/MAIF/izanami)：Izanami是一款共享配置、功能翻转和A/B测试服务器，非常适合微服务架构实现。
* [Apache Unomi](https://github.com/apache/unomi)：Unomi存储用户个人资料信息，主要用于为A/B测试和个性化提供后端服务器，由Jahia开源。

#### 验收测试

* [FitNesse](https://github.com/unclebob/fitnesse)：FitNesse是一个完全集成的独立验收测试框架和wiki，由Uncle Bob开发。
* [Thucydides](https://github.com/thucydides-webtests/thucydides)：Thucydides是一个旨在使编写自动化验收测试变得更容易、更有趣的库。
* [Gwen](https://github.com/shazam/gwen)：Gwen是一个允许使用Give-When-Then语法编写验收测试的简单库。
* [JLineup](https://github.com/otto-de/jlineup)：JLineup是一个对于网页的自动视觉回归测试非常有用的工具，特别是在持续交付管道中，它可以用作简单的命令行工具或通过REST API控制的小型Web服务。
* [Simple-DSL](https://github.com/LMAX-Exchange/Simple-DSL)：Simple-DSL是LMAX Exchange使用的一种编写验收测试的风格，旨在平衡人类和机器的可读性。
* [RestFixture](https://github.com/smartrics/RestFixture)：RestFixture是一个FitNesse测试夹具，允许开发人员和/或产品所有者以简单的方式为REST服务编写测试夹具。
* [JWebUnit](https://github.com/JWebUnit/jwebunit)：JWebUnit是一个Java框架，有助于为Web应用程序创建验收测试。
* [Kensa](https://github.com/kensa-dev/kensa)：Kensa是一个验收测试工具，支持Java、Kotlin和JUnit 5。

#### 回归测试

* [ARA](https://github.com/Decathlon/ara)：ARA可以预先分析你的非回归测试运行、跟踪和跟踪问题、保留其历史记录，甚至在质量不满足的情况下破坏你的CI构建，从而帮助你对抗回归，由迪卡侬开源。
* [AREX](https://github.com/arextest/arex-agent-java)：Arex是一个围绕利用现实世界数据(即数据库记录、服务负载、缓存项等)进行回归测试的非常简单的原则设计的框架。
* [NoraUi](https://github.com/NoraUi/NoraUi)：用户界面的非回归自动化。
* [Diffy](https://github.com/opendiffy/diffy)：Diffy使用并排运行新代码和旧代码的实例来发现服务中的潜在错误，由Twitter开源。
* [Gojira](https://github.com/flipkart-incubator/gojira)：Gojira是一个基于记录和回放的回归测试工具，由Flipkart开源。
* [Unlogged Java SDK](https://github.com/unloggedio/unlogged-sdk)：Unlogged Java SDK支持以二进制格式记录代码执行。
* [Drill4J](https://github.com/Drill4J/drill4j)：Drill4J是一款开源工具，用于识别测试差距并减少回归测试所花费的时间。

#### 契约测试

* [Pact](https://github.com/pact-foundation/pact-jvm)：Pact的JVM版本，用于编写消费者驱动的契约测试。
* [Spring Cloud Contract](https://github.com/spring-cloud/spring-cloud-contract)：Spring对消费者驱动契约的支持。
* [Stubby4j](https://github.com/azagniotov/stubby4j)：HTTP/1.1、HTTP/2和WebSocket存根服务器，用于在Docker和非容器化环境中存根分布式Web服务以进行契约测试。
* [Contract Test Runner](https://github.com/wso2/contract-test-runner)：用于契约测试的Java库，由WSO2开源。
* [ContractCase Contract Testing Framework](https://github.com/case-contract-testing/java-dsl)：这是ContractCase契约测试框架的Java绑定。
* [Specmatic](https://github.com/znsio/specmatic)：Specmatic通过利用API规范作为可执行合约来体现契约驱动开发(CDD)。
* [Cofoja](https://github.com/nhatminhle/cofoja)：Cofoja是一种Java契约编程框架和测试工具，它使用注解处理和字节码检测来提供运行时检查。

#### 渗透测试

* [Cobalt Strike](https://www.cobaltstrike.com/)：Cobalt Strike是一款基于Java的渗透测试神器。
* [TrackRay](https://github.com/iSafeBlue/TrackRay)：溯光是一个开源渗透测试框架，框架自身实现了漏洞扫描功能，集成了知名安全工具：Metasploit、Nmap、Sqlmap、AWVS、Burpsuite等。
* [Jackhammer](https://github.com/olacabs/jackhammer)：Jackhammer是一款协作工具，旨在弥合安全团队与开发团队、QA团队之间的差距，并成为TPM的促进者，以了解和跟踪投入生产的代码的质量。
* [WS-Attacker](https://github.com/RUB-NDS/WS-Attacker)：WS-Attacker是一个用于Web服务渗透测试的模块化框架，由波鸿鲁尔大学开发。

#### 快照测试

* [ApprovalTests](https://github.com/approvals/ApprovalTests.Java)：Java的ApprovalTest验证库。
* [Java Snapshot Testing](https://github.com/origin-energy/java-snapshot-testing)：Java测试的Facebook风格快照测试。
* [Selfie](https://github.com/diffplug/selfie)：快照测试是记录和指定系统及其组件行为的最快且最精确的机制。

#### 断言库

* [AssertJ](https://github.com/assertj/assertj)：AssertJ提供了一组丰富且直观的强类型断言，用于单元测试。
* [AssertJ Android](https://github.com/square/assertj-android)：一组用于测试Android的AssertJ断言，由Square开源。
* [JSONAssert](https://github.com/skyscreamer/JSONassert)：用更少的代码编写JSON单元测试，非常适合测试REST接口。
* [Truth](https://github.com/google/truth)：Google出品的流式断言库。
* [Hamcrest](https://github.com/hamcrest/JavaHamcrest)：Hamcrest是一个匹配器库，可以将其组合起来以在测试中创建灵活的意图表达。
* [Spotify Hamcrest](https://github.com/spotify/java-hamcrest)：这是一个用有用的匹配器扩展Hamcrest匹配库的库集合，由Spotify开源。
* [BeanMatcher](https://github.com/orien/bean-matchers)：用于测试Java Bean的Hamcrest匹配器。
* [Deepdive](https://github.com/jdlib/deepdive)：Deepdive是Java的流式断言库。
* [Fest](https://github.com/alexruiz/fest-assert-2.x)：FEST-Assert为断言提供了流式的接口。
* [Expekt](https://github.com/winterbe/expekt): Kotlin的BDD断言库。
* [AssertJ-DB](https://github.com/assertj/assertj-db)：Expekt是一个Kotlin BDD断言库，受到Chai.js的启发。
* [JFRUnit](https://github.com/moditect/jfrunit)：用于断言JFR事件的JUnit扩展。
* [ArchUnit](https://github.com/TNG/ArchUnit)：ArchUnit是一个免费、简单且可扩展的库，用于检查Java代码的架构。
* [Confidence](https://github.com/saynotobugsorg/confidence)：Confidence是一个声明式Java断言框架。
* [Power Assert](https://github.com/jkschneider/java-power-assert)：Power Assert通过条件评估过程中产生的值的信息来增强断言失败，并以易于理解的形式呈现它们。
* [Visible Assertions](https://github.com/rnorth/visible-assertions)：JUnit断言的替代方案，为你的测试提供更有洞察力的日志叙述。
* [LogCapture](https://github.com/jsalinaspolo/logcapture)：LogCapture是一个用于断言日志消息的测试库。
* [Log Capture](https://github.com/dm-drogeriemarkt/log-capture)：日志消息的简单断言。
* [Atrium](https://github.com/robstoll/atrium)：Atrium是一个针对Kotlin的开源多平台期望/断言库，支持JVM、JS和Android。
* [ModelAssert](https://github.com/webcompere/model-assert)：用于模型数据的断言库。
* [Valid4j](https://github.com/valid4j/valid4j)：Java的简单断言和验证库。
* [DataSource-Assert](https://github.com/ttddyy/datasource-assert)：DataSource-Assert为DataSource提供断言API以验证查询执行。
* [Strikt](https://github.com/robfletcher/strikt)：Strikt是Kotlin的断言库，旨在与JUnit、Minutest、Spek或KotlinTest等测试运行器一起使用。
* [NDD Check4J](https://gitlab.com/ndd-oss/java/ndd-check4j)：NDD Check4J通过流式或简洁的API提供简单的参数检查。
* [LambSpec](https://github.com/pholser/lambspec)：适用于Java 8的类似RSpec的断言库。
* [Hamcrest-JSON](https://github.com/hertzsprung/hamcrest-json)：用于比较JSON文档的Hamcrest匹配器。
* [Jcabi-Matchers](https://github.com/jcabi/jcabi-matchers)：一些方便的Hamcrest匹配器，主要用于XPath与XHTML以及JAXB中的字段。
* [Shazamcrest](https://github.com/shazam/shazamcrest)：适用于自动化测试的可重复使用的Hamcrest匹配器。
* [ReCheck](https://github.com/retest/recheck)：ReCheck是一个功能齐全的开源测试工具，允许替换手动断言并立即检查所有内容。

#### Mock框架

* [Mockito](https://github.com/mockito/mockito)：Mockito是Java中最流行的单元测试Mock框架。
* [PowerMock](https://github.com/powermock/powermock)：PowerMock是一个框架，它扩展了EasyMock等其他Mock库，使其具有更强大的功能。
* [TestableMock](https://github.com/alibaba/testable-mock)：TestableMock是一款轻量Mock工具，由阿里开源。
* [EasyMock](https://github.com/easymock/easymock)：EasyMock是一个Java库，它提供了一种在单元测试中使用Mock对象的简单方法。
* [Mockk](https://github.com/mockk/mockk)：用于Kotlin的Mock框架。
* [JMock](https://github.com/jmock-developers/jmock-library)：JMock是一个支持使用Mock对象进行Java代码测试驱动开发的库。
* [ScalaMock](https://github.com/paulbutcher/ScalaMock)：原生Scala Mock框架。
* [JMockit](https://github.com/jmockit/jmockit1)：JMockit是用于集成测试、Mock、伪造和代码覆盖率的高级Java库。
* [Mokkery](https://github.com/lupuuss/Mokkery)：Kotlin Multiplatform的Mock库，易于使用、无样板且由编译器插件驱动。
* [Mockative](https://github.com/mockative/mockative)：使用KSP实现Kotlin/Native和Kotlin Multiplatform的Mock。
* [Mockrunner](https://github.com/mockrunner/mockrunner)：Mockrunner是用于企业级应用程序的Mock工具。

#### 接口Mock

* [Moco](https://github.com/dreamhead/moco)：Moco是一个易于设置的存根框架。
* [WireMock](https://github.com/wiremock/wiremock)：WireMock是一种流行的API Mock测试开源工具。
* [Microcks](https://github.com/microcks/microcks)：Microcks是用于API Mock和测试的开源云原生工具。
* [MockServer](https://github.com/mock-server/mockserver)：MockServer可以轻松模拟通过HTTP或HTTPS与用Java、JavaScript和Ruby编写的客户端集成的任何系统。
* [MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver)：用于测试HTTP客户端的可编写脚本的Web服务器，由Square开源。
* [AnyMock](https://github.com/duxiaoman/AnyMock)：AnyMock是一个通用接口Mock平台，提供Mock配置和模拟响应的服务，由度小满开源。
* [Imposter](https://github.com/outofcoffee/imposter)：Imposter是REST API、OpenAPI规范、SOAP Web Services、Salesforce和HBase API的Mock服务器。
* [Flashback](https://github.com/linkedin/flashback)：Flashback旨在模拟HTTP和HTTPS资源(例如Web服务和REST API)以用于测试目的，由LinkedIn开源。
* [Mockey](https://github.com/clafonta/Mockey)：Mockey是一个用于测试通过HTTP的应用程序交互的工具，重点是测试Web服务，特别是使用XML、JSON和HTML的Web或原生应用程序。
* [SMockin](https://github.com/matthewgallina/smockin)：SMockin是一种用于动态模拟API端点、S3存储桶和电子邮件帐户的开发工具。
* [Betamax](https://github.com/betamaxteam/betamax)：Betamax是一个用于在测试中模拟外部HTTP资源的工具，该项目的灵感来自于Ruby的VCR库。
* [HttpMocker](https://github.com/speekha/httpmocker)：HttpMocker是一个非常轻量的Kotlin库，允许依赖OkHttp或Ktor客户端库来模拟HTTP调用。
* [CastleMock](https://github.com/castlemock/castlemock)：CastleMock是一个Web应用程序，提供模拟RESTful API和SOAP Web Service的功能。
* [Restito](https://github.com/mkotsur/restito)：Restito是一个用于验证代码与REST服务交互的工具。
* [DeepfakeHTTP](https://github.com/xnbox/DeepfakeHTTP)：DeepfakeHTTP是一个使用HTTP转储作为响应源的Web服务器。
* [Jadler](https://github.com/jadler-mocking/jadler)：Jadler是一个Java库，用于以声明方式存根和模拟HTTP服务器和资源。
* [SpecMock](https://github.com/specmock/specmock)：SpecMock提供了各种规格的Mock Server，提供轻量、快速且易于使用的体验。
* [Logging Interceptor](https://github.com/ihsanbal/LoggingInterceptor)：Logging Interceptor是一个OkHttp拦截器，具有漂亮的请求和响应记录器，以及Mock支持。
* [ZeroMock](https://github.com/tonivade/zeromock)：零依赖的模拟HTTP Server。
* [OKHttp Client Mock](https://github.com/gmazzo/okhttp-client-mock)：一个简单的OKHttp客户端Mock，使用可编程请求拦截器。
* [OkReplay](https://github.com/airbnb/okreplay)：OkReplay旨在通过拦截应用程序发起的HTTP连接并重放之前记录的响应，由Airbnb开发。
* [RESTMock](https://github.com/andrzejchm/RESTMock)：RESTMock是一个建立在MockWebServer之上的库，它允许你指定Hamcrest匹配器来匹配HTTP请求并指定要返回的响应。

#### Mock库

* [RabbitMQ Mock](https://github.com/fridujo/rabbitmq-mock)：RabbitMQ Java AMQP-Client的Mock库。
* [S3Mock](https://github.com/adobe/S3Mock)：S3Mock是一个轻量级服务器，它实现了部分Amazon S3 API，由Adobe开源。
* [S3Mock](https://github.com/findify/s3mock)：S3Mock是一个实现AWS S3 API的Web服务，可用于使用S3对代码进行本地测试。
* [S3Ninja](https://github.com/scireum/s3ninja)：S3Ninja模拟Amazon S3 API以用于开发和测试目的。
* [GreenMail](https://github.com/greenmail-mail-test/greenmail)：GreenMail是一个开源、直观且易于使用的电子邮件服务器测试套件。
* [DaggerMock](https://github.com/fabioCollini/DaggerMock)：用于轻松覆盖Dagger 2对象的JUnit Rule。
* [CouchbaseMock](https://github.com/couchbase/CouchbaseMock)：CouchbaseMock是一个测试Couchbase服务器。
* [Hoverfly](https://github.com/SpectoLabs/hoverfly-java)：Hoverfly的Java绑定，Hoverfly是一个允许你模拟HTTP服务的代理。
* [Keycloak Mock](https://github.com/TNG/keycloak-mock)：提供Keycloak测试支持的Java库。
* [gRPC Mock](https://github.com/Fadelis/grpcmock)：一个gRPC Java测试工具，可轻松Mock gRPC服务端点以进行集成测试或单元测试。
* [GwtMockito](https://github.com/google/gwtmockito)：GwtMockito是用于GWT应用程序的测试工具，由Google开源。
* [Mock OAuth2 Server](https://github.com/navikt/mock-oauth2-server)：可编写脚本/可自定义的Web服务器，用于使用OAuth2/OpenID Connect测试HTTP客户端或依赖于正在运行的OAuth2服务器的应用程序。
* [Thrift Mock](https://github.com/didi/thrift-mock)：Thrift Mock是用于Thrift服务的轻量级Java单元测试库，由滴滴开源。
* [Spring Data Mock](https://github.com/mmnaseri/spring-data-mock)：Spring Data Repository的Mock工具。
* [MockFtpServer](https://github.com/dx42/MockFtpServer)：MockFtpServer项目提供模拟/虚拟FTP服务器实现来测试FTP客户端代码。
* [Odo](https://github.com/groupon/odo)：Odo是一个代理服务器，可以充当模拟服务器或允许操作实时数据。

#### 数据Mock

* [Java Faker](https://github.com/DiUS/java-faker)：Java Faker是Ruby的stympy/faker gem的Java端口，用于生成假数据。
* [Instancio](https://github.com/instancio/instancio)：Instancio是一个Java库，可以自动为单元测试创建和填充对象。
* [JUnit DataProvider](https://github.com/TNG/junit-dataprovider)：DataProvider类似TestNG的JUnit数据提供者运行程序，具有许多附加功能。
* [DataFaker](https://github.com/datafaker-net/datafaker)：Datafaker是一个用于Java和Kotlin生成虚假数据的库。
* [EasyRandom](https://github.com/j-easy/easy-random)：EasyRandom是一个生成随机Java对象的库。
* [MockNeat](https://github.com/nomemory/mockneat)：Mockneat是一个用Java编写的任意数据生成器开源库。
* [jFairy](https://github.com/Devskiller/jfairy)：Java测试数据生成器。
* [Jmockdata](https://github.com/jsonzou/jmockdata)：Jmockdata是一款实现模拟Java类或对象的实例化并随机初始化对象的数据的工具框架。
* [Kotlin Faker](https://github.com/serpro69/kotlin-faker)：用Kotlin编写的流行Ruby faker gem的移植版。
* [Fixture Monkey](https://github.com/naver/fixture-monkey)：Fixture Monkey旨在轻松生成可控的任意实例，它允许你在多个测试中重复使用实例的相同配置，由Naver开源。
* [Burst](https://github.com/square/burst)：Burst是用于不同测试数据的单元测试库，由Square开源。
* [Log Synth](https://github.com/tdunning/log-synth)：Log Synth的主要用途是根据指定的模式生成数据。
* [EasyModeling](https://github.com/easymodeling/easy-modeling)：EasyModeling是一个Java注解处理器，可生成随机填充的对象以供测试使用。
* [Beanmother](https://github.com/keepcosmos/beanmother)：Beanmother有助于创建各种对象，可以非常轻松地使用用于测试的夹具。
* [Common Random](https://github.com/yindz/common-random)：Common Random是一个简单易用的随机数据生成器。
* [JFactory](https://github.com/leeonky/jfactory)：JFactory是通过工厂方法创建具有某些默认属性测试数据的工具库。
* [DataHelix](https://github.com/finos/datahelix)：DataHelix可以快速生成丰富且真实的数据用于模拟和测试。
* [Fixture Factory](https://github.com/six2six/fixture-factory)：Fixture Factory是一个帮助开发人员快速构建和组织假对象以进行单元测试的工具。
* [Wordnet Random Name](https://github.com/kohsuke/wordnet-random-name)：用于测试的人类友好随机名称生成器。
* [Podam](https://github.com/mtedone/podam)：Podam是一个Java测试工具，可以用虚构的数据自动填充POJO。
* [RandomData](https://github.com/ZieIony/RandomData)：RandomData自动用生成的随机名称、数字、图像等填充对象。
* [AutoParams](https://github.com/AutoParams/AutoParams)：AutoParams是一个专为Java参数化测试而设计的任意测试数据生成器。
* [Java Random](https://github.com/merkle-open/java-random)：Java Random提供了一种通用机制来创建Java对象的随机测试虚拟对象。
* [JFixture](https://github.com/FlexTradeUKLtd/jfixture)：JFixture是一个自动填充测试数据的Java库。
* [JavaFixture](https://github.com/Nylle/JavaFixture)：JavaFixture旨在将AutoFixture的极致易用性引入Java世界。
* [DataFactory](https://github.com/andygibson/datafactory)：用于生成测试数据的Java库。
* [Mock.java](https://gitee.com/ForteScarlet/Mock.java)：这是一个仿照Mock.js语法的Java语言使用的假数据生成工具框架。
* [Java Test Data Generator](https://github.com/binarywang/java-testdata-generator)：Java实现的各种随机测试数据生成器，包括身份证号码、银行卡号、姓名、汉字、手机号、电子邮箱地址等。
* [Test Data Supplier](https://github.com/sskorol/test-data-supplier)：该仓库包含TestNG DataProvider包装器，有助于以更灵活的方式提供测试数据。
* [Data Factory](https://github.com/houbb/data-factory)：Data Factory用于根据对象随机自动生成初始化信息。
* [Elmyr](https://github.com/xgouchet/Elmyr)：Elmyr是一个Kotlin库，提供生成随机值的工具。
* [EvoSQL](https://github.com/SERG-Delft/evosql)：EvoSQL是一种自动为SQL查询生成测试数据的工具，由代尔夫特理工大学开源。
* [RandomJson](https://github.com/mangatmodi/RandomJson)：RandomJson提供Kotlin/Java库来创建随机JSON字符串。
* [Nomen est Omen](https://github.com/igr/nomen-est-omen)：这个Java库有助于生成一些随机名称，你可以将它们用于某些唯一的ID或密码。
* [Model Citizen](https://github.com/mguymon/model-citizen)：Model Citizen是一个基于注解的Java模型工厂。
* [Test Arranger](https://github.com/ocadotechnology/test-arranger)：将测试数据排列为完全填充的对象。
* [Datagen](https://github.com/qala-io/datagen)：用于生成随机数据(数字、字符串、日期)的Java库-以便于随机测试。
* [Fabricator](https://github.com/azakordonets/fabricator)：Fabricator是随机字符串、数字等的极简生成器，有助于减少单调性。
* [Make It Easy](https://github.com/npryce/make-it-easy)：一个微型框架，可轻松用Java编写测试数据构建器。

#### BDD框架

* [Cucumber](https://github.com/cucumber/cucumber-jvm)：Cucumber是一个支持行为驱动开发(BDD)的工具。
* [Karate](https://github.com/karatelabs/karate)：Karate是将API测试自动化、Mock、性能测试甚至UI自动化整合到一个统一框架中的开源工具，由Intuit开源。
* [Quantum](https://github.com/Perfecto-Quantum/Quantum)：Quantum是一个由Perfecto设计的基于Java的开源BDD测试框架。
* [Spek](https://github.com/spekframework/spek)：Spek是Kotlin的规范框架。
* [Serenity BDD](https://github.com/serenity-bdd/serenity-core)：Serenity BDD是一个旨在使编写自动化验收测试变得更容易、更有趣的库。
* [Concordion](https://github.com/concordion/concordion)：Concordion是一个可执行规范的开源运行程序，可创建丰富的实时文档。
* [YAKS](https://github.com/citrusframework/yaks)：YAKS是一个在Kubernetes上启用云原生BDD测试的框架。
* [JBehave](https://github.com/jbehave/jbehave-core)：JBehave是一个适用于Java和所有JVM语言的BDD框架。
* [JGiven](https://github.com/TNG/JGiven)：JGiven是一个开发人员友好且实用的Java BDD工具。
* [Chorus](https://github.com/Chorus-bdd/Chorus)：Chorus是一个BDD测试解释器，具有用于测试分布式架构的额外功能。
* [ScalaTest](https://github.com/scalatest/scalatest)：ScalaTest是一个为Scala和Java程序员提供的免费开源测试工具包。
* [Lambda Behave](https://github.com/RichardWarburton/lambda-behave)：Lambda Behave是Java 8的现代测试和行为规范框架。
* [Spectrum](https://github.com/greghaskins/spectrum)：Spectrum受到BDD框架Jasmine和RSpec的启发，将它们的表达语法和功能风格引入Java测试。
* [Specs2](https://github.com/etorreborre/specs2)：Specs2是一个用于在Scala中编写可执行软件规范的库。
* [YatSpec](https://github.com/bodar/yatspec)：YatSpec是一个BDD测试框架，可以运行你的JUnit测试并生成人类可读的HTML报告。
* [SmartBDD](https://github.com/bit-smart-io/smart-bdd)：从Java代码创建交互式HTML文档/功能文件的BDD框架。
* [BDD Security](https://github.com/iriusrisk/bdd-security)：BDD Security是一个安全测试框架，它使用行为驱动开发概念来创建自我验证的安全规范。
* [J8Spec](https://github.com/j8spec/j8spec)：J8Spec是一个库，允许用Java编写的测试遵循RSpec和Jasmine引入的BDD风格。
* [Narrative](https://github.com/tim-group/narrative)：用于使用流式Java构建行为驱动测试的框架。
* [Wakamiti](https://github.com/iti-ict/wakamiti)：Wakamiti是一款受Cucumber启发的自动化测试工具，专注于使用自然语言进行黑盒测试，由瓦伦西亚理工大学开源。
* [Tiger](https://github.com/gematik/app-Tiger)：Tiger是一个接口驱动的BDD黑盒测试框架。
* [Akita](https://github.com/alfa-laboratory/akita)：基于Cucumber和Selenide的BDD测试步骤库。
* [Cuppa](https://github.com/cuppa-framework/cuppa)：Cuppa是Java 8的测试框架，它使编写测试变得高效且有趣。
* [JDave](https://github.com/jdave/JDave)：JDave是Java的BDD框架。

#### 测试生成器

* [Auto Unit Test Case Generator](https://github.com/traas-stack/auto-unit-test-case-generator)：Auto Unit Test Case Generator自动生成Java的高级代码覆盖JUnit测试套件，在蚂蚁中广泛使用。
* [Tcases](https://github.com/Cornutum/tcases)：Tcases是基于模型的测试用例生成器。
* [JCUnit](https://github.com/dakusui/jcunit)：JCUnit是一个基于模型的开源测试框架，由组合交互测试技术提供支持。
* [ChatUniTest](https://github.com/ZJU-ACES-ISE/chatunitest-core)：ChatUniTest是一个创新框架，旨在改进自动化单元测试生成，由浙江大学开源。
* [GraphWalker](https://github.com/GraphWalker/graphwalker-project)：GraphWalker是一个基于模型的测试工具，它以有向图的形式读取模型，并从这些图生成测试路径。
* [Randoop](https://github.com/randoop/randoop)：Randoop是Java的单元测试生成器，它会自动为你的类创建JUnit格式的单元测试。
* [DSpot](https://github.com/STAMP-project/dspot)：DSpot是一个在JUnit测试中生成缺失断言的工具。
* [Diffblue](https://www.diffblue.com/)：Diffblue Cover使用下一代自主AI来自动化单元测试，以便Java开发团队可以更快地构建更好的应用程序。
* [Squaretest](https://squaretest.com/)：Squaretest是一个自动为Java类生成单元测试的Intellij IDEA插件。
* [Symflower](https://symflower.com/en/)：Symflower是Java代码生成工具，旨在减少单元和集成测试的日常工作。
* [AgitarOne](http://www.agitar.com/solutions/products/agitarone.html)：用于生成Java代码测试用例的工具。
* [Jtest](https://www.parasoft.com/products/parasoft-jtest/)：通过AI优化的静态分析和AI支持的自动化单元测试加速Java软件开发，以提供可靠、安全且可维护的软件。
* [EvoSuite](https://github.com/EvoSuite/evosuite)：EvoSuite自动为Java类生成JUnit测试套件，针对代码覆盖率标准。
* [UTBotJava](https://github.com/UnitTestBot/UTBotJava)：UnitTestBot是用于自动化单元测试生成和精确代码分析的工具。
* [TestMe](https://github.com/wrdv/testme-idea)：自动为Java、Groovy、Scala生成测试用例的Intellij IDEA插件。

#### 参数化测试

* [Burst](https://github.com/cashapp/burst)：Burst是一个用于参数化单元测试的库。
* [JUnitParams](https://github.com/Pragmatists/JUnitParams)：JUnitParams项目为JUnit添加了一个新的运行器，提供了更容易、更易读的参数化测试。
* [TestParameterInjector](https://github.com/google/TestParameterInjector)：TestParameterInjector是JUnit 4和JUnit 5测试运行程序，它针对字段/参数值的不同组合运行其测试方法，由Google开源。
* [JUnit 5 FormattedSource](https://github.com/mikemybytes/junit5-formatted-source)：该库可以通过编写参数化测试的新方法对JUnit 5进行扩展。
* [JUnit JSON Params](https://github.com/joshka/junit-json-params)：一个JUnit 5库，提供在参数化测试中从JSON字符串或文件加载数据的注解。
* [Spockito](https://github.com/tools4j/spockito)：一个简单的Java库，用于以类似表格的方式定义数据。

#### Selenium库

* [Selenium](https://github.com/SeleniumHQ/selenium)：Selenium是一个伞式项目，封装了各种支持Web浏览器自动化的工具和库，由ThoughtWork开源。
* [Selenium Jupiter](https://github.com/bonigarcia/selenium-jupiter)：Selenium Jupiter是一个开源Java库，它实现了用于开发Selenium WebDriver测试的JUnit 5扩展。
* [Zalenium](https://github.com/zalando/zalenium)：Zalenium是一个灵活且可扩展的基于容器的Selenium Grid，具有视频录制、实时预览、基本身份验证和仪表板，由Zalando开源。
* [Selenide](https://github.com/selenide/selenide)：Selenide是一个用Java编写易于阅读和易于维护的自动化测试的框架。
* [WebDriverManager](https://github.com/bonigarcia/webdrivermanager)：WebDriverManager是一个开源Java库，用于对Selenium WebDriver所需的驱动程序(例如chromedriver、geckodriver、msedgedriver等)进行管理。
* [Selendroid](https://github.com/selendroid/selendroid)：Selendroid是一个测试自动化框架，它通过Selendroid驱动Android原生和混合应用程序以及移动Web的UI。
* [jBrowserDriver](https://github.com/MachinePublishers/jBrowserDriver)：与Selenium WebDriver规范兼容的可编程、可嵌入的Web浏览器驱动程序。
* [Html Elements](https://github.com/yandex-qatools/htmlelements)：Html Elements是一个Java框架，提供在网页测试中与网页元素交互的易于使用的方式。
* [FluentSelenium](https://github.com/SeleniumHQ/fluent-selenium)：FluentSelenium是Selenium 2+的包装器，添加了用于浏览器的流式界面样式，可以更轻松、更快速地编写Web UI测试。
* [Frameworkium](https://github.com/Frameworkium/frameworkium-core)：Frameworkium是用于Web、应用程序和API测试的快速启动自动化框架。
* [Aquality Selenium](https://github.com/aquality-automation/aquality-selenium-java)：Aquality Selenium是一个基于Selenium WebDriver工具构建的库，允许使用Web浏览器实现自动化工作。
* [Conductor](https://github.com/conductor-framework/conductor)：Selenium WebDriver API的包装器。
* [aShot](https://github.com/pazone/ashot)：WebDriver屏幕截图工具，截图、裁剪、美化、比较。
* [Darcy](https://github.com/darcy-framework)：Darcy是一个开源Java 8框架，用于使用声明式、与自动化库无关的DSL将用户界面建模为页面对象，由RedHat开源。
* [HtmlUnitDriver](https://github.com/SeleniumHQ/htmlunit-driver)：HtmlUnitDriver是HtmlUnit无头浏览器的WebDriver兼容驱动程序。
* [Shutterbug](https://github.com/assertthat/selenium-shutterbug)：Selenium Shutterbug是一个用Java编写的实用程序库，用于使用Selenium WebDriver制作屏幕截图。
* [Selenium Cucumber](https://github.com/selenium-cucumber/selenium-cucumber-java)：Selenium Cucumber是一种BDD方法，用于编写自动化测试脚本来测试Web。
* [Selenese Runner Java](https://github.com/vmi/selenese-runner-java)：Selenium IDE原生格式(selenese和side)解释器。
* [UTAM Java](https://github.com/salesforce/utam-java)：UI测试自动化模型(UTAM)项目允许开发人员创建和使用页面对象，通过浏览器自动化网页，由Salesforce开源。
* [Atlas](https://github.com/qameta/atlas)：另一个WebDriver包装器。
* [Ghost Driver](https://github.com/detro/ghostdriver)：Ghost Driver是PhantomJS的WebDriver Wire协议的纯JavaScript实现，它是一个使用PhantomJS作为后端的远程WebDriver。
* [Healenium-Web](https://github.com/healenium/healenium-web)：用于Selenium基于Web的测试的自我修复库。
* [Pickleib](https://github.com/Umutayb/Pickleib)：Pickleib是一个用于软件自动化项目的实用程序库，它可以帮助你以简单有效的方式使用Selenium WebDriver设计和运行测试。
* [Selenium Foundation](https://github.com/sbabcoc/Selenium-Foundation)：Selenium Foundation是一个自动化框架，旨在扩展和增强Selenium WebDriver提供的功能。
* [Page Factory 2](https://github.com/sbtqa/page-factory-2)：Page-Factory-2是一个用于自动化测试的开源Java框架，允许你以BDD风格开发自动测试，重点是使用页面工厂模式。
* [Atlassian Selenium](https://bitbucket.org/atlassian/atlassian-selenium)：由Atlassian开源的项目，旨在促进Selenium/WebDriver库中功能测试的开发。
* [Jalenium](https://lkkushan.gitbook.io/jalenium)：Jalenium是一个Selenium Java包装器，使Selenium Java中的测试自动化更加简单。
* [Selenium Grid Extras](https://github.com/groupon/Selenium-Grid-Extras)：Selenium Grid Extras是一个帮助你设置和管理本地Selenium Grid的项目。
* [Selenium Grid Router](https://github.com/seleniumkit/gridrouter)：Selenium Grid Router是一个轻量级服务器，它将Selenium Webdriver请求路由和代理到多个Selenium集线器。
* [Testy](https://github.com/RWS/Testy)：Testy是用于Web应用程序的Selenium WebDriver测试框架。

#### 自动化框架

* [Selenium](https://github.com/SeleniumHQ/selenium)：Selenium是一个伞式项目，封装了各种支持Web浏览器自动化的工具和库，由ThoughtWork开源。
* [Playwright](https://github.com/microsoft/playwright-java)：Playwright是一个Java库，可通过单个API实现Chromium、Firefox和WebKit的自动化，由Microsoft开源。
* [F2etest](https://github.com/alibaba/f2etest)：F2etest是一个面向前端、测试、产品等岗位的多浏览器兼容性测试整体解决方案，由阿里开源。
* [FluentLenium](https://github.com/FluentLenium/FluentLenium)：FluentLenium是一个Web和移动自动化框架，它扩展了Selenium以编写可靠且有弹性的UI功能测试。
* [SeLion](https://github.com/paypal/SeLion)：SeLion是Paypal开源的自动化测试工具。
* [Robotium](https://github.com/robotiumtech/robotium)：Robotium是一个Android测试自动化框架，完全支持原生和混合应用程序。
* [JDI Light](https://github.com/jdi-testing/jdi-light)：JDI Light是一个功能强大的测试自动化框架，有助于使你的测试快速、可持续，并提供明显且可预测的测试运行结果，由EPAM开源。
* [ZeroCode](https://github.com/authorjapps/zerocode)：ZeroCode是一个社区开发的免费开源自动化测试框架，用于微服务API、Kafka和负载测试。
* [Carina](https://github.com/zebrunner/carina)：Carina是一个基于Java的测试自动化框架。
* [Geb](https://github.com/geb/geb)：Geb是一种浏览器自动化解决方案，它汇集了WebDriver的强大功能、jQuery内容选择的优雅性、页面对象建模的稳健性以及Groovy语言的表现力。
* [Dagger](https://github.com/NetEase/Dagger)：Dagger是一个基于Selenium和TestNG的轻量级、健壮的Web UI自动测试框架，由网易开源。
* [RESTest](https://github.com/isa-group/RESTest)：RESTest是一个用于RESTful Web API自动化黑盒测试的框架，由塞维利亚大学开源。
* [RestTestGen](https://github.com/SeUniVr/RestTestGen)：RestTestGen是一个强大的工具和框架，专为RESTful Web API的自动化黑盒测试而设计，由维罗纳大学开源。
* [HBrowser](https://github.com/Osiris-Team/HBrowser)：无头/完整的Java浏览器，支持下载文件、使用Cookie、检索HTML和模拟真实用户输入。
* [ATS Framework](https://github.com/Axway/ats-framework)：ATS是内部开发的测试框架，广泛用于满足大多数Axway产品的测试需求。
* [Appium Client](https://github.com/appium/java-client)：用于编写符合WebDriver协议的Appium测试的Java语言绑定。
* [Ride](https://github.com/adobe/ride)：Ride是一个与服务无关、模块化、可扩展的Java REST API自动化框架，由Adobe开源。
* [Boyka Framework](https://github.com/BoykaFramework/boyka-framework)：测试自动化框架，可在任何平台上自动化任何应用程序。
* [Gepard](https://github.com/epam/Gepard)：Gepard是一个基于JUnit的测试自动化框架，由EPAM开源。
* [TestZeus](https://github.com/TestZeus/TestZeus)：TestZeus是专为Salesforce构建的开源自动化框架。
* [UI-Automation](https://github.com/mmarquee/ui-automation)：UI-Automation是一个用于自动化(通过MS UIAutomation库)基于Win32、WPF和其他Windows应用程序的富客户端应用程序的框架。
* [FastAutoTest](https://github.com/y-grey/FastAutoTest)：FastAutoTest是一个基于Appium的快速自动化框架。
* [Selcukes Java](https://github.com/selcukes/selcukes-java)：Selcukes是一个强大的开源测试库，适用于Web、移动、桌面应用程序和API端点，旨在创建可扩展的高质量自动化测试。
* [Smart Test Framework](https://github.com/HPInc/smart-test-framework)：Smart Test Framework是一个多用途测试框架，能够为网页、Web服务、桌面应用程序和移动应用程序创建自动化测试，由惠普开源。
* [TestHub](https://gitee.com/dromara/TestHub)：TestHub是一款基于流程编排的自动化测试工具，由dromara社区开源。
* [JSystem](https://github.com/Top-Q/jsystem)：JSystem是一个用于编写和管理自动化系统测试的专业开源框架。

#### 自动化工具

* [MeterSphere](https://github.com/metersphere/metersphere)：MeterSphere是一站式开源持续测试平台，涵盖测试跟踪、接口测试、UI测试和性能测试等功能，全面兼容JMeter、Selenium等主流开源标准，由飞致云开源。
* [Aqua](https://www.jetbrains.com/aqua/)：Aqua是一个专门为测试自动化创建的IDE，由JetBrains开发。
* [Sonic](https://github.com/SonicCloudOrg/sonic-server)：Sonic是一个集远程控制调试和移动设备自动化测试于一体的平台，致力于为全球开发者和测试工程师创造更好的使用体验。
* [LuckyFrameWeb](https://gitee.com/seagull1985/LuckyFrameWeb)：LuckyFrame是一款免费开源的测试平台，最大的特点是全纬度覆盖了接口自动化、WEB UI自动化、APP自动化。
* [SoloPi](https://github.com/alipay/SoloPi)：SoloPi是一个无线化、非侵入式的Android自动化工具，由阿里开源。
* [ReadyAPI](https://smartbear.com/product/ready-api/)：ReadyAPI是一个专业的API测试工具，支持Java集成和自动化测试，由SmartBear开发。
* [HydraLab](https://github.com/microsoft/HydraLab)：HydraLab是一个可以帮助你利用现有的测试设备/机器轻松构建云测试平台的框架，由Microsoft开源。
* [Hamibot](https://github.com/hamibot/hamibot)：Hamibot是一款Android平台JavaScript自动化工具，无需Root，基于Auto.js。
* [Testsigma](https://github.com/testsigmahq/testsigma)：Testsigma是一个开源、可扩展的测试自动化平台，开箱即用。
* [OpenTest](https://github.com/mcdcorp/opentest)：OpenTest是一款免费开源功能测试自动化工具，适用于Web应用程序、移动应用程序和API，麦当劳开源。
* [VIVIDUS](https://github.com/vividus-framework/vividus)：VIVIDUS是一种测试自动化工具，为测试最流行的应用程序类型提供已实施的解决方案。
* [Cerberus](https://github.com/cerberustesting/cerberus-core)：Cerberus Test是一个低代码测试自动化平台，支持测试Web、iOS、Android和API(REST、SOAP和Kafka)应用程序。
* [SHAFT](https://github.com/ShaftHQ/SHAFT_ENGINE)：SHAFT是一个统一的测试自动化引擎，由一流的框架提供支持，提供类似向导的语法来高效推动自动化、最大化你的投资回报率并最小化你的学习曲线。
* [AutoMeter](https://gitee.com/season-fan/autometer-api)：AutoMeter是一款针对分布式服务、微服务API做功能和性能一体化的自动化测试平台。
* [INGenious](https://github.com/ing-bank/INGenious)：INGenious提供了一种简单易行的方式来创建高度可靠的自动化测试，由ING银行开源。
* [AngusTester](https://github.com/xcancloud/AngusTester)：AngusTester旨在协助软件开发团队高效、持续地开展软件开发和测试活动，同时满足用户的敏捷开发和测试需求，由晓蚕云公司开发。
* [AppiumTestDistribution](https://github.com/AppiumTestDistribution/AppiumTestDistribution)：一个用于跨设备并行运行Android和iOS Appium测试的工具。
* [Neodymium](https://github.com/Xceptance/neodymium-library)：Neodymium尝试通过结合JUnit、WebDriver、BDD/Cucumber和适当的报告来解决典型且最紧迫的UI测试自动化问题，由Xceptance开发。
* [Video Recorder](https://github.com/SergeyPirogov/video-recorder-java)：该库只需添加一些注解即可轻松录制UI测试的视频。
* [Step](https://github.com/exense/step)：Step是一个统一的软件自动化平台，可让你充分利用自动化工件，同时摆脱特定工具。
* [TESTAR](https://github.com/TESTARtool/TESTAR_dev)：TESTAR是一款能够在GUI级别对桌面、Web和移动应用程序进行无脚本自动化系统测试的工具，由瓦伦西亚理工大学、乌得勒支大学和荷兰开放大学等开发。
* [Testerra](https://github.com/telekom/testerra)：Testerra是一个用于自动测试(Web)应用程序的集成框架，由德国电信开源。
* [ReVoman](https://github.com/salesforce-misc/ReVoman)：ReVoman是一个JVM API自动化工具，它通过让你在JVM程序/测试中执行Postman集合来重新构想API自动化，由Salesforce开源。
* [BotCity Framework](https://github.com/botcity-dev/botcity-framework-core)：BotCity RPA模块提供识别UI元素并使用鼠标和键盘操作与其交互的功能。
* [Sahi](https://www.sahipro.com/)：Sahi是一个成熟、业务就绪的测试自动化平台，用于对UI密集型应用程序进行功能测试。
* [T-Plan](https://www.t-plan.com/)：T-Plan是一个机器人流程自动化和GUI测试自动化项目。
* [ATP](https://atestingp.sourceforge.net/)：ATP是一个自动化测试和分析工具，由西班牙马德里理工大学开源。
* [Maveryx](https://www.maveryx.com/)：Maveryx是一款专注于减少测试维护成本、适应动态UI的高效测试自动化工具。
* [TestHubo](https://github.com/tiklab-project/tiklab-testhubo)：全栈式测试管理工具，涵盖功能测试、接口测试、Web测试、App测试及性能测试等，全面覆盖各种测试场景。

#### QA自动化

* [Stevia](https://github.com/persado/stevia)：Stevia是Persado的开源QA自动化测试框架。
* [QMetry](https://github.com/qmetry/qaf)：使用Selenium、WebDriver、TestNG和Jersey的Web、MobileWeb移动原生和Rest Web服务的质量自动化框架。
* [AET](https://github.com/wttech/aet)：AET是一个检测网站上的视觉变化并执行基本页面健康检查(如W3C合规性、可访问性、HTTP状态码、JS错误检查等)的系统。

#### 测试报告

* [Allure](https://github.com/allure-framework/allure2)：Allure Report是一种灵活的多语言测试报告工具，可向你展示已测试内容的详细表示，并从日常测试执行中提取最大程度的信息。
* [Scott](https://github.com/dodie/scott)：Scott为Java测试提供了详细的失败消息，并且无需使用复杂的断言库。
* [ReportPortal](https://github.com/reportportal/reportportal)：ReportPortal是一项TestOps服务，它提供了增强的功能，通过使用内置分析功能来加速结果分析和报告。
* [Zebrunner Reporting](https://github.com/zebrunner/reporting)：Zebrunner Reporting是一种测试自动化管理工具，可累积并表示测试结果。
* [Difido Reports](https://github.com/Top-Q/difido-reports)：该项目旨在为各种功能测试自动化框架提供灵活、实时的HTML报告。
* [Cluecumber](https://github.com/trivago/cluecumber)：用于从Cucumber BDD、Karate和其他框架生成的Cucumber兼容JSON文件创建聚合测试报告。
* [Open Test Reporting](https://github.com/ota4j-team/open-test-reporting)：与语言无关的测试报告格式和工具。
* [Cucumber Reporting](https://github.com/damianszczepanik/cucumber-reporting)：这是一个Java报告发布器，主要用于在Jenkins构建服务器上发布Cucumber报告。
* [Java TestNG](https://github.com/reportportal/agent-java-testNG)：将结果上传到ReportPortal服务器的TestNG报告器。
* [Spock Reports](https://github.com/renatoathaydes/spock-reports)：该项目是Spock的全局扩展，用于创建测试报告。

#### 多线程测试

* [Awaitility](https://github.com/awaitility/awaitility)：Awaitility是一种DSL，允许你以简洁且易于阅读的方式表达对异步系统的期望。
* [Lincheck](https://github.com/JetBrains/lincheck)：Lincheck是一个实用且用户友好的框架，用于在JVM上测试并发算法，由JetBrains开源。
* [Vmlens](https://github.com/vmlens/vmlens)：Vmlens使在JVM上对多线程应用程序进行单元测试变得容易。
* [Thread Weaver](https://github.com/google/thread-weaver)：Weaver是一个用于测试多线程代码的Java框架，由Google开源。
* [JCStress](https://github.com/openjdk/jcstress)：JCStress是实验性工具和一套测试，用于帮助研究JVM、类库和硬件中并发支持的正确性，由Oracle开源。
* [MultithreadedTC](https://code.google.com/archive/p/multithreadedtc/)：MultithreadedTC是用于测试并发Java应用程序的框架，由Google开源。
* [ConcurrentUnit](https://github.com/jhalterman/concurrentunit)：ConcurrentUnit的创建是为了帮助开发人员测试多线程或异步代码。
* [Tempus Fugit](https://github.com/tobyweston/tempus-fugit)：用于编写和测试并发代码的小型库。
* [Concutest](https://www.concutest.org/)：Concutest是一套工具，旨在使并发Java程序的开发和测试更加轻松可靠，由莱斯大学开源。
* [Threads Collider](https://github.com/stawirej/threads-collider)：Threads Collider尝试在“完全相同”的时刻对多个线程执行所需的操作，以增加出现由竞争条件或死锁引起的问题的几率。
* [VerCors](https://github.com/utwente-fmt/vercors)：VerCors是一个用于验证并发和并行程序的工具集，由特文特大学开源。
* [ThreadPoster](https://github.com/techyourchance/thread-poster)：用于单元测试和富有表现力的多线程的轻量级库。
* [Fray](https://github.com/cmu-pasta/fray)：Fray是Java的一个并发测试工具，可以帮助你查找和调试表现为断言违规、运行时异常或死锁的棘手竞争条件，由CMU开源。

#### POJO测试

* [OpenPojo](https://github.com/OpenPojo/openpojo)：用于简化POJO测试的库。
* [POJO TESTER](https://github.com/sta-szek/pojo-tester)：POJO-TESTER是一个Java测试库，可以使POJO测试变得更加容易。
* [BoundBox](https://github.com/stephanenicolas/boundbox)：BoundBox提供了一种简单的方法来测试对象，即通过访问对象的所有字段、构造函数和方法。

#### JUnit扩展

* [HiveRunner](https://github.com/HiveRunner/HiveRunner)：HiveRunner是基于JUnit 4和5的Hive查询开源单元测试框架。
* [ReRunner](https://github.com/artsok/rerunner-jupiter)：ReRunner是JUnit 5的扩展，可以立即重新运行失败的JUnit 5测试。
* [JUnit Pioneer](https://github.com/junit-pioneer/junit-pioneer)：JUnit Pioneer为JUnit 5及其Jupiter API提供扩展。
* [JUnitParams](https://github.com/Pragmatists/JUnitParams)：JUnitParams项目为JUnit添加了一个新的运行器，提供了更容易、更易读的参数化测试。
* [System Rules](https://github.com/stefanbirkner/system-rules)：System Rules是用于测试使用java.lang.System的代码的JUnit Rule集合。
* [System Lambda](https://github.com/stefanbirkner/system-lambda)：System Lambda是用于测试使用java.lang.System的代码的函数集合。
* [System Stubs](https://github.com/webcompere/system-stubs)：System Stubs用于测试依赖于java.lang.System中方法的代码。
* [JUnit5 System Exit](https://github.com/tginsberg/junit5-system-exit)：此JUnit 5扩展可帮助你为调用System.exit()的代码编写测试。
* [NoSQLUnit](https://github.com/lordofthejars/nosql-unit)：NoSQLUnit是一个JUnit扩展，可以更轻松地编写使用NoSQL后端的系统的单元和集成测试。
* [TagUnit](https://tagunit.sourceforge.net/)：TagUnit是一个用于测试JSP页面中的自定义标签的标签库。
* [JSPUnit](https://sourceforge.net/projects/jspunit/)：JSPUnit是JUnit的扩展，用于对JSP进行单元测试。
* [JSFUnit](https://jsfunit.jboss.org/)：JSFUnit是JSF应用程序的测试框架，由JBoss社区维护。
* [Docker Compose JUnit Rule](https://github.com/palantir/docker-compose-rule)：这是一个用于执行与Docker Compose托管容器交互的JUnit测试的库，由Palantir开源。
* [JGotesting](https://github.com/tastapod/jgotesting)：JGotesting是受Go测试包启发的JUnit兼容测试工具。
* [RandomizedTesting](https://github.com/randomizedtesting/randomizedtesting)：适用于JUnit、ANT和Maven的随机测试基础设施。
* [Kafka JUnit](https://github.com/charithe/kafka-junit)：Kafka JUnit提供了在测试期间启动和关闭Kafka 代理的工具程序。
* [Chronicle Test Framework](https://github.com/OpenHFT/Chronicle-Test-Framework)：该库提供了用于编写JUnit测试的支持类，支持JUnit 4和JUnit 5。
* [Loom-Unit](https://github.com/cescoffier/loom-unit)：用于检查虚拟线程是否固定载体线程的JUnit 5扩展。
* [JUnit Toolbox](https://github.com/MichaelTamm/junit-toolbox)：使用JUnit 4编写自动化测试的有用类。
* [Weld Testing](https://github.com/weld/weld-testing)：该项目的主要目标是为CDI单元/组件测试提供简单快速的工具，这些工具作为JUnit 4、JUnit 5和Spock扩展实现。
* [CDI-Unit](https://github.com/cdi-unit/cdi-unit)：CDI应用程序的单元测试库，支持Mockito来Mock依赖项。
* [Kafka JUnit](https://github.com/mguenther/kafka-junit)：Kafka JUnit使开发人员能够在JUnit测试中启动和停止由Kafka代理和分布式Kafka Connect Workers组成的完整Kafka集群。
* [FakeTime](https://github.com/faketime-java/faketime)：FakeTime使用原生Java代理将System.currentTimeMillis()实现替换为你可以使用系统属性控制的实现。

#### 其他测试库

* [JsonUnit](https://github.com/lukas-krecan/JsonUnit)：JsonUnit是一个简化测试中JSON比较的库。
* [EqualsVerifier](https://github.com/jqno/equalsverifier)：EqualsVerifier可用于Java单元测试，以验证类中equals和hashCode方法的约定是否得到满足。
* [OpenTest4J](https://github.com/ota4j-team/opentest4j)：该项目是JUnit 5团队倡议的成果。
* [HtmlUnit](https://github.com/HtmlUnit/htmlunit)：HtmlUnit是Java程序的无GUI浏览器。
* [Apache Commons Testing](https://github.com/apache/commons-testing)：用于测试的Java实用程序类包。
* [MUnit](https://github.com/scalameta/munit)：具有可操作错误和可扩展API的Scala测试库。
* [NonDex](https://github.com/TestingResearchIllinois/NonDex)：NonDex是一个用于检测和调试对未确定的Java API的错误假设的工具，由伊利诺伊大学开源。
* [XmlUnit](https://github.com/xmlunit/xmlunit)：XMLUnit是一个支持以多种方式测试XML输出的库。
* [LogCaptor](https://github.com/Hakky54/log-captor)：LogCaptor是一个能够轻松捕获用于单元和集成测试目的的日志记录条目的库。
* [ConsoleCaptor](https://github.com/Hakky54/console-captor)：ConsoleCaptor是一个可让你轻松捕获控制台的输出以进行单元测试的库。
* [SikuliRobot](https://github.com/rainmanwy/robotframework-SikuliLibrary)：Sikuli机器人框架库为Robot Framework提供关键字，可以通过Sikuli测试UI。
* [Spring Addons](https://github.com/ch4mpy/spring-addons)：提供OAuth2资源服务器配置和测试的库。
* [Tzatziki](https://github.com/Decathlon/tzatziki)：该项目是一组现成的Cucumber步骤的集合，通过专注于由外向内的测试策略，可以轻松实现TDD Java微服务，由迪卡侬开源。
* [Mutability Detector](https://github.com/MutabilityDetector/MutabilityDetector)：Mutability Detector旨在分析Java类并报告给定类的实例是否不可变。
* [StackSrc](https://github.com/laech/java-stacksrc)：该项目的目标是修饰测试失败的堆栈跟踪，使其更有用。
* [Overcast](https://github.com/xebialabs/overcast)：用于针对云中的主机编写测试的Java工具类。
* [SQL Logic Test](https://github.com/hydromatic/sql-logic-test)：SQL Logic Test是一套包含超过700万个测试的套件，用于测试SQL的核心方面。
* [Specnaz](https://github.com/skinny85/specnaz)：用于用Java、Kotlin和Groovy编写漂亮的RSpec/Jasmine/Mocha/Jest风格规范的库。
* [Hsac Fitnesse Fixtures](https://github.com/fhoeben/hsac-fitnesse-fixtures)：该项目通过提供定义和运行测试的应用程序来协助测试Web Services和Web应用程序。
* [Courgette JVM](https://github.com/prashant-ramcharan/courgette-jvm)：Courgette JVM是Cucumber的扩展，增加了在功能级别或场景级别并行运行Cucumber测试的功能。
* [Oleaster](https://github.com/mscharhag/oleaster)：Oleaster允许你像编写Jasmine测试一样编写JUnit测试。
* [Freud](https://github.com/LMAX-Exchange/freud)：用于编写静态分析测试的框架，由英国外汇交易公司LMAX开发。
* [EasyTest](https://github.com/EaseTech/easytest-core)：EasyTest是一个用于在Java中执行数据驱动测试的库。
* [Karibu Testing](https://github.com/mvysny/karibu-testing)：Vaadin服务器端无浏览器无容器单元测试。
* [SocketTest](https://github.com/akshath/SocketTest)：一个用于套接字测试的Java工具，它可用于测试任何使用TCP或UDP协议进行通信的服务器或客户端。
* [Component Test Framework](https://github.com/lydtechconsulting/component-test-framework)：允许对Spring Boot应用程序进行组件测试的库。
* [Skippy](https://github.com/skippy-io/skippy)：Skippy是JVM的测试影响分析和预测测试选择框架。
* [Taikai](https://github.com/enofex/taikai)：Taikai通过提供针对各种技术定制的一整套预定义规则来扩展流行的ArchUnit库的功能。

#### 测试套件

* [YCSB](https://github.com/brianfrankcooper/YCSB)：Yahoo云服务基准测试套件。
* [HiBench](https://github.com/Intel-bigdata/HiBench)：HiBench是一个大数据基准测试套件，可帮助评估不同大数据框架的速度、吞吐量和系统资源利用率，由Intel开源。
* [NDBench](https://github.com/Netflix/ndbench)：NDBench是一种可插拔的支持云的基准测试工具，可在任何数据存储系统中使用，由Netflix开源。
* [COSBench](https://github.com/intel-cloud/cosbench)：COSBench是一款用于衡量云对象存储服务性能的基准测试工具，由Intel开源。
* [BenchBase](https://github.com/cmu-db/benchbase)：BenchBase是一个通过JDBC的多DBMS SQL基准测试框架，由CMU开源。
* [LinkBench](https://github.com/facebookarchive/linkbench)：LinkBench是一个数据库基准测试，旨在评估类似于Facebook生产MySQL部署的工作负载的数据库性能。
* [Theodolite](https://github.com/cau-se/theodolite)：Theodolite是一个用于对Kubernetes中云原生应用程序的水平和垂直可扩展性进行基准测试的框架，由基尔大学开源。
* [Ant AST Benchmark](https://github.com/alipay/ant-application-security-testing-benchmark)：由蚂蚁安全团队联合浙江大学网络空间安全学院共同设计的xAST评价体系及其测试样本套件Benchmark。
* [Nexmark](https://github.com/nexmark/nexmark)：Nexmark是用于连续数据流查询的基准套件。
* [Spark Bench](https://github.com/CODAIT/spark-bench)：Apache Spark基准测试套件，IBM开源。
* [NoSQLBench](https://github.com/nosqlbench/nosqlbench)：NoSQLBench是一款针对NoSQL生态系统的严谨性能测试工具，由DataStax开源。
* [Industrial Benchmark](https://github.com/siemens/industrialbenchmark)：Industrial Benchmark是离线强化学习和在线强化学习的现实基准，用于寻找最适合实际应用的强化学习算法，由Siemens开源。
* [DaCapo Benchmark](https://github.com/dacapobench/dacapobench)：此基准测试套件旨在作为编程语言、内存管理和计算机架构社区对Java进行基准测试的工具，澳大利亚国立大学开源。
* [BenchmarkSQL](https://github.com/pingcap/benchmarksql)：BenchmarkSQL是一个易于使用的JDBC基准测试，由PingCAP维护。

#### 测试异味

* [Test Smell Detector](https://github.com/TestSmells/TestSmellDetector)：该项目旨在帮助开发人员了解在编写单元测试时通常会引入或遇到的单元测试异味类型。
* [JNose](https://github.com/arieslab/jnose)：JNose Test是一种用于自动检测测试代码中的测试异味并收集覆盖率指标的工具，由巴伊亚联邦大学、拉夫拉斯联邦大学和巴伊亚州立大学共同开发。

#### 代码覆盖率

* [JaCoCo](https://github.com/jacoco/jacoco)：JaCoCo是一个免费的Java代码覆盖率库。
* [Super JaCoCo](https://github.com/didi/super-jacoco)：Super-JaCoCo基于JaCoCo、Git二次开发打造的一站式Java代码全量/diff覆盖率收集平台，能够低成本、无侵入的收集代码覆盖率数据，由滴滴开源。
* [Kover](https://github.com/Kotlin/kotlinx-kover)：Kover是一组用于收集为JVM和Android平台编译的Kotlin代码的测试覆盖率的解决方案，由JetBrains开发。
* [Clover](https://bitbucket.org/atlassian/clover)：Atlassian开源的Java和Groovy代码覆盖率工具。
* [Cobertura](https://github.com/cobertura/cobertura)：Cobertura是一个免费的Java代码覆盖率报告工具。
* [JSCover](https://github.com/tntim96/JSCover)：JSCover是一个易于使用的JavaScript代码覆盖率测量工具。
* [EMMA](https://emma.sourceforge.net/)：EMMA是一个用于测量和报告Java代码覆盖率的开源工具包。
* [Codecov](https://about.codecov.io/)：Codecov是适用于任何测试套件的一体化代码覆盖率报告解决方案。
* [JCov](https://github.com/openjdk/jcov)：JCov开源项目用于收集与测试套件的生产相关的质量指标，Oracle开源。
* [Parasoft JTest](https://www.parasoft.com/)：包括多种现代QA工具，允许测量代码覆盖率，并对其进行静态和动态分析，这是一款商业工具。
* [OpenClover](https://github.com/openclover/clover)：OpenClover测量Java和Groovy的代码覆盖率并收集20多个代码指标。

## API管理

* [RAP](https://github.com/thx/RAP)：RAP是一种Web工具，允许开发人员快速定义和记录在典型的基于RESTful API的Web应用程序中使用的Web API，阿里开源。
* [Apigee](https://cloud.google.com/apigee)：Apigee是Google Cloud的原生API管理平台，可用于构建、管理和保护API。
* [Yaade](https://github.com/EsperoTech/yaade)：Yaade是一个开源、自托管、协作式API开发环境。
* [AgileTC](https://github.com/didi/AgileTC)：AgileTC是一个基于思维导图的具有多实时协作能力的测试用例管理平台，由滴滴开源。
* [CrapApi](https://gitee.com/CrapApi/CrapApi)：CrapApi是完全开源、免费使用的API接口管理系统、BUG管理系统。
* [XXL-API](https://github.com/xuxueli/xxl-api)：XXL-API是一个强大易用的API管理平台，提供API的管理、文档、Mock和测试等功能。
* [WSO2 API Manager](https://github.com/wso2/product-apim)：WSO2 API Manager是一个用于创建、管理、使用和监控Web API的强大平台。
* [Apiman](https://github.com/apiman/apiman)：Apiman是一个灵活的开源API管理平台，由RedHat开源。
* [Repose](https://github.com/rackerlabs/repose)：Repose为API处理任务提供解决方案，例如身份验证、速率限制、API验证、HTTP请求日志记录等等。
* [Postin](https://github.com/tiklab-project/tiklab-postin)：Postin提供API设计、调试、文档生成和Mock数据模拟等一站式解决方案。
* [EasyOpen](https://gitee.com/durcframework/easyopen)：EasyOpen是一个简单易用的接口开放平台，平台封装了常用的参数校验、结果返回等功能。
* [Torna](https://gitee.com/durcframework/torna)：Torna是一个接口文档解决方案，目标是让接口文档管理变得更加方便、快捷。
* [Gravitee](https://github.com/gravitee-io/gravitee-api-management)：Gravitee是一种灵活、轻量级且速度极快的开源解决方案，可帮助你的组织控制用户访问API的人员、时间和方式。
* [Apicurio Studio](https://github.com/Apicurio/apicurio-studio)：Apicurio Studio项目是一个独立的API设计工具，可用于创建新的或编辑现有的API设计(使用OpenAPI或AsyncAPI规范)，由RedHat开源。
* [Apicurio Registry](https://github.com/Apicurio/apicurio-registry)：Apicurio Registry使你能够使用远程REST API在存储中添加、更新和删除工件，由RedHat开源。
* [APK](https://github.com/wso2/apk)：APK即Kubernetes API平台，这是一种尖端的API管理解决方案，旨在利用Kubernetes的强大功能来实现无缝且可扩展的部署，WSO2开源。
* [Otoroshi](https://github.com/MAIF/otoroshi)：Otoroshi是一个轻量级API管理层，由MAIF OSS团队开发，可以处理微服务之间的所有调用，无需服务定位器，并允许你在运行时动态更改配置。
* [RESTFiddle](https://github.com/AnujaK/restfiddle)：适用于团队的企业级API管理平台，RESTFiddle帮助你设计、开发、测试和发布API。

## GraphQL

* [GraphQL Java](https://github.com/graphql-java/graphql-java)：GraphQL Java实现。
* [DGS Framework](https://github.com/netflix/dgs-framework)：DGS Framework是由Netflix开发的Spring Boot的GraphQL服务器框架。
* [Apollo Kotlin](https://github.com/apollographql/apollo-kotlin)：Apollo Kotlin是一个GraphQL客户端，可根据GraphQL查询生成Kotlin和Java模型。
* [Rejoiner](https://github.com/google/rejoiner)：Rejoiner可用于从gRPC微服务和其他Protobuf源生成统一的GraphQL模式，由Google开发。
* [Spring GraphQL](https://github.com/spring-projects/spring-graphql)：Spring GraphQL为基于GraphQL Java构建的Spring应用程序提供支持。
* [GraphQL Kotlin](https://github.com/ExpediaGroup/graphql-kotlin)：GraphQL Kotlin构建在GraphQL Java之上，可简化在Kotlin中运行GraphQL客户端和服务器，由Expedia开源。
* [GraphQL SPQR](https://github.com/leangen/graphql-spqr)：GraphQL SPQR是一个简单易用的库，用于在Java中快速开发GraphQL API。
* [GraphQL Spring Boot](https://github.com/graphql-java-kickstart/graphql-spring-boot)：集成GraphQL Java和Spring Boot的库。
* [Elide](https://github.com/yahoo/elide)：Elide是一个Java库，可以轻松设置模型驱动的GraphQL或JSON API Web服务，由Yahoo开源。
* [Lacinia](https://github.com/walmartlabs/lacinia)：Lacinia是一个实现GraphQL规范的Clojure库，由沃尔玛开源。
* [Sangria](https://github.com/sangria-graphql/sangria)：Sangria是一个Scala GraphQL库。
* [Graphcool](https://github.com/Graphcool/graphcool-framework)：Graphcool是一个开源后端开发框架，用于开发和部署GraphQL API。
* [GraphQL Java Annotations](https://github.com/Enigmatis/graphql-java-annotations)：该库为GraphQL模式定义提供基于注解的语法。
* [KGraphQL](https://github.com/aPureBase/KGraphQL)：KGraphQL是GraphQL的Kotlin实现。
* [GraphQL Calculator](https://github.com/graphql-calculator/graphql-calculator)：GraphQL Calculator是一个轻量级的GraphQL查询计算引擎。
* [Microprofile GraphQL](https://github.com/eclipse/microprofile-graphql)：MicroProfile框架中用于构建GraphQL应用程序的GraphQL服务器和客户端规范。
* [Nodes](https://github.com/americanexpress/nodes)：Nodes是一个GraphQL客户端，旨在根据标准模型定义构建查询，由美国运通开源。
* [Caliban](https://github.com/ghostdogpr/caliban)：Caliban是一个纯函数库，用于在Scala中构建GraphQL服务器和客户端。
* [GraphQL Java Generator](https://github.com/graphql-java-generator/graphql-maven-plugin-project)：GraphQL Java Generator可以轻松地以模式优先的方式在Java中使用GraphQL。
* [GraphQL Codegen](https://github.com/kobylynskyi/graphql-java-codegen)：GraphQL Codegen可以轻松地让你的Java应用程序遵循模式优先的方法。
* [GraphQL JPA Query](https://github.com/introproventures/graphql-jpa-query)：GraphQL JPA Query库使用JPA规范为你的JPA实体Java类使用GraphQL Java派生和构建GraphQL API。
* [HyperGraphQL](https://github.com/hypergraphql/hypergraphql)：HyperGraphQL是一个GraphQL接口，用于在Web上查询和提供链接数据。
* [GraphQL Java Tools](https://github.com/graphql-java-kickstart/graphql-java-tools)：该库允许使用GraphQL模式语言来构建GraphQL Java模式。
* [Mocca](https://github.com/paypal/mocca)：Mocca是JVM语言的GraphQL客户端，其目标是易于使用、灵活和模块化，由Paypal开源。
* [GraphQL APIGen](https://github.com/Distelli/graphql-apigen)：使用GraphQL模式生成Java API，促进模式优先开发。
* [Lilo](https://github.com/friatech/lilo)：Lilo是一个超快的GraphQL拼接库，该项目受到Atlassian Braid的启发。
* [GraphQL Orchestrator Java](https://github.com/graph-quilt/graphql-orchestrator-java)：GraphQL Orchestrator Java通过提供统一的GraphQL模式，简化了从各种GraphQL微服务访问数据的过程。
* [Test GraphQL Java](https://github.com/vimalrajselvam/test-graphql-java)：用于简化GraphQL测试的Java库。
* [Spring GraphQL Common](https://github.com/yandooo/spring-graphql-common)：简化在Spring框架中使用GraphQL的库。
* [GraphQL JPA](https://github.com/jcrygier/graphql-jpa)：这是一个简单的项目，用于扩展GraphQL Java并让它从JPA模型派生模式。
* [SchemaGen GraphQL](https://github.com/bpatters/schemagen-graphql)：GraphQL Java插件，增加了对企业级应用程序的模式生成和执行的支持。
* [Vertx GraphQL Client](https://github.com/graphqly/vertx-graphql-client)：代码优先GraphQL客户端的优雅实现。
* [Federation JVM](https://github.com/apollographql/federation-jvm)：Graphql Java的Apollo Federation规范的实现。
* [GraphQL Java Servlet](https://github.com/graphql-java-kickstart/graphql-java-servlet)：GraphQL Java的Servlet端口。
* [Light4j GraphQL](https://github.com/networknt/light-graphql-4j)：基于Light-4j的GraphQL实现。
* [GraphQL Java Extended Validation](https://github.com/graphql-java/graphql-java-extended-validation)：该库为Graphql Java提供了字段和字段参数的扩展验证。
* [GraphQL Java DataLoader](https://github.com/graphql-java/java-dataloader)：这个小而简单的实用程序库是Facebook DataLoader的纯Java 8端口。
* [GraphQL Java Extended Scalars](https://github.com/graphql-java/graphql-java-extended-scalars)：该库为GraphQL Java提供扩展标量。
* [GraphQL Java DateTime](https://github.com/tailrocks/graphql-java-datetime)：GraphQL ISO Date是一组与GraphQL Java一起使用的符合RFC 3339的日期/时间标量类型。
* [Nadel](https://github.com/atlassian-labs/nadel)：Nadel是一个将多个GraphQL服务组合在一起的Kotlin库，由Atlassian开源。
* [GraphQL Filter Java](https://github.com/intuit/graphql-filter-java)：该库可帮助GraphQL开发人员构建具有细粒度过滤支持的出色API。
* [GraphQLJavaGen](https://github.com/Shopify/graphql_java_gen)：为提供查询构建器和响应类的特定GraphQL模式生成代码，由Shopify开源。

## 日期时间

* [Joda Time](https://github.com/JodaOrg/joda-time)：Joda Time提供了Java日期和时间类的优质替代品。
* [Prettytime](https://github.com/ocpsoft/prettytime)：Java的社交风格日期和时间格式。
* [Time4J](https://github.com/MenoData/Time4J)：Time4J是围绕Date、Calendar和SimpleDateFormat的旧Java类的完整且高端的替代品。
* [ThreeTen Extra](https://github.com/ThreeTen/threeten-extra)：ThreeTen Extra提供了额外的日期时间类来补充JDK 8中的类。
* [XK-Time](https://gitee.com/xkzhangsan/xk-time)：XK-Time包含时间转换、时间计算、时间格式化、时间解析、日历、时间Cron表达式和时间NLP等工具。
* [Date4j](https://github.com/IanDarwin/date4j)：Date4j是Java内置日期类的轻量级替代品。
* [ThreeTen](https://github.com/ThreeTen/threetenbp)：ThreeTen-Backport提供Java 8日期时间类到Java 6和7的向后移植。
* [Jollyday](https://github.com/svendiedrichsen/jollyday)：Jollyday可以确定给定年份、国家/名称以及最终州/地区的假期。
* [Jollyday](https://github.com/focus-shift/jollyday)：Jollyday是一个查询公共假期的Java库，目前支持70多个国家/地区。
* [iCal4j](https://github.com/ical4j/ical4j)：iCal4j是一个Java库，用于读取和写入RFC2445中定义的iCalendar数据流。
* [TimeAgo](https://github.com/marlonlom/timeago)：一个简单的Java库，用于将日期显示为相对时间之前的语言。
* [Biweekly](https://github.com/mangstadt/biweekly)：Biweekly是一个用Java编写的iCalendar库。
* [Zmanim](https://github.com/KosherJava/zmanim)：Zmanim库是一个特殊日历的API，可以计算不同的天文时间，包括日出和日落以及犹太zmanim或祈祷和其他犹太宗教职责的宗教时间。
* [Adhan Kotlin](https://github.com/batoulapps/adhan-kotlin)：Adhan是一个经过充分测试和记录良好的库，用于计算伊斯兰祈祷时间。
* [DateParser](https://github.com/sisyphsu/dateparser)：DateParser是一个智能且高性能的日期时间解析器库，它支持数百种不同的模式。
* [Internet Time Utility](https://github.com/ethlo/itu)：ISO格式日期时间的极快解析器和格式化程序。
* [Lib-Recur](https://github.com/dmfs/lib-recur)：该库解析RFC 5545和RFC 2445中定义的重复字符串并迭代实例。
* [Human Duration](https://bitbucket.org/atlassian/humanduration)：该库允许从人类可读的字符串(如2分39秒、2m 39s等)解析Java 8的Duration时间，由Atlassian开源。
* [BusinessCalendar4J](https://github.com/yusuke/businessCalendar4J)：BusinessCalendar4J是一个100%纯Java业务日历库。
* [TickTock](https://github.com/ZacSweers/ticktock)：TickTock是一个时区数据管理库，适用于JVM和Android，针对Java 8或更高版本中的java.time.* API。
* [Sunrise/SunsetLib Java](https://github.com/mikereedell/sunrisesunsetlib-java)：用于计算给定纬度/经度和日期组合的当地日出和日落的Java库。
* [Lunar](https://gitee.com/6tail/lunar-java)：Lunar是一个支持阳历、阴历、佛历和道历的日历工具库。
* [Tyme4j](https://github.com/6tail/tyme4j)：Tyme4j是一个非常强大的日历工具库，可以看作Lunar的升级版，拥有更优的设计和扩展性，支持公历和农历、星座、干支、生肖、节气、法定假日等。
* [Natty](https://github.com/joestelmach/natty)：Natty是一个用Java编写的自然语言日期解析器。
* [Business Hours Java](https://github.com/dhatim/business-hours-java)：这个Java库有助于处理工作时间，例如“周一到周五上午9点到下午6点，周六上午9点到中午12点”。
* [JChronic](https://github.com/samtingleff/jchronic)：Java中的自然语言日期解析器，作为Ruby Chronic的直接移植。
* [UmmAl-Qura Calendar](https://github.com/msarhan/ummalqura-calendar)：Umm Al-Qura日历系统的java.util.Calendar实现。

## 实体解析

* [Zingg](https://github.com/zinggAI/zingg)：Zingg是一种基于ML的实体解析工具。
* [JedAI](https://github.com/scify/JedAIToolkit)：JedAI是一个开源、高可扩展性的Java实体解析工具包，由鲁汶大学、巴黎西岱大学、摩德纳雷焦艾米利亚大学开源。
* [Duke](https://github.com/larsga/Duke)：Duke是一款快速灵活的重复数据删除引擎。
* [Zentity](https://github.com/zentity-io/zentity)：Zentity是一个用于实时实体解析的Elasticsearch插件。
* [ReCiter](https://github.com/wcmc-its/ReCiter)：ReCiter是一个高度准确的系统，用于猜测某个人在PubMed上发表了哪些出版物，由康奈尔大学开源。

## 数据科学

* [CUDF](https://github.com/rapidsai/cudf/tree/branch-24.08/java)：该项目为CUDF提供了Java绑定，以便能够在GPU上处理大量数据，由NVIDIA提供。
* [Tablesaw](https://github.com/jtablesaw/tablesaw)：Tablesaw是一个数据框架和可视化库，支持加载、清理、转换、过滤和汇总数据。
* [OpenRefine](https://github.com/OpenRefine/OpenRefine)：OpenRefine是一个基于Java的强大工具，可让你加载数据、理解数据、清理数据、协调数据，并使用来自Web的数据对其进行扩充，由Google开源。
* [Enso](https://github.com/enso-org/enso)：Enso是一种屡获殊荣的交互式编程语言，具有双重视觉和文本表示形式。
* [XChart](https://github.com/knowm/XChart)：XChart是一个轻量且方便的数据绘制库，旨在在尽可能短的时间内从数据到图表，并消除自定义图表样式时的猜测工作。
* [Incanter](https://github.com/incanter/incanter)：Incanter是一个基于Clojure、类似R的JVM统计计算和图形环境。
* [Hopsworks](https://github.com/logicalclocks/hopsworks)：Hopsworks是一个ML数据平台，具有以Python为中心的特征存储和MLOps功能，由Hopsworks开源。
* [Superglue](https://github.com/intuit/superglue)：Superglue是一种谱系追踪工具，可帮助直观地了解数据在复杂管道中的传播，由Intuit开发。
* [ShinyProxy](https://github.com/openanalytics/shinyproxy)：ShinyProxy是用于Shiny和数据科学应用程序的开源企业部署软件。
* [Eclipse ICE](https://github.com/eclipse/ice)：ICE是一个科学工作台和工作流程环境，旨在改善计算科学家的用户体验。
* [Morpheus](https://github.com/zavtech/morpheus-core)：Morpheus库旨在促进涉及大型数据集的高性能分析软件的开发，以便在JVM上进行离线和实时分析。
* [SessionAnalytics](https://github.com/Tencent/SessionAnalytics)：SessionAnalytics是一个基于互联网用户Session会话的用户路径分析和挖掘框架，由腾讯开源。
* [Apache Commons Statistics](https://github.com/apache/commons-statistics)：Commons Statistics提供用于统计应用程序的工具，为常用的连续和离散分布提供支持。
* [JScience](https://github.com/javolution/jscience)：提供一组用于处理科学测量和单位的类。
* [JFreeChart](https://github.com/jfree/jfreechart)：JFreeChart是一个适用于Java平台的综合免费图表库，可在客户端(JavaFX和Swing)或服务器端使用。
* [DataMelt](https://datamelt.org/)：DataMelt是一款用于数值计算、统计、符号计算、数据分析和数据可视化的软件。
* [Dataframe](https://github.com/Kotlin/dataframe)：Dataframe旨在利用Kotlin语言的全部功能以及Jupyter Notebook和REPL中间歇性代码执行提供的机会，协调Kotlin的静态类型与数据的动态特性，JetBrains开源。
* [Krangl](https://github.com/holgerbrandl/krangl)：Krangl是一个用于数据处理的Kotlin库，通过使用现代函数式API实现数据操作语法，它允许过滤、转换、聚合和重塑表格数据。
* [ELKI](https://github.com/elki-project/elki)：ELKI是一款用Java编写的开源数据挖掘软件，由德国多特蒙德大学开发。
* [DataCleaner](https://github.com/datacleaner/DataCleaner)：DataCleaner是一个数据质量工具包，可让你分析、更正和丰富你的数据。
* [Featran](https://github.com/spotify/featran)：Featran是用于数据科学和机器学习的Scala特征转换库，由Spotify开源。
* [Datavines](https://github.com/datavane/datavines)：DataVines是一个易于使用的数据质量服务平台，支持多种指标，由Datavane大数据组织开源。
* [TMD](https://github.com/techascent/tech.ml.dataset)：TMD是一个用于表格数据处理的Clojure库，类似于Python的Pandas或R的data.table。
* [DDF](https://github.com/ddf-project/DDF)：DDF旨在通过汇集R数据科学、RDBMS/SQL和大数据分布式处理的最佳思想，使大数据变得简单而强大。
* [Koma](https://github.com/kyonifer/koma)：Koma是Kotlin的科学计算环境。
* [Joinery](https://github.com/cardillo/joinery)：Java的DataFrame。
* [JDFrame](https://github.com/burukeYou/JDFrame)：JDFrame是一个Java DataFrame的实现。
* [DFLib](https://github.com/dflib/dflib)：DFLib是通用DataFrame数据结构的轻量级纯Java实现，由ObjectStyle开源。
* [Classifai](https://github.com/CertifaiAI/classifai)：Classifai是最全面的开源数据标注平台之一。
* [DataFrame EC](https://github.com/vmzakharov/dataframe-ec)：基于Eclipse Collections框架的表格数据结构。
* [Poppy](https://github.com/tenmax/poppy)：Poppy是Java的Dataframe库，它提供常见的SQL操作来在Java中处理数据，由TenMax开源。

## 异常检测

* [ThirdEye](https://github.com/startreedata/thirdeye)：ThirdEye是一款用于实时监控时间序列和交互式根本原因分析的集成工具，最初由LinkedIn开源。
* [Sherlock](https://github.com/yahoo/sherlock)：Sherlock是一个构建在Druid之上的异常检测服务，由Yahoo开源。
* [Adaptive Alerting](https://github.com/ExpediaGroup/adaptive-alerting)：Adaptive Alerting通过自动模型选择和拟合进行流异常检测，由Expedia开源。
* [Random Cut Forest](https://github.com/aws/random-cut-forest-by-aws)：该仓库包含随机森林(RCF)概率数据结构的实现，由Amazon开发，用于流数据的非参数异常检测算法。
* [AnomalyDetection](https://github.com/JeemyJohn/AnomalyDetection)：Java实现的异常检测算法。
* [Isolation Forest](https://github.com/linkedin/isolation-forest)：这是孤立森林无监督异常值检测算法的分布式Scala/Spark实现，由LinkedIn开发。

# 指纹识别

* [Soter](https://github.com/Tencent/soter)：腾讯主导的Android下安全、快速的生物识别认证标准及平台。
* [SourceAFIS](https://github.com/robertvazan/sourceafis-java)：SourceAFIS Java是SourceAFIS(一种用于识别人类指纹的算法)的纯Java端口，它可以1:1比较两个指纹或1:N在大型数据库中搜索匹配的指纹。
* [FingerprintIdentify](https://github.com/uccmawei/FingerprintIdentify)：Android指纹验证SDK。
* [Android-Goldfinger](https://github.com/infinum/Android-Goldfinger)：用于简化生物识别身份验证实施的Android库。

## 推荐系统

* [Twitter Recommendation Algorithm](https://github.com/twitter/the-algorithm)：Twitter的推荐算法是一组服务和作业，负责在所有Twitter产品界面(例如For You时间线、搜索、探索、通知)上提供推文和其他内容的提要。
* [RankSys](https://github.com/RankSys/RankSys)：RankSys是一个用于实施和评估推荐算法和技术的新框架。
* [LibRec](https://github.com/guoguibing/librec)：LibRec是一个用于推荐系统的Java库，它实现了一套最先进的推荐算法，旨在解决两个经典的推荐任务：评级预测和项目排名。
* [LensKit](https://github.com/lenskit/lenskit)：LensKit是协同过滤算法的实现，也是一组用于对算法进行基准测试的工具，由明尼苏达大学开源。
* [CARSKit](https://github.com/irecsys/CARSKit)：CARSKit是一款基于Java的开源上下文感知推荐引擎。
* [Open-NARS](https://github.com/opennars/opennars)：Open-NARS是NARS的开源版本，NARS是一个通用AI系统，设计为推理系统框架。
* [Neo4j Reco](https://github.com/graphaware/neo4j-reco)：GraphAware Neo4j推荐引擎是一个基于Neo4j构建高性能复杂推荐引擎的库。

## 逻辑编程

* [LogicNG](https://github.com/logic-ng/LogicNG)：LogicNG是一个用于创建、操作和求解布尔和伪布尔公式的Java库，它包括MiniSAT、Glucose、PBLib或OpenWBO等流行工具的纯Java实现。
* [NeuraLogic](https://github.com/GustikS/NeuraLogic)：该框架的核心是一种自定义语言，你可以使用它来编写可微分程序来编码你的学习场景，类似于经典的深度学习框架。
* [Formulog](https://github.com/HarvardPL/formulog)：Formulog通过构建和推理SMT公式的机制以及一些一阶函数编程来扩展逻辑编程语言Datalog，由哈佛编程语言研究小组开源。
* [Alpha](https://github.com/alpha-asp/Alpha)：Alpha是一个答案集编程(ASP)系统：它读取逻辑程序(一组逻辑规则)并计算相应的答案集，由维也纳工业大学开源。
* [ASP4J](https://github.com/hbeck/asp4j)：Java的答案集编程求解器接口。

## 多智能体

* [Jason](https://github.com/jason-lang/jason)：Jason是AgentSpeak扩展版本的成熟解释器，AgentSpeak是一种面向BDI代理的逻辑编程语言，由圣卡塔琳娜联邦大学开源。
* [JaCaMo](https://github.com/jacamo-lang/jacamo)：JaCaMo项目旨在通过提供一个集成工具和语言的开发平台来推广面向多智能体编程(MAOP)方法。
* [BURLAP](https://github.com/jmacglashan/burlap)：BURLAP是一个Java代码库，用于使用和开发单智能体或多智能体规划和学习算法以及伴随它们的域，由布朗大学开源。
* [Yggdrasil](https://github.com/Interactions-HSG/yggdrasil)：使用Vert.x构建的超媒体多代理系统(MAS)平台，由圣加仑大学交互研究团队开源。

## MATLAB

* [Dimple](https://github.com/analog-garage/dimple)：Dimple是一款用于概率建模、推理和学习的开源软件工具。
* [MFL](https://github.com/HebiRobotics/MFL)：MFL是一个Java库，用于读取和写入与MATLAB的MAT文件格式兼容的MAT文件，由CMU机器人研究所开源。
* [MatFileRW](https://github.com/diffplug/matfilerw)：MatFileRW是一个允许读取和写入MAT文件的库。
* [MatConsoleCtl](https://github.com/diffplug/matconsolectl)：MatConsoleCtl是一个Java API，允许从Java调用MATLAB。

## Jupyter

* [Rapaio Jupyter Kernel](https://github.com/padreati/rapaio-jupyter-kernel)：基于JShell的Java语言Jupyter内核。
* [Kotlin Jupyter](https://github.com/Kotlin/kotlin-jupyter)：Kotlin Jupyter是一个强大的引擎，旨在增强你的Kotlin REPL体验；它提供对执行代码单元、提供基本代码完成和分析错误的支持。
* [Ganymede](https://github.com/allen-ball/ganymede)：Ganymede是基于Java Shell工具JShell的Jupyter Notebook Java内核。
* [Almond](https://github.com/almond-sh/almond)：Almond是Jupyter的Scala内核。
* [Apache Toree](https://github.com/apache/incubator-toree)：Toree是Juypter Notebook内核，主要目标是为使用Scala语言连接和使用Spark的交互式应用程序提供基础。
* [IJava](https://github.com/SpencerPark/IJava)：用于执行Java代码的Jupyter内核。
* [SciJava Kernel](https://github.com/scijava/scijava-jupyter-kernel)：基于BeakerX的已失效内核。
* [BeakerX](https://github.com/twosigma/beakerx)：BeakerX是JVM内核和交互式小部件的集合，用于绘图、表格、自动翻译以及Jupyter Notebook和Jupyter Lab版本1.2.x和2.x的其他扩展。
* [JVM Repr](https://github.com/jupyter/jvm-repr)：用于将JVM对象转换为MIME类型表示的API，适用于Jupyter生态系统。
* [JNotebook](https://github.com/cyrilou242/jnotebook)：JNotebook是一个现代的Java Notebook系统，JNotebook解释Java JShell文件并将它们呈现为笔记本。

## 元启发式框架

* [Mork](https://github.com/mork-optimization/mork)：Mork是一个使用JVM开发NP-Hard问题方法的框架，由胡安卡洛斯国王大学开源。
* [jMetal](https://github.com/jMetal/jMetal)：jMetal是一个基于Java的框架，用于使用元启发式进行多目标优化，由马拉加大学开源。
* [SEAGE](https://github.com/seage/seage)：SEAGE是一个用于元启发式算法协作的超启发式框架，由捷克理工大学开源。

## 机器人开发

* [WPILib](https://github.com/wpilibsuite/allwpilib)：该仓库包含HAL、WPILibJ和WPILibC项目，这些是为roboRIO创建机器人程序的核心库，由伍斯特理工学院开源。
* [GRIP](https://github.com/WPIRoboticsProjects/GRIP)：GRIP是一款用于快速原型设计和部署计算机视觉算法的应用程序，主要用于机器人应用，由伍斯特理工学院开源。
* [FlashLib](https://github.com/Flash3388/FlashLib)：FlashLib是一个Java机器人软件开发库，最初旨在改进和帮助FRC团队，但现在旨在支持非FRC机器人甚至其他用途。
* [OpenTCS](https://github.com/openTCS/opentcs)：OpenTCS是一个用于控制自动引导车辆(AGV)和移动机器人车队的免费平台，由弗劳恩霍夫物流研究所维护。
* [PathPlanner](https://github.com/mjansen4857/pathplanner)：PathPlanner是3015团队创建的FRC机器人运动轨迹生成器。
* [IHMC Open Robotics Software](https://github.com/ihmcrobotics/ihmc-open-robotics-software)：机器人软件具有腿式运动算法和基于动量的优化控制器核心；世界级机器人的支持软件，包括人形、跑鸟、外骨骼、机甲等，由IHMC机器人实验室开源。
* [MyRobotLab](https://github.com/MyRobotLab/myrobotlab)：用于机器人和创意机器控制的开源Java框架。
* [Bag Database](https://github.com/swri-robotics/bag-database)：Bag Database是一个基于Web的应用程序，可监视ROS bag文件的目录、解析其元数据，并提供友好的Web界面来搜索包、下载包以及在其上运行后处理脚本，由美国西南研究所开源。
* [ROS 2 Java](https://github.com/ros2-java/ros2_java)：这是一组使开发人员能够为JVM和Android编写ROS 2应用程序的项目。
* [YAGSL](https://github.com/BroncBotz3481/YAGSL)：YAGSL是一个由现任和前任BroncBotz导师为所有FRC团队开发的转向库。
* [Caliko](https://github.com/FedUni/caliko)：Caliko库是Java语言FABRIK逆运动学(IK)算法的实现，由澳大利亚联邦大学开源。
* [JBullet](http://jbullet.advel.cz/)：JBullet是Bullet物理库的Java端口。
* [FTCVision](https://github.com/lasarobotics/FTCVision)：FTCVision是基于OpenCV的FTC计算机视觉库，由FRC 418团队开源。
* [Maple SIM](https://github.com/Shenzhen-Robotics-Alliance/maple-sim)：利用物理引擎将FRC Java机器人模拟提升到新水平，由深圳市机器人产业联盟开源。
* [JROSBridge](https://github.com/rctoris/jrosbridge)：原生Java EE ROSBridge客户端，由伍斯特理工学院开源。
* [Robot Overlord](https://github.com/MarginallyClever/Robot-Overlord-App)：Robot Overlord是一款机器人3D控制软件，由Marginally Clever Robots开源。
* [BowlerStudio](https://github.com/CommonWealthRobotics/BowlerStudio)：BowlerStudio是一款机器人开发应用程序，它将脚本和设备管理与强大的控制和处理功能结合在一起。
* [EV3Dev-lang-Java](https://github.com/ev3dev-lang-java/ev3dev-lang-java)：EV3Dev-lang-Java是一个学习Java并使用EV3Dev和LeJOS方式支持的硬件为Mindstorms机器人创建软件的项目。
* [Firmata4j](https://github.com/kurbatov/firmata4j)：Firmata4j是用Java编写的Firmata客户端库，该库允许从你的Java程序控制运行Firmata协议的Arduino。
* [StuyLib](https://github.com/StuyPulse/StuyLib)：StuyLib是一个FRC库/工具包，其中包括许多不同的编程实用程序；它包括游戏手柄库、Limelight库、数字滤波器/流库以及许多其他与数学和编程相关的实用程序，由史岱文森高中的FIRST机器人团队开发。
* [GradleRIO](https://github.com/wpilibsuite/GradleRIO)：GradleRIO是一个功能强大的Gradle插件，允许参加FIRST机器人竞赛的团队生成和构建他们的代码，由伍斯特理工学院开源。
* [PhotonVision](https://github.com/PhotonVision/photonvision)：PhotonVision是FIRST机器人竞赛的免费、快速且易于使用的计算机视觉解决方案。
* [EasyOpenCV](https://github.com/OpenFTC/EasyOpenCV)：在FTC机器人上使用OpenCV的简单方法。
* [AdvantageKit](https://github.com/Mechanical-Advantage/AdvantageKit)：AdvantageKit是由Team 6328开发的日志记录、遥测和重放框架。
* [Robo4J](https://github.com/Robo4J/robo4j)：Robo4J提供了一种简单的方法来开始构建自定义硬件并为其创建在JVM上运行的软件。
* [Makelangelo](https://github.com/MarginallyClever/Makelangelo-software)：Makelangelo软件是一个Java程序，可为CNC绘图仪准备艺术品，最初是为Makelangelo艺术机器人设计的。
* [FTCLib](https://github.com/FTCLib/FTCLib)：FTCLib是一个旨在成为FTC编程所需的唯一库的库。
* [MuJoCo-Java](https://github.com/CommonWealthRobotics/mujoco-java)：MuJoCo物理系统的Java JNI绑定。
* [Road Runner](https://github.com/acmerobotics/road-runner)：一个简单的Kotlin库，用于规划专为FTC设计的2D移动机器人路径和轨迹。
* [FtcRobotController](https://github.com/FIRST-Tech-Challenge/FtcRobotController)：该仓库包含用于构建Android应用程序以控制FIRST Tech Challenge竞赛机器人的源代码。
* [IIWA STACK](https://github.com/IFL-CAMP/iiwa_stack)：适用于KUKA LBR IIWA R800/R820(7/14公斤)的ROS Indigo/Kinetic元包。
* [MASON](https://github.com/eclab/mason)：MASON是一个基于Java的快速代理模拟库核心，旨在成为大型定制Java模拟的基础，并为许多轻量级模拟需求提供足够的功能，由乔治梅森大学开源。
* [VirtualRoBot](https://github.com/Beta8397/virtual_robot)：VirtualRoBot是一款2D机器人模拟器，帮助初学者学习FTC Robotics的Java编程，由FTC Team 8397 Beta开源。
* [Pedro Pathing](https://github.com/Pedro-Pathing/PedroPathing)：Pedro Pathing是一种先进的反应矢量跟随器，由FTC团队10158开发，旨在彻底改变机器人的自主导航。
* [Third Coast](https://github.com/strykeforce/thirdcoast)：适用于FRC机器人的Third Coast转向驱动和遥测API，由Stryke Force FRC 2767机器人团队开发。
* [MAPF](https://github.com/J-morag/MAPF)：几种MAPF算法的Java实现。

## 数学库

* [SuanShu](https://github.com/aaiyer/SuanShu)：SuanShu是一个Java数学库，用于数值分析、统计、求根、线性代数、优化等。
* [Colt](https://dst.lbl.gov/ACSSoftware/colt/)：Java中用于高性能科学计算的库，它包含用于数据分析、线性代数、多维数组、傅里叶变换、统计和直方图的有效算法，由欧洲核子研究中心开发。
* [Apache Commons Math](https://github.com/apache/commons-math)：Commons Math是一个开源的数学库，提供了一系列基础数学算法和高级数学功能。
* [Apache Commons Numbers](https://github.com/apache/commons-numbers)：Commons Numbers提供数字类型和实用程序的实现。
* [Eclipse January](https://github.com/eclipse/january)：January是一组用于在Java中处理数值数据的库，它部分受到NumPy的启发，旨在提供类似的功能。
* [ELEFUNT](http://www.math.utah.edu/~beebe/software/java/)：ELEFUNT附带了一个扩展了java.lang.Math的新类库，以及用于数字输出格式化的新类库，由犹他大学开源。
* [JNT](https://math.nist.gov/jnt/)：JNT包含计算内核的坚实基础，可以帮助引导开发Java中复杂数值应用程序的工作，由美国国家标准与技术研究院开源。
* [JUMP](https://sourceforge.net/projects/jump-math/)：JUMP是一个基于Java的可扩展高精度数学包，包括对基于分数的计算的支持，支持转换为浮点数和BigDecimal。
* [JSci](https://jsci.sourceforge.net/)：JSci是一组免费的Java包，目的是以最自然的方式概括科学方法/原理，由杜伦大学开源。
* [JavaPlex](https://github.com/appliedtopology/javaplex)：JavaPlex库实现了计算和应用拓扑中的持久同源性和相关技术，由斯坦福大学开源。
* [Jampack](https://math.nist.gov/pub/Jampack/Jampack/AboutJampack.html)：Jampack是一个协作类的集合，旨在在Java应用程序中执行矩阵计算，由马里兰大学和美国国家标准与技术研究院开发。
* [JAMA](https://math.nist.gov/javanumerics/jama/)：JAMA是Java的基本线性代数包，它提供了用于构造和操作真实的稠密矩阵的用户级类，由马里兰大学和美国国家标准与技术研究院开发。
* [Symja](https://github.com/axkr/symja_android_library)：Symja是计算机代数语言和符号数学库，用纯Java实现的流行算法的集合。
* [Jeigen](https://github.com/hughperkins/jeigen)：Jeigen提供了高性能C++矩阵库Eigen的包装器。
* [Graphulo](https://github.com/Accla/graphulo)：Graphulo是一个用于Accumulo数据库的Java库，提供服务器端稀疏矩阵数学原语，支持更高级别的图形算法和分析，MIT开源。
* [Orbital](https://github.com/LS-Lab/orbital)：Orbital是一个Java类库，为逻辑、数学和计算机科学提供面向对象的表示和算法，由卡尔斯鲁厄理工学院开源。
* [Jafama](https://github.com/jeffhain/jafama)：Jafama是一个Java库，旨在提供更快版本的java.lang.Math处理，最终代价是1e-15ish精度误差，但仍能正确处理特殊情况。
* [Ryu](https://github.com/ulfjack/ryu)：该项目包含使用最短、固定%f和科学%e格式将IEEE-754浮点数转换为十进制字符串的例程。
* [Euclid](https://github.com/ihmcrobotics/euclid)：Euclid是一个解决向量数学和几何问题的通用库，由IHMC机器人实验室开源。
* [Ojalgo](https://github.com/optimatika/ojAlgo)：ojAlgo是用于数学、线性代数和优化的开源Java代码。
* [JNA GMP](https://github.com/square/jna-gmp)：GNU多精度算术库的Java JNA包装器，由Square开源。
* [KotlinGrad](https://github.com/breandan/kotlingrad)：JVM的类型安全符号微分，由蒙特利尔大学开源。
* [Apfloat](https://github.com/mtommila/apfloat)：Apfloat是一个高性能任意精度算术库，你可以用它进行数百万位精度的计算。
* [Cojac](https://github.com/Cojac/Cojac)：Cojac旨在提高Java数字的算术能力，由弗里堡大学开源。
* [BigDecimalMath](https://github.com/eobermuhlner/big-math)：使用任意精度的高级Java BigDecimal数学函数库。
* [UnCommons Maths](https://github.com/dwdyer/uncommons-maths)：Java的随机数生成器、概率分布、组合学和统计库。
* [NM Dev](https://nm.dev/)：NM Dev是一个数值库，涵盖了广泛的算法，例如线性代数、微积分、微分方程、无约束和约束优化、统计学和极值理论。
* [EJML](https://github.com/lessthanoptimal/ejml)：EJML是一个用Java编写的快速且易于使用的线性代数库，适用于稠密、稀疏、实数和复杂矩阵。
* [La4j](https://github.com/vkostyukov/la4j)：La4j是一个开源的100% Java库，提供线性代数基元(矩阵和向量)和算法。
* [Neanderthal](https://github.com/uncomplicate/neanderthal)：Neanderthal是一个用于快速矩阵和线性代数计算的Clojure库，基于针对CPU和GPU的高度优化的BLAS和LAPACK计算例程的原生库。
* [Hacktoberfest Mathematics](https://github.com/BaReinhard/Hacktoberfest-Mathematics)：数学公式和函数的脚本和/或程序库。
* [Decimal4j](https://github.com/tools4j/decimal4j)：用于基于长整型的快速定点算术的Java库，支持最多18位小数。
* [DSI Utils](https://github.com/vigna/dsiutils)：DSI Utils是过去20年在米兰大学信息科学系开发的项目中积累的工具类。
* [KMath](https://github.com/SciProgCentre/kmath)：Kotlin数学扩展库，由莫斯科物理技术学院开源。
* [ParallelColt](https://github.com/rwl/ParallelColt)：Parallel Colt是Colt的多线程版本，由欧洲核子研究组织开源。
* [LIBLINEAR](https://github.com/bwaldvogel/liblinear-java)：LIBLINEAR的Java版本，LIBLINEAR是一个用于解决大规模正则化线性问题分类、回归和异常值检测的简单包。
* [JTransforms](https://github.com/wendykierp/JTransforms)：JTransforms是第一个用纯Java编写的开源多线程FFT库。
* [Jblas](https://github.com/jblas-project/jblas)：Jblas是一个Java矩阵库，它使用现有的高性能BLAS和LAPACK库(如ATLAS)，由柏林工业大学开源。
* [Marlin](https://github.com/PasaLab/marlin)：在Spark之上构建的分布式矩阵运算库，由南京大学开发。
* [SymJava](https://github.com/yuemingl/SymJava)：SymJava是一个用于符号数值计算的Java库。
* [JAutoDiff](https://github.com/uniker9/JAutoDiff)：JAutoDiff是一个用100%纯Java编写的自动微分库。
* [Hipparchus](https://github.com/Hipparchus-Math/hipparchus)：Hiparchus项目是一个轻量级、独立的数学和统计组件库，可解决Java编程语言中无法解决的最常见问题。
* [Ptolemaeus](https://gitlab.com/lmco/ptolemaeus)：Ptolemaeus是一个Java数学库，扩展了Hipparchus库，由洛克希德马丁公司开发。
* [Universal Java Matrix Package](https://github.com/ujmp/universal-java-matrix-package)：UJMP是一个开源库，用于Java中的密集和稀疏矩阵计算以及线性代数。
* [SSJ](https://github.com/umontreal-simul/ssj)：SSJ是一个用于随机模拟的Java库，由蒙特利尔大学开发。
* [OwlPack](https://www.cs.rice.edu/%7Ezoran/OwlPack/)：OwlPack是一个多态、面向对象风格的Java通用线性代数库，基于标准Fortran LINPACK库，由莱斯大学开源。
* [Java Algebra System](https://github.com/kredel/java-algebra-system)：Java代数库，由曼海姆大学开源。
* [Matrix Toolkits Java](https://github.com/fommil/matrix-toolkits-java)：MTJ是一个用于开发线性代数应用程序的高性能库。
* [NetLib Java](https://github.com/fommil/netlib-java)：NetLib-Java是低级BLAS、LAPACK和ARPACK的包装器，其执行速度与带有纯JVM回退的C/Fortran接口一样快。
* [BigInt](https://github.com/tbuktu/bigint)：这是java.math.BigInteger的改进版本，它使用快速算法来乘除大数。
* [Vectorz](https://github.com/mikera/vectorz)：用于Java的快速双精度向量和矩阵数学库，基于N维数组的概念。
* [BigDecimal Utils](https://github.com/mortezaadi/bigdecimal-utils)：用于比较BigDecimal的工具库。
* [FastDoubleParser](https://github.com/wrandelshofer/FastDoubleParser)：该项目提供了double、float、BigDecimal和BigInteger值的解析器，double和float解析器针对最常见输入的速度进行了优化。
* [Tensorics](https://github.com/tensorics/tensorics-core)：Tensorics是一个用于多维数据处理的Java框架。
* [GLPK](https://winglpk.sourceforge.net/)：GLPK软件包提供了用于大规模线性规划(LP)和混合整数规划(MIP)的求解器。
* [Jape](https://github.com/RBornat/jape)：Jape是一个可配置的证明计算器，支持推理系统中形式证明的交互式发现。
* [Math](https://github.com/SpongePowered/math)：用于Java的不可变数学库，提供数学类型、快速三角函数、向量、矩阵、复数、四元数和操作链之间的轻松转换，重点关注游戏和计算机图形。
* [F2J](https://sourceforge.net/projects/f2j)：F2J项目的目标是为最初用Fortran编写的数值库(特别是BLAS和LAPACK)提供Java API，由田纳西大学开源。
* [NumJ](https://github.com/J-Libraries/numJ)：NumJ是一个受NumPy启发的Java库，提供对多维数组和数学运算的支持。
* [Netlib](https://github.com/luhenry/netlib)：该项目提供了BLAS、LAPACK和ARPACK子例程的多个Java实现，支持Java 8+，它通过OpenBLAS和Intel MKL等本机实现提供BLAS、LAPACK和ARPACK的硬件加速。
* [DDoggeg](https://github.com/lessthanoptimal/ddogleg)：DDoggeg是一个高性能Java库，用于非线性优化、稳健模型拟合、多项式求根、排序等。
* [Catalano](https://github.com/DiegoCatalano/Catalano-Framework)：Catalano框架是一个用于Java和Android的科学计算框架。
* [Shared Scientific Toolbox](https://carsomyr.github.io/shared/)：SST是基础科学库的集合，其主要目的是充当所涉及的科学计算的高度特定需求与Java编程语言的更传统方面之间的桥梁。
* [Jspline+](https://www.mathematik.hu-berlin.de/~lamour/software/JAVA/J_Spline/overview-summary.html)：Jspline+是新西伯利亚计算数学和数学地球物理研究所开发的Java样条逼近库，它包含离散网格上的单变量和多元样条近似的类，以及核心矩阵和线性系统解类。
* [DynaHist](https://github.com/dynatrace-oss/dynahist)：Java动态直方图库。
* [Curves API](https://github.com/virtuald/curvesapi)：实现在一组控制点上定义自身的各种数学曲线。
* [NdArray Java](https://github.com/tensorflow/java-ndarray)：NdArray公开了用于在Java中操作N维空间中的数据的实用程序，由Tensorflow开源。
* [Histogram](https://github.com/bigmlcom/histogram)：该项目是Ben-Haim的流式并行决策树中描述的流式一次性直方图的实现。
* [Java Math Library](https://github.com/TilmanNeumann/java-math-library)：一个专注于数论和整数分解的Java数学库。
* [LAML](https://sites.google.com/site/qianmingjie/home/toolkits/laml)：LAML是一个独立的纯Java线性代数和机器学习库。
* [JLinAlg](https://github.com/JLinAlg/JLinAlg)：JLinAlg是一个开源且易于使用的线性代数Java库。

## 本体库

* [OWLAPI](https://github.com/owlcs/owlapi)：OWL API是用于创建、操作和序列化OWL本体的Java API，由曼彻斯特大学开源。
* [Apache Jena](https://github.com/apache/jena)：Jena是一个免费的开源Java框架，用于构建语义Web和链接数据应用程序，最初由惠普实验室开发。
* [Karma](https://github.com/usc-isi-i2/Web-Karma)：Karma是一种信息集成工具，使用户能够快速轻松地集成来自各种数据源的数据，由南加州大学信息科学研究所开源。
* [Widoco](https://github.com/dgarijo/Widoco)：WIDOCO是一个带有本体文档的HTML模板逐步生成器，它使用LODE环境来创建部分模板，由芬欧汇川大学本体工程组开发。
* [Ontop](https://github.com/ontop/ontop)：Ontop是一个虚拟知识图谱系统，它将任意关系数据库的内容公开为知识图，由博尔扎诺自由大学开源。
* [Scowl](https://github.com/phenoscape/scowl)：Scowl提供了Scala DSL，允许使用OWL API以声明式方法编写OWL表达式和公理。
* [DL-Learner](https://github.com/SmartDataAnalytics/DL-Learner)：DL-Learner是一个用于执行丰富语义背景知识的机器学习框架，由德累斯顿工业大学开源。
* [ROBOT](https://github.com/ontodev/robot)：ROBOT是一个用于自动化本体开发任务的命令行工具和库，重点是开放生物和生物医学本体。
* [SciGraph](https://github.com/SciGraph/SciGraph)：SciGraph旨在将本体和使用本体描述的数据表示为Neo4j图。
* [OWL2VOWL](https://github.com/VisualDataWeb/OWL2VOWL)：转换WebVOWL的本体。
* [LogMap](https://github.com/ernestojimenezruiz/logmap-matcher)：LogMap是一个高度可扩展的本体匹配系统，具有“内置”推理和不一致修复功能，由伦敦大学城市学院开源。
* [Openllet](https://github.com/Galigator/openllet)：Openllet提供了检查本体一致性、计算分类层次结构、解释推论以及回答SPARQL查询的功能，由巴黎第十一大学开源。
* [ELK](https://github.com/liveontologies/elk-reasoner)：ELK是一个本体推理器，旨在支持OWL 2 EL配置文件，由乌尔姆大学人工智能研究所和牛津大学计算机科学系知识表示和推理小组开发。
* [AMIE](https://github.com/dig-team/amie)：AMIE是一个在知识库上挖掘Horn规则的系统，由巴黎电信学院开源。
* [OWLTools](https://github.com/owlcollab/owltools)：OWLTools是OWL API之上的便捷Java API。
* [Slib](https://github.com/sharispe/slib)：Slib是一个致力于基于文本和/或本体处理的语义数据挖掘的Java库。
* [OBOGraphs](https://github.com/geneontology/obographs)：该仓库包含用于本体交换的JSON/YAML格式规范，以及参考Java对象模型和OWL转换器。
* [Ontmalizer](https://github.com/srdc/ontmalizer)：Ontmalizer自动执行XML模式(XSD)和XML数据到RDF/OWL的全面转换。
* [O'FAIRe](https://github.com/agroportal/fairness)：O'FAIRe是一种开源公平性评估方法和工具，适用于D2KAB和FooSIN项目中开发的本体、词汇和语义资源，由蒙彼利埃大学开源。
* [LODE](https://github.com/essepuntato/LODE)：实时OWL文档环境，用于将OWL本体转换为HTML人类可读页面，由博洛尼亚大学开源。
* [CEL](https://github.com/julianmendez/cel)：CEL是一种用于大规模生物医学本体的轻量级描述逻辑推理器。
* [JCEL](https://github.com/julianmendez/jcel)：JCEL是描述逻辑EL+的推理器，它使用OWL API，可以用作Protege的插件。
* [Racer](https://github.com/ha-mo-we/Racer)：Racer是一个知识表示系统，它为描述逻辑SRIQ(D)实现了高度优化的表格演算。
* [Pellet](https://github.com/stardog-union/pellet)：Pellet是Java中的OWL 2推理机，提供开源和商业许可、商业支持，由Complexible开发。
* [OPPL 2](https://sourceforge.net/projects/oppl2/)：OPPL 2是OPPL(本体预处理语言)的第二个版本，它是一种旨在修改OWL本体的语言。
* [RDF Toolkit](https://github.com/edmcouncil/rdf-toolkit)：RDF Toolkit是一个用于读写多种格式的RDF文件的工具。
* [IDMP](https://github.com/edmcouncil/idmp)：该项目包含基于药品识别ISO标准构建的OWL本体。
* [OntoBrowser](https://github.com/Novartis/ontobrowser)：OntoBrowser是一个基于Web的应用程序，用于管理本体。
* [OntoGraph](https://github.com/NinePts/OntoGraph)：OWL本体绘图程序。
* [Ontology Modeling Language](https://github.com/opencaesar/oml)：该仓库用于OML抽象语法、文本语法、图形语法、API和规范。
* [ONT-API](https://github.com/owlcs/ont-api)：ONT-API是一个以RDF为中心的Java库，可与OWL2配合使用，由曼彻斯特大学开源。
* [Phenol](https://github.com/monarch-initiative/phenol)：表型组学和基因组学本体库。
* [STATO](https://github.com/ISA-tools/stato)：STATO是一个通用的统计本体，由牛津大学开源。
* [Sigma](https://github.com/ontologyportal/sigmakee)：Sigma是逻辑理论的集成开发环境，扩展了建议的上层合并本体(SUMO)。
* [IFCtoLBD](https://github.com/jyrkioraskari/IFCtoLBD)：IFCtoLBD将IFC STEP格式的文件转换为链接建筑数据本体。
* [MELT](https://github.com/dwslab/melt)：MELT是一个强大的Maven框架，用于开发、调整、评估和打包本体匹配系统，由德国曼海姆大学开源。
* [JOPA](https://github.com/kbss-cvut/jopa)：JOPA是一个Java OWL持久层框架，旨在以Java方式高效地以编程方式访问OWL2本体和RDF图，由布拉格捷克技术大学开源。
* [KOMMA](https://github.com/komma/komma)：KOMMA是一个RDF对象映射器，也是一个用于管理和编辑RDF、RDFS和OWL的框架，由弗劳恩霍夫机床和成形技术研究所开源。
* [ZOOMA](https://github.com/EBISPOT/zooma)：ZOOMA是一款用于发现最佳本体映射的应用程序，由欧洲分子生物学实验室开发。

## 语义Web

* [Eclipse RDF4J](https://github.com/eclipse-rdf4j/rdf4j)：RDF4J是一个强大的Java框架，用于处理RDF数据，这包括使用RDF和链接数据创建、解析、可扩展存储、推理和查询。
* [VIVO](https://github.com/vivo-project/VIVO)：VIVO是一个可扩展的语义Web应用程序，用于研究发现和展示学术工作。
* [SPARQL](https://github.com/SPARQL-Anything/sparql.anything)：SPARQL Anything是一个用于语义Web重新设计的系统，允许用户使用SPARQL查询任何内容。
* [Strabon](https://strabon.di.uoa.gr/)：Strabon是一个完全实现的语义地理空间数据库系统，由雅典大学开源。
* [LinkedGeoData](https://github.com/GeoKnow/LinkedGeoData)：LinkedGeoData致力于向数据网/语义网添加空间维度，LinkedGeoData使用OpenStreetMap项目收集的信息，并根据关联数据原则将其作为RDF知识库提供，由莱比锡大学开源。
* [YAGO](https://github.com/yago-naga/yago3)：YAGO是一个大型知识库，包含有关人物、城市、国家、电影和组织的一般知识，由巴黎高科电信大学、马克斯普朗克信息研究所以及Ambiverse联合开发。
* [Ripple](https://github.com/joshsh/ripple)：Ripple是一种基于堆栈的函数式查询语言，适用于关联数据和其他RDF数据源。
* [Vitro](https://github.com/vivo-project/Vitro)：Vitro是一个通用的基于Web的本体和实例编辑器，具有可定制的公共浏览功能。
* [Apache Commons RDF](https://github.com/apache/commons-rdf)：Commons RDF旨在为RDF 1.1提供一个通用库，并实现常见Java RDF框架(如RDF4J、Jena)以及其他库(如OWLAPI、Clerezza和其他JVM语言)的实现。
* [Corese](https://github.com/Wimmics/corese)：Corese是一个实现和扩展语义网标准的软件平台，它允许创建、操作、解析、序列化、查询、推理和验证RDF数据。
* [NeoSemantics](https://github.com/neo4j-labs/neosemantics)：NeoSemantics是一个允许在Neo4j中使用RDF的插件。
* [Wikidata Toolkit](https://github.com/Wikidata/Wikidata-Toolkit)：Wikidata Toolkit是一个用于访问Wikidata和其他Wikibase安装的Java库。
* [D2RQ](https://github.com/d2rq/d2rq)：D2RQ平台是一个用于以虚拟只读RDF图形式访问关系数据库的系统，由约翰开普勒林茨大学、HP实验室等组织开源。
* [HeFQUIN](https://github.com/LiUSemWeb/HeFQUIN)：HeFQUIN是一种用于图数据源异构联合的查询联合引擎，目前由林雪平大学开发。
* [RDFUnit](https://github.com/AKSW/RDFUnit)：RDFUnit在测试驱动数据验证本体之上实现，旨在读取和生成仅符合该本体的RDF，由莱比锡大学开源。
* [RDF-File](https://github.com/alipay/rdf-file)：RDF-File是一个处理结构化文本文件的工具组件，由支付宝开源。
* [LodView](https://github.com/LodLive/LodView)：LodView是一个基于Spring和Jena的Java Web应用程序，它是一个能够提供符合W3C标准的IRI解引用的工具。
* [Empire](https://github.com/mhgrove/Empire)：Empire使用SPARQL为RDF数据库提供标准JPA风格的接口。
* [HDT](https://github.com/rdfhdt/hdt-java)：HDT-lib是一个Java库，它实现了RDF HDT(标头-字典-三元组)二进制格式的W3C提交。
* [CARML](https://github.com/carml/carml)：CARML是一个Java库，根据RML规范，将结构化源转换为RDF，如RML映射中声明的那样。
* [JSON2RDF](https://github.com/AtomGraph/JSON2RDF)：流式的通用JSON到RDF转换器。
* [Trellis](https://github.com/trellis-ldp/trellis)：Trellis是一个用于构建可扩展的关联数据应用程序的平台。
* [Apache Rya](https://github.com/apache/rya)：Rya是一个可扩展的RDF存储，它构建在列式索引存储(例如Accumulo)之上，它作为RDF4J的扩展来实现，以提供简单的查询机制(SPARQL、SERQL等)和RDF数据存储(RDF/XML、NTriples等)。
* [ESMF SDK](https://github.com/eclipse-esmf/esmf-sdk)：ESMF SDK包含旨在使用、扩展语义切面元模型(SAMM)或与语义切面元模型集成的所有各方(例如解决方案开发人员、领域专家或OEM)的工件和资源。
* [Pinto](https://github.com/stardog-union/pinto)：Pinto是一个Java框架，用于将JavaBean转换为RDF。
* [Asquare](https://github.com/cognizone/asquare)：语义开发库。
* [qEndpoint](https://github.com/the-qa-company/qEndpoint)：qEndpoint是一个高度可扩展的三重存储，具有全文和GeoSPARQL支持。
* [Parliament](https://github.com/raytheonbbn/parliament)：Parliament是专为语义网设计的高性能三重存储和推理器，由雷神公司开源。
* [jRDF2Vec](https://github.com/dwslab/jRDF2Vec)：jRDF2Vec是RDF2Vec的Java实现，它支持多线程、内存中(或基于磁盘访问)的步行生成和训练，由德国曼海姆大学开源。
* [Mobi](https://github.com/inovexcorp/mobi)：Mobi是一个协作知识图谱平台，供团队和社区开发和发布语义数据和模型。

## 知识图谱

* [DMX](https://github.com/dmx-systems/dmx-platform)：DMX是一个知识构建平台。
* [Renku](https://github.com/SwissDataScienceCenter/renku)：Renku是一个将各种工具捆绑在一起的平台，用于可重复和协作的数据分析项目，由瑞士数据科学中心开源。
* [OpenSPG](https://github.com/OpenSPG/openspg)：OpenSPG是蚂蚁集团与OpenKG合作开发的基于SPG(语义增强可编程图)框架的知识图引擎。
* [Wandora](https://github.com/wandora-team/wandora)：Wandora是一个基于主题图和Java的通用信息提取、管理和发布应用程序。
* [LinkedDataHub](https://github.com/AtomGraph/LinkedDataHub)：LinkedDataHub是一款开源软件，可用于管理数据、创建可视化以及在RDF知识图上构建应用程序。

## 生物信息学

* [Nextflow](https://github.com/nextflow-io/nextflow)：Nextflow是一个工作流程系统，用于创建可扩展、可移植和可重复的工作流程，由西班牙巴塞罗那的生物医学和基因组学研究中心CRG开发。
* [NF TencentCloud](https://github.com/Tencent/nf-tencentcloud)：NF TencentCloud是一个Nextflow插件，旨在为Nextflow工作流引擎添加腾讯云COS对象存储和腾讯健康组学平台执行器适配支持。
* [Cromwell](https://github.com/broadinstitute/cromwell)：Cromwell是一个用于生物信息学的开源工作流程管理系统，由麻省理工学院和哈佛大学布罗德研究所开源。
* [GATK](https://github.com/broadinstitute/gatk)：由麻省理工学院和哈佛大学布罗德研究所开源的下一代基因组分析工具包。
* [QuPath](https://github.com/qupath/qupath)：QuPath是用于生物图像分析的开源软件，由英国女王大学开源。
* [BioJava](https://github.com/biojava/biojava)：BioJava是一个开源项目，致力于提供处理生物数据的Java框架。
* [ADAM](https://github.com/bigdatagenomics/adam)：ADAM是一个库和命令行工具，支持使用Spark跨集群/云计算环境并行进行基因组数据分析，由加州大学伯克利分校、西奈山伊坎医学院、微软研究院等开源。
* [WDL](https://github.com/openwdl/wdl)：WDL是一种开放标准，用于使用人类可读和可写的语法来描述数据处理工作流，最初由布罗德研究所开发。
* [Jvarkit](https://github.com/lindenb/jvarkit)：用于生物信息学的Java实用程序。
* [Picard](https://github.com/broadinstitute/picard)：一组用于操作高通量测序(HTS)数据和格式的Java命令行工具，由麻省理工学院和哈佛大学布罗德研究所开源。
* [CDK](https://github.com/cdk/cdk)：CDK是一个用于化学信息学和生物信息学的开源Java库。
* [InterMine](https://github.com/intermine/intermine)：InterMine是一个强大的开源数据仓库系统，允许用户以最少的努力集成不同的数据源，InterMine为生命科学领域一些最大的数据仓库提供支持，由剑桥大学开源。
* [BBMap](https://github.com/BioInfoTools/BBMap)：用于DNA/RNAseq的BBMap短读对齐器和其他生物信息学工具。
* [AliView](https://github.com/AliView/AliView)：用于对齐、查看和编辑DNA/氨基酸序列的软件，直观、快速且轻量，由乌普萨拉大学开源。
* [ASTRAL](https://github.com/smirarab/ASTRAL)：ASTRAL是一种根据一组无根基因树来估计无根物种树的工具。
* [Pegasus](https://github.com/pegasus-isi/pegasus)：Pegasus是一个可配置系统，用于在各种计算基础设施上映射和执行科学工作流程，由南加州大学信息科学研究所、威斯康星大学麦迪逊分校开源。
* [GloBI](https://github.com/globalbioticinteractions/globalbioticinteractions)：GloBI提供对现有物种相互作用数据集的访问。
* [MOLGENIS](https://github.com/molgenis/molgenis)：MOLGENIS是一个协作开源项目，其目的是为生命科学研究生成出色的软件基础设施，由格罗宁根大学开源。
* [SIRIUS](https://github.com/sirius-ms/sirius)：SIRIUS是一款使用串联质谱法从头鉴定代谢物的软件，由耶拿大学开源。
* [LibLevenshtein](https://github.com/universal-automata/liblevenshtein-java)：有关Levenshtein传感器的各种实用程序。
* [Jannovar](https://github.com/charite/jannovar)：Java中的功能变体文件注释，Jannovar提供了一个用于VCF文件注释的程序，并通过库API公开其功能，由柏林计算生物学开源。
* [PeptideShaker](https://github.com/compomics/peptide-shaker)：PeptideShaker是一个独立于搜索引擎的平台，用于解释来自多个搜索和de novo引擎的蛋白质组学鉴定结果，由根特大学开源。
* [OME](https://github.com/ome/openmicroscopy)：OME开发用于存储和操作生物光学显微镜数据的开源软件和数据格式标准，是欧洲和美国大学、研究机构和行业之间的联合项目。
* [SearchGUI](https://github.com/compomics/searchgui)：SearchGUI是一个高度适应性的开源通用界面，用于配置和运行蛋白质组学搜索和de novo引擎，由根特大学开源。
* [Bio-Formats](https://github.com/ome/bioformats)：Bio-Formats是一个独立的Java库，用于读取和写入生命科学图像文件格式，由开放显微镜环境联盟开发，其中包括威斯康星大学麦迪逊分校、邓迪大学等开发团队。
* [Bio4j](https://github.com/bio4j/bio4j)：Bio4j是一个生物信息学图数据平台，集成了Uniprot KB(SwissProt+Trembl)、Gene Ontology、UniRef(50,90,100)、NCBI Taxonomy和Expasy Enzyme DB中的大部分可用数据。
* [PathVisio](https://github.com/PathVisio/pathvisio)：PathVisio是一款免费的开源通路分析和绘图软件，可以绘制、编辑和分析生物通路，由马斯特里赫特大学和格莱斯顿研究所开发。
* [LibSBOLj](https://github.com/SynBioDex/libSBOLj)：LibSBOLj为合成生物学开放语言(SBOL)提供核心Java接口及其实现。
* [GBIF IPT](https://github.com/gbif/ipt)：IPT是由全球生物多样性信息设施(GBIF)提供的免费开源软件工具，用于通过GBIF网络发布和共享生物多样性数据集。
* [Tetrad](https://github.com/cmu-phil/tetrad)：Tetrad是一款拥有30年历史的免费工具，用于分析因果系统，根据已知的数据和因果操作推断“什么导致什么”。由CMU大学哲学系开发，并在美国国立卫生研究院与匹兹堡大学生物信息学系的支持下进行了改进。
* [MZmine 3](https://github.com/mzmine/mzmine3)：MZmine是一款用于质谱数据处理的开源软件，由芬兰VTT技术研究中心、图尔库生物技术中心开源。
* [Eclipse ChemClipse](https://github.com/eclipse/chemclipse)：Eclipse ChemClipse项目提供化学信息学和生物信息学领域的数据读取和处理功能，由Eclipse科学工作组开源。
* [JNBIS](https://github.com/mhshams/jnbis)：JNBIS是一个用Java编写的库，用于提取和解码NIST(美国国家标准与技术研究所)压缩文件和WSQ(小波标量量化)图像。
* [MesquiteCore](https://github.com/MesquiteProject/MesquiteCore)：Mesquite是一款模块化、可扩展的进化生物学软件，旨在帮助生物学家组织和分析有关生物体的比较数据。
* [Bacting](https://github.com/egonw/bacting)：Bacting是一个基于Bioclipse的化学和生物信息学开源平台，它定义了许多公共领域对象并包装了公共功能，提供了一个独立于工具包的、可编写脚本的解决方案来处理来自生命科学的数据。
* [BridgeDb](https://github.com/bridgedb/BridgeDb)：BridgeDb是一个在各种生物数据库和相关资源之间映射标识符的框架，由曼彻斯特大学、赫瑞瓦特大学、马斯特里赫特大学开源。
* [Wildbook](https://github.com/WildMeOrg/Wildbook)：Wildbook是一个开源软件框架，支持标记重新捕获、分子生态学和社会生态学研究。
* [JSBML](https://github.com/sbmlteam/jsbml)：JSBML是一个社区驱动的项目，旨在创建一个免费、开源、纯Java库，用于读取、写入和操作SBML文件和数据流。
* [BBTools](https://jgi.doe.gov/data-and-tools/software-tools/bbtools/)：BBTools是一套快速、多线程生物信息学工具，专为分析DNA和RNA序列数据而设计。
* [Dockstore](https://github.com/dockstore/dockstore)：Dockstore是一个免费的开源平台，用于共享可重用且可扩展的分析工具和工作流程，由癌症基因组合作实验室开源
* [BioFormats2Raw](https://github.com/glencoesoftware/bioformats2raw)：Bio-Formats图像文件格式到原始格式转换器。
* [Icy](https://gitlab.pasteur.fr/bia/icy)：Icy是一款图像分析软件，主要面向生物图像的分析，由巴斯德研究所开源。
* [OpenChrom](https://github.com/Openchrom/openchrom)：OpenChrom是Lablicate GmbH开发的一款用于分析和可视化质谱和色谱数据的开源工具。
* [MOLGENIS](https://github.com/molgenis/systemsgenetics)：MOLGENIS是一个协作开源项目，其使命是为生命科学研究生成出色的软件基础设施，由格罗宁根大学开源。
* [Micro-Manager](https://github.com/micro-manager/micro-manager)：Micro-Manager是一个控制显微镜硬件的应用程序，例如相机、xy平台、滤光轮等，由加利福尼亚大学旧金山分校开源。
* [FairSIM](https://github.com/fairSIM/fairSIM)：结构照明显微镜(SIM)为荧光显微镜提供了一种快速、温和的超分辨率方法，FairSIM项目旨在为使用SIM的科学家提供一系列免费开源工具和资源，由比勒费尔德大学开源。
* [GC4S](https://github.com/sing-group/GC4S)：GC4S是一个开源库，为Swing提供面向生物信息学的GUI组件集合，由维戈大学开源。
* [VCell](https://github.com/virtualcell/vcell)：VCell是一个建立在中央数据库之上并以Web应用程序形式传播的用于建模细胞生物系统的综合平台，由康涅狄格大学开源。
* [Eoulsan](https://github.com/GenomiqueENS/eoulsan)：Eoulsan是一个基于MapReduce算法的Hadoop实现的多功能框架，致力于分布式计算机上的高吞吐量测序数据分析，由巴黎高等师范学院生物研究所开源。
* [Opal](https://github.com/obiba/opal)：Opal是OBiBa用于生物样本库或流行病学研究的核心数据库应用程序。
* [BioMedICUS](https://github.com/nlpie/biomedicus)：BioMedICUS是一个用于生物医学和临床报告的大规模文本分析和处理的系统，由明尼苏达大学开发。

## 基因组学

* [cBioPortal](https://github.com/cBioPortal/cbioportal)：cBioPortal提供大规模癌症基因组学数据集的可视化、分析和下载，由纪念斯隆-凯特琳癌症中心、丹娜法伯癌症研究院、毕尔肯大学、多伦多玛格丽特公主癌症中心等组织开源。
* [DNAnalyzer](https://github.com/VerisimilitudeX/DNAnalyzer)：致力于彻底改变DNA分析领域，目标是使DNA分析工具的使用更加民主化。
* [IGV](https://github.com/igvteam/igv)：用于Mac、Windows和Linux的桌面基因组可视化工具，由加州大学圣地亚哥分校、麻省理工学院和哈佛大学开源。
* [GKL](https://github.com/Intel-HLS/GKL)：GKL包含GATK和HTSJDK等基因组学应用程序中使用的计算内核的优化版本，由Intel开源。
* [HTSJDK](https://github.com/samtools/htsjdk)：HTSJDK是统一Java库的实现，用于访问用于高通量测序数据的常见文件格式，例如SAM和VCF，由哈佛医学院开源。
* [GRIDSS](https://github.com/PapenfussLab/gridss)：GRIDSS是一个模块软件套件，包含可用于检测基因组重排的工具，由沃尔特和伊丽莎·霍尔医学研究所开源。
* [Artemis](https://github.com/sanger-pathogens/Artemis)：Artemis软件是一套用于基因组浏览和注释的软件工具，由威康桑格研究所开源。
* [MISO](https://github.com/miso-lims/miso-lims)：MISO是一个开源实验室信息管理系统(LIMS)，始于厄勒姆研究所，最近由安大略癌症研究所开发，专门用于跟踪下一代测序实验。
* [P2Rank](https://github.com/rdk/p2rank)：P2Rank是一个独立的命令行程序，可根据蛋白质结构预测配体结合口袋。
* [Apollo](https://github.com/GMOD/Apollo)：Apollo是一个基于Web的协作、实时基因组注释编辑器。
* [OpenCGA](https://github.com/opencb/opencga)：OpenCGA是一个开源项目，旨在为数百TB甚至PB级的基因组规模数据分析提供大数据存储引擎和分析框架，由剑桥大学计算生物学开源。
* [GORpipe](https://github.com/gorpipe/gor)：GORpipe是一种基于基因组有序关系架构的工具，允许在并行执行引擎中使用声明性查询语言分析大量基因组和表型表格数据，由Genuity Science开发。
* [Exomiser](https://github.com/exomiser/Exomiser)：Exomiser是一个Java程序，可以从全外显子组或全基因组测序数据中查找潜在的致病变异，由柏林夏里特大学、桑格研究所开发。
* [HMFTools](https://github.com/hartwigmedical/hmftools)：该存储库包含哈特维格医学基金会全基因组、靶向DNA和全转录组分析流程中使用的工具套件。
* [Drop-seq](https://github.com/broadinstitute/Drop-seq)：用于分析Drop-seq数据的Java和R工具，由麻省理工学院和哈佛大学布罗德研究所开源。
* [Cytoscape](https://cytoscape.org/)：Cytoscape是一个开源的生物信息学软件平台，用于可视化分子相互作用网络并与基因表达谱和其他状态数据集成，最初由西雅图系统生物学研究所开发。
* [BioTapestry](https://biotapestry.systemsbiology.net/)：BioTapestry是一种交互式工具，用于通过网络构建、可视化和共享基因调控网络模型，由西雅图系统生物学研究所与加州理工学院戴维森实验室合作创建。
* [InterProScan](https://github.com/ebi-pf-team/interproscan)：InterPro通过将蛋白质归类为家族并预测域和重要位点来提供蛋白质的功能分析，由欧洲分子生物学实验室开源。
* [Cloud-Pipeline](https://github.com/epam/cloud-pipeline)：与云无关的基因组学分析、科学计算和存储平台。
* [IRIDA](https://github.com/phac-nml/irida)：IRIDA是加拿大基因组流行病学综合快速传染病分析平台。
* [SnpEff](https://github.com/pcingola/SnpEff)：基因组变异注释和功能效果预测工具包。
* [PharmCAT](https://github.com/PharmGKB/PharmCAT)：一种从遗传数据集(以VCF文件表示)中提取所有CPIC指南基因变异、解释变异等位基因并生成报告的工具，由斯坦福大学和宾夕法尼亚大学维护。
* [NGB](https://github.com/epam/NGB)：NGB是一种基于Web的NGS数据查看器，具有独特的结构变异(SV)可视化功能、高性能、可扩展性和云数据支持。
* [Cellbase](https://github.com/opencb/cellbase)：Cellbase是一个集中式数据库，集成了来自多个主要基因组和生物数据库的大量信息，用于基因组注释和临床变异优先级排序，由剑桥大学计算生物学开源。
* [VarSim](https://github.com/bioinform/varsim)：VarSim是用于癌症应用的高通量基因组测序的高保真模拟验证框架，由罗氏开源。
* [FastQC](https://github.com/s-andrews/FastQC)：FastQC是一个旨在发现高通量测序数据集中潜在问题的程序，由巴布拉汉姆研究所生物信息学小组开源。
* [Osprey](https://github.com/donaldlab/OSPREY3)：OSPREY软件包提供了蛋白质设计工具，这是连续灵活性建模、集成建模和具有可证明保证的算法的独特组合，由杜克大学开源。
* [FragPipe](https://github.com/Nesvilab/FragPipe)：FragPipe是一个Java GUI，用于一套计算工具，可对基于质谱的蛋白质组数据进行全面分析，由密歇根大学开源。
* [NGSEP](https://github.com/NGSEP/NGSEPcore)：NGSEP提供了一个对象模型来支持不同类型的DNA高通量测序(HTS)数据分析。
* [GEDCOM X Java](https://github.com/FamilySearch/gedcomx-java)：该项目托管GEDCOM X项目的Java实现，并作为GEDCOM X的参考实现，由FamilySearch开源。
* [GeneMANIA](https://github.com/GeneMANIA/genemania)：GeneMANIA可以帮助你预测你最喜欢的基因和基因组的功能，由多伦多大学开源。
* [Compomics Utilities](https://github.com/compomics/compomics-utilities)：用于计算蛋白质组学的开源Java库，由根特大学开源。
* [PIA](https://github.com/medbioinf/pia)：PIA是一个用于基于MS的蛋白质推断和识别分析的工具箱，由波鸿鲁尔大学开源。

## NGS

* [GATK](https://github.com/broadinstitute/gatk)：由麻省理工学院和哈佛大学布罗德研究所开源的下一代基因组分析工具包。
* [MiXCR](https://github.com/milaboratory/mixcr)：MiXCR是一款通用软件，可快速准确地分析原始T细胞或B细胞受体组测序数据。
* [FgBio](https://github.com/fulcrumgenomics/fgbio)：FgBio是一个用于处理基因组数据尤其是下一代测序数据的命令行工具包。
* [HTSJDK](https://github.com/samtools/htsjdk)：HTSJDK是统一Java库的实现，用于访问用于高通量测序数据的常见文件格式，例如SAM和VCF，由哈佛医学院开源。
* [MISO](https://github.com/miso-lims/miso-lims)：MISO是一个开源实验室信息管理系统(LIMS)，始于厄勒姆研究所，最近由安大略癌症研究所开发，专门用于跟踪下一代测序实验。
* [NGB](https://github.com/epam/NGB)：NGB是一种基于Web的NGS数据查看器，具有独特的结构变异(SV)可视化功能、高性能、可扩展性和云数据支持，由EPAM开源。
* [Eoulsan](https://github.com/GenomiqueENS/eoulsan)：Eoulsan是一个基于MapReduce算法的Hadoop实现的多功能框架，致力于分布式计算机上的高吞吐量测序数据分析，由巴黎高等师范学院生物研究所开源。
* [AdamaJava](https://github.com/AdamaJava/adamajava)：AdamaJava包含与新一代测序(NGS)分析相关的变异调用程序和流程工具的代码，由昆士兰医学研究院伯格霍夫医学研究所的基因组信息学小组开发和维护。

## 医疗平台

* [HAPI FHIR](https://github.com/hapifhir/hapi-fhir)：HAPI FHIR是HL7 FHIR标准的完整实现，用于Java中的医疗保健互操作性。
* [Mirth Connect](https://github.com/nextgenhealthcare/connect)：Mirth Connect是一个基于开源标准的医疗保健集成引擎，它通过接受传入的信息包并根据你提供的规则处理它们来实现两个系统之间的互操作性。
* [Open Hospital](https://github.com/informatici/openhospital)：Open Hospital是一款免费的开源健康信息管理系统(HIMS)软件应用程序。
* [Clinical Quality Language](https://github.com/cqframework/clinical_quality_language)：CQL是用于表达临床知识的HL7标准，可在广泛的临床领域中使用，包括临床决策支持(CDS)和临床质量测量(CQM)。
* [IPF](https://github.com/oehf/ipf)：Camel路由和中介引擎的扩展，为医疗保健领域的消息处理和连接信息系统提供全面支持。
* [HMIS](https://github.com/hmislk/hmis)：HMIS是一个医院信息管理系统，自2004年推出以来一直积极为40多家医疗机构提供服务。
* [HAPI FHIR Core](https://github.com/hapifhir/org.hl7.fhir.core)：适用于FHIR规范的Java核心对象处理代码，带有实用程序(包括验证器)。
* [OpenMRS](https://github.com/openmrs/openmrs-core)：OpenMRS是一个基于患者的医疗记录系统，专注于为提供商提供免费的可定制电子医疗记录系统(EMR)。
* [EHRbase](https://github.com/ehrbase/ehrbase)：EHRbase是一个OpenEHR临床数据存储库，为可互操作的临床应用程序提供基于标准的后端，由德国布伦瑞克工业大学及汉诺威医学院联合开发。
* [DCM4che](https://github.com/dcm4che/dcm4che)：Java中的DICOM实现。
* [Synthea](https://github.com/synthetichealth/synthea)：Synthea是一款合成患者群体模拟器，目标是以各种格式输出合成的、真实的(但不是真实的)患者数据和相关的健康记录。
* [Weasis](https://github.com/nroduit/Weasis)：Weasis是一款独立的基于Web的软件，用于可视化从医疗成像设备获得的图像。
* [OpenICE](https://github.com/mdpnp/mdpnp)：OpenICE是一项旨在创建集成临床环境的社区实施的倡议，该计划不仅包括软件实现，还包括更广泛的临床生态系统的架构，以实现临床研究的新途径。
* [Dicoogle](https://github.com/bioinformatics-ua/dicoogle)：Dicoogle是一款可扩展、独立于平台的开源PACS归档软件，它以更敏捷的索引和检索机制取代了传统的集中式数据库，由阿威罗大学开源。
* [Phoenix CTMS](https://github.com/phoenixctms/ctsms)：Phoenix CTMS是一款大型Web应用程序，将临床研究中使用的数据库软件的功能结合在一个模块化系统中。
* [SIMRS Khanza](https://github.com/mas-elkhanza/SIMRS-Khanza)：适用于医院、诊所、保健中心、私人医生的软件，已在印度尼西亚1000多家医院使用。
* [OpenELIS Global 2](https://github.com/I-TECH-UW/OpenELIS-Global-2)：OpenELIS Global是专为公共卫生实验室量身定制的开放式企业级实验室信息系统软件，由华盛顿大学开源。
* [LinuxForHealth FHIR](https://github.com/LinuxForHealth/FHIR)：LinuxForHealth FHIR服务器(以前称为IBM FHIR服务器)是HL7 FHIR规范的模块化Java实现，支持版本R4和R4B，重点关注性能和可配置性。
* [Pathling](https://github.com/aehrc/pathling)：Pathling是一组工具，可让你在健康数据分析中更轻松地使用FHIR和临床术语，由澳大利亚电子健康研究中心开源。

## 化学库

* [CDK](https://github.com/cdk/cdk)：CDK是一个用于化学信息学和生物信息学的开源Java库。
* [Eclipse ChemClipse](https://github.com/eclipse/chemclipse)：Eclipse ChemClipse项目提供化学信息学和生物信息学领域的数据读取和处理功能，由Eclipse科学工作组开源。
* [JChemPaint](https://github.com/JChemPaint/jchempaint)：JChemPaint是使用CDK开发的2D化学结构编辑器和查看器。
* [OpenChemLib](https://github.com/Actelion/openchemlib)：OpenChemLib是基于Java的框架，提供化学信息学核心功能和用户界面组件。
* [ChemicalTagger](https://github.com/BlueObelisk/chemicaltagger)：ChemicalTagger是化学领域语义文本挖掘的工具。
* [JMol](https://github.com/BobHanson/Jmol-SwingJS)：JMol是一个开源Java/SwingJS应用程序，用于可视化和分析具有化学品特征的3D分子结构、晶体、材料和生物分子。
* [SMSD](https://github.com/asad/smsd)：SMSD是一个基于Java的软件库，用于查找小分子之间的最大公共子图(MCS)/子结构，由欧洲生物信息学研究所开源。
* [Indigo](https://github.com/epam/Indigo)：通用化学信息学工具包、实用程序和数据库搜索工具。
* [LaMa4J](https://lama4j.di.unimi.it/)：LaMa4J是一组实现许多晶格和晶格运算的Java类。
* [Toxtree](https://toxtree.sourceforge.net/)：Toxtree是一个功能齐全、灵活、用户友好的开源应用程序，它能够通过应用决策树方法来估计毒性危害。
* [OPSIN](https://github.com/dan2097/opsin)：OPSIN是一个用于IUPAC名称到结构转换的Java库，可为有机化学命名法提供高召回率和精确度，由剑桥大学化学系开源。
* [MolVec](https://github.com/ncats/molvec)：NCATS(化学) OCR引擎，可以将化学图像矢量化为化学对象，并尽可能保留2D布局，由国家转化科学促进中心开源。
* [AMBIT](https://ambit.sourceforge.net/)：AMBIT为化学物质、结构和纳米材料提供化学信息学数据管理。
* [DataWarrior](https://github.com/thsa/datawarrior)：DataWarrior是一个用于交互式数据分析和可视化的程序。

## 工作流

* [Camunda](https://github.com/camunda/camunda-bpm-platform)：Camunda Platform是一个灵活的工作流程和流程自动化框架，其核心是在JVM内运行的原生BPMN 2.0流程引擎。
* [Activiti](https://github.com/Activiti/Activiti)：Activiti是一个轻量级工作流程和BPM平台，面向业务人员、开发人员和系统管理员，由Alfresco开源。
* [Flowable](https://github.com/flowable/flowable-engine)：Flowable为开发人员、系统管理员和业务用户提供紧凑且高效的工作流程和BPM平台。
* [jBPM](https://github.com/kiegroup/jbpm)：jBPM是一个用于构建业务应用程序以帮助自动化业务流程和决策的工具包，JBoss社区开源。
* [jDMN](https://github.com/goldmansachs/jdmn)：jDMN为DMN中指定的决策模型提供执行引擎，这些决策可以解释或翻译为Java并在JVM上执行，由高盛银行开源。
* [JDEasyFlow](https://github.com/JDEasyFlow/jd-easyflow)：JDEasyFlow是京东开源的一个通用流程编排组件，适用于服务编排、工作流、审计等，具有易用、灵活、易扩展的特点。
* [ActiveMatrix BPM](https://docs.tibco.com/products/tibco-activematrix-bpm-4-3-0)：TIBCO ActiveMatrix BPM用于开发、部署、执行和管理以业务流程管理为中心的应用程序。
* [Piper](https://github.com/runabol/piper)：Piper是一个基于Spring Boot构建的开源分布式工作流引擎，设计非常简单。
* [Maestro](https://github.com/lucidity-labs/maestro)：Maestro是一个简单但功能强大、持久的工作流库。
* [ByteChef](https://github.com/bytechefhq/bytechef)：ByteChef是一个开源、低代码、可扩展的API集成和工作流自动化平台。
* [Compileflow](https://github.com/alibaba/compileflow)：Compileflow是一个非常轻量级、高性能、可集成和可扩展的流程引擎，由阿里开源。
* [Azkaban](https://github.com/azkaban/azkaban)：Azkaban是LinkedIn创建的批处理工作流作业调度程序，用于运行Hadoop作业。
* [Apromore](https://github.com/apromore/ApromoreCore)：Apromore用于流程挖掘和预测流程分析，由墨尔本大学、塔尔图大学等开源。
* [Turbo](https://github.com/didi/turbo)：Turbo是一款轻量级流程引擎服务框架，可作为底层服务支持各类流程设计、低代码设计、工作流、服务编排等场景，由滴滴开源。
* [JEHC BPM](https://gitee.com/jehc/JEHC-BPM)：JEHC BPM是小诗科技公司研发的一套开源工作流平台。
* [Bulbasaur](https://github.com/alibaba/bulbasaur)：Bulbasaur是阿里开源的可插拔精简流程引擎，可快速实现流程、审批、业务失败重试等场景。
* [Imixs Workflow](https://github.com/imixs/imixs-workflow)：Imixs Workflow是一个开源工作流引擎，用于在灵活而强大的框架上构建以人为中心的工作流应用程序。
* [Bonita](https://github.com/bonitasoft/bonita-engine)：部署、执行、管理使用Bonita Studio或通过Engine API制作的基于流程的应用程序。
* [JFlow](https://gitee.com/opencc/JFlow)：基于AI的低代码BPM开发平台，由济南驰骋公司开发。
* [WINGS](https://github.com/KnowledgeCaptureAndDiscovery/wings)：WINGS是一个语义工作流系统，可帮助科学家设计计算实验，由南加州大学开源。
* [BAMOE](https://github.com/IBM/bamoe)：IBM BAMOE是用于工作流和决策管理的企业自动化软件。
* [行云流程引擎](https://gitee.com/bestfeng/oa_git_free)：行云流程引擎具备Activiti的常用功能，上手更容易。
* [Emissary](https://github.com/NationalSecurityAgency/emissary)：Emissary是一种基于P2P的数据驱动工作流引擎，运行在异构的、可能广泛分散的多层P2P计算资源网络中，由美国国家安全局开源。
* [Digdag](https://github.com/treasure-data/digdag)：简单、开源、多云工作流程引擎。
* [LittleHorse](https://github.com/littlehorse-enterprises/littlehorse)：LittleHorse是一个高性能的微服务编排引擎，允许开发人员构建可扩展、可维护和可观察的应用程序。
* [AgileBPM](https://gitee.com/agile-bpm/agile-bpm-basic)：快速、简洁且强大的低代码流程开发平台，国产开源。
* [Schedulis](https://github.com/WeBankFinTech/Schedulis)：Schedulis是一个基于LinkedIn的开源项目Azkaban开发的工作流任务调度系统，由微众开源。
* [UFLO2](https://github.com/youseries/uflo)：UFLO是一款基于Spring的纯Java流程引擎，支持并行、动态并行、串行、会签等各种流转方式。
* [NFlow](https://github.com/NitorCreations/nflow)：NFlow是一种经过验证的用于编排业务流程的解决方案，它可以用作微服务编排器(Saga模式)、业务流程引擎或持久有限状态机。
* [FlowLong](https://gitee.com/aizuda/flowlong)：由爱组搭开源的工作流引擎。
* [Nextflow](https://github.com/nextflow-io/nextflow)：Nextflow是一个工作流程系统，用于创建可扩展、可移植和可重复的工作流程，由西班牙巴塞罗那的生物医学和基因组学研究中心CRG开发。
* [Concord](https://github.com/walmartlabs/concord)：Concord是一个工作流服务器，它是使用用户创建的场景和插件将不同系统连接在一起的编排引擎，由沃尔玛开源。
* [DataBuilder](https://github.com/flipkart-incubator/databuilderframework)：DataBuilder框架是一个高级逻辑执行引擎，可用于执行多步骤工作流，该引擎目前为Flipkart的结账系统以及诊断和其他工作流提供支持。
* [RuoYi Activiti](https://gitee.com/shenzhanwang/RuoYi-activiti)：基于Activiti 6.0，集流程设计、流程部署、流程执行、任务办理、流程监控于一体的开源工作流开发平台。
* [F2BPM](https://www.f2bpm.com/)：F2BPM是一款纯国产工作流引擎，遵循WFMC/BPMN2.0的规范，由致博软件公司开发。
* [盘古BPM](https://gitee.com/pangu-dm/pangubpm-dmn)：盘古BPM工作流平台是国内首款开源的互联网决策引擎系统，拥有独立的DMN1.3标准设计器、解析器、决策引擎、支持决策表、DRD、DRG。
* [JsonFlow](https://gitee.com/jackrolling/jsonflow-ui)：简单但强大易用易扩展且适应复杂场景的中国式审批的工作流引擎系统。
* [Smart Flow](https://gitee.com/smartboot/smart-flow)：SmartFlow是一个轻量、灵活的业务流程编排框架，支持业务流程中常见的条件分支控制、子流程、业务组件异步和降级等功能。
* [COPPER](https://github.com/copper-engine/copper-engine)：COPPER是一个开源、强大、轻量且易于配置的工作流引擎，它使用Java作为工作流的描述语言。
* [FlyFlow](https://gitee.com/junyue/flyflow)：FlyFlow借鉴了钉钉与飞书的界面设计理念，致力于打造一款用户友好、快速上手的工作流程工具。
* [Easy Flows](https://github.com/j-easy/easy-flows)：Easy Flows是Java的工作流引擎，它提供简单的API和构建块，使创建和运行可组合工作流程变得轻松。
* [FoxBPM](https://github.com/FoxBPM/FoxBPM)：FoxBPM是一款开源的基于BPMN 2.0标准的工作流引擎，引擎底层直接支持BPMN 2.0国际标准。
* [Taverna](https://github.com/apache/incubator-taverna-engine)：Taverna是一个用于设计和执行工作流的开源软件工具，最初由曼彻斯特大学创建。
* [Yaoqiang BPMN Editor](https://bpmn.sourceforge.net/)：Yaoqiang BPMN Editor是一款开源的业务流程图图形编辑器，符合OMG规范(BPMN 2.0)。
* [Flo](https://github.com/spotify/flo)：Flo是一个轻量级的工作流定义库，由Spotify开发。
* [JWorkflow](https://github.com/danielgerlag/jworkflow)：JWorkflow是一个轻量级的Java工作流库，它支持可插拔的持久性和并发性提供程序，以支持多节点集群。
* [YAWL](https://github.com/yawlfoundation/yawl)：YAWL是一个BPM/工作流系统，基于简洁而强大的建模语言，可处理复杂的数据转换，并与组织资源和外部Web服务完全集成。
* [WarmFlow](https://gitee.com/dromara/warm-flow)：此项目是极其简单的工作流，没有太多设计，代码量少，并且只有6张表，由dromara社区开发。
* [Titanoboa](https://github.com/commsor/titanoboa)：Titanoboa是一个面向JVM的低代码工作流编排平台。
* [Automatiko](https://github.com/automatiko-io/automatiko-engine)：Automatiko可以帮助你基于以众所周知的标准表达的工作流程和决策构建更好的服务和功能。
* [Application Engine](https://github.com/netgrif/application-engine)：Application Engine是一个完全支持低代码语言Petriflow的工作流管理系统。
* [Fixflow](https://github.com/fixteam/fixflow)：Fixflow是一款开源的基于BPMN2.0标准的工作流引擎，由北京百特云享公司开发。
* [HelloDATA BE](https://github.com/kanton-bern/hellodata-be)：HelloDATA BE是一个基于现代数据堆栈的开源工具构建的企业数据平台。
* [Operaton](https://github.com/operaton/operaton)：Operaton是一个原生的BPMN 2.0流程引擎，运行在Java虚拟机中。
* [ZephFlow](https://github.com/fleaktech/zephflow-core)：ZephFlow是一个精简的框架，旨在使无状态数据处理变得简单、可靠和高效。
* [EximeeBPMS](https://github.com/EximeeBPMS/eximeebpms)：EximeeBPMS是一个灵活的工作流和流程自动化框架。

## 编排引擎

* [Conductor](https://github.com/conductor-oss/conductor)：Conductor是Netflix创建的一个平台，用于编排微服务和事件。
* [Kestra](https://github.com/kestra-io/kestra)：Kestra是一个通用的开源编排器，可以简化计划和事件驱动的工作流程。
* [Apache DolphinScheduler](https://github.com/apache/dolphinscheduler)：DolphinScheduler是现代数据编排平台，以低代码敏捷创建高性能工作流程，由易观开源。
* [Maestro](https://github.com/Netflix/maestro)：Maestro是一款通用工作流编排器，为Netflix的数据平台用户提供完全托管的工作流即服务(WAAS)。
* [SmartEngine](https://github.com/alibaba/SmartEngine)：SmartEngine是一个轻量级的业务编排引擎，在阿里内部广泛使用，可以用于在微服务架构中编排多个服务，也可以用于传统的流程审批场景。
* [Cadence](https://github.com/uber/cadence-java-client)：Cadence是分布式、可扩展、持久且高度可用的编排引擎，用于以可扩展和弹性的方式执行异步长时间运行的业务逻辑，由Uber开发。
* [Flowret](https://github.com/americanexpress/unify-flowret)：Flowret是一个基于Java的轻量级编排引擎，由美国运通开源。
* [CloudSlang](https://github.com/CloudSlang/cloud-slang)：CloudSlang是一种基于YAML的语言，用于为CloudSlang编排引擎编写易于理解的工作流。
* [Rill Flow](https://github.com/weibocom/rill-flow)：Rill Flow是一种高性能、可扩展的分布式工作流编排服务，由微博开源。
* [ProActive Workflows](https://github.com/ow2-proactive/scheduling)：ProActive是一款功能全面的开源作业调度和编排器，同时还具备工作流和资源管理功能，由Activeeon开发。
* [Kstry](https://gitee.com/kstry/kstry-core)：Kstry可以将原本存在于代码中错综复杂的方法调用关系以可视化流程图的形式更直观的展示出来。
* [Flux](https://github.com/flipkart-incubator/flux)：Flux是一个异步、可扩展、可选的多租户、分布式且可靠的基于状态机的编排器，由Flipkart开源。
* [Solon Flow](https://gitee.com/opensolon/solon-flow)：Java Flow通用流编排应用开发框架，支持已知流编排的各种场景。

## 规则引擎

* [Apache Drools](https://github.com/apache/incubator-kie-drools)：Drools是Java的规则引擎、DMN引擎和复杂事件处理(CEP)引擎，由JBoss社区开源。
* [Easy Rules](https://github.com/j-easy/easy-rules)：Easy Rules是一个简单但功能强大的Java规则引擎。
* [Radar](https://gitee.com/freshday/radar)：Radar是一款使用Spring Boot、MongoDB、Groovy、ES等框架搭建的轻量级实时风控引擎。
* [Liteflow](https://gitee.com/dromara/liteFlow)：LiteFlow是一个轻量且强大的国产规则引擎框架，可用于复杂的组件化业务的编排领域，由dromara社区开源。
* [RuleBook](https://github.com/deliveredtechnologies/rulebook)：RuleBook提供了一个简单但强大且灵活的规则抽象，其学习曲线非常短。
* [Clara](https://github.com/oracle-samples/clara-rules)：Clara是一个用Clojure(Script)编写并具有Java互操作性的前向链接规则引擎，由Oracle开发。
* [Nected](https://www.nected.ai/)：Nected通过用户友好的界面和声明性规则语言简化了规则表示。
* [RuleEngine](https://github.com/Hale-Lee/RuleEngine)：非常好用的规则引擎，可以直接使用SQL语句定义规则，简化了编码的负荷，也可以使用XML、drl文件配置规则，还支持drools文件导入。
* [Evrete](https://github.com/evrete/evrete)：Evrete是一个前向链接Java规则引擎，它实现RETE算法并完全符合Java规则引擎规范(JSR 94)。
* [OpenL Tablets](https://github.com/openl-tablets/openl-tablets)：OpenL Tablets是一个用于Java的开源业务规则和决策管理系统。
* [ICE](https://github.com/zjn-zjn/ice)：Java规则引擎，针对复杂/灵活变动业务，提供一个新的抽象编排解决方案，轻量级、高性能并提供可视化操作页面。
* [Jess](http://alvarestech.com/temp/fuzzyjess/Jess60/Jess70b7/docs/index.html)：Jess是最早能够轻松与Java集成的规则引擎之一，由桑迪亚国家实验室开源。
* [RuleEngine](https://gitee.com/aizuda/rule-engine-open)：RuleEngine基于Web可视化配置，简单高效快捷，爱组搭开源。
* [JRuleEngine](https://jruleengine.sourceforge.net/)：JRuleEngine是一个Java规则引擎，基于JSR 94，版本1.1。
* [URule](https://github.com/youseries/urule)：URule是一款基于RETE算法的纯Java规则引擎，提供规则集、决策表、决策树、评分卡、规则流等各种规则表现工具及基于网页的可视化设计器。
* [DataFrames](https://github.com/databrickslabs/dataframe-rules-engine)：用于自定义数据框/数据集验证的可扩展规则引擎。
* [JVS-Rules](https://gitee.com/software-minister/jvs-rules)：本项目是基于JVS逻辑引擎构建的规则引擎，将JVS低代码开发平台的逻辑引擎简化，交互优化，从而形成侧重于金融风控、场景规则计算、在线决策的JVS-Rules。
* [YARE](https://github.com/SabreOSS/yare)：YARE是一个用Java编写的规则引擎。
* [Naga](https://github.com/threatgrid/naga)：Naga是一个在图数据库上执行规则的库。
* [Rule Engine](https://github.com/jetlinks/rule-engine)：基于流程的流式规则引擎。

## 报表引擎

* [JimuReport](https://github.com/jeecgboot/JimuReport)：一款免费的数据可视化报表，含报表和大屏设计，功能涵盖数据报表、打印设计、图表报表、大屏设计等，由北京国炬公司开发。
* [UReport2](https://github.com/youseries/ureport)：UReport2是一个基于Spring架构的高性能纯Java报表引擎，可以通过迭代单元格来准备复杂的中式报表和报表。
* [EasyReport](https://github.com/xianrendzw/EasyReport)：EasyReport是一个简单易用的Web报表工具，它的主要功能是把SQL语句查询出的行列结构转换成HTML表格，并支持表格的跨行与跨列。
* [VeryReport](https://www.veryreport.com/)：VeryReport是专业级企业Web报表软件，易学易用，轻松解决中国式复杂报表，这是中创微软件公司的商业产品。
* [JasperReports](https://github.com/TIBCOSoftware/jasperreports)：JasperReports库是世界上最流行的开源报告引擎，由TIBCO软件公司开源。
* [Eclipse BIRT](https://github.com/eclipse-birt/birt)：BIRT是商业智能报告工具，可以从许多不同的数据源(数据库、文件、Java、Javascript、Web服务等)提取和组合数据，并将这些数据用于报告和图表，由安讯公司开源。
* [Ferris Wheel](https://github.com/littleorca/ferris-wheel)：Ferris Wheel是一个很棒的电子表格框架，由携程开发。
* [YARG](https://github.com/cuba-platform/yarg)：YARG是一个Java开源报告库，由Haulmont开发。
* [Pentaho](https://github.com/pentaho/pentaho-reporting)：Pentaho是用于生成报告的Java类库，它使用来自多个来源的数据提供灵活的报告和打印功能，并支持输出到显示设备、打印机、PDF、Excel、XHTML、纯文本、XML和CSV文件。
* [DynamicJasper](https://github.com/intive-FDV/DynamicJasper)：DynamicJasper是一个隐藏JasperReports复杂性的API，它可以帮助开发人员在设计简单/中等复杂性报表时节省时间，自动生成报表元素的布局。
* [ExtentReports](https://github.com/extent-framework/extentreports-java)：使用ExtentReports库，可以为你的测试创建美观、交互式且详细的报告。
* [FineReport](https://www.finereport.com/)：FineReport是一款用于报表制作，分析和展示的工具，这是帆软公司的商业产品。
* [Logi Report](https://devnet.logianalytics.com/hc/en-us/categories/1500001227442-Logi-Report)：Logi Report被设计为作为独立服务器执行，但可以将其集成到现有WAR项目中。
* [Report Mill](http://www.reportmill.com/product/)：ReportMill可以平滑地嵌入到每个Java应用程序中，此外，与BIRT一样它非常灵活：可以在运行时自定义报告。
* [iReport](https://community.jaspersoft.com/project/ireport-designer)：一个开源报表设计器，对于JasperReports库和JasperReports服务器免费。
* [OpenReports](https://sourceforge.net/projects/oreports/)：基于Web的报告解决方案，允许用户通过浏览器动态查看XLS、HTML或PDF格式创建的报告。
* [AJ-Report](https://gitee.com/anji-plus/report)：AJ-Report是一个完全开源，拖拽编辑的可视化设计工具。
* [R3-Query](https://gitee.com/aagagagag/R3-Query)：R3 Query整合了企业报表领域各个周期的支持，其中包括报表设计、报表发布、报表生成、报表管理、订阅发布和报表监控等报表的整个生命周期的步骤。
* [Pentaho Platform](https://github.com/pentaho/pentaho-platform)：该项目构成了Pentaho的核心平台和业务分析服务器。
* [UReport-kepp](https://gitee.com/summer-T/ureport-keep)：UReport的替代项目。
* [Skyeye Report](https://gitee.com/doc_wei01/skyeye-report)：Skyeye
  Report是一款高性能的Java报表引擎，提供完善的基于网页的报表设计器，可快速做出各种复杂的中式报表。
* [Dynamic Reports](https://github.com/dynamicreports/dynamicreports)：DynamicReports是一个基于JasperReports的开源Java报表库，它允许创建动态报表设计，并且不需要可视化报表设计器。
* [SpringReport](https://gitee.com/springreport/springreport)：SpringReport是一款企业级的报表系统，支持在线设计报表，并绑定动态数据源，无需写代码即可快速生成想要的报表。
* [Clear Reports](https://www.inetsoftware.de/products/clear-reports)：Clear Reports是一种灵活且功能强大的报告解决方案，可用于多种不同用途。

## 商业智能

* [Metabase](https://github.com/metabase/metabase)：Metabase是一个开源的商业智能工具，你可以通过它理解数据、分析数据，进行数据查询并获取格式化结果，以数据驱动决策。
* [Poli](https://github.com/shzlw/poli)：Poli是一款易于使用的SQL报告应用程序，专为SQL爱好者打造。
* [FineBI](https://www.finebi.com/)：FineBI是帆软公司推出的一款商业智能产品。
* [SuperSonic](https://github.com/tencentmusic/supersonic)：SuperSonic是下一代LLM支持的数据分析平台，集成了ChatBI和HeadlessBI，由腾讯音乐娱乐开源。
* [CBoard](https://gitee.com/tuiqiao/CBoard)：CBoard由上海楚果信息技术有限公司主导开源，它不仅仅是一款自助BI数据分析产品，还是开放的BI产品开发平台。
* [DataRoom](https://gitee.com/gcpaas/DataRoom)：DataRoom是一款基于SpringBoot、MyBatis Plus、Vue、ElementUI、G2Plot、Echarts等技术栈的大屏设计器，由科大国创云网科技公司开源。
* [BI Platform](https://github.com/baidu/BIPlatform)：百度开源，业内领先的Holap敏捷BI分析平台，提供高性能、准实时、可扩展的、一站式的BI建模、分析平台。
* [Abixen Platform](https://github.com/abixen/abixen-platform)：Abixen是一个基于微服务的软件平台，用于构建企业应用程序，通过创建特定的微服务并通过提供的CMS集成来提供功能。
* [Helical Insight](https://github.com/helicalinsight/helicalinsight)：Helical Insight是世界上第一个开源商业智能框架，可以帮助你从一个或多个数据源中获取见解。
* [Wren Engine](https://github.com/Canner/wren-engine)：Wren Engine被设计为独立的语义引擎，你可以轻松地使用任何AI代理实现它，可以将其用作语义层的通用语义引擎。
* [Guitar](https://github.com/iflytek/Guitar)：Guitar是一款简单、高效的分布式多维BI报表分析引擎，由科大讯飞开源。
* [LinceBI](https://github.com/LinceBI/lincebi)：LinceBI是一个完整的商业智能解决方案，包括ETL、元数据、大数据和机器学习等模块。
* [Knowage](https://github.com/KnowageLabs/Knowage-Server)：Knowage是开源分析和商业智能套件，可让你将传统数据和大/云数据源组合成有价值且有意义的信息，由OW2开源。
* [Datafor](https://datafor.com.cn/)：Datafor是一款自助式敏捷BI工具，旨在为用户提供直观易用的数据可视化和探索式分析功能，帮助用户快速进行数据探索、分析和决策。
* [JRelax-BI](https://gitee.com/zengchao/JRelax-BI)：BI商业智能，自定义表单+自定义流程+自定义报表。
* [ART](https://art.sourceforge.net/)：ART是一种报告和商业智能解决方案，它可以快速部署SQL查询结果，支持表格报告、图表、仪表板、调度。
* [FusionView](https://gitee.com/hitsoft1995/fusion-view)：富表智能数据可视化平台是一款面向企业和行业级别的用户，具有AI特色的可视化与BI商业智能敏捷分析的平台。
* [ReportServer](https://github.com/infofabrik/reportserver)：ReportServer是一个现代、多功能的开源商业智能(BI)平台，具有强大的报告功能，由ReportServer公司开发。
* [Logi Analytics](https://insightsoftware.com/logi-analytics/)：Logi Analytics提供具有自助分析功能的商业智能和数据发现平台，这是InsightSoftware公司的商业产品。
* [睿思BI](https://gitee.com/ruisibi/rsbi-pom)：睿思BI是由成都睿思商智公司研发的企业数据分析系统。

## SMT求解器

* [Alloy](https://github.com/AlloyTools/org.alloytools.alloy)：Alloy是一个独立的可执行文件，其中包括Kodkod模型查找器和各种SAT求解器，由MIT开源。
* [Sat4j](https://gitlab.ow2.org/sat4j/sat4j)：Sat4j是一个用于解决布尔满足和优化问题的Java库，它可以解决SAT、MAXSAT、伪布尔、最小不可满足子集问题，由阿尔多瓦大学和法国国家科学研究院开发。
* [DuaLip](https://github.com/linkedin/DuaLip)：DuaLip是一种基于Spark的超大规模线性规划(LP)求解器，由LinkedIn开发。
* [EUROPA](https://github.com/nasa/europa)：EUROPA是一个用于建模和解决规划、调度和约束规划问题的框架，NASA艾姆斯研究中心开源。
* [CVC5](https://github.com/cvc5/cvc5)：CVC5是一种用于确定一阶公式对一阶理论(或此类理论的组合)的可满足性的工具，由伯克利大学和三星研究院开发。
* [MiniCP](https://github.com/minicp/minicp)：MiniCP是一个用Java实现的轻量级CP求解器，由康涅狄格大学、鲁汶大学、佐治亚理工学院的三位教授共同开发。
* [JavaSMT](https://github.com/sosy-lab/java-smt)：JavaSMT是用于访问各种SMT求解器的通用API层，由德国慕尼黑大学开源。
* [SMTInterpol](https://github.com/ultimate-pa/smtinterpol)：SMTInterpol是弗莱堡大学开发的插值SMT求解器。
* [Statix Solver](https://mvnrepository.com/artifact/org.metaborg/statix.solver)：由代尔夫特理工大学开源的约束求解器。
* [JSCIPOpt](https://github.com/scipopt/JSCIPOpt)：SCIP是目前混合整数规划(MIP)和混合整数非线性规划(MINLP)最快的非商业求解器之一，由柏林自由大学开源。
* [CLP Java](https://github.com/quantego/clp-java)：CLP线性求解器的Java接口，针对快速模型构建和快速解析进行了优化。
* [LINE](https://line-solver.sourceforge.net/)：LINE是一个开源软件包，用于通过分析方法或模拟来分析排队模型，由伦敦帝国理工学院的QORE实验室开发。
* [KSMT](https://github.com/UnitTestBot/ksmt)：满足各种SMT求解器的统一Kotlin/Java API。

## 脚本

* [Blaze](https://github.com/fizzed/blaze)：用于JVM的快速、灵活、通用脚本和应用程序启动堆栈。
* [Apache Commons BSF](https://github.com/apache/commons-bsf)：BSF是一组Java类，它在Java应用程序中提供脚本语言支持，并通过脚本语言访问Java对象和方法，IBM开源。
* [Nicobar](https://github.com/Netflix/Nicobar)：Nicobar是一个Java动态脚本框架，由基于JBoss Modules的强大模块加载系统驱动，Netflix开源。
* [jPowerShell](https://github.com/profesorfalken/jPowerShell)：允许与PowerShell控制台交互的简单Java API。

## CLI框架

* [Picocli](https://github.com/remkop/picocli)：Picocli是一个现代框架，用于轻松构建功能强大、用户友好、支持GraalVM的命令行应用程序。
* [Clikt](https://github.com/ajalt/clikt)：Clikt是一个多平台Kotlin库，它使编写命令行界面变得简单直观。
* [Apache Commons CLI](https://github.com/apache/commons-cli)：Commons CLI提供了一个简单的API，用于呈现、处理和验证命令行界面。
* [Spring Shell](https://github.com/spring-projects/spring-shell)：Spring Shell可帮助你创建基于Spring的、针对CLI空间的生产级应用程序。
* [Airline](https://github.com/rvesse/airline)：Airline是一个Java库，提供基于注解的框架来解析命令行接口。
* [JLine](https://github.com/jline/jline3)：JLine是一个用于处理控制台输入的Java库。
* [Kotlinx CLI](https://github.com/Kotlin/kotlinx-cli)：通用命令行解析器的纯Kotlin实现，由JetBrains开发。
* [Text-IO](https://github.com/beryx/text-io)：Text-IO是一个用于创建Java控制台应用程序的库，它可用于需要读取用户交互式输入的应用程序。
* [Progressbar](https://github.com/ctongfei/progressbar)：Progressbar是一个基于控制台的Java进度条。
* [Jexer](https://gitlab.com/AutumnMeowMeow/jexer)：该库实现了一个基于文本的窗口系统。
* [Kotlin Inquirer](https://github.com/kotlin-inquirer/kotlin-inquirer)：受Inquirer.js启发编写的常见交互式命令行用户界面集合。

## CLI工具

* [JBang](https://github.com/jbangdev/jbang)：JBang是一个命令行开发工具，用于以脚本形式运行Java程序。
* [Babun](https://github.com/babun/babun)：Babun是Windows上类似Linux的控制台。
* [J2ObjC](https://github.com/google/j2objc)：J2ObjC是Google的开源命令行工具，可将Java源代码转换为适用于iOS(iPhone/iPad)平台的Objective-C。
* [Just](https://github.com/maciejwalkowiak/just)：Just是一个智能零配置命令行接口，用于在开发模式下运行Spring Boot应用程序。
* [BuildCLI](https://github.com/BuildCLI/BuildCLI)：BuildCLI是一个CLI工具，用于管理和自动化Java项目开发中的常见任务。
* [Signal CLI](https://github.com/AsamK/signal-cli)：Signal CLI是Signal Messenger的命令行接口，它支持注册、验证、发送和接收消息。
* [Open PDF Sign](https://github.com/open-pdf-sign/open-pdf-sign)：Open PDF Sign CLI应用程序允许从命令行轻松签署PDF文件，签名可以是不可见的(默认)或可见的(可以自定义)。
* [JReleaser](https://github.com/jreleaser/jreleaser)：JReleaser是一个用于Java和非Java项目的自动化发布工具。
* [Crash](https://github.com/crashub/crash)：CRaSH是一个为扩展Java程序和Java虚拟机而设计的shell。
* [MCS](https://github.com/mthmulders/mcs)：MCS是一个小型CLI，用于从命令行查询Maven Central。
* [JD CLI](https://github.com/intoolswetrust/jd-cli)：JD-CLI是JD-Core项目的简单命令行包装器。
* [SQLLine](https://github.com/julianhyde/sqlline)：SQLLine是一个用于通过JDBC向关系型数据库发出SQL的命令行Shell。
* [JMXTerm](https://github.com/jiaqi/jmxterm)：Jmxterm是一个用Java编写的基于开源命令行的交互式JMX客户端。
* [Certificate Ripper](https://github.com/Hakky54/certificate-ripper)：用于提取服务器证书的CLI工具。
* [Unix4j](https://github.com/tools4j/unix4j)：Unix4j是Unix命令行工具的Java实现，你可以在Java程序中使用你在Unix中了解的命令。

## 命令行参数解析

* [JCommander](https://github.com/cbeust/jcommander)：JCommander是一个非常小的Java框架，可以轻松解析命令行参数。
* [Args4j](https://github.com/kohsuke/args4j)：Args4j是一个小型Java类库，可以轻松解析CUI应用程序中的命令行选项/参数。
* [Aesh](https://github.com/aeshell/aesh)：Aesh是一个用于处理控制台输入的Java库。
* [Cloud](https://github.com/Incendo/cloud)：Cloud是一个通用Java命令调度程序和框架，它允许程序员定义命令链，然后从用户提供的字符串输入中解析和调用这些命令链，以执行预定义的操作。
* [JOpt Simple](https://github.com/jopt-simple/jopt-simple)：JOpt Simple是一个用于解析命令行选项的Java库。
* [Argparse4j](https://github.com/argparse4j/argparse4j)：Argparse4j是一个基于Python argparse模块的Java命令行参数解析器库。
* [JaCoLine](https://github.com/chriswhocodes/JaCoLine)：为开发人员提供有用的工具来理解和验证他们的Java命令行选项。
* [Docopt.Java](https://github.com/docopt/docopt.java)：Docopt的Java移植版本，Docopt是一个命令行参数解析库。
* [CLI Parser](https://github.com/spullara/cli-parser)：CLI Parser是一个小型、超级易于使用的库，用于解析各种命令行参数或属性列表。
* [JewelCLI](https://github.com/lexicalscope/jewelcli)：JewelCli使用带注解的接口或类定义来自动解析和呈现命令行参数。
* [CmdOption](https://github.com/ToToTec/CmdOption)：CmdOption是一个简单的注解驱动的命令行解析器工具包，适用于通过注解配置的Java 6+应用程序。
* [JBock](https://github.com/jbock-java/jbock)：JBock是一个命令行解析器。
* [Google Options](https://github.com/pcj/google-options)：这是Bazel项目的命令行参数解析器。
* [RecordArgs](https://github.com/nipafx/record-args)：RecordArgs是一个简单的命令行参数解析器，适用于依赖记录和密封接口的Java应用程序。
* [Rop](https://github.com/ryenus/rop)：用Java编写的轻量级命令行参数解析器。
* [Lanat](https://github.com/darvil82/lanat)：Lanat是Java 17的命令行参数解析器，具有易用性和高度可定制性。
* [Barclay](https://github.com/broadinstitute/barclay)：Barclay是一组用于注释、解析、验证和生成命令行选项文档的类，由麻省理工学院和哈佛大学布罗德研究所开源。

## cURL

* [cURL](https://github.com/libetl/curl)：在Java中使用cURL的库。
* [Java cURL](https://github.com/rockswang/java-curl)：Java cURL是一个基于标准JDK中的HttpURLConnection实现的纯Java HTTP实用程序。

## Git工具

* [Eclipse JGit](https://eclipse.dev/jgit/)：JGit是一个纯Java类库，实现了Git版本控制系统。
* [GitBucket](https://github.com/gitbucket/gitbucket)：GitBucket是由Scala提供支持的Git平台，具有易于安装、高扩展性和GitHub API兼容性。
* [Gitblit](https://github.com/gitblit-org/gitblit)：Gitblit是一个开源、纯Java Git解决方案，用于管理、查看和服务Git仓库。
* [Gitiles](https://github.com/google/gitiles)：Gitiles是一个简单的Git仓库浏览器，基于JGit构建，由Google开源。
* [Agit](https://github.com/rtyley/agit)：Agit是适用于Android设备的开源Git客户端，允许你查看任何Git仓库的完整历史记录、查看更改并存储以进行完全离线访问。
* [Gitective](https://github.com/kevinsawicki/gitective)：Gitective是一个基于JGit构建的Java库，它使得调查Git仓库变得更简单、更容易。
* [Copybara](https://github.com/google/copybara)：Copybara是Google内部使用的工具，用于在仓库之间转换和移动代码。
* [Gitnuro](https://github.com/JetpackDuba/Gitnuro)：适合新手和专业人士的FOSS Git多平台客户端。
* [BFG Repo-Cleaner](https://github.com/rtyley/bfg-repo-cleaner)：BFG是git-filter-branch的更简单、更快速的替代方案，用于清除Git仓库历史记录中的不良数据。
* [RepoSense](https://github.com/reposense/RepoSense)：RepoSense是Git仓库的贡献分析工具。
* [GitSolo](https://gitee.com/zhiqim/gitsolo)：GitSolo是知启蒙团队开源的极简Git服务器，纯Java开发。
* [MeGit](https://github.com/eclipsesource/megit)：基于EGit的独立Git GUI。
* [Giter8](https://github.com/foundweekends/giter8)：Giter8是一个命令行工具，用于从GitHub或任何其他Git仓库上发布的模板生成文件和目录。
* [GitHub Search](https://github.com/seart-group/ghs)：用于从GitHub爬取、存储和呈现项目以及与其相关的任何统计信息的平台，由瑞士卢加诺的意大利大学软件研究所开源。
* [Coming](https://github.com/SpoonLabs/coming)：Coming是一个用于挖掘Git仓库的工具，由法国国立计算机及自动化研究院、里尔大学开源。
* [SCM Manager](https://github.com/scm-manager/scm-manager)：共享和管理Git、Mercurial和Subversion仓库的最简单方法。
* [Git Changelog Lib](https://github.com/tomasbjerre/git-changelog-lib)：该库可以从Git仓库生成变更日志或发行说明，并且可以根据自上次发布以来的提交格式确定下一个版本。
* [JGitFS](https://github.com/centic9/JGitFS)：JGitFS提供对Git分支/标签/提交的访问，就像它们通过FUSE用户层文件系统是单独的目录一样。
* [MOE](https://github.com/google/MOE)：MOE是一个用于同步、翻译和清理源代码仓库的系统，由Google开源。

## 函数式编程

* [Vavr](https://github.com/vavr-io/vavr)：Vavr是Java 8的对象功能语言扩展，旨在减少代码行数并提高代码质量。
* [Arrow](https://github.com/arrow-kt/arrow)：Arrow是Kotlin中类型化函数式编程的库。
* [JavaTuples](https://github.com/javatuples/javatuples)：JavaTuples提供一组允许你使用元组的Java类。
* [Functional Java](https://github.com/functionaljava/functionaljava)：Function Java是一个促进Java函数式编程的开源库。
* [Apache Commons Functor](https://github.com/apache/commons-functor)：Commons Functor库定义了通用函子和函子相关的接口、实现和实用程序。
* [Cyclops](https://github.com/aol/cyclops)：用于构建现代Java 8应用程序的强大流和函数数据类型，由AOL开源。
* [Linq4j](https://github.com/julianhyde/linq4j)：LINQ的Java实现库。
* [Totallylazy](https://github.com/bodar/totallylazy)：TotallyLazy补充了标准库，如持久数据结构、解析器组合器等。
* [Retrolambda](https://github.com/luontola/retrolambda)：Retrolambda允许你在Java 7、6或5上运行带有Lambda表达式、方法引用和try-with-resources语句的Java 8代码。
* [Underscore Java](https://github.com/javadev/underscore-java)：Underscore.js的Java版本。
* [LINQ](https://github.com/timandy/linq)：LINQ到对象转换的Java库。
* [Purefun](https://github.com/tonivade/purefun)：Java函数式编程库。
* [DataEnum](https://github.com/spotify/dataenum)：DataEnum允许你在Java中使用代数数据类型，由Spotify开源。
* [Pragmatic](https://github.com/siy/pragmatica)：在实践中应用实用函数式Java方法所需的最小Java类集。
* [Typeof](https://github.com/nurkiewicz/typeof)：Java 8中的instanceof运算符和访问者模式替代品。
* [Tail](https://github.com/nrktkt/tail)：使用尾调用优化启用无限递归。
* [HighJ](https://github.com/highj/highj)：HighJ试图克服Java缺乏高阶类型多态性的问题，并将几个众所周知的类型类(包括Applicative、Monad和Foldable)和数据结构从Haskell转换为Java。
* [Vallang](https://github.com/usethesource/vallang)：Vallang是JVM上高度集成且大部分封闭的相互递归基本数据类型的集合。
* [KamilaLisp](https://github.com/kspalaiologos/kamilalisp)：受Haskell和APL等启发的实用、灵活且简洁的Lisp。
* [Fluent](https://github.com/rogerkeays/fluent)：Fluent允许你像调用对象方法一样调用静态Java方法。
* [Jamaica](https://www.patreon.com/Jamaica440)：Java的一种方言，添加了被Oracle拒绝的流行功能。
* [JKScope](https://github.com/evpl/jkscope)：受Kotlin启发的Java作用域函数。

#### 函数式异常处理

* [Throwing Function](https://github.com/pivovarit/throwing-function)：支持受检异常的Java 8函数接口+适配器。
* [NoException](https://github.com/robertvazan/noexception)：NoException是Java异常处理程序的函数式编程库。
* [Try](https://github.com/lambdista/try)：Java的Try-Success-Failure Scala API的实现。
* [Unchecked](https://github.com/rogerkeays/unchecked)：Unchecked允许你将Java的受检异常视为非受检异常。
* [SneakyThrow](https://github.com/rainerhahnekamp/sneakythrow)：SneakyThrow是一个用于忽略受检异常的Java库。
* [Faux Pas](https://github.com/zalando/faux-pas)：Faux Pas是一个简化Java函数式编程错误处理的库，由Zalando开源。
* [Either](https://github.com/jbock-java/either)：Java的函数式错误处理库。
* [Catch Exception](https://github.com/Codearte/catch-exception)：Catch Exception库在单行代码中捕获异常，并使它们可用于进一步分析。
* [Maybe](https://github.com/JoseLion/maybe)：Maybe是一个类似于java.util.Optional的单子包装器，但意图不同。
* [Result](https://github.com/leakyabstractions/result)：Result的目的是为可能成功或失败的操作结果提供类型安全的封装，而不是抛出异常。
* [Fluent Result](https://github.com/gorandalum/fluent-result)：一个Java结果库，帮助你摆脱异常，实现更流式的编码风格。
* [Fugue](https://bitbucket.org/atlassian/fugue)：Fugue提供补充Google Guava库的函数结构，包括Option、Either、Pair和其他类似有用的类，由Atlassian开源。
* [Dichotomy](https://github.com/xyzsd/dichotomy)：Java的Result、Either、Try和Maybe类型的单子类型。
* [Either.Java](https://github.com/spencerwi/Either.java)：Java的“Either a b”的右偏实现，使用Java 8进行映射/折叠和类型推断。
* [Ambivalence](https://github.com/poetix/ambivalence)：Java 8的Either类型。

#### Stream工具库

* [StreamEx](https://github.com/amaembo/streamex)：StreamEx是对Java Stream API的增强库。
* [Parallel Collector](https://github.com/pivovarit/parallel-collectors)：Parallel Collectors是一个工具包，可使用Stream API简化Java中的并行收集处理。
* [StreamSupport](https://github.com/stefan-zobel/streamsupport)：StreamSupport是适用于Android和Java 6或7用户的Java 8 java.util.function和java.util.stream API的向后移植。
* [Lightweight Stream API](https://github.com/aNNiMON/Lightweight-Stream-API)：Java 8中的Stream API在Java 7及更低版本的迭代器上重写。
* [Kool](https://github.com/davidmoten/kool)：java.util.stream.Stream替代方案(仅同步)，可重用、更快、更多操作符、更易于使用。
* [Streams Utils](https://github.com/JosePaumard/streams-utils)：Streams Utils是一组基于Java 8 Stream编写的操作，它允许一些Java 8中不可用的基本操作。
* [Streams](https://github.com/palantir/streams)：用于处理Java 8 Stream的实用程序，Palantir开源。
* [Protonpack](https://github.com/poetix/protonpack)：Java Stream API的实用工具库。
* [Mug](https://github.com/google/mug)：Google开源的一个小型Java 8工具库，与Guava(BiStream、Substring、MoreStreams、Parallelizer)互补。
* [CombinatoricsLib 3](https://github.com/dpaukov/combinatoricslib3)：适用于Java 8+的组合对象流生成器。
* [Stream Utils](https://github.com/Conductor/stream-utils)：该库包含处理Stream的常用实用程序。
* [Gatherers4j](https://github.com/tginsberg/gatherers4j)：适用于Java 23+的有用的流收集器(自定义中间操作)库。
* [More Gatherers](https://github.com/pivovarit/more-gatherers)：此库旨在通过提供一组Gatherers来更灵活地从流中收集数据。
* [Packrat](https://github.com/jhspetersson/packrat)：Packrat是一个Java库，它为Stream API提供了各种Gatherer实现。
* [Streamplify](https://github.com/beryx/streamplify)：该库的目标是提供有用的Java 8流并帮助你构建允许高效并行处理的新流。
* [Seq](https://github.com/wolray/seq)：提供一个强大而完备的流式编程API，并为Java添加类似生成器的编程机制。

#### Lambda扩展

* [jOOL](https://github.com/jOOQ/jOOL)：jOOL为Java 8 Lambda提供了一些有用的扩展。
* [Bijection](https://github.com/twitter/bijection)：Bijection是一种可逆函数，可以在两种类型之间来回转换，并约定通过双射进行往返将带回原始对象，由Twitter开发。
* [Functions](https://github.com/mintern-java/functions)：Functions为每个可能的0到3个参数的函数提供@FunctionalInterface。
* [Functional](https://github.com/io-fairy/functional)：Functional提供更简单、更好用的Java函数式编程接口。
* [Lambda](https://github.com/palatable/lambda)：Lambda的诞生是因为希望使用其他语言中惯用的一些相同的规范函数和函数模式，并使它们可用于Java。
* [More Lambdas](https://github.com/PhantomThief/more-lambdas-java)：Java 8的一些有用的Lambda实现。
* [Koryphe](https://github.com/gchq/koryphe)：Koryphe是一个可扩展函数库，用于基于Java Function API过滤、聚合和转换数据，由英国政府通讯总部开源。
* [FunctionalJ](https://github.com/NawaMan/FunctionalJ)：FunctionalJ是一个用Java编写函数式代码的库。

#### LINQ

* [LINQ](https://github.com/timandy/linq)：LINQ到对象转换的Java库。
* [JINQ](https://github.com/vivekragunathan/JINQ)：JINQ是一个极简的Java库，模仿.NET LINQ。
* [Linq4j](https://github.com/julianhyde/linq4j)：LINQ的Java实现库。

#### 模式匹配

* [JPML](https://github.com/klappdev/jpml)：JPML是一个Java模式匹配库。
* [Motif](https://github.com/johnlcox/motif)：Motif在Java 8中提供类似Scala的模式匹配。
* [SuitCase](https://github.com/d-plaindoux/suitcase)：SuitCase是一个方便的Java库，专用于使用模式匹配机制进行对象操作。
* [Procrastination](https://github.com/gdejohn/procrastination)：Procrastination是一个小型、简单的库，将函数式编程的优势引入Java 11。

#### 定理证明

* [Aya](https://github.com/aya-prover/aya-dev)：Aya是一种编程语言和交互式证明助手，专为类型导向编程和形式化数学而设计。
* [Arend](https://github.com/JetBrains/arend-lib)：Arend是一个基于同伦类型论的定理证明器，由JetBrains开源。

## 字节码操作

* [ASM](https://gitlab.ow2.org/asm/asm)：ASM是一个Java字节码操作框架，它能用来动态生成类或者增强既有类的功能。
* [Byte Buddy](https://github.com/raphw/byte-buddy)：Byte Buddy是一个代码生成和操作库，用于在Java应用程序运行时创建和修改Java类，而无需编译器的帮助。
* [JDK ClassFile](https://openjdk.org/jeps/457)：JDK提供的用于解析、生成和转换Java字节码文件的标准API。
* [Byteman](https://github.com/bytemanproject/byteman)：Byteman是一个可以轻松跟踪、监视和测试Java应用程序和JDK运行时代码的行为的工具，由JBoss社区开源。
* [Apache Commons BCEL](https://github.com/apache/commons-bcel)：Commons BCEL旨在为用户提供一种便捷的方式来分析、创建和操作Java字节码文件。
* [Javassist](https://github.com/jboss-javassist/javassist)：Javassist使Java字节码操作变得简单，它是Java中用于编辑字节码的类库，由东京⼯业⼤学开源。
* [CGLIB](https://github.com/cglib/cglib)：CGLIB是一个功能强大、高性能和高质量的代码生成库。
* [ByteX](https://github.com/bytedance/ByteX)：ByteX是一个基于Android Gradle Transform API和ASM的字节码插件平台，由字节开源。
* [Dexmaker](https://github.com/linkedin/dexmaker)：Dexmaker是用于针对Android的Dalvik VM进行编译或运行时代码生成的实用程序，由LinkedIn开源。
* [Allocation Instrumenter](https://github.com/google/allocation-instrumenter)：Allocation Instrumenter是使用java.lang.instrument API和ASM编写的Java代理，由Google开源。
* [Soot](https://github.com/soot-oss/soot)：Soot是一个Java优化框架，提供了多种用于分析和转换Java字节码的中间表示形式，由麦吉尔大学开源。
* [Mixin](https://github.com/SpongePowered/Mixin)：Mixin是一个使用ASM的Java特征/混合框架，并通过一组可插入的内置或用户提供的服务挂钩到运行时类加载过程。
* [ByteKit](https://github.com/alibaba/bytekit)：Java字节码工具包，由阿里开发。
* [ProGuard](https://github.com/Guardsquare/proguard-core)：ProGuard是一个免费的库，用于读取、分析、修改和写入Java字节码文件，由GuardSquare开源。
* [Titan Dex](https://github.com/baidu/titan-dex)：Titan-Dex是面向Android Dalvik(ART)字节码格式的操纵框架，可以在二进制格式下实现修改已有的类，或者动态生成新的类，由百度开源。
* [DroidAssist](https://github.com/didi/DroidAssist)：DroidAssist是一个轻量级的Android字节码编辑插件，基于Javassist对字节码操作，由滴滴开源。
* [Jitescript](https://github.com/qmx/jitescript)：用于字节码生成的Java API。
* [InjKit](https://github.com/facebookincubator/InjKit)：InjKit是一个基于ASM库的字节码操作框架，由Facebook开发。
* [Perses](https://github.com/nick-kanakis/perses)：Perses允许你在字节码级别动态注入故障/延迟，无需添加任何依赖项或重新启动/部署目标应用程序。
* [Maker](https://github.com/cojen/maker)：Maker库是一个轻量级、功能齐全的低级动态Java字节码生成器，其设计易于使用。
* [Ja-Netfilter](https://gitee.com/ja-netfilter/ja-netfilter)：Java Instrumentation框架。
* [Bytecode](https://github.com/airlift/bytecode)：Bytecode是一个用于生成JVM字节码的高级Java库。
* [AsmTools](https://wiki.openjdk.org/display/CodeTools/asmtools)：AsmTools项目是用于生成正确和不正确的Java class文件的工具。
* [Serp](https://serp.sourceforge.net/)：Serp是一个用于操作Java字节码的开源框架。
* [Gizmo](https://github.com/quarkusio/gizmo)：Gizmo旨在简化字节码生成，Quarkus大量使用它。

## 字节码工具库

* [JD Core](https://github.com/java-decompiler/jd-core)：JD Core是一个独立的Java库，包含Java Decompiler项目的Java反编译器。
* [JD Core Java](https://github.com/nviennot/jd-core-java)：Java反编译器JD Core库。
* [JDart](https://github.com/psycopaths/jdart)：JDart是一个在Java程序上执行一致执行的工具，它是作为NASA Java Pathfinder(JPF)的扩展编写的。
* [CFR](https://github.com/leibnitz27/cfr)：CFR可以很好地将class文件从其他JVM语言转回Java。
* [UnLuac](https://github.com/HansWessels/unluac)：UnLuac是Lua 5.1的反编译器，它在使用标准Lua编译器编译的Lua块上运行。
* [AabResGuard](https://github.com/bytedance/AabResGuard)：AabResGuard是一款由抖音Android团队提供支持的资源混淆工具。
* [Native Obfuscator](https://github.com/radioegor146/native-obfuscator)：用于JNI的Java class到cpp转换器。
* [YGuard](https://github.com/yWorks/yGuard)：YGuard是yWorks推出的与Ant和Gradle配合使用的开源Java混淆工具。
* [SpecialSource](https://github.com/md-5/SpecialSource)：SpecialSource是jar混淆映射的自动生成器和重命名器。
* [Allatori](https://allatori.com/)：Allatori是第二代Java混淆器，它为你的知识产权提供全方位的保护。
* [Simple String Obfuscator](https://github.com/shamanland/simple-string-obfuscator)：Simple String Obfuscator可以隐藏真实的字符串常量以防止反编译。
* [Branchlock](https://branchlock.net/)：Branchlock为Java桌面应用程序、Android应用程序以及Kotlin和Groovy等JVM语言的二进制文件提供混淆。
* [Jar Protect](https://gitee.com/chejiangyi/jar-protect)：Jar Protect是一个jar加密加壳工具，对class文件进行加密防护，对Properties、YAML等配置文件进行加密，避免反编译破解。
* [CAFEDOOD](https://github.com/Col-E/CAFED00D)：CAFEDOOD是一个专注于混淆支持的类库。
* [Avaj](https://github.com/cg-dot/avaj)：Avaj是一个Java混淆器。
* [Paramorphism](https://paramorphism.dev/)：Paramorphism是一个快速、现代的混淆器，支持Java 8到13、Kotlin和其他JVM语言。
* [Serianalyzer](https://github.com/mbechler/serianalyzer)：Serianalyzer是一个静态字节码分析器，跟踪反序列化期间调用的方法进行的原生方法调用。
* [Kex](https://github.com/vorpal-research/kex)：Kex是一个用于分析Java字节码的平台，由Jetbrains开发。

## 图像处理

* [Thumbnailator](https://github.com/coobird/thumbnailator)：Thumbnailator是一个Java缩略图生成库。
* [Thumbnails4j](https://github.com/elastic/thumbnails4j)：使用JVM生成文件缩略图的项目，由Elastic开源。
* [Pngtastic](https://github.com/depsypher/pngtastic)：Pngtastic是一个纯Java PNG图像优化和操作库。
* [Aspose.Imaging](https://products.aspose.com/imaging/java/)：Aspose.Imaging是一个库，为你提供用于图像和照片处理的高级工具。
* [Grid](https://github.com/guardian/grid)：Grid是卫报的图像管理系统，它提供了访问组织的媒体的通用且快速的体验，并以经济实惠的方式使用它来生成高质量的内容。
* [Apache Commons Imaging](https://github.com/apache/commons-imaging)：Commons Imaging是一个纯Java图像库。
* [SmartCropper](https://github.com/pqpo/SmartCropper)：SmartCropper是一个简单易用的智能图片裁剪库，适用于身份证，名片，文档等照片的裁剪。
* [TwelveMonkeys](https://github.com/haraldk/TwelveMonkeys)：TwelveMonkeys ImageIO通过javax.imageio.*包的插件为Java平台提供扩展图像文件格式支持。
* [ImgLib2](https://github.com/imglib/imglib2)：ImgLib2是一个通用的多维图像处理库。
* [ImgScalr](https://github.com/rkalla/imgscalr)：ImgScalr是一个简单高效(硬件加速)的图片缩放“最佳实践”类库，纯Java 2D实现。
* [Marvin](https://github.com/gabrielarchanjo/marvin-framework)：Marvin图像处理框架提供实时处理图像和视频的功能。
* [Picasso](https://github.com/square/picasso)：Picasso是一个强大的Android图像下载和缓存库，由Square开源。
* [EasyImage](https://github.com/aviyehuda/EasyImage)：EasyImage可让你执行所有基本图像操作-转换、裁剪、调整大小、旋转、翻转等。
* [JMagick](https://github.com/techblue/jmagick)：JMagick是ImageMagick的开源Java接口。
* [Jrawio](https://github.com/tidalwave-it/jrawio-src)：Java Image I/O API的服务提供者，用来处理数码相机拍摄的RAW格式的图片，包括NEF、CRW、CR2、PEF、SRF、MRW。
* [JJIL](https://github.com/litmanowicziv/jjil)：JJIL是一个Java图像处理库，它包括图像处理架构和60多个用于各种图像处理任务的例程。
* [PNGJ](https://github.com/leonbloy/pngj)：PNGJ是一个纯Java库，用于高性能读写PNG图像。
* [ICAFE](https://github.com/dragon66/icafe)：用于读取、写入、转换和操作图像和元数据的Java库。
* [Image4j](https://github.com/imcdonagh/image4j)：Image4j库允许你以100%纯Java读取和写入某些图像格式。
* [Pollexor](https://github.com/square/pollexor)：Thumbor图像服务的Java客户端，允许你使用流式的API以富有表现力的方式构建URI，由Square开源。
* [JAI ImageIO](https://github.com/jai-imageio/jai-imageio-core)：Java高级图像I/O工具项目。
* [LEADTOOLS](https://www.leadtools.com/sdk/java)：LEADTOOLS是一个Java图像处理库，提供了文档清理、医学图像增强、边缘检测、颜色转换和校正、降噪等功能。
* [SimpleImage](https://github.com/alibaba/simpleimage)：SimpleImage是阿里开源的一个Java图片处理的类库，可以实现图片缩略、水印等处理。
* [Image Comparison](https://github.com/romankh3/image-comparison)：Image Comparison可以比较2个相同大小的图像，并通过绘制矩形直观地显示差异。
* [OptimizedImageEnhance](https://github.com/26hzhang/OptimizedImageEnhance)：该库包含一组由Java实现的图像/视频增强方法，用于解决一些常见任务，例如去雾、去噪、水下后向散射消除、低照度增强、特征化、平滑等。
* [Scrimage](https://github.com/sksamuel/scrimage)：Scrimage是一个用于图像操作的不可变、函数式、高性能的JVM库。
* [CV4j](https://github.com/imageprocessor/cv4j)：CV4j是用纯Java实现的高质量、实时的图像处理和机器学习库。
* [ImgLib](https://github.com/nackily/imglib)：ImgLib是一个轻量级的Java图像处理库，致力于简化对图像的常见处理。
* [GIFEncoder](https://github.com/square/gifencoder)：GIFEncoder是一个实现GIF89a规范的纯Java库，由Square开源。
* [AndroidLibyuvImageUtils](https://github.com/myrao/AndroidLibyuvImageUtils)：Android上的图像处理库。
* [ImageCombiner](https://gitee.com/dromara/image-combiner)：ImageCombiner是一个专门用于Java服务端图片合成的工具，由dromara社区开源。
* [ImageTool](https://gitee.com/xshuai/imagetool)：一个简单的图片处理工具，支持图片压缩、图片水印、图片裁剪、图片旋转、图片格式转换等功能。
* [LIRE](https://github.com/dermotte/LIRE)：LIRE是一个用于基于内容的图像检索的开源库，这意味着你可以使用LIRE来实现搜索看起来相似的图像的应用程序。
* [Fiji](https://github.com/fiji/fiji)：Fiji是一个图像处理包，捆绑了许多有助于科学图像分析的插件，由威斯康星大学麦迪逊分校等机构开源。
* [CognitiveJ](https://github.com/CognitiveJ/cognitivej)：CognitiveJ是一个开源流式Java API，可管理和编排Java应用程序与Microsoft的Cognitive(牛津项目)机器学习和图像处理库之间的交互，并允许你查询和分析图像。
* [JImageHash](https://github.com/KilianB/JImageHash)：JImageHash是一个完全用Java编写的高性能感知图像指纹库。
* [Image Scaling](https://github.com/mortennobel/java-image-scaling)：该库的目的是提供更好的图像缩放选项。
* [Eclipse ImageN](https://github.com/eclipse/imagen)：ImageN项目提供了一个可扩展的按需图像处理库，对光栅大小或波段数量没有人为限制。
* [Blurry](https://github.com/wasabeef/Blurry)：Blurry是一个简单的Android模糊库。
* [Cantaloupe](https://github.com/cantaloupe-project/cantaloupe)：Cantaloupe是一个开源动态图像服务器，用于按需生成高分辨率源图像的衍生品。
* [Open Imaging](https://github.com/DhyanB/Open-Imaging)：Open Imaging是用于图像创建和处理的工具和库。
* [ImageIO-Ext](https://github.com/geosolutions-it/imageio-ext)：ImageIO-Ext是一个开源项目，为标准Oracle Java Image I/O项目提供扩展、修复和改进。
* [SCIFIO](https://github.com/scifio/scifio)：SCIFIO是一个可扩展的Java框架，用于读取和写入图像，特别是N维科学图像。
* [Kim](https://github.com/Ashampoo/kim)：Kim是一个用于读取和写入图像元数据的Kotlin多平台库。
* [Java Image Filters](http://www.jhlabs.com/ip/filters/index.html)：Java Image Filters是由Jhlabs开发的一组用来处理Java图像的类库，提供各种常用的图像处理效果，例如反转色、扭曲、水波纹、凹凸、黑白效果等等数十种效果。
* [JAI EXT](https://github.com/geosolutions-it/jai-ext)：JAI EXT是一个开源项目，旨在扩展JAI API，由GeoSolutions开源。
* [Animated GIF Library](https://github.com/rtyley/animated-gif-lib-for-java)：Java的动画GIF库。
* [GifAnimation](https://github.com/extrapixel/gif-animation)：GifAnimation是一个用于播放和导出GIF动画的Processing库。

## 水印

* [AndroidWM](https://github.com/huangyz0918/AndroidWM)：一个支持隐写技术的Android图像水印库。
* [OpenStego](https://github.com/syvaidya/openstego)：OpenStego是一款隐写术应用程序。
* [RubberStamp](https://github.com/vinaygaba/RubberStamp)：RubberStamp是一个Android库，可让你轻松地为图像添加水印。

## SVG库

* [JFreeSVG](https://github.com/jfree/jfreesvg)：JFreeSVG是一个用于Java平台的图形库，允许你使用标准Java2D绘图API生成SVG格式的内容。
* [Apache Batik](https://github.com/apache/xmlgraphics-batik)：Batik是一个基于Java的工具包，适用于处理SVG格式的图像各种目的，例如观看、生成或操纵。
* [SVG Salamander](https://github.com/blackears/svgSalamander)：SVG Salamander是一个用于Java的SVG引擎，设计小巧、速度快。
* [VectorGraphics2D](https://github.com/eseifert/vectorgraphics2d)：VectorGraphics2D提供Java Graphics2D接口的实现，并以各种矢量文件格式导出图形。
* [JSVG](https://github.com/weisJ/jsvg)：JSVG是一个使用AWT图形的SVG用户代理。
* [WebVector](https://github.com/radkovo/WebVector)：WebVector是一个HTML到SVG、PDF或PNG转换器。
* [WMF2SVG](https://github.com/hidekatsu-izuno/wmf2svg)：适用于Java的WMF到SVG转换工具和库。
* [Safe SVG](https://github.com/bgalek/safe-svg)：简单且轻量级的库，有助于以安全方式验证SVG文件。

## TIFF库

* [TIFF Java](https://github.com/ngageoint/tiff-java)：TIFF是一个用于读写标记图像文件格式文件的Java库，由美国国家地理空间情报局开源。
* [AlgART TIFF](https://github.com/Daniel-Alievsky/algart-tiff)：AlgART TIFF是一个Java库，提供TIFF文件的完整读/写支持。

## 验证码

* [AJ Captcha](https://gitee.com/anji-plus/captcha)：AJ-Captcha行为验证码，包含滑动拼图、文字点选两种方式，UI支持弹出和嵌入两种方式。
* [EasyCaptcha](https://gitee.com/ele-admin/EasyCaptcha)：Java图形验证码，支持GIF、中文、算术等类型，可用于Java Web、Java SE等项目。
* [Tianai Captcha](https://gitee.com/tianai/tianai-captcha)：非常好用的开源行为验证码(滑块验证码、点选验证码、行为验证码、旋转验证码， 滑动验证码)。
* [Happy Captcha](https://gitee.com/ramostear/Happy-Captcha)：Happy Captcha是一款易于使用的Java验证码软件包。
* [Kaptcha](https://github.com/penggle/kaptcha)：Kaptcha是一个可高度配置的实用验证码生成工具。
* [JCaptcha](https://mvnrepository.com/artifact/com.octo.captcha/jcaptcha/1.0)：一个可以生成图片、声音式验证码的Java库。
* [Captcha Plus](https://github.com/xingyuv/captcha-plus)：Captcha Plus行为验证码，包含滑动拼图、文字点选两种方式，UI支持弹出和嵌入两种方式。
* [NanoCaptcha](https://github.com/logicsquad/nanocaptcha)：NanoCaptcha是一个用于生成图像和音频验证码的Java库。

## 二维码生成器

* [ZXing](https://github.com/zxing/zxing)：ZXing是一个用Java实现的开源、多格式1D/2D条形码图像处理库。
* [QR Code Generator](https://github.com/nayuki/QR-Code-generator)：Java、TypeScript/JavaScript、Python、Rust、C++、C语言的高质量QR码生成器库。
* [ZXingLite](https://github.com/jenly1314/ZXingLite)：ZXing的精简极速版，优化扫码和生成二维码/条形码，内置闪光灯等功能。
* [AwesomeQRCode](https://github.com/sumimakito/AwesomeQRCode)：适用于Android的超棒二维码生成器。
* [Aspose.BarCode](https://products.aspose.com/barcode/java/)：Aspose.BarCode是一个强大且可靠的API，为Java应用程序提供条形码生成和识别功能。
* [Spire.Barcode](https://www.e-iceblue.com/Introduce/barcode-for-java.html)：Spire.Barcode是一款专业的条形码组件，专为开发人员在Java应用程序上生成、读取和扫描1D和2D条形码而设计。
* [QArt4J](https://github.com/dieforfree/qart4j)：提供ASCII Art输出图像的QR码生成器。
* [Barbecue](https://barbecue.sourceforge.net/)：Barbecue是一个开源的Java库，它提供了创建用于在Java应用程序中打印和显示的条形码的方法。
* [Barcode4J](https://barcode4j.sourceforge.net/)：Barcode4J是一个用Java编写的灵活的条形码生成器。
* [QRGen](https://github.com/kenglxn/QRGen)：基于ZXING构建的简单的Java二维码生成API。
* [ZXingGenerator](https://github.com/vivian8725118/ZXingGenerator)：花式二维码生成库，提供了6种样式。
* [Visual QR Code](https://gitee.com/boat824109722/visual-qr-code)：Visual QR Code可以创建出设置了虚拟背景图片的二维码。
* [QRext4j](https://gitee.com/BYSRepo/qrext4j)：一个简单易用的二维码生成工具，可自定义二维码颜色和码眼样式。
* [QRCode Kotlin](https://github.com/g0dkar/qrcode-kotlin)：QRCode Kotlin旨在提供一种简单、直接且可自定义的二维码创建方式，尤其是在后端。
* [FiwanQRCode](https://gitee.com/frogchou/FiwanQRCode)：飞网开发的二维码生成工具。
* [WeChatQRCode](https://github.com/jenly1314/WeChatQRCode)：WeChatQRCode是一个基于OpenCV开源的微信二维码引擎移植封装的二维码识别库。
* [QRCode Generator](https://github.com/kazuhikoarase/qrcode-generator)：以JavaScript、Java等语言实现的QR码生成器。
* [QRGenerator](https://github.com/androidmads/QRGenerator)：二维码生成器库。
* [QRCode Utils](https://github.com/binarywang/qrcode-utils)：二维码生成工具。
* [Okapi Barcode](https://github.com/woo-j/OkapiBarcode)：Okapi Barcode是一款完全用Java编写的开源条形码生成器，支持50多种编码标准，包括所有ISO标准。
* [EMV QRCode](https://github.com/mvallim/emv-qrcode)：基于Java的EMV二维码生成器和解析器(MPM、CPM)。
* [Barcode Java](https://github.com/barnhill/barcode-java)：该库旨在为开发人员提供一个简单的类，供开发人员在需要从一串数据生成条形码图像时使用。
* [SwissQRBill](https://github.com/manuelbl/SwissQRBill)：用于生成和解码瑞士二维码钞票的开源Java库。

## 压缩库

* [Brotli](https://github.com/google/brotli)：Brotli是一种通用无损压缩算法，它结合使用LZ77算法的现代变体、霍夫曼编码和二阶上下文建模来压缩数据，其压缩率可与目前最好的通用压缩方法相媲美，由Google开源。
* [Aspose.ZIP](https://products.aspose.com/zip/java/)：Aspose.ZIP是一个针对标准ZIP格式的灵活的文档压缩和档案操作API。
* [CompressHelper](https://github.com/nanchen2251/CompressHelper)：文件、图片压缩工具类。
* [AdvancedLuban](https://github.com/shaohui10086/AdvancedLuban)：AdvancedLuban是一个方便简单的Android图像压缩工具库。
* [Zstd](https://github.com/luben/zstd-jni)：Zstd是一种新的无损压缩算法，它可以为你的标准压缩需求提供良好的压缩比和速度。
* [Luban](https://github.com/Curzibn/Luban)：Luban是Android图片压缩工具，仿微信朋友圈压缩策略。
* [Archive Patcher](https://github.com/google/archive-patcher)：Archive Patcher是一个开源项目，允许对zip存档进行节省空间的修补，由Google开源。
* [Apache Commons Compress](https://github.com/apache/commons-compress)：Commons Compress定义了一个用于处理压缩和存档格式的API。
* [LZF Compressor](https://github.com/ning/compress)：LZF Compressor是一个用于编码和解码LZF格式数据的Java库。
* [JZlib](https://github.com/ymnk/jzlib)：JZlib是zlib在纯Java中的重新实现。
* [HPACK](https://github.com/twitter/hpack)：HPACK库提供将header列表压缩为header块的支持，由Twitter开源。
* [MiGz](https://github.com/linkedin/migz)：MiGz是一个支持多线程的使用GZIP格式的压缩库，由LinkedIn开发。
* [JArchiveLib](https://github.com/thrau/jarchivelib)：JArchiveLib是一个简单的Java归档和压缩库，它在Apache Commons Compress之上提供了一个精简且易于使用的API层。
* [ParallelGZIP](https://github.com/shevek/parallelgzip)：该库包含一个并行GZIP实现，它是标准java.util.zip类的高性能替代品。
* [Snappy Java](https://github.com/xerial/snappy-java)：Snappy的Java移植版，Snappy是Google开发的快速C++压缩器/解压缩器。
* [LZMA](https://github.com/jponge/lzma-java)：该库为在Java平台上运行的应用程序提供LZMA压缩。
* [LZO](https://github.com/shevek/lzo-java)：liblzo2 LZO压缩算法的纯Java实现。
* [LZ4](https://github.com/lz4/lz4-java)：用于Java的LZ4压缩库。
* [Compress](https://gitee.com/yu120/compress)：基于gzip、deflate、lz4、snappy、lzo等算法实现数据压缩，主要用于RPC通讯数据的压缩。
* [Zip4j](https://github.com/srikanth-lingala/zip4j)：Zip4j是最全面的zip文件或流Java库。
* [JavaFastPFOR](https://github.com/lemire/JavaFastPFOR)：Java中的一个简单的整数压缩库。
* [ZT ZIP](https://github.com/zeroturnaround/zt-zip)：Java zip库，构建于java.util.zip包之上。
* [ZIP Forge](https://github.com/helpermethod/zip-forge)：一个小型的、格式化程序友好的Java DSL，用于创建ZIP文件。
* [AirCompressor](https://github.com/airlift/aircompressor)：该库包含用纯Java编写的Zstandard(Zstd)、LZ4、Snappy和LZO的实现，它们通常比原生库的JNI包装器快10-40%。
* [Junrar](https://github.com/junrar/junrar)：纯Java解压缩库。
* [LLJ-ZIP](https://github.com/Col-E/LL-Java-Zip)：LLJ-ZIP是一个用于可查找文件的zip格式读取器，可以容忍前导和尾随垃圾，并容忍针对前导垃圾调整内部偏移。
* [Brotli4j](https://github.com/hyperxpro/Brotli4j)：Brotli4j为Java提供Brotli压缩和解压缩。
* [WebGraph](https://github.com/vigna/webgraph)：WebGraph是一个旨在研究网络图的图压缩框架，它利用现代压缩技术提供了管理非常大的图形的简单方法，由米兰大学开发。
* [Qat-Java](https://github.com/intel/qat-java)：Qat-Java库使用英特尔QuickAssist技术QATzip库提供加速压缩和解压缩。
* [XZ Java](https://github.com/tukaani-project/xz-java)：XZ Java是纯Java中XZ数据压缩的完整实现。
* [LibDeflate Java](https://github.com/astei/libdeflate-java)：该项目为libdeflate库提供了安全、高性能的JNI绑定。
* [HtmlCompressor](https://github.com/hazendaz/htmlcompressor)：HtmlCompressor通过删除多余的空格、注释和其他不需要的字符来缩小给定的HTML或XML源，而不会破坏内容结构。
* [7 Zip JBinding](https://github.com/borisbrodski/sevenzipjbinding)：7 Zip JBinding是7-Zip免费压缩/解压缩库的免费跨平台Java绑定。
* [Zip4jvm](https://github.com/oleg-cherednik/zip4jvm)：用于处理Zip文件的Java库。

## 反射库

* [Reflections](https://github.com/ronmamo/reflections)：Reflections会扫描项目的类路径元数据并为其建立索引，从而允许在运行时对类型系统进行反向传递查询。
* [jOOR](https://github.com/jOOQ/jOOR)：jOOR是用于反射的流式API库，可以以更直观的方式访问Class类结构。
* [ReflectASM](https://github.com/EsotericSoftware/reflectasm)：ReflectASM是一个非常小的Java库，它通过使用代码生成来提供高性能反射。
* [Objenesis](https://github.com/easymock/objenesis)：Objenesis是一个专门用于在创建对象时绕过构造函数的库。
* [Mirror](https://github.com/Genymobile/mirror)：Java和Android的轻松反射。
* [Intimate](https://github.com/eleme/Intimate)：Intimate提供了友好的API让Java反射的使用更加简单平滑，由饿了么开源。
* [Apache Commons ClassScan](https://commons.apache.org/sandbox/commons-classscan)：ClassScan可以提供有关运行时可用的所有类的信息，无论该类是否已加载。
* [FEST-Reflect](https://github.com/alexruiz/fest-reflect)：FEST-Reflect提供了直观、紧凑且类型安全的流式API，使Java反射非常易于使用：不再需要强制转换、检查异常、PriviledgedActions或setAccessible调用。
* [Lambda-Factory](https://github.com/Hervian/lambda-factory)：Lambda-Factory是一个Java实用程序项目，它提供了基于反射的方法调用的快速替代方案。
* [Mirror](http://projetos.vidageek.net/mirror/mirror/)：Mirror的创建是为了解决一个简单的问题，通常命名为ReflectionUtil，它几乎适用于所有依赖反射来完成高级任务的项目。
* [Reflection Util](https://github.com/cronn/reflection-util)：简化Java反射常见用例的工具类。
* [Paranamer](https://github.com/paul-hammant/paranamer)：Paranamer是一个允许在运行时访问非私有方法和构造函数的参数名称的库，由ThoughtWorks开源。
* [Mirror](https://github.com/vidageek/mirror)：Java反射API上的简单DSL层。
* [Jeflect](https://github.com/RomanQed/jeflect)：一组旨在与反射交互并加速反射的实用程序。
* [Reflecto](https://github.com/cariochi/reflecto)：Reflecto是一个功能强大的Java反射库，旨在简化深度反射任务。

## 注解库

* [JetBrains Annotations](https://github.com/JetBrains/java-annotations)：Annotations是一组可在基于JVM的语言中使用的Java注解，由JetBrains开源。
* [Atlassian Annotations](https://bitbucket.org/atlassian/atlassian-annotations)：用于声明API状态和属性的标准注解类型，由Atlassian开源。

## 批处理框架

* [Spring Batch](https://github.com/spring-projects/spring-batch)：Spring Batch是一个轻量级、全面的批处理框架，旨在支持开发对企业系统日常运营至关重要的健壮批处理应用程序。
* [Spring Cloud Data Flow](https://github.com/spring-cloud/spring-cloud-dataflow)：Spring Cloud Data Flow是一个基于微服务的工具包，用于在Cloud Foundry和Kubernetes中构建流式和批量数据处理管道。
* [Asakusa](https://github.com/asakusafw/asakusafw)：Asakusa是一个面向分布式/并行计算的全栈框架，提供了支持各种分布式/并行计算环境的开发平台和运行时库，例如Hadoop、Spark、用于批处理的M3等。
* [JBeret](https://github.com/jberet/jsr352)：JBeret是Jakarta Batch的实现，它还包含在WildFly中，以在Jakarta EE环境中提供便携式批处理支持。
* [JBatch](https://github.com/WASdev/standards.jsr352.jbatch)：JBatch是Jakarta Batch规范的兼容实现，由IBM开源。
* [Easy Batch](https://github.com/j-easy/easy-batch)：Easy Batch是一个旨在简化Java批处理的框架，它专为简单的单任务ETL作业而设计。

## 注解处理器

* [Lombok](https://github.com/projectlombok/lombok)：Lombok是对Java语法非常有用的补充，消除大量样板代码。
* [Immutables](https://github.com/immutables/immutables)：Immutables是用于创建不可变对象和构建器的注解处理器。
* [Derive4j](https://github.com/derive4j/derive4j)：Derive4j是一个Java 8注解处理器，用于派生代数数据类型构造函数、模式匹配等。
* [AndroidAnnotations](https://github.com/androidannotations/androidannotations)：AndroidAnnotations是一个开源框架，可加速Android开发。
* [Compile Testing](https://github.com/google/compile-testing)：javac和注解处理器的测试工具，由Google开源。
* [PaperParcel](https://github.com/grandstaish/paperparcel)：PaperParcel可以自动生成Java和Kotlin的Parcelable实现。
* [RecordBuilder](https://github.com/Randgalt/record-builder)：RecordBuilder是用于Java记录的记录构建器。
* [PojoBuilder](https://github.com/mkarneim/pojobuilder)：PojoBuilder是一个符合Java 6的注解处理器，可为POJO生成流式的构建器类。
* [Annotation Command Framework](https://github.com/aikar/commands)：ACF是一个极其强大的命令框架，它几乎采用了命令处理程序中常见的样板代码的所有概念，并将它们抽象到注解后面。
* [Hugo](https://github.com/JakeWharton/hugo)：调试版本的注解触发方法调用日志记录。
* [Jackdaw](https://github.com/vbauer/jackdaw)：Jackdaw是一个Java注解处理器，可以简化Java/Android开发并防止编写繁琐的代码。
* [ParcelablePlease](https://github.com/sockeqwe/ParcelablePlease)：ParcelablePlease是用于生成Android Parcelable样板代码的注解处理器。
* [BeanKnife](https://github.com/vipcxj/beanknife)：BeanKnife是用于自动生成DTO的注解处理器库。
* [Rest.Vertx](https://github.com/zandero/rest.vertx)：类似JAX-RS的注解处理器，适用于Vert.x Vertical。
* [FreeBuilder](https://github.com/inferred/FreeBuilder)：FreeBuilder可以自动生成Java的Builder模式，由Google开源。
* [Airline](https://github.com/airlift/airline)：Airline是一个基于Java注解的框架，用于解析类似命令行结构的Git。
* [Config-Builder](https://github.com/TNG/config-builder)：Config-Builder使用注解和反射来构建自定义类的配置实例。
* [CallBuilder](https://github.com/google/CallBuilder)：CallBuilder是一个Java代码生成器，可以使创建构建器类变得容易，由Google开源。
* [Domino-Jackson](https://github.com/DominoKit/domino-jackson)：Domino-Jackson是一个基于注解处理器的JSON映射器。
* [Domino-Rest](https://github.com/DominoKit/domino-rest)：Domino-Rest是一个用于从JaxRs兼容接口生成REST客户端的库。
* [Duzzt](https://github.com/misberner/duzzt)：Duzzt是一个Java注解处理器(库)，可轻松生成Java的嵌入式DSL。
* [Gson Path](https://github.com/LachlanMcKee/gsonpath)：一个注解处理器库，在编译时生成Gson类型适配器，也使用基本的JsonPath功能。
* [Pojo Analyzer](https://github.com/almogtavor/pojo-analyzer)：Pojo Analyzer是一个Java库，旨在为POJO的每个字段生成包含Getter、Setter和字符串名称的List或Map。
* [Sundrio](https://github.com/sundrio/sundrio)：一系列基于APT的代码生成工具，包括高级生成器生成器、DSL生成器、Velocity转换器等。
* [Viper](https://github.com/civitz/viper)：用于通过Java EE的CDI注入配置的生成器和框架。
* [APTK](https://github.com/toolisticon/aptk)：APTK可帮助你以更有效的方式构建注解处理器的工具包。
* [ADT4J](https://github.com/sviperll/adt4j)：该库为Java实现了代数数据类型。
* [Elementary](https://github.com/Pante/elementary)：Elementary是一套可简化注解处理器的创建和单元测试的库。
* [AutoMatter](https://github.com/danielnorberg/auto-matter)：AutoMatter用于从定义为最小接口的值类型具体化值类和构建器。
* [Jilt](https://github.com/skinny85/jilt)：Jilt是一个Java注解处理器，用于自动生成实现Builder设计模式的类。
* [Deoplice](https://github.com/chriskiehl/Deoplice)：Deoplice是一个Java库，它会自动生成用于转换不可变POJO的API。
* [Kotlin Compile Testing](https://github.com/tschuchortdev/kotlin-compile-testing)：用于测试Kotlin和Java注解处理器、编译器插件和代码生成的库。
* [DistributeMe](https://github.com/anotheria/distributeme)：DistributeMe是一个自动分发Java代码的框架，DistributeMe直接操作你的Java代码，带注解的接口由DistributeMe apt预处理器处理，生成分发相关代码。
* [Cute](https://github.com/toolisticon/cute)：Java编译测试库，允许你测试注解处理器。
* [Coat](https://github.com/poiu-de/coat)：Coat是一个注解处理器，用于生成用于将配置值读取到类型安全对象中的类。
* [Incap](https://github.com/tbroyer/gradle-incap-helper)：用于构建增量注解处理器的辅助库和注解处理器。

## 字符串工具库

* [Joda Convert](https://github.com/JodaOrg/joda-convert)：Joda-Convert是一个小型的、高度集中的库，提供对象和字符串之间的往返转换。
* [SimMetrics](https://github.com/Simmetrics/simmetrics)：相似度和距离度量的Java库，例如Levenshtein距离和余弦相似度。
* [Java String Similarity](https://github.com/tdebatty/java-string-similarity)：实现不同字符串相似度和距离测量的库。
* [Apache Commons Text](https://github.com/apache/commons-text)：Commons Text是一个专注于字符串算法的库。
* [Strman](https://github.com/shekhargulati/strman-java)：Java 8字符串操作库。
* [Java String Similarity](https://github.com/rrice/java-string-similarity)：Java String Similarity是一个实现了多种计算字符串之间相似度算法的Java库。
* [Aho Corasick](https://github.com/robert-bor/aho-corasick)：用于高效字符串匹配的Aho-Corasick算法的Java实现。
* [JavaWuzzy](https://github.com/xdrop/fuzzywuzzy)：FuzzyWuzzy模糊字符串匹配算法的Java实现。
* [Type Parser](https://github.com/drapostolos/type-parser)：解析字符串并将其转换为另一种类型，支持所有适用的Java库类。
* [String Format](https://github.com/JoanZapata/string-format)：当涉及大字符串时，String.format()的替代方案。

## 字符串插值

* [Phrase](https://github.com/square/phrase)：Phrase是一个Android字符串资源模板库，由Square开源。
* [Better Strings](https://github.com/antkorwin/better-strings)：Better Strings是用于Java字符串插值的插件。

## 9-22特性

* [Bach](https://github.com/sormuras/bach)：Bach是一个编排JDK工具以构建模块化Java项目的工具。
* [ModiTect](https://github.com/moditect/moditect)：ModiTect项目旨在提供使用Java模块系统的生产力工具。
* [OpenWebStart](https://github.com/karakun/OpenWebStart)：OpenWebStart提供了一个用户友好的安装程序，可以在更高的Java版本中使用Web Start/JNLP功能。
* [IcedTeaWeb](https://github.com/AdoptOpenJDK/IcedTea-Web)：IcedTeaWeb是JSR-56(Java Web Start)的开源实现。
* [Jabel](https://github.com/bsideup/jabel)：可在Java 8上使用现代Java 9-14语法。
* [Permit Reflection](https://github.com/nqzero/permit-reflect)：用于使用Java 11模块的实用程序。
* [Reified](https://github.com/Auties00/Reified)：用于在Java 11及更高版本中实现具体化。
* [Java REPL](https://github.com/albertlatacz/java-repl)：Java语言的简单REPL，考虑到Java 9已经包含，因此不再维护。
* [InvokeBinder](https://github.com/headius/invokebinder)：用于绑定方法处理的Java DSL向前移植。
* [Virtual Thread Bridge](https://github.com/thunkware/virtual-threads-bridge)：该库允许你在Java 8+中使用Java 21的虚拟线程API。
* [JvmDowngrader](https://github.com/unimined/JvmDowngrader)：将现代Java字节码降级为旧版本。
* [JDK Classfile Preview](https://github.com/dmlloyd/jdk-classfile-preview)：这是JDK 21及更高版本中新ClassFile API到JDK 17的非官方向后移植。
* [ModuleFS](https://github.com/xpipe-io/modulefs)：ModuleFS库提供了一个简单的文件系统实现，以统一的方式访问Java模块的内容。

## 接口文档

* [Swagger](https://github.com/swagger-api/swagger-core)：Swagger是OpenAPI规范的Java实现。
* [Swagger Codegen](https://github.com/swagger-api/swagger-codegen)：Swagger Codegen允许根据OpenAPI Spec自动生成API客户端库(SDK生成)、服务器存根和文档。
* [Knife4j](https://gitee.com/xiaoym/knife4j)：Knife4j是一个集Swagger 2和OpenAPI 3为一体的增强解决方案。
* [TypeSpec](https://github.com/microsoft/typespec)：TypeSpec是一种用于定义云服务API和形状的语言，由Microsoft开发。
* [Springfox](https://github.com/springfox/springfox)：Springfox库旨在自动生成使用Spring系列项目编写的JSON API的机器和人类可读规范。
* [Swagger Parser](https://github.com/swagger-api/swagger-parser)：Swagger Parser可以将JSON或YAML格式的OpenAPI定义解析为Java POJO的Swagger-Core表示形式，返回任何验证警告/错误。
* [SpringDoc OpenAPI](https://github.com/springdoc/springdoc-openapi)：SpringDoc OpenAPI库有助于使用Spring Boot项目自动生成API文档。
* [OpenAPI Generator](https://github.com/OpenAPITools/openapi-generator)：OpenAPI Generator允许在给定OpenAPI规范的情况下自动生成API客户端库(SDK生成)、服务器存根、文档和配置。
* [Spring Boot Starter Swagger](https://github.com/SpringForAll/spring-boot-starter-swagger)：该项目主要利用Spring Boot的自动配置特性来实现快速的将Swagger 2引入Spring Boot应用来生成API文档，简化原生使用Swagger 2的整合代码。
* [JApiDocs](https://github.com/YeDaxia/JApiDocs)：JApiDocs是一个Spring Boot无注解API文档生成器。
* [JSONDoc](https://github.com/fabiomaffioletti/jsondoc)：JSONDoc是一个Java库，可用于构建RESTful服务的文档。
* [OpenAPI4J](https://github.com/openapi4j/openapi4j)：OpenAPI 3解析器、JSON模式和请求验证器。
* [Swagger2Word](https://github.com/JMCuixy/swagger2word)：一个Swagger API文档转Word文档的工具项目。
* [Spring REST Docs](https://github.com/spring-projects/spring-restdocs)：该项目的主要目标是通过将使用Asciidoctor手写的内容与使用Spring MVC测试框架生成的自动生成的示例相结合，轻松记录RESTful服务。
* [Springwolf](https://github.com/springwolf/springwolf-core)：使用Spring Boot构建的异步API的自动化文档。
* [Spring Auto REST Docs](https://github.com/ScaCap/spring-auto-restdocs)：Spring REST Docs的扩展。
* [RESTDocs API Spec](https://github.com/ePages-de/restdocs-api-spec)：为Spring REST Docs添加API规范支持。
* [KaiZen OpenApi Parser](https://github.com/RepreZen/KaiZen-OpenApi-Parser)：KaiZen OpenApi Parser是一个基于Java的OpenAPI 3.0验证解析器，提供高度统一的读/写编程API。
* [OpenAPI-diff](https://github.com/OpenAPITools/openapi-diff)：用于比较两个OpenAPI规范的实用程序。
* [Swagger Socket](https://github.com/swagger-api/swagger-socket)：Swagger Socket协议允许在WebSocket协议之上执行任何现有的REST资源。
* [Swagger-Play](https://github.com/swagger-api/swagger-play)：这是一个在Play框架控制器中支持Swagger注解的模块。
* [Swagger Validator Badge](https://github.com/swagger-api/validator-badge)：该项目在网站上显示“valid swagger”徽章，支持Swagger/OpenAPI 2.0和OpenAPI 3.x规范。
* [OpenAPI Style Validator](https://github.com/OpenAPITools/openapi-style-validator)：可定制的样式验证器，可确保你的OpenAPI规范遵循你组织的标准。
* [Smart-Doc](https://gitee.com/TongchengOpenSource/smart-doc)：一款同时支持Java REST API和Dubbo RPC接口文档生成的工具，由同程开源。
* [Doc APIs](https://gitee.com/easy-es/doc-apis)：Doc APIs是一款零侵入接口文档生成工具，由dromara社区开源。
* [Zally](https://github.com/zalando/zally)：Zally是一个简约、易于使用的OpenAPI 2和3 linter。
* [Swaggy Swagger](https://github.com/Swaggy-Swagger/swagger-custom-java)：Swaggy Swagger是一个库，旨在增强API文档工具Swagger的功能和用户体验。
* [Swagger Dubbo](https://github.com/Sayi/swagger-dubbo)：Dubbo的Swagger服务文档。
* [Swagger-Coverage](https://github.com/viclovsky/swagger-coverage)：基于OAS(Swagger) v2和v3生成API测试覆盖率全貌的工具。
* [Swagger Maven Plugin](https://github.com/kongchen/swagger-maven-plugin)：该插件使你的Swagger注解项目能够在Maven构建阶段生成Swagger规范和可定制的模板化静态文档。
* [Swagger2Markup](https://github.com/Swagger2Markup/swagger2markup)：Swagger到AsciiDoc或Markdown转换器，通过将手写文档与自动生成的API文档相结合，简化最新RESTful API文档的生成。
* [AssertJ-Swagger](https://github.com/RobWin/assertj-swagger)：AssertJ-Swagger是一个AssertJ库，它将契约优先的Swagger YAML/JSON文件与代码优先的Swagger JSON输出进行比较。
* [OpenAPI JSON Schema Generator](https://github.com/openapi-json-schema-tools/openapi-json-schema-generator)：OpenAPI JSON Schema Generator允许自动生成API客户端库，重点关注给定OpenAPI文档的JSON模式。
* [OpenAPI v3 Generator Spring Boot](https://github.com/qaware/openapi-generator-for-spring)：该库在运行时自动为Spring Boot应用程序生成OpenApi v3规范。
* [SpringDoc OpenAPI Maven Plugin](https://github.com/springdoc/springdoc-openapi-maven-plugin)：该插件的目的是在运行时生成JSON和YAML OpenAPI描述。
* [xDoc](https://gitee.com/treeleaf/xDoc)：基于Java注释生成接口文档，对代码无侵入，无需注解，纯代码注释。
* [Apigen](https://github.com/apiaddicts/apigen.springboot)：Apigen允许使用OpenAPI文件作为OpenAPI定义和数据库之间的映射工具来生成Spring Boot原型。
* [AutoRest Java](https://github.com/Azure/autorest.java)：用于生成Java代码的AutoRest扩展。
* [Swagger Brake](https://github.com/redskap/swagger-brake)：Swagger-Brake是一个简单的工具，可以验证新版本的API是否会破坏现有版本。
* [OpenAPI Processor Spring](https://github.com/openapi-processor/openapi-processor-spring)：适用于Spring Boot的OpenAPI 3.0和3.1接口和模型Java代码生成器。
* [Swagger Codegen Generators](https://github.com/swagger-api/swagger-codegen-generators)：Swagger Codegen Generators项目是Swagger Codegen 3.0.0项目在其针对特定语言或语言框架的代码生成过程中使用的一组类和模板。
* [Swagger Inflector](https://github.com/swagger-api/swagger-inflector)：该项目使用Swagger规范来驱动API实现。
* [Guardrail](https://github.com/guardrail-dev/guardrail)：Guardrail是一个代码生成工具，能够读取OpenAPI/Swagger规范文件并生成Scala和Java源代码。
* [Swagger Gradle Codegen](https://github.com/Yelp/swagger-gradle-codegen)：用于从Swagger规范文件生成网络代码的Gradle插件。
* [Swagger Maven Plugin](https://github.com/openapi-tools/swagger-maven-plugin)：该插件旨在使用Swagger Core库从基于JAX-RS的REST服务生成OpenAPI文档，并尽可能减少更改。
* [Swagger Codegen Maven Plugin](https://github.com/garethjevans/swagger-codegen-maven-plugin)：一个支持Swagger代码生成项目的Maven插件。
* [Swagger Butler](https://github.com/dyc87112/swagger-butler)：Swagger Butler是一个基于Swagger与Zuul构建的API文档汇集工具。
* [Swagger Diff](https://github.com/Sayi/swagger-diff)：比较两个Swagger API规范(1.x或v2.0)，并将差异呈现到HTML文件或Markdown文件中。
* [Restdocs Spec](https://github.com/BerkleyTechnologyServices/restdocs-spec)：一个使用Spring Restdocs生成Open API和Postman Collection规范的Maven插件。
* [Light Codegen](https://github.com/networknt/light-codegen)：基于Rocker的代码生成器，可以用作命令行实用程序或Web服务。
* [OpenAPI Maven Plugin](https://github.com/kbuntrock/openapi-maven-plugin)：OpenAPI Maven Plugin分析REST控制器Java类并生成相应的OpenAPI 3.0.3文档，它支持Spring MVC、Javax RS和Jakarta RS注解。
* [Swagger Schema Validator](https://github.com/bjansen/swagger-schema-validator)：该库根据Swagger 2规范的definitions部分中定义的模型校验JSON对象。
* [Swagger Spring Boot Starter](https://github.com/battcn/swagger-spring-boot)：Swagger Spring Boot Starter是一款建立在Swagger基础之上的工具包，利用Spring Boot自动装配的特性，简化了传统Swagger的繁琐配置。

## 技术文档

* [AsciidoctorJ](https://github.com/asciidoctor/asciidoctorj)：AsciidoctorJ是在JVM上运行Asciidoctor的官方库，使用AsciidoctorJ，你可以转换AsciiDoc内容或分析来自Java和其他JVM语言的已解析AsciiDoc文档的结构。
* [DocToolchain](https://github.com/docToolchain/docToolchain)：DocToolchain是一个脚本集合，可以轻松创建和维护强大的技术文档。
* [DITA-OT](https://github.com/dita-ot/dita-ot)：DITA-OT是一个开源发布引擎，用于在Darwin信息类型架构中创作的内容。

## Javadoc

* [Markdown Doclet](https://github.com/Abnaxos/markdown-doclet)：Markdown Doclet允许在Javadoc注释中使用Markdown的Doclet。
* [UMLDoclet](https://github.com/talsma-ict/umldoclet)：UMLDoclet可以在Javadoc中自动生成PlantUML图。
* [Asciidoclet](https://github.com/asciidoctor/asciidoclet)：Asciidoclet是一个基于Asciidoctor的Javadoc Doclet，它允许你使用AsciiDoc语法编写Javadoc。
* [Multiline](https://github.com/benelog/multiline)：使用Javadoc注释在Java中实现多行字符串文本。
* [TherAPI Runtime Javadoc](https://github.com/dnault/therapi-runtime-javadoc)：该库可以在运行时读取Javadoc注释。
* [Javadoc Themer](https://github.com/nisrulz/javadoc-themer)：Javadoc Themer用于为Javadoc着色。
* [Codesnippet Javadoc Doclet](https://github.com/jtulach/codesnippet4javadoc)：Codesnippet Doclet可帮助你在文档中包含真实的代码片段，确保它们始终可编译。
* [Deploy Publish JavaDoc](https://github.com/MathieuSoysal/Javadoc-publisher.yml)：自动从Java项目生成Javadoc并将其发布到GitHub Page。

## 文件解析

#### PDF库

* [Apache PDFBox](https://github.com/apache/pdfbox)：PDFBox库是一个用于处理PDF文档的开源Java工具。
* [Stirling PDF](https://github.com/Frooodle/Stirling-PDF)：Stirling PDF是一款功能强大、基于Web的本地托管PDF处理工具。
* [iText](https://github.com/itext/itext7)：iText是一个经过考验的高性能库，可创建、改编、检查和维护PDF文档。
* [Aspose.PDF](https://products.aspose.com/pdf/java/)：Aspose.PDF是一个原生库，可让开发人员为其应用程序添加PDF处理功能。
* [Spire.PDF](https://www.e-iceblue.com/Introduce/pdf-for-java.html)：Spire.PDF是一个PDF API，它使Java应用程序无需使用Adobe Acrobat即可读取、写入和保存PDF文档。
* [OpenPDF](https://github.com/LibrePDF/OpenPDF)：OpenPDF是一个Java库，用于创建和编辑PDF文件。
* [PD4ML](https://pd4ml.com/)：PD4ML提供强大的工具和API，用于生成高质量、定制化、可访问的文档，用于打印和长期电子存档。
* [PDF Clown](https://pdfclown.org/)：PDF Clown是一个用于Java的通用PDF库，专注于严格执行PDF 1.7规范(ISO 32000-1)。
* [X-EasyPDF](https://gitee.com/dromara/x-easypdf)：X-EasyPDF是一个基于PDFBox/FOP二次封装的框架，由dromara社区开源。
* [PDFsam](https://github.com/torakiki/pdfsam)：PDFsam是一款用于拆分、合并、混合、旋转PDF文件和提取页面的桌面应用程序。
* [Tabula](https://github.com/tabulapdf/tabula-java)：Tabula是一个用于从PDF文件中提取表格的库。
* [Apache FOP](https://xmlgraphics.apache.org/fop/)：FOP是由XSL-FO驱动的打印格式化程序和独立于输出的格式化程序。
* [PdfCompare](https://github.com/red6/pdfcompare)：一个简单的Java库，用于比较两个PDF文件。
* [Boxable](https://github.com/dhorions/boxable)：Boxable是一个可用于轻松在PDF文档中创建表格的库。
* [PDF Studio Viewer](https://www.qoppa.com/pdfstudioviewer/)：PDF Studio Viewer是一款可靠且易于使用的跨平台PDF阅读器。
* [EasyTable](https://github.com/vandeseer/easytable)：这是一个基于PDFBox构建的小项目，允许你以相当简单的方式创建表格。
* [Science Parse](https://github.com/allenai/science-parse)：Science Parse解析科学论文(PDF形式)并以结构化形式返回，由艾伦人工智能研究院开源。
* [Sejda](https://github.com/torakiki/sejda)：Sejda SDK是一个用Java编写的面向任务的PDF编辑SDK库。
* [TrapRange](https://github.com/thoqbk/traprange)：TrapRange是一种可用于检测表格内容并将其提取到PDF文件中的数据方法。
* [PDF-Util](https://github.com/vinsguru/pdf-util)：PDF比较工具库。
* [JSignPdf](https://github.com/intoolswetrust/jsignpdf)：JSignPdf是一个为PDF文档添加数字签名的Java应用程序。
* [Briss](https://github.com/mbaeuerle/Briss-2.0)：Briss是一个用于裁剪PDF文件的小型应用程序。
* [PDFrenderer](https://github.com/katjas/PDFrenderer)：使用Java2D将PDF文档渲染到屏幕的Java库。
* [Staplr](https://github.com/pridiltal/staplr)：该库提供了操作PDF文件的函数。
* [VeraPDF](https://github.com/veraPDF/veraPDF-library)：行业支持的开源PDF/A验证库。
* [ICEpdf](https://github.com/pcorless/icepdf)：ICEpdf是一个纯Java PDF文档渲染和查看解决方案。
* [OrsonPDF](https://github.com/jfree/orsonpdf)：OrsonPDF是一个适用于Java平台的PDF生成库，允许你使用标准Java2D绘图API(Graphics2D)创建PDF格式的内容。
* [ComPDFKit](https://github.com/ComPDFKit/compdfkit-api-java)：ComPDFKit提供强大稳定的PDF库和完整的PDF功能来构建PDF查看器和编辑器，允许预览、编辑、注释、签名、加密和解密PDF文件。
* [PDF Test](https://github.com/codeborne/pdf-test)：PDF测试库。

#### Excel库

* [Apache POI](https://github.com/apache/poi)：POI是用于读写Office二进制和OOXML文件格式的Java库。
* [EasyExcel](https://github.com/alibaba/easyexcel)：EasyExcel是一个基于Java的、快速、简洁、解决大文件内存溢出的Excel处理工具，由阿里开源。
* [FastExcel](https://github.com/CodePhiliaX/fastexcel)：EasyExcel最新升级版本，快速、简洁、解决大文件内存溢出的Java处理Excel工具。
* [Aspose.Cells](https://products.aspose.com/cells/java/)：Aspose.Cells提供Excel文件生成、转换和操作。
* [Spire.XLS](https://www.e-iceblue.com/Introduce/xls-for-java.html)：Spire.XLS是一个专业的Java Excel API，使开发人员无需使用Microsoft Office或Microsoft Excel即可创建、管理、操作、转换和打印Excel工作表。
* [Docx4j](https://github.com/plutext/docx4j)：Docx4j是一个开源库，用于创建、编辑和保存OpenXML包，包括docx、pptx和xslx。
* [MyExcel](https://github.com/liaochong/myexcel)：MyExcel是一个集导入导出、加密Excel等多项功能的工具包。
* [EasyPOI](https://gitee.com/wupaas/easypoi)：EasyPOI是一个POI工具库，提供了Excel的快速导入导出、Excel模板导出、Word模板导出。
* [Excel4j](https://gitee.com/Crab2Died/Excel4J)：Excel4j是基于POI的Excel和Commons-CSV的CSV操作组件。
* [EasyFile](https://gitee.com/openquartz/easy-file)：EasyFile是一整套Web大文件导出解决方案，可以轻松导出千万以上数据。
* [FastExcel](https://github.com/dhatim/fastexcel)：FastExcel可用于快速生成和读取大Excel文件。
* [JXLS](https://github.com/jxlsteam/jxls)：Jxls是一个小型且易于使用的Java库，用于使用Excel模板文件生成Excel报告。
* [Poiji](https://github.com/ozlerhakan/poiji)：Poiji是一个小型线程安全Java库，提供从Excel工作表到Java类的一种映射方式。
* [AutoPOI](https://github.com/jeecgboot/autopoi)：AutoPOI是Excel和Word的简易工具类，由北京国炬公司开发。
* [XresLoader](https://github.com/xresloader/xresloader)：XresLoader是一组用于把Excel数据结构化并导出为程序可读的数据文件的导表工具集。
* [AutoExcel](https://github.com/feng-haitao/auto-excel)：AutoExcel是Excel的快速导入和导出工具。
* [Excel Streaming Reader](https://github.com/monitorjbl/excel-streaming-reader)：使用POI的流式Excel读取器的易于使用的实现。
* [ZeroCell](https://github.com/creditdatamw/zerocell)：ZeroCell提供了一个简单的API，用于使用注解将Excel中的数据加载到POJO中，将Excel中的列映射到Java类中的字段。
* [EEC](https://github.com/wangguanquan/eec)：EEC是一款轻量且高效的Excel读写工具，它具有包体小、接入代码量少和运行时消耗资源少等优点。
* [ExcelKit](https://gitee.com/wuwenze/ExcelKit)：简单、好用且轻量级的海量Excel文件导入导出解决方案。
* [ExcelUtil](https://github.com/SargerasWang/ExcelUtil)：用于导入导出Excel的Util包，基于Java的POI。
* [POI Excel](https://gitee.com/stupid1t/poi-excel)：POI-Excel是一个基于POI的Java工具，旨在简化新手在处理Excel表格时的操作。
* [Chimm.Excel](https://gitee.com/chimmhuang/chimm.excel)：Chimm.Excel是一个用Java写的Excel生成工具，基于模板操作，简单、快捷、易上手。
* [ExcelCompare](https://github.com/na-ka-na/ExcelCompare)：ExcelCompare是一个命令行工具，用于比较Excel/Open document(ods)电子表格。
* [ZK Spreadsheet](https://github.com/zkoss/zkspreadsheet)：ZK Spreadsheet是一个开源的、可嵌入的、基于Web的在线电子表格，它使用纯Java在浏览器中提供Excel的丰富功能。
* [ToolGood.Algorithm](https://github.com/toolgood/ToolGood.Algorithm)：ToolGood.Algorithm是一个功能强大、轻量级、兼容Excel公式的算法类库，旨在提高开发人员在不同业务场景中的生产力。
* [Keikai](https://github.com/keikai/dev-ref)：Keikai是一个轻松构建电子表格驱动的Web应用程序。
* [Xcelite](https://github.com/eBay/xcelite)：Xcelite是一个类似ORM的Java库，它允许你轻松地将Java Bean序列化到Excel电子表格或从Excel电子表格反序列化Java Bean，由eBay开源。
* [Java Excel API](https://jexcelapi.sourceforge.net/)：Java Excel API是一个成熟的开源Java API，使开发人员能够动态读取、写入和修改Excel电子表格。
* [HY Common Report](https://github.com/HY-Org/hy.common.report)：报表、Excel操作类库。
* [DsExcel Java](https://github.com/GrapeCity/DsExcel-Java)：高速Java Excel电子表格API库。
* [Excel Boot](https://github.com/programmeres/excel-boot)：Excel Boot是一款Excel导入导出解决方案组成的轻量级开源组件。
* [MemPOI](https://github.com/firegloves/MemPOI)：使用POI简化从数据库导出到Excel文件的库。
* [Excel Plus](https://github.com/hellokaton/excel-plus)：提高Excel操作库的生产力。

#### CSV库

* [Apache Commons CSV](https://github.com/apache/commons-csv)：Commons CSV库提供了一个简单的接口，用于读取和写入各种类型的CSV文件。
* [AdaptiveTableLayout](https://github.com/Cleveroad/AdaptiveTableLayout)：可以读取、编辑和写入CSV文件的库。
* [MyExcel](https://github.com/liaochong/myexcel)：MyExcel是一个集导入导出、加密Excel等多项功能的工具包，支持CSV文件。
* [Super CSV](https://github.com/super-csv/super-csv)：Super CSV是一个快速、程序员友好的开源库，用于使用Java读写CSV文件。
* [FastCSV](https://github.com/osiegmar/FastCSV)：FastCSV是一个快如闪电、无依赖的Java CSV库，符合RFC标准。
* [Excel4j](https://gitee.com/Crab2Died/Excel4J)：Excel4j是基于POI的Excel和Commons-CSV的CSV操作组件。
* [Charred](https://github.com/cnuernber/charred)：Charred是针对CSV和JSON格式的高效基于字符的文件解析库。
* [Jackson Dataformats Text](https://github.com/FasterXML/jackson-dataformats-text)：支持通过Jackson抽象读取和写入CSV编码数据。
* [UniVocity Parsers](https://github.com/uniVocity/univocity-parsers)：UniVocity Parsers是速度最快功能最全的CSV开发库之一，同时支持CSV与固定宽度记录的读写。
* [Scala CSV](https://github.com/tototoshi/scala-csv)：用于Scala的CSV读写库。
* [OpenCSV](https://opencsv.sourceforge.net/)：OpenCSV是一个易于使用的Java CSV解析器库。
* [kotlin CSV](https://github.com/doyaaaaaken/kotlin-csv)：纯Kotlin CSV读写库。
* [FlatPack](https://flatpack.sourceforge.net/)：FlatPack是一个Java文件解析器，用于处理CSV、固定长度和自定义分隔符。
* [CSVeed](https://github.com/42BV/CSVeed)：CSVeed是一个Java库，用于CSV文件并将其公开为行或Java Bean。
* [Java CSV](http://sourceforge.net/projects/javacsv)：Java CSV是一个小型快速开源Java库，用于读写CSV和纯分隔文本文件。
* [DeCS](https://github.com/diergo/decs)：DeCS是一个简单的Java 8 CSV解析器和生成器。
* [Daff](https://github.com/paulfitz/daff)：这是一个用于比较表格、生成其差异摘要并将此类摘要用作补丁文件的库。
* [CSV Utils](https://ostermiller.org/utils/CSV.html)：用于读取和写入CSV文本文件的工具类。
* [CsvJdbc](https://github.com/simoc/csvjdbc)：CsvJdbc是一个只读JDBC驱动程序，它使用CSV文件或DBF文件作为数据库表，非常适合编写数据导入程序或分析日志文件。
* [Deephaven CSV](https://github.com/deephaven/deephaven-csv)：Deephaven CSV库是一个高性能、面向列、类型推断的CSV解析器。
* [PicoCSV](https://github.com/nbbrd/picocsv)：Java的轻量级CSV库。

#### Word库

* [Aspose.Words](https://products.aspose.com/words/java/)：Aspose.Words是一个原生库，它为开发人员提供了丰富的功能来创建、编辑和转换Word、PDF、Web文档，而无需在系统上安装Microsoft Word环境。
* [Spire.Doc](https://www.e-iceblue.com/Introduce/doc-for-java.html)：Spire.Doc是一个专业的Word API，它使Java应用程序能够创建、转换、处理和打印Word文档，而无需依赖Microsoft Word。
* [POI-TL](https://github.com/Sayi/poi-tl)：POI-TL是一个Word模板引擎，可以根据Word模板和数据生成新文档。
* [Docx4j](https://github.com/plutext/docx4j)：Docx4j是一个开源库，用于创建、编辑和保存OpenXML“包”，包括docx、pptx和xslx。
* [Docx Stamper](https://github.com/thombergs/docx-stamper)：Docx Stamper是一个用于docx文档的Java模板引擎。
* [WordGO](https://github.com/qrpcode/wordgo)：让Java生成word文档更容易。

#### PPT库

* [Aspose.Slides](https://products.aspose.com/slides/java/)：Aspose.Slides是一个用于演示文稿操作和管理的Java PowerPoint API。
* [Spire.Presentation](https://www.e-iceblue.com/Introduce/presentation-for-java.html)：Spire.Presentation是一个专业的PowerPoint API，它使开发人员能够在Java应用程序中创建、读取、编写、转换和保存PowerPoint文档。
* [PPTShow](https://github.com/qrpcode/pptshow)：Java生成PPT文档工具包，支持2010版PPTX新功能。
* [PPT Template](https://github.com/Coreoz/PPT-Templates)：PPT Template是一个用于生成PowerPoint演示文稿的小型模板库。

#### ODS库

* [jOpenDocument](https://www.jopendocument.org/)：用于OASIS Open Document文件操作的纯Java库。
* [SODS](https://github.com/miachm/SODS)：Java中用于处理ODS文件的简单库。
* [FastODS](https://github.com/jferard/fastods)：FastODS是一个非常快速且轻量级的库，用于在Java中创建ODS(开放文档电子表格，主要用于Calc)文件。

#### DBF库

* [JDBF](https://github.com/iryndin/jdbf)：用于读写DBF文件的Java实用程序。
* [JavaDBF](https://github.com/albfernandez/javadbf)：JavaDBF是一个用于读写XBase文件的Java库。
* [DBF](https://github.com/jamel/dbf)：用于快速读取/写入DBF文件的Java库。

#### INI库

* [Ini4j](https://github.com/facebookarchive/ini4j)：Ini4j是一个简单的Java API，用于处理Windows .ini格式的配置文件，由Facebook开发。
* [JIniFile](https://github.com/SubZane/JIniFile)：JIniFile从INI文件中存储和检索特定于应用程序的信息和设置。
* [Java INI Parser](https://github.com/vincentrussell/java-ini-parser)：Java INI Parser帮助你在Java中解析ini文件。

#### Toml库

* [TomlJ](https://github.com/tomlj/tomlj)：TomlJ是一个完整的TOML解析器。
* [Toml4j](https://github.com/mwanji/toml4j)：Toml4j是一个用于Java的TOML 0.4.0解析器。

#### HTML库

* [Aspose.HTML](https://products.aspose.com/html/java/)：Aspose.HTML是一种高级HTML操作API，用于在Java应用程序内操作和生成HTML。
* [JFiveParse](https://github.com/digitalfondue/jfiveparse)：一个符合Java HTML 5的解析器。
* [JsoupXpath](https://github.com/zhegexiaohuozi/JsoupXpath)：纯Java实现的支持W3C Xpath 1.0标准语法的HTML解析器。
* [JTidy](https://github.com/jtidy/jtidy)：JTidy是HTML Tidy的Java端口，一个HTML语法检查器和漂亮的打印机。
* [OWASP Java HTML Sanitizer](https://github.com/owasp/java-html-sanitizer)：一个用Java编写的快速且易于配置的HTML Sanitizer，可让你在Web应用程序中包含第三方编写的HTML，同时防止XSS，由OWASP开源。
* [J2Html](https://github.com/tipsy/j2html)：Java到HTML生成器。
* [Jspoon](https://github.com/DroidsOnRoids/jspoon)：Jspoon是一个Java库，它提供基于CSS选择器将HTML解析为Java对象的功能。
* [NekoHTML](https://github.com/codelibs/nekohtml)：HTML解析器和标签平衡器。
* [Fruit](https://github.com/graycreate/Fruit)：Fruit是一个Java库，可用于将HTML转换为Java对象。
* [CSSBox](https://github.com/radkovo/CSSBox)：CSSBox是一个用纯Java编写的(X)HTML/CSS渲染引擎。
* [HTMLParser](https://github.com/validator/htmlparser)：Validator.nu HTML解析器是HTML解析算法的Java实现。
* [jWebForm](https://github.com/jochen777/jWebForm)：jWebForm可以以简洁的方式定义HTML表单，用请求变量填充它们，验证并从中构建HTML。
* [AttoParser](https://github.com/attoparser/attoparser)：AttoParser是一个用于XML和HTML标签的Java解析器。
* [RenderSnake](https://github.com/emicklei/rendersnake)：RenderSnake是一个Java库，用于创建仅使用Java生成HTML的组件和页面。
* [Ultralight Java](https://github.com/LabyMod/ultralight-java)：Ultralight Web引擎的Java包装器，Ultralight是一款轻量级、跨平台的HTML渲染引擎，适用于桌面应用和游戏。

#### EPUB库

* [FolioReader Android](https://github.com/FolioReader/FolioReader-Android)：FolioReader Android是一款用Java和Kotlin编写的EPUB读取器。
* [Epublib](https://github.com/psiegman/epublib)：Epublib是一个用于读取/写入/操作epub文件的Java库。
* [EPUBCheck](https://github.com/w3c/epubcheck)：EPUBCheck是EPUB出版物的官方一致性检查器，由W3C开源。
* [EpubParser](https://github.com/mertakdut/EpubParser)：EpubParser是一个用于解析epub文件的Java库。

#### Outlook库

* [Aspose.Email](https://products.aspose.com/email/java/)：Aspose.Email可以在Java应用程序内创建Outlook电子邮件、解析电子邮件或转换消息格式，如MSG、EML、MBOX、PST、OST和MHT。
* [Jotlmsg](https://github.com/ctabin/jotlmsg)：这是一个简单的API，用于轻松生成Microsoft Outlook消息文件(.msg)。
* [OST2PST](https://github.com/mkorthof/ost2pst)：将Outlook OST文件转换为PST格式。
* [OLMReader](https://github.com/teverett/OLMReader)：用于读取MS Outlook for Mac OLM档案的Java库。
* [Outlook Message Parser](https://github.com/bbottema/outlook-message-parser)：Outlook Message Parser是一个小型开源Java库，用于解析Outlook .msg文件。
* [LibPST](https://github.com/rjohnsondev/java-libpst)：一个使用Java读取PST文件的库。

#### License库

* [TrueLicense](https://github.com/christian-schlichtherle/truelicense)：用于JVM上许可证管理的开源引擎。
* [FOSSLight](https://github.com/fosslight/fosslight)：FOSSLight Hub通过管理开源、许可证和漏洞，帮助你合规、安全地使用开源软件。
* [Licensius](https://github.com/decebals/licensius)：Java微型许可框架。
* [HawkEye](https://github.com/korandoru/hawkeye)：简单的许可证头检查器和格式化程序，有多种分发形式。
* [License3j](https://github.com/verhas/License3j)：License3j是一个免费开源Java库，用于管理需要技术许可证管理强制支持的Java程序中的许可证文件。
* [License](https://github.com/kobeyk/license)：软件许可证书生成+验证。
* [Candlepin](https://github.com/candlepin/candlepin)：Candlepin是一个开源订阅和授权引擎，旨在从供应商和客户的角度管理软件订阅。
* [Solicitor](https://github.com/devonfw/solicitor)：Solicitor是一款能够管理软件依赖项许可证的工具。
* [Smart License](https://gitee.com/smartboot/smart-license)：Smart-License是一款用于安全加固的开源项目，主要服务于非开源产品、商业软件、具备试用功能的付费软件等，为软件提供授权制的使用方式。

#### Markdown库

* [Txtmark](https://github.com/rjeschke/txtmark)：Java Markdown处理器。
* [MarkdownJ](https://github.com/myabc/markdownj)：MarkdownJ是Markdown(John Gruber编写的文本到HTML转换工具)的纯Java端口。
* [Markwon](https://github.com/noties/Markwon)：Android Markdown库。
* [MarkedJ](https://github.com/gitbucket/markedj)：优雅Markdown处理器marked.js的JVM端口。
* [Commonmark-Java](https://github.com/commonmark/commonmark-java)：用于根据CommonMark规范解析和渲染Markdown文本的Java库，由Atlassian开源。
* [Java Markdown Generator](https://github.com/Steppschuh/Java-Markdown-Generator)：用于生成Markdown的Java库。
* [Pegdown](https://github.com/sirthias/pegdown)：基于parboiled PEG解析器的纯Java Markdown处理器，支持多种扩展。
* [Intellij Markdown](https://github.com/JetBrains/markdown)：用Kotlin编写的多平台Markdown处理器，由JetBrains开源。
* [MarkupDocBuilder](https://github.com/Swagger2Markup/markup-document-builder)：一个支持AsciiDoc、Markdown和Confluence Wiki的标签文档生成器。
* [Nutz](https://github.com/sangupta/nutz)：JVM的Markdown处理器，手工编码的解析器生成AST，并允许轻松添加扩展。
* [MarkdownPapers](https://github.com/lruiz/MarkdownPapers)：用Java实现的Markdown解析器和转换器。
* [Markdown To AsciiDoc](https://github.com/markdown-asciidoc/markdown-to-asciidoc)：一个小型、轻量级的Markdown到AsciiDoc转换器，用Java编写，基于Pegdown。
* [MDTool](https://github.com/cevin15/MDTool)：一个可以将Markdown转换为HTML的工具。
* [KeenWrite](https://gitlab.com/DaveJarvis/KeenWrite)：免费、开源、跨平台桌面Markdown文本编辑器，具有实时预览、字符串插值和数学功能。

#### 文件库

* [CDC](https://gitlab.com/cdc-java/cdc-office)：与Office文档相关的工具类。
* [DotEnv](https://github.com/cdimascio/dotenv-java)：Ruby DotEnv项目的无依赖、纯Java端口，用于从.env文件加载环境变量。
* [Dotenv Kotlin](https://github.com/cdimascio/dotenv-kotlin)：用于Java和Kotlin的Ruby dotenv项目的端口。
* [HWPLib](https://github.com/neolord0/hwplib)：Java的HWP库。
* [MPXJ](https://github.com/joniles/mpxj)：该库使你能够从各种文件格式和数据库中读取项目计划(有时称为进度表或项目集)，还可以将该信息写入各种文件格式。
* [OSMPBF](https://github.com/openstreetmap/OSM-binary)：OSMPBF是一个用于读取和写入OpenStreetMap PBF文件的Java/C++库。
* [WaveAccess](https://github.com/sintrb/WaveAccess)：波形文件(.wav)的Java读写操作库。
* [JPMML Evaluator](https://github.com/jpmml/jpmml-evaluator)：用于生成和使用PMML文档的Java库。
* [Org Java](https://github.com/orgzly/org-java)：Org模式文件Java解析器。
* [JElf](https://github.com/fornwall/jelf)：用于解析ELF文件的Java库。
* [ODF Toolkit](https://github.com/tdf/odftoolkit)：ODF Toolkit是一组Java模块，允许以编程方式创建、扫描和操作ODF文档。
* [OFDRW](https://gitee.com/ofdrw/ofdrw)：开源的OFD处理库，支持文档生成、数字签名、文档保护、文档合并、转换、导出等功能。
* [jHDF](https://github.com/jamesmudd/jhdf)：该项目是用于访问HDF5文件的纯Java实现。
* [MSLinks](https://github.com/DmitriiShamrikov/mslinks)：用于解析和创建Windows快捷方式文件(.lnk)的库。
* [HCL4j](https://github.com/bertramdev/hcl4j)：HCL4j是JVM上Hashicorp配置语言的解析器。
* [Apron](https://github.com/poiu-de/apron)：Apron是一个用于读写Java .properties文件的小型库。
* [Obj](https://github.com/javagl/Obj)：Obj是一个简单的Wavefront OBJ文件加载器和写入器。
* [Java Date Front](https://github.com/mokiat/java-data-front)：用于读取Wavefront 3D模型资源(OBJ、MTL)的Java库。
* [JglTF](https://github.com/javagl/JglTF)：与glTF相关的Java库。
* [LASzip4j](https://github.com/mreutegg/laszip4j)：LASzip库的Java移植。
* [Java netCDF](https://github.com/Unidata/netcdf-java)：netCDF Java库提供了科学数据访问的接口，它可用于从各种文件格式读取科学数据，包括netCDF、HDF、GRIB、BUFR等，由美国国家科学基金会开源。
* [DD PList](https://github.com/3breadt/dd-plist)：一个Java库，提供对ASCII、XML和二进制属性列表的支持。
* [Meico](https://github.com/cemfi/meico)：Meico是MEI文件的转换器框架，由帕德博恩大学开源。
* [OneBusAway GTFS](https://github.com/OneBusAway/onebusaway-gtfs-modules)：用于读取和写入GTFS源的Java库，包括数据库支持。
* [Jcabi Manifests](https://github.com/jcabi/jcabi-manifests)：用于方便读取类路径中可用的MANIFEST.MF文件的Java库。
* [X12 Parser](https://github.com/imsweb/x12-parser)：用于ANSI ASC X12文档的解析器。
* [Samchika](https://github.com/MayankPratap/Samchika)：Samchika是Java语言的一个可重复、易于使用且速度极快的文件处理库。

#### 文件转换

* [PDF2JSON](https://github.com/modesty/pdf2json)：PDF2JSON是一个PDF文件解析器，可将PDF二进制文件转换为基于文本的JSON。
* [Retrosheet](https://github.com/theapache64/retrosheet)：将Google电子表格转换为JSON端点。
* [OPENHTMLTOPDF](https://github.com/danfickle/openhtmltopdf)：OPENHTMLTOPDF是一个纯Java库，用于使用CSS 2.1进行布局和格式化，输出为PDF或图像，呈现格式良好的XML/XHTML的合理子集。
* [PDFLayoutTextStripper](https://github.com/JonathanLink/PDFLayoutTextStripper)：将PDF文件转换为文本文件，同时保留原始PDF的布局。
* [Java WkHtmlToPdf Wrapper](https://github.com/jhonnymertz/java-wkhtmltopdf-wrapper)：WkHtmlToPdf命令行工具的基于Java的包装器。
* [Pdf2Dom](https://github.com/radkovo/Pdf2Dom)：Pdf2Dom是一个PDF解析器，可将文档转换为HTML DOM表示形式。
* [HTMLToPDF](https://github.com/wooio/htmltopdf-java)：该项目基于WkHtmlToPdf，它将HTML文档转换为PDF。
* [PDF Converter](https://github.com/jmrozanec/pdf-converter)：一个Java库，用于将.pdf文件转换为.epub、.txt、.png、.jpg、.zip格式。
* [Documents4j](https://github.com/documents4j/documents4j)：Documents4j是一个用于将文档转换为另一种文档格式的Java库。
* [Mammoth](https://github.com/mwilliamson/java-mammoth)：Mammoth旨在转换.docx文档，例如由Microsoft Word、Google Docs和LibreOffice创建的文档，并将其转换为HTML。
* [Html2Image](https://github.com/hkirk/java-html2image)：这个简单的Java库将纯HTML标签转换为图像，并使用HTML元素提供客户端图像映射。
* [XmlToJson](https://github.com/smart-fun/XmlToJson)：用于将XML转换为JSON以及将JSON转换为XML的Android库。
* [Jettison](https://github.com/jettison-json/jettison)：一个用于在StAX的帮助下将XML和JSON相互转换的Java库，它实现XMLStreamWriter和XMLStreamReader并支持Mapped和BadgerFish约定。
* [Apache XML Graphics](https://xmlgraphics.apache.org/)：从XML到图形输出的转换工具。
* [Json2Xml](https://github.com/lukas-krecan/json2xml)：Json2Xml项目是JSON到XML转换的简单实现。
* [Fugerit Doc](https://github.com/fugerit-org/fj-doc)：从XML文档元模型开始生成不同输出格式的文档的框架。
* [Silencio](https://github.com/damianszczepanik/silencio)：Silencio是一个用于转换XML、JSON、YAML、Properties和其他格式的Java库。
* [JODConverter](https://github.com/jodconverter/jodconverter)：JODConverter使用LibreOffice或OpenOffice自动执行文档转换。

## 集群管理

* [Apache Aurora](https://github.com/apache/aurora)：Aurora是一个用于长期运行服务和cron作业的Mesos框架，由Twitter开源。
* [Chronos](https://github.com/mesos/chronos)：Chronos是cron的替代品，它是一个分布式容错调度程序，运行在Mesos之上，可用于作业编排。
* [Terracotta](https://github.com/Terracotta-OSS/terracotta-core)：Terracotta Server为Terracotta产品(例如Ehcache和TCStore)提供强大的分布式内存数据管理功能，是Terracotta集群的骨干。
* [Singularity](https://github.com/HubSpot/Singularity)：Singularity是一种API和Web应用程序，用于运行和调度Mesos任务，包括长时间运行的进程、计划作业和一次性任务。
* [Haven](https://github.com/codeabovelab/haven-platform)：Haven是一个Docker集群管理系统，用户可以通过用户友好且功能强大的用户界面和命令行工具控制整个平台。
* [Norbert](https://github.com/rhavyn/norbert)：Norbert是一个提供简单集群管理和集群感知客户端/服务器网络API的库，由Linkedin开源。
* [Apache Helix](https://github.com/apache/helix)：Helix是一个通用集群管理框架，用于自动管理节点集群上托管的分区、复制和分布式资源，由LinkedIn开源。
* [Apache Airavata](https://airavata.apache.org/)：用于在分布式计算资源(包括本地集群、超级计算机、国家电网、学术和商业云)上执行和管理计算作业和工作流程的软件框架。
* [Fenzo](https://github.com/Netflix/Fenzo)：适用于Mesos框架的调度程序Java库，支持调度优化插件并促进集群自动扩展，由Netflix开源。
* [Apache REEF](https://github.com/apache/reef)：用于为集群资源管理器(例如Hadoop YARN或Mesos)开发可移植应用程序的库。例如，Microsoft Azure流分析是基于REEF和Hadoop构建的。
* [Orion](https://github.com/pinterest/orion)：Orion是一个适用于有状态分布式系统的通用可插拔管理和自动化平台，由Pinterest开源。
* [Apache Myriad](https://github.com/apache/incubator-myriad)：Myriad是一个Mesos框架，旨在扩展Mesos上的YARN集群，由eBay、MapR和Mesosphere开源。
* [Declarative Cluster Management](https://github.com/vmware/declarative-cluster-management)：DCM使程序员能够使用高级声明性语言(SQL)构建调度程序和集群管理器，由VMware开源。
* [Marathon](https://github.com/mesosphere/marathon)：Marathon是经过生产验证的用于容器编排的Mesos框架。

## 容器化工具

* [Jib](https://github.com/GoogleContainerTools/jib)：Jib无需Docker守护进程即可为Java应用程序构建优化的Docker和OCI镜像，它可作为Maven和Gradle的插件以及Java库使用，由Google开源。
* [Dockerfile Maven](https://github.com/spotify/dockerfile-maven)：该库包含一组用于处理Dockerfile的Maven工具，由Spotify开源。
* [Docker Maven Plugin](https://github.com/spotify/docker-maven-plugin)：用于构建和推送Docker镜像的Maven插件，Spotify开源，该项目不再活跃。
* [Helios](https://github.com/spotify/helios)：Helios是一个Docker编排平台，用于跨整个服务器群部署和管理容器，由Spotify开源。
* [Docker Maven Plugin](https://github.com/fabric8io/docker-maven-plugin)：用于运行和创建Docker镜像的Maven插件。
* [Terrakube](https://github.com/AzBuilder/terrakube)：Terrakube是一个开源协作平台，用于使用Terraform或OpenTofu将远程基础设施作为代码操作运行。
* [Eclipse JKube](https://github.com/eclipse/jkube)：JKube是插件和库的集合，用于使用Docker、JIB或S2I构建策略构建容器镜像。
* [Cattle](https://github.com/rancher/cattle)：Cattle是为Rancher提供支持的编排引擎，它的主要作用是元数据管理和外部系统的编排。
* [Stargate](https://github.com/ppdaicorp/stargate)：Stargate是一个基于Kubernetes和Docker的应用发布平台，由信也科技开源。
* [HyScale](https://github.com/hyscale/hyscale)：HyScale是基于K8s的以应用程序为中心的抽象框架。
* [Styx](https://github.com/spotify/styx)：Styx是一项用于触发Docker容器定期调用的服务，由Spotify开源。
* [Blox](https://github.com/blox/blox)：Blox提供针对在Amazon ECS上运行应用程序进行优化的开源调度程序，由Amazon开源。
* [Haven](https://github.com/codeabovelab/haven-platform)：Haven是一个开源Docker容器管理系统，它将容器、应用程序、集群、镜像和注册表管理集成在一处。
* [StackGres](https://github.com/ongres/stackgres)：StackGres是Kubernetes的全栈PostgreSQL发行版，打包到一个简单的部署单元中，拥有一组精心挑选和调整的周边PostgreSQL组件。
* [Kubernetes Operators](https://github.com/operator-framework/java-operator-sdk)：Java Operator SDK是一个生产就绪的框架，可以轻松地在Java中实现Kubernetes Operator。
* [Dekorate](https://github.com/dekorateio/dekorate)：用于生成Kubernetes相关清单的工具。
* [KubeHelper](https://github.com/KubeHelper/kubehelper)：KubeHelper通过Web界面简化了许多日常Kubernetes集群任务，搜索、分析、运行命令、Cron作业、报告、过滤器、Git同步等等。
* [Titus](https://github.com/Netflix/titus-control-plane)：Titus是Netflix容器管理平台，用于管理容器并提供与基础设施生态系统的集成。

## DevOps

* [Ice](https://github.com/Teevity/ice)：Ice可以从使用情况和成本的角度鸟瞰庞大而复杂的云环境，由Netflix开源。
* [DHorse](https://github.com/512team/dhorse)：DHorse是一个轻量级、简单易用的云应用管理平台，具有多云和多环境管理、应用管理和部署、服务治理等功能。
* [MQCloud](https://github.com/sohutv/mqcloud)：RocketMQ企业级一站式服务平台，由搜狐开源。
* [EazyBuilder](https://github.com/iSoftStoneGroup/EazyBuilder)：EazyBuilder是一套完整的云原生架构下的DevOps平台项目，由软通动力开源。
* [OpsCloud4](https://github.com/ixrjog/opscloud4)：OpsCloud4是用于云上运维的工具，提供持续交付、多实例动态数据源、堡垒机等功能。
* [SREWorks](https://github.com/alibaba/SREWorks)：SREWorks专注于以应用为中心的开发模式，提供一站式云原生数智化运维SaaS管理套件，由阿里开源。
* [WGCLOUD](https://github.com/tianshiyeben/wgcloud)：WGCLOUD是Linux运维监控工具，支持系统硬件信息、内存、CPU、温度、磁盘空间及IO、硬盘smart、系统负载、网络流量等监控。
* [MoSKito](https://github.com/anotheria/moskito)：MoSKito是一个开源系统，用于监控Java Web应用程序的性能和行为。
* [Choerodon](https://gitee.com/choerodon/choerodon)：Choerodon数智化开发管理平台，提供体系化方法论和协作、测试、DevOps及容器工具，由甄知科技开源。
* [Dubbo Admin](https://github.com/apache/dubbo-admin)：Dubbo Admin是为了更好地可视化Dubbo服务而设计的控制台，由阿里开发。
* [Dubbokeeper](https://github.com/dubboclub/dubbokeeper)：Dubbokeeper是一个开源版本基于Spring MVC开发的社区版DubboAdmin。
* [CloudExplorer Lite](https://github.com/CloudExplorer-Dev/CloudExplorer-Lite)：CloudExplorer Lite脱胎于飞致云创始软件产品CloudExplorer多云管理平台，支持对接纳管主流的公有云和私有云基础设施，提供开箱即用的云主机管理、云账单、运营分析和安全合规等基本功能。
* [Ward](https://github.com/Rudolf-Barbu/Ward)：Ward是一个简单简约的服务器监控工具，支持自适应设计系统。
* [Yunyi](https://gitee.com/openeuler/yunyi)：云翼数据缓存中间件云原生管理平台通过将数据缓存中间件服务纳入管理，增加数据缓存产品的云服务能力，由华为开发。
* [Kaelthas](https://github.com/tiklab-project/tiklab-kaelthas)：支持主机、网络、数据库、容器等各种监控，支持灵活多样的告警通知方式。
* [Jianmu](https://gitee.com/jianmu-dev/jianmu)：建木是一个面向DevOps领域的极易扩展的开源无代码(图形化)/低代码(GitOps)工具。
* [DomeOS](https://github.com/domeos/server)：DomeOS是搜狐北京研发中心打造的一款基于Docker的企业级应用编排运维管理系统。
* [Kardio](https://github.com/tmobile/kardio)：Kardio是一个简单的工具，可以配置为在任何端点上执行运行状况检查，由T-Mobile开源。
* [Pallet](https://github.com/pallet/pallet)：Pallet用于在云和虚拟机基础设施上配置和维护服务器，旨在解决跨各种云提供一致配置的运行镜像的问题。
* [Orion Visor](https://gitee.com/dromara/orion-visor)：Orion Visor是一款高颜值、现代化的智能运维&轻量堡垒机平台，由dromara社区开源。
* [Rundeck](https://github.com/rundeck/rundeck)：Rundeck是一种开源自动化服务，具有Web控制台、命令行工具和Web API，它使你可以轻松地跨一组节点运行自动化任务。
* [Uyuni](https://github.com/uyuni-project/uyuni)：Uyuni是一个开源系统管理解决方案，源自Spacewalk。
* [WeCube](https://github.com/WeBankPartners/wecube-platform)：WeCube是一套开源、一站式IT架构管理和运维管理工具，主要用于简化分布式架构IT管理，并可以通过插件进行功能扩展，由微众开源。
* [MSEC](https://github.com/Tencent/MSEC)：MSEC由腾讯QQ团队开源，它是一个后端Devops引擎，包括RPC、名称查找、负载均衡、监控、发布和容量管理。
* [Phoenix](https://gitee.com/monitoring-platform/phoenix)：Phoenix是一个灵活可配置的开源监控平台，主要用于监控应用程序、服务器、Docker、数据库、网络、TCP端口和HTTP接口。
* [Frostmourne](https://github.com/AutohomeCorp/frostmourne)：Frostmourne是汽车之家经销商技术部监控系统的开源版本，用于帮助监控几乎所有数据库数据(包括Elasticsearch、Prometheus、SkyWalking、MySQL等等)。
* [Cubic](https://github.com/dromara/cubic)：Cubic一站式问题定位平台，分布式实例监控、线程栈监控、线程池监控、动态Arthas命令集、依赖分析等等，由dromara社区开源。
* [Suricate](https://github.com/michelin/suricate)：Suricate是一款开源应用程序，它允许IT团队通过由可在电视上显示的小部件组成的仪表板来监控其环境，由米其林开源。
* [UAVStack](https://github.com/uavorg/uavstack)：UAVStack是智能化服务技术栈，是研发运维一体化的解决方案。
* [Ovirt](https://github.com/oVirt/ovirt-engine)：Ovirt是一个开源的虚拟化管理平台，RedHat虚拟化管理平台RHEV的开源版本。
* [CloudUnit](https://github.com/end-of-game/cloudunit)：CloudUnit是Treeptik开源的DevOps平台。
* [OneOps](https://github.com/oneops/oneops)：OneOps是一个自动化运维开发的云管理平台，由沃尔玛赞助。
* [Jpom](https://gitee.com/dromara/Jpom)：Jpom是一款原生Ops软件，由dromara社区开源。
* [Nginx WebUI](https://gitee.com/cym1102/nginxWebUI)：Nginx WebUI是一款图形化管理Nginx配置的工具。
* [Nginx Admin](https://github.com/jonatansalemes/nginx-admin)：Nginx Admin是一个开源的多平台Nginx软件管理器。
* [SimpleDocker](https://gitee.com/taoes_admin/SimpleDocker)：SimpleDocker是一个简单的Docker控制面板，可以让用户更方便、舒适的使用Docker，其界面简洁、操作便捷，功能强大，可以带来更好地运维体验。
* [Gitaction Board](https://github.com/otto-de/gitactionboard)：Github Actions的仪表板。
* [HeartBeat](https://gitee.com/mkk/HeartBeat)：心跳检测各类应用服务器(如Tomcat、Jetty)，Web服务器(如Apache、Nginx)的Java Web应用程序。
* [Bk Job](https://github.com/TencentBlueKing/bk-job)：蓝鲸作业平台是一套运维脚本管理系统，具备海量任务并发处理能力，腾讯开源。
* [DQOps](https://github.com/dqops/dqo)：DQOps是一款DataOps友好的数据质量监控工具，具有可定制的数据质量检查和数据质量仪表板。
* [OpenNMS](https://github.com/OpenNMS/opennms)：OpenNMS是一个开源网络监控平台，可帮助你可视化和监控本地和分布式网络上的所有内容。
* [Easy Manager Tool](https://gitee.com/aizuda/easy-manager-tool)：Easy Manager Tool集成各类工具的核心使用方法，打造集成化程度高且专业的开、测、维一体化管理工具，由爱组搭开源。
* [RackShift](https://github.com/fit2cloud/rackshift)：RackShift是开源的裸金属服务器管理平台，功能覆盖裸金属服务器的发现、带外管理、RAID配置、固件更新、操作系统安装等，由飞致云开源。
* [Hinemos](https://github.com/hinemos/hinemos)：Hinemos是一款开源集成系统管理软件，提供监控和作业管理(工作负载调度)功能，实现系统操作自动化，由NTT Data开源。
* [NeatLogic](https://gitee.com/neat-logic/neatlogic-itom-all)：NeatLogic是一套渐进式ITOM平台，致力为不同类型、不同规模用户提供完整的ITOM解决方案。
* [Hyperic HQ](https://github.com/hyperic/hq)：Hyperic HQ是一个基于Java的软件资源监测和管理平台。
* [Apitally](https://apitally.io/spring-boot)：Apitally可帮助你了解API的使用方式，并在出现问题时提醒你，只需向你的项目添加两行代码即可。
* [Lite Monitor](https://github.com/haueosc/lite-monitor)：Lite Monitor是一个快速、准确、轻量化的服务器监控系统，拥有秒级的监控粒度，支持历史数据查看便于拥有多平台服务器的用户集中管理，由河南工程学院开发。

## 云服务

* [Nomulus](https://github.com/google/nomulus)：Nomulus是一种开源、可扩展、基于云的服务，用于运营顶级域名(TLD)，由Google开源。
* [ZStack](https://github.com/zstackio/zstack)：ZStack是开源IaaS软件，旨在实现数据中心自动化，通过API管理计算、存储和网络资源，由云轴科技开源。
* [Gaia](https://github.com/gaia-app/gaia)：Gaia是一个用于Terraform模块和自助服务基础设施的Terraform UI。
* [AWS SaaS Boost](https://github.com/awslabs/aws-saas-boost)：AWS SaaS Boost为组织提供即用型核心软件元素，以便在云中成功运行SaaS工作负载，由Amazon开源。
* [Compute Nest SaaS Boost](https://github.com/aliyun/alibabacloud-compute-nest-saas-boost)：计算巢SaaS Boost是由阿里云推出的一款开发工具和框架，旨在帮助(SaaS)开发者快速构建、部署、扩展和售卖SaaS应用程序。
* [Wemirr Platform](https://gitee.com/battcn/wemirr-platform)：优秀、简单、漂亮的开源SaaS、多租户云平台架构。
* [HZERO](https://gitee.com/open-hand/hzero)：HZERO是基于微服务架构开源免费的企业级PaaS平台，由上海汉得公司开发。
* [J2PaaS](https://gitee.com/j2paas/j2paas-framework)：J2PaaS是一个集成开发平台，以参数驱动为核心，为开发者提供可视化、组件化、低代码、拖拽式在线敏捷开发平台，由吉鼎科技开源。
* [Apache Stratos](https://github.com/apache/stratos)：Stratos包括多语言和环境支持，以及在多个IaaS运行时上运行的能力。
* [Myria](https://github.com/uwescience/myria)：Myria是华盛顿大学的分布式、无共享大数据管理系统和云服务。
* [Eclipse Jemo](https://github.com/eclipse/jemo)：Jemo旨在为基于JVM的语言提供真正的多云FaaS实现。
* [Eclipse Dirigible](https://github.com/eclipse/dirigible)：Dirigible是一种高生产力PaaS，它提供了一个由预选执行引擎和内置Web开发工具组成的应用程序服务器，它也适合利用低代码/无代码技术来快速开发业务应用程序，由SAP开源。
* [Eucalyptus](https://github.com/eucalyptus/eucalyptus)：Eucalyptus是用于构建与Amazon Web Services兼容的私有云和混合云的开源软件。
* [Kalix](https://www.kalix.io/)：Kalix是一个PaaS平台，它抽象了事件驱动的微服务的复杂性，团队可以专注于构建应用程序背后的业务逻辑，由Lightbend开源。
* [Paladin Cloud](https://github.com/PaladinCloud/CE)：Paladin Cloud是一个免费的开源云安全平台，致力于帮助你发现云安全中的盲点。
* [Eclipse Winery](https://github.com/eclipse/winery)：Winery是一个基于Web的环境，用于以图形方式对TOSCA拓扑进行建模并计划管理这些拓扑。
* [Alfresco](https://github.com/Alfresco/alfresco-community-repo)：Alfresco平台提供全面的云原生内容服务。
* [XGVela](https://github.com/XGVela/XGVela)：XGVela是由中国移动主导发起的5G云原生PaaS平台开源项目。
* [Asgard](https://github.com/Netflix/asgard)：Asgard是一个基于Web的工具，用于管理基于云的应用程序和基础设施，由Netflix开源。
* [Poja](https://github.com/hei-school/poja)：Poja是一个完整的Java基础设施，由马达加斯加计算机科学高中开源。
* [Eclipse Xpanse](https://github.com/eclipse-xpanse/xpanse)：Xpanse是一个开源项目，允许在任何云服务提供商上轻松实施本机托管服务。

## 软件质量

* [Stan4j](http://stan4j.com/)：STAN是领先的基于Eclipse的Java结构分析工具，以自然的方式将开发和质量保证结合在一起。
* [Socomo](https://github.com/gdela/socomo)：Socomo是一个简单的工具，可以可视化源代码的组成，并在你的Java项目开发过程中跟踪组成的更改。
* [Structure101](https://structure101.com/)：Structure101是一个敏捷体系结构开发环境，它允许软件开发团队组织代码库，这是一个收费软件。
* [Lattix](https://www.lattix.com/)：Lattix使你能够快速识别和修复架构问题。
* [JDepend](https://github.com/clarkware/jdepend)：JDepend遍历Java类和源文件目录，并为每个Java包生成设计质量指标。
* [Macker](https://innig.net/macker/)：Macker是一个为Java开发人员提供的构建时架构规则检查实用程序。
* [Architexa](https://www.architexa.com/)：Architexa帮助你理解和记录大型/复杂的代码库。
* [USL4j](https://github.com/codahale/usl4j)：USL4j是Neil Gunther博士的通用可扩展性定律的Java建模器。

## 编码规范

* [Google Java Format](https://github.com/google/google-java-format)：一个重新格式化Java源代码以符合Google Java风格的程序。
* [Spring Java Format](https://github.com/spring-io/spring-javaformat)：一组可应用于任何Java项目以提供一致的Spring风格的插件。
* [Square Java Code Style](https://github.com/square/java-code-styles)：Square的Java和Android项目的IntelliJ IDEA代码样式设置。
* [Spotless](https://github.com/diffplug/spotless)：Spotless是支持多种语言的代码格式化工具。
* [Prettier Java](https://github.com/jhipster/prettier-java)：Prettier是一个代码格式化程序，它通过解析代码并使用自己的规则重新打印代码来强制执行一致的样式。
* [P3C](https://github.com/alibaba/p3c)：阿里巴巴Java编码指南PMD实现和IDE插件。
* [Cornell Java Code Style](https://www.cs.cornell.edu/courses/JavaAndDS/JavaStyle.html)：康奈尔大学Java编程风格指南。
* [Oracle Java Code Convention](https://www.oracle.com/java/technologies/javase/codeconventions-contents.html)：Oracle官方Java代码约定。
* [Android AOSP Style Guide](https://source.android.com/docs/setup/contribute/code-style?hl=zh-cn)：面向贡献者的AOSP Java代码样式指南。
* [Twitter Java Code Style](https://github.com/twitter-archive/commons/blob/master/src/java/com/twitter/common/styleguide.md)：Twitter提供的一组鼓励优秀代码的约定。
* [Hackday Conventions Java](https://github.com/naver/hackday-conventions-java)：Naver使用的Java开发风格指南。
* [JavaRanch Style Guide](https://coderanch.com/wiki/718799/Style)：Oracle编码风格的替代方案。
* [GVSU Java Coding Style](https://www.gvsu.edu/computing/java-coding-style-guide-37.htm)：大峡谷州立大学的Java编码规范。
* [CMU Java Code Style](https://www.cs.cmu.edu/~rdriley/121/resources/styleguide/)：CMU 15-121源代码编码标准的完整定义。
* [Chromium Java Style Guide](https://chromium.googlesource.com/chromium/src/+/HEAD/styleguide/java/java.md)：Chromium Java编码风格指南。
* [CIS Java Style Guide]()：CIS 120 Java风格指南。
* [Palantir Java Format](https://github.com/palantir/palantir-java-format)：一个现代、Lambda友好、120个字符的Java格式化器。
* [CodingStyle](https://github.com/uhafner/codingstyle)：慕尼黑应用科技大学使用的Java编码风格和模板项目。

## 依赖分析

* [DependencyCheck](https://github.com/jeremylong/DependencyCheck)：OWASP DependencyCheck是一种软件组合分析实用程序，可检测应用程序依赖中公开披露的漏洞。
* [Depends](https://github.com/multilang-depends/depends)：Depends是一个源代码依赖提取工具，旨在从各种编程语言推断源代码实体(例如文件和方法)之间的语法关系。
* [Jarviz](https://github.com/ExpediaGroup/jarviz)：Jarviz是专为Java应用程序设计的依赖分析和可视化工具，由Expedia开源。
* [Classycle](https://classycle.sourceforge.net/)：Classycle可以分析Java应用程序或库中的静态类和包依赖关系。
* [DepAn](https://github.com/google/depan)：DepAn是一个直接操作工具，用于可视化、分析和重构大型应用程序中的依赖关系，Google开源。
* [JDependency](https://github.com/tcurdt/jdependency)：可帮助分析类级别依赖关系、冲突和缺失的类。
* [Dependency-Track](https://github.com/DependencyTrack/dependency-track)：Dependency-Track是一个智能组件分析平台，允许组织识别并降低软件供应链中的风险，由OWASP开源。
* [Degraph](https://github.com/riy/degraph)：Degraph是一个用于可视化和测试JVM应用程序中的类和包依赖关系的工具。
* [Depgraph Maven Plugin](https://github.com/ferstl/depgraph-maven-plugin)：该Maven插件在单个模块上生成依赖关系图，或者在多模块项目上以聚合形式生成依赖关系图。
* [FASTEN](https://github.com/fasten-project/fasten)：在调用图级别分析包依赖网络的工具。
* [MissingLink](https://github.com/spotify/missinglink)：用于检测Java项目中链接问题的构建时工具，由Spotify开源。
* [JResolve-CLI](https://github.com/bowbahdoe/jresolve-cli)：用于解决JVM依赖的命令行工具。
* [Dependency-Analysis](https://github.com/project-ncl/dependency-analysis)：该项目是一项服务，它提供有关已构建工件的信息并分析项目的依赖关系。
* [OSS Review Toolkit](https://github.com/oss-review-toolkit/ort)：OSS Review Toolkit是一个FOSS策略自动化和编排工具包，你可以使用它以战略、安全和高效的方式管理你的软件依赖项。

## 污点分析

* [Gadget Inspector](https://github.com/JackOfMostTrades/gadgetinspector)：GadgetInspector是一个自动化反序列化链挖掘工具，它通过对字节码形式的Java项目进行污点分析，挖掘可能存在的反序列化链。
* [Phosphor](https://github.com/gmu-swe/phosphor)：Phosphor是一个在JVM和商用JVM上执行动态污点跟踪的系统。

## 审计框架

* [Audit4j](https://github.com/audit4j/audit4j-core)：Audit4j是一个开源审计框架，专门设计用于捕获整个企业应用程序中各个组件生成和触发的审计事件。
* [JaVers](https://github.com/javers/javers)：Java的对象审计和差异框架。
* [Java Object Diff](https://github.com/SQiShER/java-object-diff)：Java Object Diff是一个简单但功能强大的库，用于查找Java对象之间的差异。

## 原型工具

* [Penpot](https://github.com/penpot/penpot)：Penpot是第一个用于设计和代码协作的开源设计工具。
* [ForeUI](https://www.foreui.com/)：ForeUI是一款易于使用的UI原型工具，旨在为你想要的任何应用程序或网站创建模型/线框/原型。

## 工件仓库

* [Maven Central](https://central.sonatype.com/)：Maven Central是JVM语言最大的开源组件仓库。
* [JitPack](https://github.com/jitpack/jitpack.io)：JitPack是一个新颖的JVM和Android项目包仓库。
* [Artifactory](https://jfrog.com/artifactory/)：Artifactory是用于容纳和管理整个软件供应链中使用的所有工件、二进制文件、包、文件、容器和组件的单一解决方案。
* [Sonatype Nexus](https://github.com/sonatype/nexus-public)：Sonatype Nexus是一个由Sonatype开发的仓库管理工具，用于管理和托管各种软件构件。
* [Bintray](https://github.com/bintray)：Bintray是一个软件包分发平台，它提供了存储、发布和分发软件包的功能。
* [Indy](https://github.com/Commonjava/indy)：Indy是一个简单的仓库管理器，适用于Maven和其他使用Maven仓库格式的构建工具。
* [Artipie](https://github.com/artipie/artipie)：Artipie是一个二进制工件管理工具，类似于Artifactory、Nexus、Archiva、ProGet等。
* [BK Repo](https://github.com/Tencentblueking/bk-repo)：BK Repo是一个基于微服务架构设计的制品管理平台，由腾讯开源。
* [Cloudsmith](https://cloudsmith.io/)：完全托管的包管理SaaS，支持Maven/Gradle/SBT，并提供免费套餐。
* [Apache Archiva](https://github.com/apache/archiva)：Archiva是一个功能丰富的仓库管理器。
* [Strongbox](https://github.com/strongbox/strongbox)：Strongbox是一个现代OSS工件仓库管理器。
* [Reposilite](https://github.com/dzikoysk/reposilite)：Reposilite是Maven工件的轻量级仓库管理器，这是一个替代Nexus、Archiva或Artifactory等管理器的简单解决方案。
* [CloudRepo](https://cloudrepo.io/)：CloudRepo是管理、共享和分发私有Maven和Python仓库的简单选项。
* [PackageCloud](https://packagecloud.io)：Packagecloud为你的软件工件提供了一个统一、开发人员友好的包管理平台。
* [Repsy](https://repsy.io/)：Repsy是下一代通用包仓库。
* [Gemfury](https://gemfury.com/)：Gemfury是一项用于安全存储和部署代码包的托管服务。
* [Hadess](https://github.com/tiklab-project/tiklab-hadess)：Hadess是一款开源、免费的制品管理工具，提供了高效、安全的制品存储和版本控制。

## Java环境管理

* [SDKMAN](https://github.com/sdkman/sdkman-cli)：SDKMAN是一个用于在任何基于Unix的系统上管理多个软件开发套件的并行版本的工具。
* [jEnv](https://github.com/jenv/jenv)：Java环境管理器。
* [jEnv](https://github.com/linux-china/jenv)：jEnv是一个用于在任何系统(例如Linux、Mac和Windows)上管理Java开发套件并行版本的工具。
* [JC jEnv](https://github.com/chroblert/JC-jEnv)：Java版本切换工具，可以很方便的在Java的多个版本之间切换。
* [JEnv Windows](https://github.com/FelixSelter/JEnv-for-Windows)：只需一行命令即可更改当前的Java版本。
* [Jabba](https://github.com/shyiko/jabba)：Java版本管理工具，由Go语言开发。
* [IDE](https://github.com/devonfw/ide)：该工具旨在帮助开发人员设置开发环境，并能够在整个团队中共享相同的项目设置。
* [JVMS](https://github.com/ystyle/jvms)：适用于Windows的JDK版本管理器。

## 代码属性图

* [Code Property Graph](https://github.com/ShiftLeftSecurity/codepropertygraph)：CPG是一种可扩展且与语言无关的程序代码表示形式，专为增量和分布式代码分析而设计。
* [Plume](https://github.com/plume-oss/plume)：Plume是一个代码表示基准测试库，可以选择从Java字节码中提取AST并将结果存储在各种图形数据库中，由Amazon开源。

## API变更管理

* [Revapi](https://github.com/revapi/revapi)：Revapi是一个用于API分析和变更跟踪的工具。
* [Japicmp](https://github.com/siom79/japicmp)：Japicmp是一个比较Jar存档的两个版本的工具。
* [Clirr](https://clirr.sourceforge.net/)：Clirr是一个检查Java库与旧版本的二进制和源代码兼容性的工具。
* [@API Guardian](https://github.com/apiguardian-team/apiguardian)：提供@API注解的库，用于标注框架或应用程序中的公共类型、方法、构造函数和字段，以便发布它们的状态和稳定性级别，并指示它们的使用者如何使用API。

## 源代码浏览器

* [Sourcetrail](https://github.com/CoatiSoftware/Sourcetrail)：Sourcetrail是一个免费的开源跨平台源代码浏览器，可帮助你高效地处理不熟悉的源代码。
* [OpenGrok](https://github.com/oracle/opengrok)：OpenGrok是一个快速且可用的源代码搜索和交叉引用引擎，可以帮助你搜索、交叉引用和导航源树，由Oracle开源。
* [PySonar2](https://github.com/yinwang0/pysonar2)：PySonar2是Python的语义索引器库，专为大型代码库的批处理而设计，生成的索引可用于构建代码浏览器和代码搜索引擎。

## 脚手架

* [Pig](https://gitee.com/log4j/pig)：Pig是基于Spring Boot 3.0、Spring Cloud 2022 & Alibaba、SAS OAuth2的微服务RBAC权限管理系统。
* [RuoYi](https://gitee.com/zhijiantianya/ruoyi-vue-pro)：RuoYi是基于Spring Boot、MyBatisPlus、Vue实现的后台管理系统、微信小程序。
* [RuoYi Cloud](https://gitee.com/zhijiantianya/yudao-cloud)：RuoYi Cloud是基于Spring Cloud Alibaba、Gateway、Nacos、RocketMQ、Vue实现的后台管理系统、用户小程序。
* [Zheng](https://gitee.com/shuzheng/zheng)：Zheng是基于Spring、Spring MVC、Mybatis分布式敏捷开发系统架构，提供整套公共微服务模块。
* [Jmix](https://github.com/jmix-framework/jmix)：Jmix是一组库和工具，用于加速Spring Boot以数据为中心的应用程序开发，由Haulmont开源。
* [Qingzhou](https://gitee.com/openeuler/qingzhou)：轻舟是一款开源的轻量级软件开发平台，其愿景是优化通用型Web管理软件的开发质量与效率，并实现不同类型软件的集中化统一管理，由华为开发。
* [MicroServices Platform](https://gitee.com/zlt2000/microservices-platform)：MicroServices Platform是基于Spring Boot 2.x、Spring Cloud和Spring Cloud Alibaba并采用前后端分离的企业级微服务多租户系统架构。
* [Cloud Platform](https://gitee.com/geek_qi/cloud-platform)：Cloud Platform是基于Spring Cloud微服务化RBAC的管理平台。
* [SpringBlade](https://gitee.com/smallc/SpringBlade)：SpringBlade是一个由商业级项目升级优化而来的微服务架构。
* [Open Capacity Platform](https://gitee.com/dromara/open-capacity-platform)：OCP是基于Spring Cloud的企业级微服务框架，其目标是帮助企业搭建一套类似百度能力开放平台的微服务框架，由dromara社区开源。
* [JeeSpringCloud](https://gitee.com/JeeHuangBingGui/jeeSpringCloud)：基于Spring Boot 2.0的后台权限管理系统界面简洁美观敏捷开发系统架构。
* [Hope Boot](https://github.com/java-aodeng/hope-boot)：Hope Boot是一款现代化的脚手架项目。
* [Spring Boot Plus](https://github.com/geekidea/spring-boot-plus)：Spring Boot Plus是一个简单易用、高速、高效、功能丰富的开源Spring Boot脚手架。
* [X-SpringBoot](https://github.com/yzcheng90/X-SpringBoot)：X-SpringBoot是一个轻量级的Java快速开发平台。
* [Lenosp](https://gitee.com/zzdevelop/lenosp)：Lenosp是基于Spring Boot的脚手架，提供完善社区文档教程。
* [BootDo](https://gitee.com/lcg0124/bootdo)：BootDo是高效率、低封装、面向学习型、微服务的开源Java EE开发框架。
* [Web Flash](https://github.com/enilu/web-flash)：Web Flash是一个基于Spring Boot和Vue.js的Web系统。
* [SpringCloud](https://github.com/zhoutaoo/SpringCloud)：基于Spring Cloud 2.1的微服务开发脚手架。
* [Liugh](https://github.com/qq53182347/liugh-parent)：实现RESTful快速开发的后端脚手架。
* [ES](https://github.com/zhangkaitao/es)：ES是一个Java EE企业级项目的快速开发的脚手架，提供了底层抽象和通用功能。
* [SOP](https://gitee.com/durcframework/SOP)：SOP是一个开放平台解决方案项目，基于Spring Cloud实现，目标让用户快速搭建自己的开放平台。
* [BallCat](https://github.com/ballcat-projects/ballcat)：BallCat是一个快速开发脚手架，快速搭建企业级后台管理系统，并提供多种便捷Starter进行功能扩展。
* [Mall Tiny](https://github.com/macrozheng/mall-tiny)：Mall Tiny是一款基于Spring Boot、MyBatisPlus的快速开发脚手架。
* [AgileBoot](https://github.com/valarchie/AgileBoot-Back-End)：AgileBoot是一套开源的全栈精简快速开发平台。
* [Spring Boot API Project Seed](https://github.com/lihengming/spring-boot-api-project-seed)：Spring Boot API Project Seed是一个基于Spring Boot、MyBatis的种子项目，用于快速构建中小型API、RESTful API项目。
* [Vole](https://github.com/gavenwangcn/vole)：Vole是一个基于最新的Spring Cloud 2.0的微服务商业开发脚手架。
* [Cola Cloud](https://gitee.com/leecho/cola-cloud)：Cola Cloud基于Spring Boot、Spring Cloud构建微服务架构企业级开发平台，集成OAuth2认证、集成短信验证码登录、FlyWay数据库版本管理、网关集成Swagger聚合所有服务API文档。
* [Spring Boot V2](https://gitee.com/bdj/SpringBoot_v2)：Spring Boot V2项目是努力打造Spring Boot框架的极致细腻的脚手架。
* [Slife](https://gitee.com/jamen/slife)：Slife是一个使用Spring Boot搭建的企业级快速开发脚手架。
* [Vhr](https://gitee.com/lenve/vhr)：Spring Boot + Vue前后端分离的人力资源管理项目，可做常规企业级应用脚手架。
* [Maozi](https://github.com/1095071913/maozi-cloud-parent)：Maozi基于Spring Cloud Alibaba、Dubbo二开封装。
* [JBone](https://github.com/417511458/jbone)：JBone基于Spring Cloud框架开发，旨在为中小企业提供稳定的微服务解决方案，为开发人员提供基础开发骨架。
* [HsWeb](https://github.com/hs-web/hsweb-framework)：HsWeb是一个基于Spring Boot开发，使用全响应式编程的企业级后台管理系统基础项目。
* [Source Vue](https://gitee.com/open-source-byte/source-vue)：Source Vue是基于Spring Boot + Vue前后端分离的Java快速开发框架。
* [Pangu](https://gitee.com/pulanos/pangu-framework)：盘古开发框架是一套轻量稳健的工业级前、中、后台三维多端行业数字化赋能开发基座。
* [SaPlus](https://gitee.com/click33/sa-plus)：SaPlus是一个基于Spring Boot的快速开发框架，内置代码生成器。
* [XXL-DEEP](https://github.com/xuxueli/xxl-deep)：XXL-DEEP是一个快速开发平台，核心目标是开发迅速、学习简单、能力丰富、开箱即用。
* [JavaFX Falsework](https://gitee.com/lwdillon/fx-falsework)：基于JavaFX、Spring Boot开发的客户端与服务端系统开发脚手架。
* [XBoot](https://github.com/Exrick/xboot)：XBoot是基于Spring Boot 2.x的一站式前后端分离快速开发平台。
* [RenRen Security](https://gitee.com/renrenio/renren-security)：RenRen Security是采用Spring Boot、MyBatisPlus、Shiro、Vue 3、ElementPlus等框架开发的一套权限系统。
* [Snowy](https://gitee.com/xiaonuobase/snowy)：Snowy是国内首个国密前后端分离快速开发平台，集成国密加解密插件，软件层面完全符合等保测评要求，同时实现国产化机型、中间件、数据库适配。
* [FCat](https://gitee.com/softnetcat/FCat)：FCAT是企业级基础功能框架，软件巢工作室出品。
* [AXBoot Framework](https://github.com/axboot/ax-boot-framework)：AXBoot是使用Java和HTML5的全栈Java Web应用程序框架。
* [Essencium Backend](https://github.com/Frachtwerk/essencium-backend)：Essencium是一个构建在Spring Boot之上的软件库，允许开发人员快速开始新的软件项目。
* [J2eeFAST](https://gitee.com/dromara/J2EEFAST)：J2eeFAST是一个Java EE企业级快速开发平台，致力于打造中小企业最好用的开源免费的后台框架平台，由dromara社区开源。
* [JVS](https://gitee.com/software-minister/jvs)：JVS是企业级应用构建的基础脚手架，提供开箱即用的基础功能集成，其中集成了账户管理、租户管理、用户权限体系、三方登录、环境配置、各种业务日志等功能，还提供了对接低代码、数据中台的能力。
* [MJGA](https://github.com/ccmjga/mjga-scaffold)：MJGA是一款现代化Java Web脚手架。
* [VBoot](https://gitee.com/zsvg/vboot-java)：VBoot是一个开箱即用的快速开发平台。
* [LikeAdmin](https://gitee.com/likeadmin/likeadmin_java)：LikeAdmin是一套快速开发管理后台，使用Spring Boot 2.5、MyBatis Plus、TypeScript、Vue 3、Vite 2、Element Plus 1.2。
* [Hawaii Framework](https://github.com/hawaiifw/hawaii-framework)：Hawaii Framework是一个用于开发基于Spring的应用程序的Java框架，由ilionx开源。
* [Pear Admin Boot](https://gitee.com/pear-admin/Pear-Admin-Boot)：Pear Admin Boot是基于Spring Boot生态、权限、工作流的开发平台。
* [金合技术中台](https://gitee.com/ikingtech/iking-platform)：现代化的下一代企业级技术中台，简洁、高效、稳定、开源。
* [MLDong](https://gitee.com/mldong/mldong)：MLDong是基于Spring Boot + Vue 3的快速开发平台、自研工作流引擎。
* [ContiNew Admin](https://github.com/continew-org/continew-admin)：ContiNew Admin是持续迭代优化的前后端分离中后台管理系统框架。
* [Cool Admin](https://github.com/cool-team-official/cool-admin-java)：Cool Admin后台权限管理系统，开源免费、AI编码、流程编排、模块化、插件化，用于快速构建后台应用程序。
* [TwelveT](https://github.com/twelvet-projects/twelvet)：基于Spring Boot 3.X的Spring Cloud Alibaba/Spring Cloud Tencent + React的微服务框架。

## 低代码

* [APITable](https://github.com/apitable/apitable)：APITable是一个面向API的低代码平台，用于构建协作应用程序。
* [JeeSite](https://gitee.com/thinkgem/jeesite4)：JeeSite快速开发平台，不仅仅是一个后台开发框架，它是一个企业级快速开发解决方案。
* [Guns](https://gitee.com/stylefeng/guns)：Guns是一个现代化的Java应用开发框架，基于主流技术Spring Boot 2 + Vue 3。
* [MakuBoot](https://gitee.com/makunet/maku-boot)：MakuBoot是采用Spring Boot 3.1、Spring Security 6.1、MybatisPlus等框架开发的一套Spring Boot低代码开发平台。
* [MateCloud](https://gitee.com/matevip/matecloud)：MateCloud是一款基于Spring Cloud Alibaba的微服务架构，支持多租户的低代码平台。
* [JeecgBoot](https://gitee.com/jeecg/jeecg-boot)：JeecgBoot是一款基于代码生成器的低代码开发平台，前后端分离架构Spring Boot 2.x、Spring Cloud、Ant Design & Vue、MybatisPlus、Shiro、JWT，支持微服务，由北京国炬公司开发。
* [DiBoot](https://gitee.com/dibo_software/diboot)：为开发人员打造的低代码开发平台。
* [Mendix](https://www.mendix.com/)：Mendix是一个低代码平台，由Siemens开发。
* [OPSLI](https://github.com/hiparker/opsli-boot)：OPSLI是一款快速的低代码平台，零代码开发，致力于做更简洁的后台管理系统。
* [Structr](https://github.com/structr/structr)：Structr是一个使用图数据库的集成低代码开发和运行时环境，使用Structr，可以比传统开发方法更快地构建企业Web应用程序。
* [DBAPI](https://gitee.com/freakchicken/db-api)：DBAPI是一个面向数仓开发人员的低代码工具，只需在页面上编写SQL，并配置好参数，就可以自动生成HTTP接口。
* [Citrus](https://github.com/Yiuman/citrus)：低代码快速开发脚手架，灵活、高效。
* [Convertigo](https://github.com/convertigo/convertigo)：Convertigo是一个开源低代码平台，包括用于全栈移动和Web应用程序开发的无代码应用程序构建器。
* [Orienteer](https://github.com/OrienteerBAP/Orienteer)：Orienteer是一个灵活的业务应用平台，它可以让你构建功能齐全的专用企业应用程序，例如CRM、ERP、供应链管理应用程序等。
* [Open Lowcode](https://github.com/openlowcode/Open-Lowcode)：特定企业软件快速开发解决方案。
* [LAMP Cloud](https://github.com/dromara/lamp-cloud)：LAMP Cloud基于JDK 11、Spring Cloud、Spring Boot开发的微服务中后台快速开发平台，专注于多租户(SaaS架构)解决方案，由dromara社区开源。
* [Portofino](https://github.com/ManyDesigns/Portofino)：Portofino是一个低代码工具，用于构建模型驱动的REST API和Web应用程序。
* [Skyeye](https://gitee.com/doc_wei01/skyeye)：智能制造一体化，采用Spring Boot + WinUI的低代码平台开发模式。
* [Erupt](https://gitee.com/erupt/erupt)：Erupt是一个低代码全栈类框架，使用Java注解动态生成页面以及增删改查、权限控制等后台功能。
* [UBML](https://gitee.com/ubml/ubml-sdk)：UBML是开放原子开源基金会下的项目，它是一种用于快速构建应用软件的低代码领域建模语言，来源于浪潮海岳iGIX团队。
* [ApiBoot](https://gitee.com/minbox-projects/api-boot)：ApiBoot是接口服务的落地解决方案，提供了一系列开箱即用的组件，通过封装来简化主流第三方框架的集成。
* [Appsmith](https://github.com/appsmithorg/appsmith)：用于构建管理面板、内部工具和仪表板的平台。
* [Openkoda](https://github.com/openkoda/openkoda)：即用型开发平台，可加速构建业务应用程序和内部工具的过程。
* [MetaLowCode](https://gitee.com/MetaLowCode/MetaLowCode)：美乐是一个低代码开发平台，包含一个前端项目Meta LowCode和一个后端项目Meta Server，由极昇数科开源。
* [VLife](https://gitee.com/wwwlike/vlife)：VLife是一套采用前后端分离(Java + React)架构的企业级低代码研发平台。
* [Skyve](https://github.com/skyvers/skyve)：Skyve是一个开源低代码平台，可让你访问构建复杂、强大且可扩展的云解决方案所需的所有关键功能。
* [WaveMaker](https://www.wavemaker.com/)：WaveMaker是一个基于Java的低代码开发平台。
* [SmartAdmin](https://gitee.com/lab1024/smart-admin)：SmartAdmin是1024创新实验室使用Spring Boot 2和Vue 3开发出的一套简洁、易用的低代码中后台解决方案。
* [EOVA](https://gitee.com/eova/eova)：国产开源的简单快速开发平台。
* [Dashjoin](https://github.com/dashjoin/platform)：Dashjoin是一个开源和云原生低代码开发和集成平台，可帮助团队更快地交付应用程序。
* [悟空无代码平台](https://github.com/WuKongOpenSource/Wukong_nocode)：通过悟空无代码平台开发工具，企业可自主地快速开发出适合企业需要的信息化系统，开发过程只需要业务人员参与，开发效率极高，维护性很强。
* [云程低代码平台](http://www.yunchengxc.com/)：云程是一款支撑企业级应用系统快速开发的低代码开发平台。
* [EasyTrans](https://gitee.com/dromara/easy_trans)：EasyTrans是一款用于做数据翻译的代码辅助插件，由dromara社区开源。
* [Nop-Entropy](https://gitee.com/canonical-entropy/nop-entropy)：Nop Platform 2.0是基于可逆计算理论实现的采用面向语言编程范式的新一代低代码开发平台。
* [Orange Admin](https://gitee.com/orangeform/orange-admin)：橙单中台化低代码生成器，可完整支持多应用、多租户、多渠道、工作流、在线表单、自定义数据同步、自定义Job、多表关联、跨服务多表关联、框架技术栈自由组合等。
* [Entfrm](https://gitee.com/entfrm/entfrm-boot)：Entfrm是一个以模块化为核心的无代码开发平台。
* [NBCIO](https://gitee.com/nbacheng/nbcio-boot)：亿事达企业管理平台后端代码。
* [Joget](https://github.com/jogetworkflow/jw-community)：Joget是下一代开源无代码/低代码应用程序平台，可实现更快、更简单的数字化转型(DX)。
* [VisionX](https://visionx.sibvisions.com/)：VisionX是一个开放的低代码平台，没有限制，由SIB Visions公司开发。
* [心通达OA](https://gitee.com/xtdoa/xtdoa)：心通达低代码开发平台，由北京高速波软件公司开源。
* [LsFusion](https://github.com/lsfusion/platform)：LsFusion是一个免费的开源平台，用于基于同名第五代编程语言的信息系统开发。
* [JECloud](https://gitee.com/ketr/jecloud)：JECloud平台后端采用微服务架构，前端采用微应用架构，可做到不同服务使用不同数据库独立运行，由北京凯特伟业公司开源。
* [Bsin-PaaS](https://gitee.com/s11e-DAO/bsin-paas-all-in-one)：Bsin-PaaS是一套企业级的低代码、零代码去中心化应用搭建平台，可帮助企业快速搭建基于云原生的有竞争力的业务中台、流程中台、业务前台。
* [PagePlug](https://github.com/cloudtogo/pageplug)：PagePlug是Appsmith的中国化项目，是一个开源、声明式、可视化的前端低代码框架，可以用来制作Web应用、微信小程序。
* [Crabc](https://gitee.com/linebyte/crabc)：Crabc是低代码接口开发平台，企业级API管理系统，深度整合Spring Boot和Mybatis实现动态数据源和动态SQL。
* [iPLAss](https://github.com/dentsusoken/iPLAss)：iPLAss是一个基于Java的低代码开发平台，主要目的是提高企业级系统开发的生产力，由电通综合研究所开源。
* [AsmoBoot](https://github.com/RotaNova/asmoboot)：AsmoBoot是基于Spring Boot开发的快速开发框架，由福建新航科技公司开源。
* [Dont-code](https://dont-code.net/)：Dont-code是一个低代码/无代码平台，允许用户基于IT开发的完整功能集创建自己的应用程序。
* [Nussknacker](https://github.com/TouK/nussknacker)：Nussknacker是一款低代码可视化工具，可供领域专家构建、运行和监控实时决策算法，而不是在代码中实现它们。
* [JeeLowCode](https://gitee.com/jeelowecode/JeeLowCode)：JeeLowCode是一款基于芋道源码精心二次封装的企业级低代码开发框架。
* [Gemini](https://github.com/gemini-projects/gemini)：云原生和低代码平台，可在几分钟内创建全栈Web管理应用程序。

## BaaS

* [Apache Usergrid](https://github.com/apache/usergrid)：Usergrid是一个基于RESTful API的用于Web和移动应用程序的多租户后端即服务堆栈，由Apigee开发。
* [Para](https://github.com/Erudika/para)：Para是一个可扩展的多租户后端服务器/框架，用于对象持久化和检索。
* [BaasBox](https://github.com/baasbox/baasbox)：BaasBox是一个开源项目，旨在为移动和Web应用程序提供后端。
* [Appwrite](https://github.com/appwrite/sdk-for-android)：Appwrite是一个用于开发Web、移动和Flutter应用程序的后端平台。

## Data API

* [DB2Rest](https://github.com/kdhrubo/db2rest)：DB2Rest是一个现代低代码REST Data API平台，可以轻松构建智能应用程序。
* [Spring Data REST](https://github.com/spring-projects/spring-data-rest)：该项目的目标是提供一种灵活且可配置的机制来编写可以通过HTTP公开的简单服务。

## POS

* [Salespoint Framework](https://github.com/st-tu-dresden/salespoint)：Salespoint是一个用于开发销售点应用程序的框架，由德累斯顿工业大学、慕尼黑联邦国防军大学共同开发。
* [MinPOS](https://sourceforge.net/projects/minpos/)：MinPOS是一款销售点免费软件，它基于Openbravo POS开发，但经过大量修改。
* [Floreant POS](https://floreant.org/)：Floreant POS是一个用Java编写的独立于平台的销售点应用程序。
* [Openbravo POS](https://github.com/iMartinezMateu/openbravo-pos)：Openbravo Java POS是一款专为触摸屏设计的销售点应用程序，支持票证打印机、客户显示器和条形码扫描仪。
* [POSper](https://sourceforge.net/projects/posper/)：POSper是专为小型企业设计的销售点系统。
* [SmartPOS](https://sourceforge.net/projects/smart-pos/)：SmartPOS是一个完整的ERP、POS，它具有ERP的所有功能，但创建了一个直观、敏捷且易于学习的销售点100% Web。
* [POSSUM](https://github.com/target/POSSUM)：POSSUM是一个Spring Boot框架Java项目，旨在将遵循JavaPOS规范的所有USB连接的POS外围设备(扫描仪、打印机、通道灯、线路显示器、现金抽屉、支票阅读器、秤等)的功能呈现为Web服务。
* [ChromisPOS](https://github.com/ChromisPos/ChromisPOS)：ChromisPOS是用于零售和餐饮的销售点系统。
* [POS System](https://github.com/hanlinag/point-of-sale-system)：用JavaFX编写的销售点系统。
* [NORD POS](https://github.com/nordpos/nordpos)：NORD POS是Openbravo POS应用的一个分支，专为触摸屏设计。

## 业务

* [CDM](https://github.com/microsoft/CDM)：CDM是一种声明性规范，是标准实体的定义，代表业务和生产力应用程序中常用的概念和活动，并且也正在扩展到观察和分析数据，由Microsoft开源。
* [EventHub](https://github.com/Codecademy/EventHub)：EventHub使公司能够进行跨设备事件跟踪。
* [Apache ODE](https://ode.apache.org/)：ODE是一个WS-BPEL兼容的Web服务编配引擎，它可以使开发人员根据以BPEL XML语法写成的过程描述来编配Web服务。
* [Spring Web Flow](https://github.com/spring-projects/spring-webflow)：Spring Web Flow有助于构建需要引导导航的Web应用程序-例如购物车、航班登记、贷款申请等等。
* [EZ-vCard](https://github.com/mangstadt/ez-vcard)：EZ-vCard是一个用于Java的vCard解析器库。
* [SCMR1](https://github.com/doublechaintech/scm-biz-suite)：供应链中台系统基础版，集成零售管理、电子商务、供应链管理、财务管理、订单管理等，由成都双链科技开源。
* [OpenXava](https://sourceforge.net/projects/openxava/)：用于快速将业务应用程序推向市场的Java平台。
* [eXo Platform](https://github.com/exoplatform/platform-public-distributions)：eXo Platform是面向成长型团队和企业的开源数字工作场所解决方案。
* [Lattice](https://github.com/hiforce/lattice)：Lattice是一个强大、轻量级的业务扩展调用框架。

## 电商

* [Broadleaf Commerce](https://github.com/BroadleafCommerce/BroadleafCommerce)：Broadleaf Commerce CE是一个完全用Java编写并利用Spring框架的电子商务框架。
* [SAP Commerce](https://www.sap.com/products/crm/commerce-cloud.html)：SAP Commerce是一个使用Java、基于Spring MVC框架的电子商务平台。
* [Lilishop](https://gitee.com/beijing_hongye_huicheng/lilishop)：Lilishop是基于Spring Boot的开源商城，由北京宏业汇成科技公司开发。
* [YouLai Mall](https://gitee.com/youlaitech/youlai-mall)：YouLai Mall是基于Spring Boot 3 、Spring Cloud & Alibaba 2022、Vue 3、Element Plus、UniApp等全栈主流技术栈构建的开源商城项目。
* [YShop](https://github.com/guchengwuyue/yshopmall)：YShop是基于当前流行技术组合的前后端分离商城系统，由郑州意象网络科技公司开发。
* [Q-Calculator](https://github.com/CyrilFeng/Q-calculator)：Stateless高性能优惠叠加计算框架。
* [Shopizer](https://github.com/shopizer-ecommerce/shopizer)：Java开源电子商务软件。
* [Axelor Open Platform](https://github.com/axelor/axelor-open-suite)：Axelor开放平台是一个开源Java框架，用于创建现代商业应用程序。
* [Scipio ERP](https://github.com/ilscipio/scipio-erp)：Scipio ERP是一个基于Java 11+和内置Tomcat应用服务器的开源业务应用程序工具包。
* [YesCart](https://github.com/inspire-software/yes-cart)：YesCart是一个使用Java技术构建的电子商务平台。
* [Qalingo](https://github.com/qalingo/qalingo-engine)：Qalingo是一个为B2C和B2B业务开发的开源Java电子商务平台。
* [Mayocat](https://github.com/jvelo/mayocat-shop)：Mayocat是一个Java开源市场和电子商务平台。
* [Kadro Merchant](https://kadro.com/ecommerce/)：Kadro Merchant是一个基于Java的全功能电子商务框架，专为电子商务业务生命周期的任何阶段而设计。
* [SoftSlate](https://www.softslate.com/)：SoftSlate是一个开源Java购物车，提供全套电子商务功能以及高性能。
* [Avetti Commerce](https://avetticommerce.com/)：Avetti Commerce是一个可靠的基于Java的电子商务解决方案，它具有针对B2C和B2B业务以及多商店电子商务的一系列深入功能。
* [Elastic Path](https://www.elasticpath.com/products/architecture/technologies/java-ecommerce)：基于Spring、OpenJPA、Eclipse、Solr、Velocity、Groovy、jQuery等开源技术的Java电子商务平台。
* [JadaSite](https://jadasite.com/)：JadaSite是一个基于Java、开源、易于使用且功能丰富的内容管理和电子商务系统。
* [KonaKart](https://www.konakart.com/)：KonaKart是一个针对大中型在线零售商的Java电子商务系统。
* [ShoppingCart](https://github.com/ikismail/ShoppingCart)：基于Spring MVC和多模块开发的一个端到端电子商务Web应用程序。
* [POP Commerce](https://github.com/moqui/PopCommerce)：POP Commerce是一个简单的电子商务应用程序和一个管理应用程序。

## 支付

* [WxJava](https://gitee.com/binary/weixin-java-tools)：微信开发Java SDK，支持微信支付、开放平台、公众号、企业号/企业微信、小程序等的后端开发。
* [Jeepay](https://gitee.com/jeequan/jeepay)：Jeepay是一套适合互联网企业使用的开源支付系统，支持多渠道服务商和普通商户模式。
* [IJPay](https://gitee.com/javen205/IJPay)：封装了微信支付、QQ支付、支付宝支付、京东支付、银联支付、PayPal支付等常用的支付方式以及各种常用的接口。
* [Roncoo-Pay](https://gitee.com/roncoocom/roncoo-pay)：龙果支付系统是国内首款开源的互联网支付系统，拥有独立的账户体系、用户体系、支付接入体系、支付交易体系、对账清结算体系。
* [Pay-Java](https://gitee.com/egzosn/pay-java-parent)：全能第三方支付对接Java开发工具包。
* [Pay-SDK](https://github.com/Pay-Group/best-pay-sdk)：支付宝、微信支付SDK。
* [PayPal](https://github.com/paypal/PayPal-Android-SDK)：接入PayPal支付的Android SDK。
* [KillBill](https://github.com/killbill/killbill)：KillBill在过去10年中一直是领先的开源订阅计费和支付平台，该平台的存在是为了帮助扩展计费和支付基础设施并发展业务。
* [微信支付Java SDK](https://github.com/YClimb/wxpay-sdk)：最新最全微信支付集成SDK，一行代码调用微信支付，包含基础支付功能。
* [微信支付API v3](https://github.com/wechatpay-apiv3/wechatpay-java)：微信支付API v3的官方Java SDK。
* [Paypal](https://github.com/paypal/Checkout-Java-SDK)：PayPal结账Java SDK。
* [Alipay](https://github.com/alipay/alipay-easysdk)：支付宝开放平台服务端SDK。
* [YunGouOS-PAY-SDK](https://gitee.com/YunGouOS/YunGouOS-PAY-SDK)：微信/支付宝官方服务商接口(支持个人、个体户、企业)签约开通。
* [Payment-Spring-Boot](https://gitee.com/dromara/payment-spring-boot)：Java微信支付V3支付Spring Boot Starter，支持微信优惠券，代金券、商家转账到零钱、公众号支付、微信小程序支付、电商收付通等全部微信支付功能API，由dromara社区开源。
* [EMV-NFC-Paycard-Enrollment](https://github.com/devnied/EMV-NFC-Paycard-Enrollment)：用于从NFC EMV信用卡读取和提取公共数据的Java库。
* [Alipay Java SDK](https://github.com/alipay/alipay-sdk-java-all)：支付宝开放平台Java SDK。
* [XPay](https://github.com/Exrick/xpay)：XPay个人收款支付系统。
* [J2PAY](https://github.com/tranxactive/J2PAY)：J2Pay是一个用于Java的开源多网关支付处理库。
* [Braintree Java](https://github.com/braintree/braintree_java)：Braintree Java库提供对Braintree网关的集成访问。
* [Adyen Java API](https://github.com/Adyen/adyen-java-api-library)：官方支持使用Adyen API的Java库。
* [WXPay-SDK](https://github.com/jkrains/wxpay-sdk-v3)：实现了微信支付V3版本的SDK，包括命令式和异步编程SDK。
* [Easy-Pay](https://github.com/easy-pay/easy-pay)：一行代码解决支付宝和微信的二维码生成，支付回调、退款、H5支付等功能。
* [Wallee Java SDK](https://github.com/wallee-payment/java-sdk)：Wallee Java库封装了wallee API，该库方便你与各种服务(例如交易、帐户和订阅)进行交互。
* [Razorpay Java SDK](https://github.com/razorpay/razorpay-java)：Razorpay API的官方Java绑定。
* [DaxPay](https://gitee.com/bootx/dax-pay)：DaxPay是一套基于Bootx-Platform脚手架构建的一套开源支付网关系统，已经对接支付宝、微信支付相关的接口，以及扩展了钱包支付、储值卡支付、现金支付等新的支付方式。
* [Ping++ Java](https://github.com/PingPlusPlus/pingpp-java)：Ping++是为移动端应用以及PC网页量身打造的下一代支付系统。

## 云服务SDK

* [Aliyun Java SDK](https://github.com/aliyun/aliyun-openapi-java-sdk)：阿里云Java SDK。
* [Aliyun Log](https://github.com/aliyun/aliyun-log-java-sdk)：阿里云日志服务Java SDK。
* [Azure Java SDK](https://github.com/Azure/azure-sdk-for-java)：Azure Java SDK。
* [Tencent SDK Java](https://github.com/TencentCloud/tencentcloud-sdk-java)：腾讯云API 3.0 Java SDK。
* [Aliyun ODPS Java SDK](https://github.com/aliyun/aliyun-odps-java-sdk)：面向Java开发者的ODPS SDK。
* [Volcengine Java SDK](https://github.com/volcengine/volcengine-java-sdk)：火山引擎Java SDK。
* [AWS Java SDK](https://github.com/aws/aws-sdk-java-v2)：AWS官方的Java SDK。
* [AWS IoT](https://github.com/aws/aws-iot-device-sdk-java)：用于从设备连接到AWS IoT的Java SDK。
* [AWS X-Ray](https://github.com/aws/aws-xray-sdk-java)：适用于Java的官方AWS X-Ray记录器SDK。
* [AWS C3R](https://github.com/aws/c3r)：C3R加密客户端和SDK。
* [Huawei SDK Java](https://github.com/huaweicloud/huaweicloud-sdk-java-v3)：华为云Java SDK。
* [Google App Engine](https://github.com/GoogleCloudPlatform/appengine-java-standard)：Google App Engine标准Java运行时：Prod运行时、本地devappserver、Cloud SDK Java组件、GAE API和GAE API模拟器。
* [Google Cloud Java](https://github.com/googleapis/google-cloud-java)：适用于Java的Google Cloud客户端库。
* [Google Cloud BigQuery Java](https://github.com/googleapis/java-bigquery)：Cloud BigQuery的Java客户端。
* [Google Map](https://github.com/googlemaps/android-maps-utils)：Android地图SDK工具类库。
* [CloudEvents Java](https://github.com/cloudevents/sdk-java)：CloudEvents是一种以通用格式描述事件数据的规范，以提供跨服务、平台和系统的互操作性。
* [Firebase Android](https://github.com/firebase/firebase-android-sdk)：Firebase安卓SDK。
* [OCI Java SDK](https://github.com/oracle/oci-java-sdk)：用于Java的Oracle云基础设施SDK。
* [SA Java SDK](https://github.com/sensorsdata/sa-sdk-java)：神策数据官方Java埋点SDK，是一款轻量级用于Java端的数据采集埋点SDK。
* [BCE SDK Java](https://github.com/baidubce/bce-sdk-java)：百度云Java语言版SDK，可基于该SDK使用Java语言接入百度云的各项产品。
* [Microsoft Graph Java SDK](https://github.com/microsoftgraph/msgraph-sdk-java)：适用于Java的Microsoft Graph SDK。
* [JD Cloud SDK](https://github.com/jdcloud-api/jdcloud-sdk-java)：京东云开发者Java工具套件。
* [讯飞开放平台AI能力Java SDK](https://github.com/iFLYTEK-OP/websdk-java)：提供各种讯飞开放平台能力的Java SDK。

## 微信开发

* [Weixin SDK](https://github.com/borball/weixin-sdk)：Weixin SDK是对微信公众平台(订阅号、服务号、企业号、小程序)、微信开放平台和微信支付的Java版封装。
* [WxJava](https://github.com/Wechat-Group/WxJava)：微信开发Java SDK，支持包括微信支付、开放平台、小程序、企业微信、公众号等的后端开发。
* [Gewechat](https://github.com/Devo919/Gewechat)：个人微信免费开源框架，支持二次开发、任意语言都可接入，REST API接入。
* [Weixin Java Tools](https://github.com/chanjarster/weixin-java-tools)：微信公众号、企业号Java SDK。
* [JFinal Weixin](https://gitee.com/jfinal/jfinal-weixin)：JFinal Weixin是基于JFinal的微信公众号极速开发SDK，只需浏览Demo代码即可进行极速开发。
* [Java Wechaty](https://github.com/wechaty/java-wechaty)：Java Wechaty是一个用Kotlin编写的聊天机器人开发者对话式SDK。
* [FastBootWeixin](https://gitee.com/kingshine/FastBootWeixin)：基于Spring Boot的注解驱动式公众号极速开发框架，用注解重新定义公众号开发。
* [Weixin Popular](https://github.com/liyiorg/weixin-popular)：微信Java SDK(公众平台、开放平台、商户平台、服务商平台)。
* [WeiXin4j](https://github.com/foxinmy/weixin4j)：WeiXin4j是一个用Java编写针对微信开发的工具包。
* [WeiXin4j](https://github.com/jeecgboot/weixin4j)：微信和钉钉开发Java SDK，主要提供微信公众号、企业微信、钉钉、微信小程序、支付的Java封装，降低集成难度，由北京国炬公司开发。
* [WeChat4j](https://github.com/sword-org/wechat4j)：WeChat4j是一个微信开发框架。
* [WX Dump4j](https://github.com/xuchengsheng/wx-dump-4j)：WX Dump4j是一款基于Java开发的微信数据分析工具。
* [JEEWX-API](https://github.com/jeecgboot/jeewx-api)：JEEWX-API是一款Java版的微信开发SDK，支持微信公众号、小程序、微信企业号、支付宝生活号SDK和微博SDK，由北京国炬公司开发。
* [QYWX](https://github.com/shuaidd/qywx)：企业微信API封装。
* [ItChat4j](https://github.com/yaphone/itchat4j)：ItChat4j提供了简单易用的API，可以很方便地对个人微信号进行扩展，实现自动回复，微信挂机机器人等。
* [WeChat-API](https://github.com/hellokaton/wechat-api)：WeChat-API是微信个人号的Java版本API，让个人号具备更多能力，提供方便的接口调用。
* [WeCOM SDK](https://gitee.com/felord/wecom-sdk)：WeCOM SDK是开源的企业微信开放API的Java实现。
* [FastWeixin](https://github.com/sd4324530/fastweixin)：FastWeixin可以简化微信公众平台服务端开发。

## 推送SDK

* [MPush](https://github.com/mpusher/mpush)：MPush是一款开源实时消息推送系统。
* [Austin](https://gitee.com/zhongfucheng/austin)：Austin是统一的接口发送各种类型消息，对消息生命周期全链路追踪。
* [Dinger](https://github.com/AnswerAIL/dingtalk-spring-boot-starter)：Dinger(叮鸽)，Spring Boot集成钉钉/企业微信/飞书群机器人实现消息通知中间件。
* [MixPush](https://github.com/taoweiji/MixPush)：Android混合推送SDK，快速集成6个厂商推送，共享系统推送通道。
* [JPush API Java](https://github.com/jpush/jpush-api-java-client)：这是JPush REST API的Java版本封装开发包，由极光推送官方提供。
* [个推PUSH Java SDK](https://github.com/GetuiLaboratory/getui-pushapi-java-client-v2)：个推官方提供的推送服务端SDK，基于全新的REST API V2接口。
* [WxPusher](https://github.com/wxpusher/wxpusher-sdk-java)：WxPusher是一个轻量级企业消息推送平台，旨在提供企业内部沟通和协作的便捷解决方案。
* [Deliver](https://gitee.com/OS-Zero/deliver)：Deliver是一个面向企业的全面消息推送平台，旨在提供企业内部沟通和协作的便捷解决方案。
* [Message Gateway](https://github.com/openMF/message-gateway)：Message Gateway是Fineract提供商的推送消息服务，可以轻松地通过短信和电子邮件推送通知。
* [MEIZUPUSH](https://github.com/MEIZUPUSH/JavaSdk)：魅族开放平台PUSH系统Java版本SDK。

## API&客户端

* [REST Countries](https://github.com/apilayer/restcountries)：REST Countries服务通过REST API提供有关国家/地区的通用信息。
* [JIRA Client](https://github.com/bobcarroll/jira-client)：JIRA Client是一个简单的Java JIRA REST客户端。
* [JiraRestClient](https://github.com/micromata/JiraRestClient)：一个用于访问Jira REST API的简单Java客户端。
* [Twitch4j](https://github.com/twitch4j/twitch4j)：模块化异步/同步/响应式Twitch API客户端/IRC客户端。
* [Riot API Java](https://github.com/taycaldwell/riot-api-java)：一个易于使用的Java Riot Games API包装器。
* [Google Maps Services Java](https://github.com/googlemaps/google-maps-services-java)：Google Maps API Web服务的Java客户端库。
* [Apple App Store Server Java Library](https://github.com/apple/app-store-server-library-java)：App Store Server API和App Store Server Notifications的Java服务器库。
* [Spring Social Google](https://github.com/spring-social/spring-social-google)：Spring Social扩展，具有连接支持和Google API绑定。
* [Java Youtube Downloader](https://github.com/sealedtx/java-youtube-downloader)：用于检索Youtube视频元数据的简单Java解析器。
* [Jenkins Java Client](https://github.com/jenkinsci/java-client-api)：适用于Java的Jenkins API客户端。
* [Cobalt](https://github.com/Auties00/Cobalt)：适用于Java和Kotlin的独立非官方全功能Whatsapp Web和移动API。
* [Artifactory Java Client](https://github.com/jfrog/artifactory-client-java)：Artifactory Java客户端在你的Java代码中提供简单而强大的Artifactory连接和管理。

#### Git

* [Github Java Client](https://github.com/spotify/github-java-client)：Spotify开源的Github API的Java客户端。
* [Jcabi Github](https://github.com/jcabi/jcabi-github)：GitHub API的Java面向对象包装器，带有整个GitHub API的假实现。
* [GitLab4J](https://github.com/gitlab4j/gitlab4j-api)：GitLab4J API提供了功能齐全且易于使用的Java库，用于通过GitLab REST API使用GitLab仓库。
* [Github Java API](https://github.com/hub4j/github-api)：该库定义了GitHub API的面向对象表示。
* [Java Gitlab API](https://github.com/timols/java-gitlab-api)：用Java编写的Gitlab API包装器。
* [Tea4j](https://codeberg.org/gitnex/tea4j-autodeploy)：适用于Gitea API的Java SDK。
* [Bitbucket REST](https://github.com/cdancy/bitbucket-rest)：使用Bitbucket REST API的客户端库。

#### Twitter

* [Twitter4J](https://github.com/Twitter4J/Twitter4J)：Twitter4J是Twitter API的100%纯Java库，没有外部依赖。
* [JTwitter](https://github.com/winterstein/JTwitter)：JTwitter是一个强大且易于使用的Twitter库。
* [Twittered](https://github.com/redouane59/twittered)：面向Java开发人员的Twitter API客户端。
* [Spring Social](https://github.com/spring-attic/spring-social)：Spring Social是Spring框架的扩展，可帮助你将应用程序与Facebook和Twitter等SaaS提供商连接起来。
* [Twitter API Java Client](https://github.com/xdevplatform/twitter-api-java-sdk)：Java版Twitter API客户端库。

#### Facebook

* [RestFB](https://github.com/restfb/restfb)：RestFB是一个纯Java Facebook Graph API客户端，没有外部依赖。
* [Facebook4J](https://github.com/roundrop/facebook4j)：Facebook4J是Java语言的Facebook Graph API绑定库。
* [Facebook Business SDK](https://github.com/facebook/facebook-java-business-sdk)：用于Meta营销API的Java SDK。

#### Instagram

* [Instagram4j](https://github.com/instagram4j/instagram4j)：使用OkHttpClient作为Instagram私有API的Java包装器。
* [JInstagram](https://github.com/sachin-handiekar/jInstagram)：Instagram API的Java库。
* [Instagram Java Scraper](https://github.com/postaddictme/instagram-java-scraper)：Instagram Java Scraper可以获取Instagram帐户信息、照片、视频和评论。

#### Slack

* [Slack Java SDK](https://github.com/slackapi/java-slack-sdk)：Slack Java SDK以Java惯用方式支持Slack平台。
* [Slack Client](https://github.com/HubSpot/slack-client)：Slack Web API的异步HTTP客户端，由HubSpot开源。
* [Simple Slack API](https://github.com/Itiviti/simple-slack-api)：该库允许应用程序连接到Slack以从任何通道接收和发送消息。

#### Reddit

* [JRAW](https://github.com/mattbdean/JRAW)：JRAW是JVM的Reddit API包装器。
* [Reddit4J](https://github.com/masecla22/Reddit4J)：Reddit4J是一个全面覆盖Reddit API的库。
* [JReddit](https://github.com/jReddit/jReddit)：JReddit是用Java编写的Reddit API的包装器。

#### TikTok

* [DyJava](https://gitee.com/sxwdmjy/dy-java)：DyJava是一款功能强大的抖音Java开发工具包，支持多种抖音开发功能模块的后端开发，包括但不限于移动/网站应用、开放平台、抖店和小程序等。
* [TikTok4j](https://github.com/cyrus07424/tiktok4j)：Java版非官方TikTok/DouYin API。
* [TikTok Live Java](https://github.com/jwdeveloper/TikTokLiveJava)：用于从TikTok LIVE实时接收直播事件(评论、礼物等)。

#### Spotify

* [Librespot Java](https://github.com/librespot-org/librespot-java)：最新的开源Spotify客户端。
* [Spotify Web API Java](https://github.com/spotify-web-api-java/spotify-web-api-java)：这是Spotify Web API的Java包装器/客户端。

## Docker客户端

* [Docker Java](https://github.com/docker-java/docker-java)：Java Docker官方客户端。
* [Spotify Docker Client](https://github.com/spotify/docker-client)：这是一个用Java编写的Docker客户端，之前被用于Spotify的许多关键生产系统。
* [Docker Java API](https://github.com/amihaiemil/docker-java-api)：另一个轻量级的Docker客户端库。
* [Docker Client](https://github.com/gesellix/docker-client)：用Groovy编写的JVM的Docker HTTP客户端。
* [Jocker](https://github.com/ndeloof/jocker)：Jocker是用于访问Docker API的Docker客户端库。

## Consul客户端

* [Consul](https://github.com/Ecwid/consul-api)：Consul的Java客户端。
* [Spring Cloud Consul](https://github.com/spring-cloud/spring-cloud-consul)：Spring Cloud Consul为Spring Boot应用程序提供Consul集成。
* [Consul Java](https://github.com/rickfast/consul-client)：Consul HTTP API的Java客户端。
* [Consultant](https://github.com/Magnetme/consultant)：Consultant是一个Java库，允许从Consul存储中检索其配置。

## Kubernetes客户端

* [Kubernetes Java](https://github.com/kubernetes-client/java)：Kubernetes官方Java客户端库。
* [Kubernetes & OpenShift Client](https://github.com/fabric8io/kubernetes-client)：该客户端通过流式的DSL提供对完整Kubernetes和OpenShift REST API的访问。
* [Spring Cloud Kubernetes](https://github.com/spring-cloud/spring-cloud-kubernetes)：Spring Cloud Kubernetes提供了消费Kubernetes原生服务的Spring Cloud通用接口实现。
* [YAKC](https://github.com/manusa/yakc)：YAKC是Kubernetes API的较低级别Java REST客户端。
* [Amdatu Kubernetes](https://bitbucket.org/amdatulabs/amdatu-kubernetes/src/master/)：Amdatu Kubernetes是Kubernetes的客户端库。
* [OpenShift Java](https://github.com/openshift/openshift-restclient-java)：基于Kubernetes的OpenShift版本3架构的Java REST客户端。

## 消息队列客户端

* [RabbitMQ Java](https://github.com/rabbitmq/rabbitmq-java-client)：RabbitMQ Java客户端库。
* [Lyra](https://github.com/jhalterman/lyra)：Lyra是一个拥抱故障的RabbitMQ客户端，可在发生意外故障时自动恢复AMQP资源，帮助你实现服务的高可用性。
* [Hop](https://github.com/rabbitmq/hop)：适用于Java、Groovy和其他JVM语言的RabbitMQ HTTP API客户端。
* [Spring AMQP](https://github.com/spring-projects/spring-amqp)：Spring AMQP项目将核心Spring概念应用于基于AMQP的消息传递解决方案的开发。
* [ReliableRMQ](https://github.com/levy-tech-spark/ReliableRMQ)：ReliableRMQ是一个Spring Boot框架，用于使用RabbitMQ的可靠消息传递实现分布式事务。
* [HiveMQ MQTT Client](https://github.com/hivemq/hivemq-mqtt-client)：MQTT 5.0和3.1.1兼容且功能丰富的高性能Java客户端库，具有不同的API风格和背压支持。
* [NSQ-J](https://github.com/sproutsocial/nsq-j)：NSQ实时分布式消息传递平台的Java客户端。
* [NATS Java](https://github.com/nats-io/nats.java)：NATS消息系统的Java客户端。

## Etcd客户端

* [JEtcd](https://github.com/etcd-io/jetcd)：JEtcd是etcd v3的官方Java客户端。
* [Boon Etcd](https://github.com/boonproject/boon/tree/master/etcd)：Boon etcd是etcd的Java客户端。
* [Etcd Java](https://github.com/IBM/etcd-java)：IBM开源的etcd v3 Java客户端库。
* [JEtcd](https://github.com/justinsb/jetcd)：一个简单的Java etcd客户端库。
* [Etcd4j](https://github.com/jurmous/etcd4j)：Etcd4j是etcd的客户端库。

## 即时通讯

* [Signal Server](https://github.com/signalapp/Signal-Server)：Signal是一款开源、端到端加密的通讯应用程序，可用于发送文本、语音、图片和视频等多种形式的消息，由Open Whisper Systems开发。
* [TIMSDK](https://github.com/TencentCloud/TIMSDK)：腾讯云聊天拥有全球接入、一对一聊天、群聊、消息推送、档案和关系链托管、账户认证等一整套解决方案。
* [CIM](https://github.com/crossoverJie/cim)：CIM是一款面向开发者的IM系统，同时提供了一些组件帮助开发者构建一款属于自己可水平扩展的IM。
* [野火IM](https://github.com/wildfirechat/im-server)：野火IM是专业级的即时通讯和实时音视频整体解决方案，由北京野火无限网络科技有限公司维护和支持。
* [FshareIM](https://github.com/fsharechat)：FshareIM是一个技术自主可控即时IM通讯系统，适于私有化部署。
* [Aurora IMUI](https://github.com/jpush/aurora-imui)：Aurora IMUI是一个通用的IM UI组件库，不依赖于任何特定的IM SDK，极光开源。
* [Actor Platform](https://github.com/actorapp/actor-platform)：Actor是一个即时通讯平台。
* [NettyChat](https://github.com/FreddyChen/NettyChat)：基于Netty、TCP、Protobuf实现的Android IM库。
* [Turms](https://github.com/turms-im/turms)：Turms是全球最先进的开源即时通讯引擎，支持100K~10M并发用户。
* [InChat](https://github.com/AwakenCN/InChat)：InChat是一个轻量级、高效、分布式的异步通信框架，支持聊天和物联网。
* [Smack](https://github.com/igniterealtime/Smack)：Smack是一个开源、高度模块化、易于使用的XMPP客户端库，用Java编写，适用于兼容Java SE的JVM和Android。
* [J-IM](https://gitee.com/xchao/j-im)：J-IM是用Java语言开发的轻量、高性能、单机支持几十万至百万在线用户IM。
* [Bytedesk](https://github.com/Bytedesk/bytedesk)：企业即时通讯解决方案，具有人工智能实时聊天、电子邮件支持、全渠道客户服务和团队即时通讯功能。
* [Openfire](https://github.com/igniterealtime/Openfire)：Openfire是一个即时通信和群聊服务器，它是使用Java编写的XMPP服务器。
* [CIM](https://gitee.com/farsunset/cim)：CIM是一套完善的消息推送框架，可应用于信令推送，即时聊天，移动设备指令推送等领域。
* [MobileIMSDK](https://github.com/JackJiang2011/MobileIMSDK)：MobileIMSDK是一个原创多端IM通信层框架，轻量级、高度提炼，支持UDP + TCP + WebSocket三种协议。
* [OIM](https://gitee.com/oimchat/oim-fx)：OIM是一套即时通讯的聊天系统，可以用于公司内网、外网通讯、客服系统等。
* [CometD](https://github.com/cometd/cometd)：用于网络消息传递的可扩展Comet(服务器推送)实现。
* [云信IM](https://github.com/netease-kit/nim-uikit-android)：云信IM UIKit基于网易云信IM SDK开发的一款即时通讯UI组件库，包括聊天、会话、圈组、搜索、群管理等组件。
* [QIQIIM](https://gitee.com/qiqiim/qiqiim-server)：QIQIIM提供简单快捷的IM方案，可用于公司内网、外网通讯，客服系统等。
* [Spark](https://github.com/igniterealtime/Spark)：Spark是一款针对企业和组织进行优化的开源跨平台IM客户端。
* [Gifsockets](https://github.com/videlalvaro/gifsockets)：使用Gif动画作为传输的实时通信库。
* [RongCloud Server SDK](https://github.com/rongcloud/server-sdk-java)：Java版融云即时通讯服务器SDK。
* [SONA](https://github.com/BixinTech/sona)：SONA是一个由比心语音技术团队开发，用于快速搭建语音房产品的全端解决方案，支撑了比心聊天室、直播、游戏房等业务。
* [Tencent Cloud IM Server SDK](https://github.com/doocs/qcloud-im-server-sdk-java)：腾讯云IM服务端SDK Java版。
* [Ant Media Server](https://github.com/ant-media/Ant-Media-Server)：Ant Media Server是一款直播流引擎软件，通过使用WebRTC技术提供自适应、超低延迟流媒体，延迟约为0.5秒。
* [Conversations](https://codeberg.org/iNPUTmice/Conversations)：Conversations是适用于Android的开源XMPP/Jabber客户端。
* [Xiaper](https://github.com/xiaper/xiaper)：Xiaper是一款开源企业IM解决方案。
* [盒子IM](https://gitee.com/bluexsx/box-im)：盒子IM是一个仿微信实现的网页版聊天软件，不依赖任何第三方收费组件。

## 视频会议

* [BigBlueButton](https://github.com/bigbluebutton/bigbluebutton)：BigBlueButton是一个开源虚拟教室，旨在帮助教师教学和学习者学习。
* [Apache OpenMeetings](https://github.com/apache/openmeetings)：Openmeetings提供视频会议、即时消息、白板、协作文档编辑和其他群件工具。
* [OpenVidu Call](https://github.com/OpenVidu/openvidu-call)：OpenVidu是一个方便在Web或移动应用程序中添加视频通话的平台。

## Web3

* [Web3j](https://github.com/web3j/web3j)：Web3j是一个轻量级、高度模块化、响应式、类型安全的Java和Android库，用于处理智能合约并与以太坊网络上的客户端集成。
* [Hyperledger Quilt](https://github.com/hyperledger-archives/quilt)：Quilt是Interledger协议的Java实现。
* [Web3signer](https://github.com/Consensys/web3signer)：Web3Signer是一种开源签名服务，能够使用存储在外部保管库中或加密在磁盘上的私钥在多个平台(Ethereum1和2、Filecoin)上进行签名。
* [Universal Resolver](https://github.com/decentralized-identity/universal-resolver)：通用解析器实现和驱动程序。
* [OmniJ](https://github.com/OmniLayer/OmniJ)：Omni Layer的Java/JVM实现，Omni Layer是一个基于比特币区块链构建的开源、完全去中心化的资产创建平台。
* [Convex](https://github.com/Convex-Dev/convex)：Convex是价值互联网的去中心化网络和执行引擎。
* [Waltid Identity](https://github.com/walt-id/waltid-identity)：多平台库、强大的API和易于使用的白标应用程序来构建身份和钱包解决方案。

#### 区块链

* [AntChain](https://github.com/AntChainOpenLabs)：蚂蚁链是蚂蚁集团自主研发的具备高性能、强隐私保护的区块链技术平台。
* [JD Chain](https://gitee.com/jdchain/jdchain)：京东区块链是一个企业级的区块链框架系统，具有简洁、易用、可扩展和高性能的特点。
* [IRI](https://github.com/iotaledger/iri)：IRI是一款开源Java软件，可在IOTA主网和Devnet上运行。
* [TrustSQL](https://cloud.tencent.com/document/product/663/38243)：腾讯推出的区块链平台，专注于金融领域的区块链应用，包括身份验证、资产管理等。
* [WeIdentity](https://github.com/WeBankBlockchain/WeIdentity)：WeIdentity是一套分布式多中心的技术解决方案，可承载实体对象(人或者物)的现实身份与链上身份的可信映射、以及实现实体对象之间安全的访问授权与数据交换，由微众银行开源。
* [TRON](https://github.com/tronprotocol/java-tron)：Tron白皮书的Java实现。
* [Corda](https://github.com/corda/corda)：Corda是一个开源区块链项目，由R3开发。
* [NEM](https://github.com/NemProject/nem)：NEM是一个去中心化的区块链平台。
* [MD BlockChain](https://gitee.com/tianyalei/md_blockchain)：MD开源Java区块链平台，可做联盟链、私链使用，不适用于公链。
* [Nxt](https://www.jelurida.com/nxt)：Nxt是一个开源区块链平台，也是第一个完全依赖权益证明共识协议的平台，由Jelurida开发。
* [Nuls](https://github.com/nuls-io/nuls-v1)：Nuls是一个全球区块链开源项目，是一个高度可定制的模块化区块链基础设施。
* [Manuscript](https://github.com/chainbase-labs/manuscript-core)：Manuscript不仅仅是一个语言规范，更是一个协议、框架和工具包，旨在简化和统一数据访问和处理方法。
* [Aion](https://github.com/aionnetwork/aion)：Aion是一个多层区块链网络。
* [Blockj](https://gitee.com/blackfox/blockj)：Blockj是Java实现的一个简易区块链(联盟链)项目，包括加密工具、钱包、P2P传输、区块同步、网络共识等基础实现。
* [J2Chain](https://gitee.com/ld/J2Chain)：J2Chain是Java开发区块链的开源项目。
* [SimBlock](https://github.com/dsg-titech/simblock)：SimBlock是一款开源区块链网络模拟器，由东京工业大学分布式系统组开发。
* [FingerNFT](https://github.com/fingerchar/fingernft)：FingerNFT是一款开源NFT市场，兼容Opensea、Rarible。
* [MultiChain](https://www.multichain.com/)：Multichain提供了一个用Java编写的区块链解决方案，主要用于构建私有链和联盟链。
* [Scorex](https://github.com/hyperledger-labs/Scorex)：Scorex 2是模块化区块链框架，采用Scala语言，允许对各种设计进行自由和无限制的实验。
* [Ultra](https://onultra.io/)：Ultra是一个区块链平台，其核心组件用Java编写，专注于去中心化应用的构建和部署。
* [Waves](https://github.com/wavesplatform/Waves)：Waves是一个基于社区的去中心化开源技术堆栈，用于构建可扩展、用户友好的应用程序。
* [Token Core](https://github.com/consenlabs/token-core-android)：TokenCore是一个区块链库，提供了相对一致的API，允许你同时管理钱包并在BTC、ETH和EOS链上签署交易。
* [Apache Tuweni](https://github.com/apache/incubator-tuweni)：Tuweni是一组库和其他工具，可帮助使用Java和其他JVM语言开发区块链和其他去中心化软件，由ConsenSys开源。
* [Emerald Dshackle](https://github.com/emeraldpay/dshackle)：Emerald Dshackle是区块链API的容错负载均衡器，由EmeraldPay开源。
* [WeBASE](https://github.com/WeBankBlockchain/WeBASE)：WeBASE是在区块链应用和FISCO BCOS节点之间搭建的一套通用组件，由微众银行开源。
* [WeCross](https://github.com/WeBankBlockchain/WeCross)：WeCross是由微众银行开源的区块链跨链协作平台，致力于促进跨行业、机构和地域的跨区块链信任传递和商业合作。
* [RepChain](https://gitee.com/BTAJL/repchain)：RepChain是第一款采用响应式编程实现的自主可控的区块链基础组件，由广州软件应用技术研究院、中国科学院软件所、贵阳信息技术研究院、中科智城信息科技有限公司、中科软科技股份有限公司和北京连琪科技有限公司共同研发。
* [Universa](https://github.com/UniversaBlockchain/universa)：Universa网络、节点、客户端和API。
* [Sun Network](https://github.com/tronprotocol/sun-network)：Sun Network是一个致力于构建TRON区块链可信去中心化侧链的项目。
* [COTI Node](https://github.com/coti-io/coti-node)：COTI是第一个基于DAG的链协议，针对企业和稳定币进行了优化。
* [Minima](https://github.com/minima-global/Minima)：Minima是一个新的区块链，强调每个用户都能够运行完整的节点。
* [Semux](https://github.com/semuxproject/semux-core)：Semux是一个实验性高性能区块链平台，为去中心化应用程序提供支持。
* [Apollo](https://github.com/ApolloFoundation/Apollo)：该仓库包含Apollo区块链平台的核心类和Apollo区块链组件的主要可执行文件。
* [TokenCore](https://github.com/GalaxySciTech/tokencore)：Tokencore是区块链钱包后端的核心组件，支持多种区块链地址生成和离线签名。
* [Prizm](https://github.com/prizmspace/PrizmCore)：Prizm是一个基于NXT项目构建的去中心化区块链项目，为去中心化金融平台提供工具。

#### 以太坊

* [Ethereumj](https://github.com/ethereum/ethereumj)：以太坊黄皮书的Java实现。
* [Besu](https://github.com/hyperledger/besu)：Besu是一个兼容MainNet的、用Java编写的以太坊客户端。
* [Teku](https://github.com/Consensys/teku)：以太坊2.0信标链的Java实现。
* [FundRequest](https://github.com/FundRequest/platform)：FundRequest是一个去中心化市场。
* [AlphaWallet](https://github.com/AlphaWallet/alpha-wallet-android)：AlphaWallet是一个开源可编程区块链应用程序平台。
* [Eventeum](https://github.com/eventeum/eventeum)：弹性以太坊事件监听器，可连接你的智能合约事件和后端微服务。
* [Presto Ethereum Connector](https://github.com/xiaoyao1991/presto-ethereum)：这是连接以太坊区块链数据的Presto连接器，有了这个连接器，就可以开始进行以太坊区块链分析工作，而无需知道如何使用Javascript API的细节。
* [Trust](https://github.com/trustwallet/trust-wallet-android-source)：Android版以太坊钱包。
* [ETHWallet](https://github.com/DwyaneQ/ETHWallet)：一款模仿imToken实现的ETH钱包。
* [Securify](https://github.com/eth-sri/securify)：以太坊智能合约安全扫描器。
* [BitcoinWallet](https://github.com/terryjiao/BitcoinWallet)：比特币和以太坊钱包。
* [EtherJar](https://github.com/emeraldpay/etherjar)：适用于以太坊区块链的框架无关的模块化Java 17+集成库。
* [Wuhan Chain](https://github.com/BSN-DDC/wuhanchain)：BSN官方DDC智能合约和SDK基于开放许可的区块链-武汉链(以太坊)。

#### 比特币

* [Bitcoinj](https://github.com/bitcoinj/bitcoinj)：Bitcoinj库是比特币协议的Java实现，它允许它维护钱包并发送/接收交易，而不需要Bitcoin Core的本地副本。
* [Bisq](https://github.com/bisq-network/bisq)：去中心化的比特币交易网络。
* [Eclair](https://github.com/ACINQ/eclair)：Eclair是闪电网络的Scala实现。
* [XChange](https://github.com/knowm/XChange)：一个Java库，提供简化的API，用于与60多个比特币和山寨币交易所进行交互，为交易和访问市场数据提供一致的接口。
* [Bitcoin Wallet](https://github.com/bitcoin-wallet/bitcoin-wallet)：适用于Android设备的比特币钱包应用程序。
* [Coin Trader](https://github.com/timolson/cointrader)：Coin Trader是一个基于Java的加密货币交易后端。
* [Exchange-core](https://github.com/exchange-core/exchange-core)：使用Java编写的超快速匹配引擎，基于LMAX Disruptor、Eclipse Collections、Agrona、OpenHFT、LZ4 Java和Adaptive Radix Trees。
* [Crypto-Exchange](https://github.com/jammy928/CoinExchange_CryptoExchange_Java)：基于Spring Cloud微服务开发，可用于数字货币交易所的搭建和二次开发。
* [OBAndroid](https://github.com/omnilaboratory/OBAndroid)：适用于Android设备的自我托管OmniBOLT闪电钱包。
* [Sparrow](https://github.com/sparrowwallet/sparrow)：Sparrow是一款现代桌面比特币钱包应用程序，支持大多数硬件钱包，并基于PSBT等通用标准构建，强调透明度和可用性。
* [Drongo](https://github.com/sparrowwallet/drongo)：一个Java比特币库。
* [BitHub](https://github.com/signalapp/BitHub)：BitHub是一项服务，它会为每次向GitHub仓库提交的内容自动支付一定比例的比特币资金，由Open Whisper Systems开发。
* [BX-bot](https://github.com/gazbert/bxbot)：用Java编写的简单比特币交易机器人。
* [Mycelium Bitcoin Wallet](https://github.com/mycelium-com/wallet-android)：Android版Mycelium比特币钱包。
* [DiabloMiner](https://github.com/Diablo-D3/DiabloMiner)：比特币OpenCL矿工。
* [Bither](https://github.com/bither/bither-android)：简单安全的比特币钱包。
* [Warp Exchange](https://github.com/michaelliao/warpexchange)：简单、超快的7 x 24交易。
* [Boilr](https://github.com/drpout/boilr)：比特币、加密货币、加密资产、期货和期权的价格警报。
* [CoinExchange](https://gitee.com/cexchange/CoinExchange)：开源数字货币合约交易所，基于Java开发的比特币交易所、BTC交易所、ETH交易所、数字货币交易所、交易平台、撮合交易引擎。
* [CoinGecko-Java](https://github.com/Philipinho/CoinGecko-Java)：CoinGecko API的Java包装器。
* [GitBitEX](https://github.com/gitbitex/gitbitex-new)：GitBitEX是一个开源的加密货币交易所。
* [Haveno](https://github.com/haveno-dex/haveno)：Haveno是一个开源平台，用于将Monero兑换为美元、欧元和英镑等法定货币或BTC、ETH和BCH等其他加密货币。
* [Orko](https://github.com/gruelbox/orko)：Orko是一款自托管Web应用程序，它提供统一的仪表板来控制众多加密货币交易所。
* [Thunder](https://github.com/blockchain/thunder)：lightning.network P2P协议的钱包/节点实现。
* [XDAGJ](https://github.com/XDagger/xdagj)：XDAGJ是XDAG在Java中的实现。
* [Snowblossom](https://github.com/snowblossomcoin/snowblossom)：Snowblossom是一种简单的加密货币。
* [Arbitrader](https://github.com/agonyforge/arbitrader)：Arbitrader是一个在两个不同的加密货币交易所之间寻找交易机会并执行自动低风险交易的程序。
* [BITISAN](https://github.com/bitisanop/CryptoExchange_TradingPlatform_CoinExchange)：BITISAN交易所支持多种数字资产的交易，涵盖加密货币、代币化资产以及其他数字化资产。

#### 区块链SDK

* [Fabric SDK Java](https://github.com/hyperledger/fabric-sdk-java)：该项目提供了一个用于与Hyperledger Fabric区块链网络交互的低级API。
* [EOSIO](https://github.com/EOSIO/eosio-java)：用于与基于EOSIO的区块链集成的API。
* [Cardano](https://github.com/bloxbean/cardano-client-lib)：Java中的Cardano客户端库。
* [Hashgraph Java SDK](https://github.com/hashgraph/hedera-sdk-java)：适用于Java的Hedera Hashgraph SDK。
* [Sol4k](https://github.com/sol4k/sol4k)：Sol4k是Solana的Kotlin客户端，可与Java或任何其他JVM语言以及Android一起使用。

#### 智能合约

* [RskJ](https://github.com/rsksmart/rskj)：RskJ是Rootstock节点的Java实现。
* [Neow3j](https://github.com/neow3j/neow3j)：Neow3j是一个开发工具包，提供简单可靠的工具来使用Java平台构建Neo dApp和智能合约。
* [Hedera Services](https://github.com/hashgraph/hedera-services)：Hedera公共账本的加密货币、代币、共识、文件和智能合约服务。
* [SmartJ](https://github.com/signum-network/signum-smartj)：Signum的Java智能合约。
* [Java4Ever](https://github.com/deplant/java4ever-framework)：Java4Ever是一个功能丰富的框架，用于智能合约开发、测试和访问TVM兼容的区块链，例如Everscale、Venom、GOSH等。

## 物联网

这里包含物联网领域相关软件，MQTT、Modbus等。

#### 物联网框架/工具

* [ThingsBoard](https://github.com/thingsboard/thingsboard)：ThingsBoard是一个开源物联网平台，用于数据收集、处理、可视化和设备管理。
* [JetLinks](https://gitee.com/jetlinks/jetlinks-community)：JetLinks是一个开箱即用，可二次开发的企业级物联网基础平台。
* [AWS IoT Greengrass](https://aws.amazon.com/greengrass/)：AWS IoT Greengrass是一种开源边缘运行时系统和云服务，用于构建、部署和管理设备软件。
* [Physical Web](https://github.com/google/physical-web)：Physical Web旨在将Web的超能力(URL)扩展到日常物理对象，由Google开源。
* [Eclipse Milo](https://github.com/eclipse/milo)：Milo是OPC UA的开源实现，它包括高性能堆栈(通道、序列化、数据结构、安全性)以及构建在堆栈顶部的客户端和服务器SDK。
* [Apache PLC4X](https://github.com/apache/plc4x)：PLC4X致力于创建一组库，用于以统一的方式与工业级可编程逻辑控制器(PLC)进行通信。
* [Eclipse SmartHome](https://github.com/eclipse-archived/smarthome)：旨在创建一个构建智能家居解决方案的框架，其重点是异构环境，即各种协议和标准集成。
* [OpenRemote](https://github.com/openremote/openremote)：OpenRemote是一个直观、用户友好的100%开源物联网平台。
* [OpenHAB](https://github.com/openhab/openhab-core)：OpenHAB是一个开源、与技术无关的家庭自动化平台，作为智能家居的中心运行。
* [SmartThings](https://www.samsung.com/us/smartthings/)：SmartThings是一款免费应用程序，它使用Wi-Fi连接基于Matter协议的智能设备，无论其制造商是哪家公司，这是三星的产品。
* [FastBee](https://gitee.com/kerwincui/wumei-smart)：FastBee开源物联网平台，简单易用，更适合中小企业和个人学习使用。适用于智能家居、智慧办公、智慧社区、农业监测、水利监测、工业控制等。
* [Enjoy IoT](https://gitee.com/open-enjoy/enjoy-iot)：Enjoy IoT是一个开源物联网平台。
* [Eclipse Californium](https://github.com/eclipse-californium/californium)：Californium是RFC7252(物联网云服务的约束应用协议)的Java实现。
* [Zeus IoT](https://github.com/zmops/zeus-iot)：Zeus IoT是一个分布式物联网采集、分析、存储平台，是全球第一个基于zabbix二次开发的物联网开源平台。
* [Eclipse Leshan](https://github.com/eclipse-leshan/leshan)：Leshan是OMA轻量级M2M服务器和客户端Java实现。
* [Groza](https://github.com/IoT-Technology/Groza)：开源物联网平台-物联网解决方案的设备管理、数据收集、处理。
* [TcMenu](https://github.com/TcMenu/tcMenu)：TcMenu是一个模块化、物联网就绪的多级菜单库，适用于Arduino、mbed、Pico-SDK和许多其他平台，支持多种输入、显示和物联网/远程接口。
* [SiteWhere](https://github.com/sitewhere/sitewhere)：SiteWhere是一个具有工业实力的开源物联网应用支持平台，可促进大规模物联网设备数据的摄取、存储、处理和集成。
* [SolarNode](https://github.com/SolarNetwork/solarnetwork-node)：SolarNode是用于收集数据和控制设备的分布式SolarNetwork组件。
* [ThingLinks](https://gitee.com/mqttsnet/thinglinks)：采用Spring Cloud微服务架构，一款高性能、高吞吐量、高扩展性的物联网平台。
* [Eclipse Ditto](https://github.com/eclipse-ditto/ditto)：Ditto是物联网中的一项技术，实现了一种称为“数字孪生”的软件模式。
* [Eclipse Kura](https://github.com/eclipse-kura/kura)：Kura是一个多功能软件框架，旨在增强你的边缘设备的性能。
* [IoTLink](https://gitee.com/sdyunze/iotlink)：IoTLink是一个基于Spring Boot、Vue、Mybatis、RabbitMQ、MySQK、Redis等开发的物联网平台，支持对物联网卡、物联网模组以及卡+模组的融合管理。
* [Apache StreamPipes](https://github.com/apache/streampipes)：StreamPipes是一个自助物联网工具箱，使非技术用户能够连接、分析和探索物联网数据流。
* [Eclipse HawkBit](https://github.com/eclipse/hawkbit)：HawkBit是一个独立于域的后端解决方案，用于向受限边缘设备以及连接到基于IP的网络基础设施的更强大的控制器和网关推出软件更新。
* [OpenMUC](https://www.openmuc.org/openmuc/)：OpenMUC是一个基于Java和OSGi的软件框架，可简化定制监控、日志记录和控制系统的开发，由弗劳恩霍夫太阳能系统研究所开发。
* [DeviceHive](https://github.com/devicehive/devicehive-java-server)：DeviceHive将任何连接的设备变成物联网的一部分。它提供通信层、控制软件和多平台库，以引导智能能源、家庭自动化、遥感、遥测、远程控制和监控软件等的开发。
* [Freedomotic](https://github.com/freedomotic/freedomotic)：Freedomotic是一个开源、灵活、安全的IoT应用程序框架，可用于构建和管理现代智能空间。
* [Tigase Server](https://github.com/tigase/tigase-server)：Tigase XMPP Server是用Java编写的高度优化、高度模块化且非常灵活的XMPP/Jabber服务器。
* [Eclipse Vorto](https://github.com/eclipse/vorto)：Vorto提供了一种用于描述IoT数字孪生模型和接口的语言。
* [IoT DC3](https://gitee.com/pnoker/iot-dc3)：基于Spring Cloud的开源、分布式的IoT平台，用于快速开发物联网项目和管理物联设备，是一整套物联系统解决方案。
* [S7Connector](https://github.com/s7connector/s7connector)：用于Java的S7 PLC连接器。
* [Eclipse Tahu](https://github.com/eclipse/tahu)：Tahu提供各种语言和各种设备的客户端库和参考实现，以显示设备/远程应用程序必须如何使用下面解释的Sparkplug规范连接和断开与MQTT服务器的连接。
* [NetXMS](https://github.com/netxms/netxms)：NetXMS是一款开源网络和基础设施监控和管理解决方案，为IT基础设施的所有层提供性能和可用性监控以及灵活的事件处理、警报、报告和图表。
* [World Avatar](https://github.com/cambridge-cares/TheWorldAvatar)：基于知识图谱的世界数字孪生，由新加坡剑桥高级研究与教育中心开源。
* [OpenIita](https://gitee.com/open-iita/iotkit-parent)：铱塔智联开源平台是一个开源的物联网基础开发平台，提供了物联网及相关业务开发的常见基础功能，能帮助你快速搭建自己的物联网相关业务平台。
* [HA-Bridge](https://github.com/bwssytems/ha-bridge)：将Philips Hue API模拟到其他家庭自动化网关，例如Amazon Echo/Dot或支持Philips Hue本地网络发现的其他系统。
* [OpenHAB Add-ons](https://github.com/openhab/openhab-addons)：该库包含在OpenHAB核心API之上实现的官方附加组件集。
* [Amazon Echo Bridge](https://github.com/armzilla/amazon-echo-ha-bridge)：Amazon Echo Bridge允许你快速模拟Phillips Hue桥，从而能够将Amazon Echo无缝集成到各种家庭自动化系统中。
* [Eclipse Kapua](https://github.com/eclipse/kapua)：Kapua是一个模块化平台，提供管理物联网网关和智能边缘设备所需的服务。
* [Eclipse Hono](https://github.com/eclipse-hono/hono)：Hono提供统一(远程)服务接口，用于将大量IoT设备连接到(云)后端。
* [Azure IoT SDK](https://github.com/Azure/azure-iot-sdk-java)：用于将设备连接到Microsoft Azure IoT服务的Java SDK。
* [Eclipse Sparkplug](https://github.com/eclipse-sparkplug/sparkplug)：Sparkplug为网络边缘网关(Sparkplug边缘节点)或支持本机MQTT的终端设备与Sparkplug主机应用程序如何在MQTT基础设施内进行双向通信提供了开放且免费的规范。
* [Eclipse Arrowhead](https://github.com/eclipse-arrowhead/core-java-spring)：Arrowhead是一个用于构建自动化和数字化解决方案的框架和实施平台。
* [IOTGate](https://gitee.com/willbeahero/IOTGate)：Java版基于Netty的物联网高并发智能网关。
* [Indriya](https://github.com/unitsofmeasurement/indriya)：JSR 385参考实现。
* [Sentilo](https://github.com/sentilo/sentilo)：Sentilo是一个架构，它隔离了为利用“城市生成”的信息而开发的应用程序和部署在城市各处以收集和广播该信息的传感器层。
* [WSO2 IoT Server](https://github.com/wso2/product-iots)：WSO2 IoT Server是一个完整的解决方案，使设备制造商和企业能够连接和管理其设备、构建应用程序、管理事件、保护设备和数据以及以可扩展的方式可视化传感器数据。
* [MyController](https://github.com/mycontroller-org/mycontroller-v1-legacy)：MyController是一个适用于家庭、办公室或任何地方的物联网自动化控制器。
* [IoT-Ucy](https://gitee.com/iteaj/iot)：IoT-Ucy是使用Java开发的物联网网络中间件，支持udp、tcp、串口通讯等底层协议和http、mqtt、websocket、modbus(tcp,rtu)、plc、dtu等上层协议。
* [ESPlorer](https://github.com/4refr0nt/ESPlorer)：面向ESP8266开发人员的集成开发环境。
* [Eclipse BaSyx](https://github.com/eclipse-basyx)：BaSyx是下一代自动化的开源平台，它实现了工业4.0平台定义的关键概念，例如作为标准化数字孪生的资产管理shell。
* [Eclipse AAS4J](https://github.com/eclipse-aas4j/aas4j)：AAS4J实现了Asset Administration Shell(AAS)的规范，例如基于AAS规范的元模型、子模型、序列化和反序列化模块、验证器和转换库。
* [GRASSMARLIN](https://github.com/nsacyber/GRASSMARLIN)：GRASSMARLIN提供工业控制系统以及监控和数据采集(SCADA)网络的IP网络态势感知，以支持网络安全，由美国国家安全局网络安全局开源。
* [Scada-LTS](https://github.com/SCADA-LTS/Scada-LTS)：Scada-LTS是一个基于Web的开源多平台解决方案，用于构建你自己的SCADA(监控和数据采集)系统。
* [Apache Edgent](https://github.com/apache/incubator-retired-edgent)：Edgent是一种适用于边缘设备的开源编程模型和运行时，使你能够分析设备上的数据和事件。
* [Grid eXchange Fabric](https://github.com/OSGP/open-smart-grid-platform)：GXF是一个软件平台，可在公共空间中实现硬件监控和控制。
* [Aura Tower](https://github.com/blumek/aura-tower)：Aura Tower是一个开源项目，旨在通过直观的仪表板控制和监控物联网设备。

#### 车联网

* [JT808 Server](https://gitee.com/yezhihao/jt808-server)：JT808、JT808协议解析；支持TCP、UDP，实时兼容2011、2013、2019版本协议，支持分包。
* [JT Framework](https://github.com/hylexus/jt-framework)：基于Spring Boot的JT-808协议服务端。
* [Eclipse MOSAIC](https://github.com/eclipse/mosaic)：MOSAIC是智能互联移动领域的多尺度仿真框架，它允许将来自不同领域的模拟器耦合到综合模拟工具。
* [JTT1078 Video Server](https://gitee.com/matrixy/jtt1078-video-server)：基于JT/T 1078协议实现的视频转播服务器。

#### 嵌入式

* [MICROEJ](https://developer.microej.com/)：MicroEJ的使命是将虚拟化和OOP民主化到嵌入式世界。
* [Arduino](https://github.com/arduino/Arduino)：Arduino是一个开源嵌入式硬件平台，用来供用户制作可交互式的嵌入式项目。
* [Eclipse Mita](https://github.com/eclipse/mita)：Mita是一种用于嵌入式物联网的新型编程语言。
* [CocktailPi](https://github.com/alex9849/CocktailPi)：基于树莓派的DIY鸡尾酒制作机的Web界面和控制软件。
* [Pi4J](https://github.com/Pi4J/pi4j-v1)：Pi4J旨在为Java程序员提供一个友好的面向对象的I/O API和实现库，以访问Raspberry Pi平台的完整I/O功能，起源于瑞士西北应用科学与艺术大学。
* [Ardulink 2](https://github.com/Ardulink/Ardulink-2)：Ardulink 2是一个完整的开源Java解决方案，用于控制和协调Arduino板。
* [Diozero](https://github.com/mattjlewis/diozero)：Diozero是用Java编写的设备I/O库，为连接到单板计算机的一系列GPIO/I2C/SPI设备提供面向对象的接口。
* [R2Cloud](https://github.com/dernasherbrezon/r2cloud)：R2Cloud可以跟踪和解码来自卫星的各种无线电信号。
* [RedBear Duo](https://github.com/redbear/Duo)：RedBear Duo是一款拇指大小的开发板，旨在简化构建物联网产品的过程。

#### MQTT

* [Eclipse Paho](https://github.com/eclipse/paho.mqtt.java)：Paho Java是一个用Java编写的MQTT客户端库，由Eclipse IoT组织开发。
* [HiveMQ](https://github.com/hivemq/hivemq-community-edition)：HiveMQ是一个基于Java的开源MQTT代理，完全支持MQTT 3.x和MQTT 5。
* [Moquette](https://github.com/moquette-io/moquette)：Moquette的目标是成为符合MQTT标准的Broker，代理支持QoS 0、QoS 1和QoS 2。
* [BifroMQ](https://github.com/baidu/bifromq)：BifroMQ是一种高性能、分布式MQTT代理实现，可无缝集成原生多租户支持，由百度开源。
* [MQTT-Client](https://github.com/fusesource/mqtt-client)：MQTT-Client为MQTT提供API，如果发生任何网络故障，它会自动重新连接到MQTT服务器并恢复客户端会话。
* [AndrOBD](https://github.com/fr3ts0n/AndrOBD)：AndrOBD允许你的Android设备通过任何ELM327兼容的OBD适配器连接到汽车的车载诊断系统，显示各种信息并执行操作。
* [ActiveMQ Artemis](https://github.com/apache/activemq-artemis)：ActiveMQ Artemis是ActiveMQ的下一代消息代理。
* [Mica MQTT](https://gitee.com/dromara/mica-mqtt)：Mica-MQTT是低延迟、高性能的MQTT物联网组件。
* [SMQTT](https://gitee.com/quickmsg/mqtt-cluster)：SMQTT是一款高性能、高吞吐量、高扩展性的物联网MQTT集群Broker。
* [MqttWk](https://github.com/Wizzercn/MqttWk)：MqttWk是由Netty实现的高并发高可用MQTT服务Broker。
* [JMQTT](https://github.com/Cicizz/jmqtt)：JMQTT是一个MQTT Broker，由Java和Netty实现，支持持久化和集群。
* [TBMQ](https://github.com/thingsboard/tbmq)：TBMQ是一个开源MQTT消息代理，能够处理4M+并发客户端连接，支持每个集群节点每秒至少3M消息吞吐量，并具有低延迟交付。
* [MoP](https://github.com/streamnative/mop)：MoP是为了在Pulsar上原生支持MQTT协议而开发的。
* [EnMasse](https://github.com/EnMasseProject/enmasse)：EnMasse在Kubernetes和OpenShift上提供了一个自助消息传递平台，具有统一的界面来管理不同的消息传递基础设施。
* [Smart MQTT](https://gitee.com/smartboot/smart-mqtt)：Smart MQTT是一款开源的云原生分布式MQTT Broker服务器，支持海量物联网设备互联互通。
* [RocketMQ MQTT](https://github.com/apache/rocketmq-mqtt)：全新的MQTT协议架构模型，基于该模型RocketMQ可以更好地支持来自物联网设备、手机APP等终端的消息。
* [HelloIoT](https://github.com/adrianromero/helloiot)：HelloIoT是一个MQTT仪表板应用程序，你可以使用HelloIoT作为MQTT客户端应用程序来发布和订阅主题，也可以使用HelloIoT作为客户端平台来创建自己的仪表板。
* [SMQTTX](https://gitee.com/quickmsg/smqttx)：基于Java实现的物联网分布式MQTT消息代理服务器。
* [WeMQ](https://gitee.com/dromara/WeMQ)：WeMQ是一款面向物联网设备运营商的开源物联网设备调试系统，提供完整的物联网设备调试方案，集成设备管理、MQTT服务器管理、客户管理等功能，由dormara社区开源。

#### 串口

* [JSerialComm](https://github.com/Fazecast/jSerialComm)：JSerialComm是一个独立于平台的Java串行端口访问库。
* [RXTX](https://github.com/rxtx/rxtx)：RXTX是Java中串口的本机接口。
* [JSSC](https://github.com/scream3r/java-simple-serial-connector)：Java中用于使用串行端口的库。
* [SerialPundit](https://github.com/RishiGupta12/SerialPundit)：SerialPundit是一个用于串行端口和HID通信的SDK。
* [PureJavaComm](https://github.com/nyholku/purejavacomm)：PureJavaComm是一个用于从Java访问串行端口的API。
* [NRJavaSerial](https://github.com/NeuronRobotics/nrjavaserial)：Java串行端口系统，这是RXTX项目的一个分支，用于本地代码的jar加载。
* [jRxTx](https://github.com/openmuc/jrxtx)：jRxTx是一个Java串行通信库，它可用于使用众所周知的基于UART的串行协议进行通信。
* [JavaCAN](https://github.com/pschichtel/JavaCAN)：Linux内核提供的socketcan API的简单JNI包装器。

#### Modbus

* [Modbus4j](https://github.com/MangoAutomation/modbus4j)：Modbus4j是由Infinite Automation和Serotonin用Java编写的Modbus协议的高性能且易于使用的实现。
* [JLibModbus](https://github.com/kochedykov/jlibmodbus)：JLibModbus是Modbus协议的Java语言实现。
* [J2mod](https://github.com/steveohara/j2mod)：J2mod是Jamod的一个分支，进行了大量的重构和代码修复。
* [Jamod](https://jamod.sourceforge.net/)：Jamod是100% Java的Modbus实现。
* [Modbus](https://github.com/digitalpetri/modbus)：适用于Java 17+的Modbus TCP、Modbus RTU/TCP和Modbus RTU/串行的现代、高性能、易于使用的客户端和服务器实现。
* [EasyModbus4j](https://github.com/zengfr/easymodbus4j)：EasyModbus4j是一个高性能和易用的Modbus协议的Java实现，基于Netty开发。
* [Iot Modbus](https://gitee.com/takeoff/iot-modbus)：物联网通讯协议，基于Netty框架，支持COM(串口)和TCP协议，支持服务端和客户端两种模式。
* [Modbus4Android](https://github.com/zgkxzx/Modbus4Android)：这是适用于Android的Modbus库。
* [Modbus4Android](https://github.com/licheedev/Modbus4Android)：Modbus的Android实现，添加对Android串口(RTU)的支持。

#### USB库

* [USB4Java](https://github.com/usb4java/usb4java)：该库可用于在Java中访问USB设备。
* [Javax USB](https://github.com/KeyBridge/lib-javax-usb3)：用于访问USB设备的Java库。
* [USB Drive Detector](https://github.com/samuelcampos/usbdrivedetector)：一个Java库，用于获取连接到计算机的所有USB存储设备的列表。
* [Java HID-API](https://github.com/nyholku/purejavahidapi)：HID-API是一个跨平台API，用于从Java访问USB HID设备。
* [JavaDoesUSB](https://github.com/manuelbl/JavaDoesUSB)：Java Does USB是一个用于处理USB设备的Java库，它允许查询有关所有连接的USB设备的信息，并使用自定义/供应商特定协议与USB设备进行通信。
* [Hid4Java](https://github.com/gary-rowe/hid4java)：libusb/hidapi库的跨平台JNA包装器，在Windows/Mac/Linux上开箱即用。

## 金融

* [Apache Fineract](https://github.com/apache/fineract)：Fineract是一个具有开放API的成熟平台，可为金融机构提供可靠、强大且价格实惠的核心银行解决方案，为全球30亿银行服务不足和无银行账户的人口提供服务。
* [Portfolio](https://github.com/portfolio-performance/portfolio)：Portfolio是一个开源程序，用于根据实时加权回报率和内部回报率计算整个投资组合(跨不同投资组合和账户)的绩效。
* [Northstar](https://gitee.com/dromara/northstar)：这是一个面向程序员的专业级量化交易软件，用于期货、股票、外汇、炒币等多种交易场景，实现自动交易，由dromara社区开源。
* [OBP-API](https://github.com/OpenBankProject/OBP-API)：OBP是一个面向银行的开源API，使账户持有人能够使用更广泛的应用程序和服务与银行进行交互。
* [Strata](https://github.com/OpenGamma/Strata)：Strata是OpenGamma的开源分析和市场风险库。
* [JavaMoney](https://github.com/JavaMoney/javamoney-lib)：JavaMoney提供基于JSR 354(兼容实现)构建的扩展和库。
* [QuickFIX/J](https://github.com/quickfix-j/quickfixj)：QuickFIX/J是适用于FIX协议的全功能消息传递引擎。
* [CoralFIX](https://www.coralblocks.com/index.php/category/coralfix/)：CoralFIX是一款功能齐全、超低延迟、无垃圾的FIX引擎，具有非常直观的API。
* [CDM](https://github.com/finos/common-domain-model)：CDM是金融产品、这些产品的交易以及这些交易的生命周期事件的模型，由金融科技开源基金会FINOS托管。
* [Moneta](https://github.com/JavaMoney/jsr354-ri)：Moneta是JSR 354货币API的参考实现。
* [Ta4j](https://github.com/ta4j/ta4j)：Ta4j是一个用于技术分析的开源Java库，它提供了创建、评估和执行交易策略的基本组件。
* [DROP](https://github.com/lakshmiDRIP/DROP)：DROP实现的库针对固定收益、信贷、商品、股票、外汇和结构性产品内部和之间的分析/风险、交易成本分析、资产负债分析、资本、风险敞口和保证金分析、估值调整分析和投资组合构建分析。
* [FinMath Library](https://github.com/finmath/finmath-lib)：FinMath Library库提供了与数学金融相关但适用于其他领域的方法的(JVM)实现。
* [Stripe](https://github.com/stripe/stripe-java)：Stripe API的Java库。
* [Philadelphia](https://github.com/paritytrading/philadelphia)：Philadelphia是一个用于JVM的快速FIX协议库。
* [Parity](https://github.com/paritytrading/parity)：Parity是一个用于交易场所的开源软件平台，它可用于运行金融市场、开发算法交易代理或研究市场微观结构。
* [Cassandre](https://github.com/cassandre-tech/cassandre-trading-bot)：Cassandre交易机器人框架允许你在多个加密货币交易所快速创建和执行交易策略。
* [Open Banking](https://github.com/wso2/financial-open-banking)：WSO2开放银行加速器是一系列技术的集合，可提高开放银行合规性的速度并降低其复杂性。
* [Joda-Money](https://github.com/JodaOrg/joda-money)：Joda-Money提供了一个类库来存储大量资金。
* [Prowide ISO 20022](https://github.com/prowide/prowide-iso20022)：Prowide ISO 20022是一个用于管理ISO 20022消息的开源Java框架。
* [Prowide](https://github.com/prowide/prowide-core)：Prowide Core是一个用于管理SWIFT FIN消息的开源Java框架。
* [Artio](https://github.com/real-logic/artio)：Artio是高性能FIX和FIXP网关。
* [jPOS](https://github.com/jpos/jPOS)：jPOS是一个开源的Java平台，用于构建和部署高度可扩展、事务处理、基于ISO-8583标准的金融交易处理系统。
* [Sailfish](https://github.com/exactpro/sailfish-core)：Sailfish是一个测试自动化工具，其主要目的是测试分布式交易平台和市场数据交付系统中的双向消息流。
* [Plaid-Java](https://github.com/plaid/plaid-java)：Plaid API的Java绑定。
* [Bateman](https://github.com/fearofcode/bateman)：Bateman是一个非常简单的交易系统，旨在筛选美国股票市场的子集。
* [Open Banking Gateway](https://github.com/adorsys/open-banking-gateway)：提供RESTful API、工具、适配器和连接器，用于透明访问开放银行API(适用于支持PSD2和XS2A以及HBCI/FinTS的银行)。
* [XS2A](https://github.com/adorsys/xs2a)：XS2A是一个完全符合PSD2标准的XS2A接口，支持所有强制和大多数可选的PSD2 XS2A流程。
* [Finance Quotes API](https://github.com/sstrickx/yahoofinance-api)：该库提供了一些方法，可以轻松地与Yahoo Finance API进行通信，它允许你请求股票的详细信息、一些统计数据和历史报价。
* [ISO8583 Message Client](https://github.com/imohsenb/ISO8583-Message-Client-java)：一个轻量级ISO8583库，适用于Java和Android，基于构建器模式。
* [JReactive-8583](https://github.com/kpavlov/jreactive-8583)：适用于ISO8583和Netty的Kotlin/Java客户端和服务器。
* [J8583](https://bitbucket.org/chochos/j8583)：J8583是ISO8583协议的Java实现。
* [SubMicroTrading](https://github.com/Richard-Rose/SubMicroTrading)：SubMicroTrading是一个高度并发的基于组件的算法交易框架。
* [Accounting](https://github.com/Nick-Triller/accounting)：Accounting是一个用Java编写的内存中复式记账组件。
* [JBanking](https://github.com/marcwrobel/jbanking)：JBanking是一个帮助开发银行功能的实用程序库，专注但不限于欧洲银行业。
* [Alpaca Java](https://github.com/Petersoj/alpaca-java)：这是Alpaca API的Java实现，Alpaca让你可以使用算法进行交易、与应用程序连接并通过免佣金的股票交易API构建服务。
* [IBC](https://github.com/IbcAlpha/IBC)：IBC可以自动化运行盈透证券交易者工作站和网关的许多方面。
* [Univocity Trader](https://github.com/uniVocity/univocity-trader)：Univocity Trader是一个开源交易框架，旨在使任何具有基本编程技能的人都能有效地创建和测试用于买卖股票、加密货币或任何其他类型工具的交易算法。
* [TA-Lib](https://github.com/BYVoid/TA-Lib)：TA-Lib是用于市场分析的多平台工具。
* [Eclipse Tradista](https://github.com/eclipse-tradista/tradista)：Tradista是一种轻量级的金融风险管理解决方案，使你能够使用单一工具管理你的日常财务和风险管理任务。
* [Quandl4J](https://github.com/jimmoores/quandl4j)：Quandl是一个通过开放REST API提供数百万个免费数据集的来源，涵盖金融、经济、社会和国家数据。
* [Trading Backtest](https://github.com/lukstei/trading-backtest)：这是一个用Java编写的通用轻量级股票回溯测试引擎。

## 智能卡

* [Eclipse Keypop](https://github.com/eclipse-keypop)：Keypop提供一套灵活的API来处理通用智能卡读卡器操作，同时还集成了针对特定卡技术(如Calypso标准)量身定制的专用API。
* [JavaEMVReader](https://github.com/sasc999/javaemvreader)：一种与EMV智能卡通信并读取其数据的工具。
* [Apdu4j](https://github.com/martinpaljak/apdu4j)：Apdu4j是一些命令行工具和有用的Java类库，用于通过JSR 268处理智能卡和智能卡读卡器。
* [JNASmartCardIO](https://github.com/jnasmartcardio/jnasmartcardio)：javax.smartcardio API的重新实现，它允许你从Java内部与智能卡(在APDU级别)进行通信。
* [JMultiCard](https://github.com/ctt-gob-es/jmulticard)：100% Java智能卡访问抽象层。

## 电子发票

* [NFE](https://github.com/wmixvideo/nfe)：Java中的电子发票。
* [Phase4](https://github.com/phax/phase4)：Phase4是一个可嵌入的轻量级Java库，用于发送和接收不同配置文件的AS4消息。
* [Java_NFe](https://github.com/Samuel-Oliveira/Java_NFe)：用于使用NFe/NFCe WebService的Java库。
* [RSHK jsifenlib](https://github.com/roshkadev/rshk-jsifenlib)：RSHK jsifenlib是一个开源库，无需外部依赖，通过Java与SIFEN(全国综合电子发票系统)进行交互。
* [Billy](https://github.com/premium-minds/billy)：Billy是一个应用程序计费库，为应用程序提供创建、管理和存储计费工件(例如发票和贷方票据)的能力。
* [Mustang](https://github.com/ZUGFeRD/mustangproject)：Mustang使你能够读取、写入和验证(例如重新计算)机器可读的发票、订单或交货通知。

## 短信

* [Twilio Java](https://github.com/twilio/twilio-java)：用于与Twilio REST API通信并生成TwiML的Java库。
* [SMS4J](https://gitee.com/dromara/sms4j)：SMS4J为短信聚合框架，可以轻松集成多家短信服务，解决接入多个短信SDK的繁琐流程。
* [Guerlab](https://gitee.com/guerlab_net/guerlab-sms)：Guerlab是基于Spring Boot的短信服务支持，通过引用不同的Starter启用不同的短信通道支持，支持多通道下的负载均衡，支持同步/异步方式发送。
* [SMSGate](https://github.com/Lihuanghe/SMSGate)：SMSGate是Netty 4框架实现的三网合一短信网关核心框架。
* [SMSCGateway](https://github.com/RestComm/smscgateway)：SMSC用于向移动运营商网络(GSM、SS7 MAP)、SMS聚合器(SMPP)和互联网电话服务提供商(SIP、SMPP)发送/接收SMS。
* [SMS](https://github.com/yunpian/sms)：云通讯、国际短信、短信API、短信SDK，短信平台，短信验证码，短信接口。
* [Cloudhopper SMPP](https://github.com/twitter-archive/cloudhopper-smpp)：Cloudhopper SMPP是SMPP的高效、可扩展且灵活的Java实现，由Twitter开源。
* [jSMPP](https://github.com/opentelecoms-org/jsmpp)：jSMPP是SMPP协议的Java实现，它提供与消息中心或ESME通信的接口，并且能够处理每秒3000-5000条消息的流量。
* [OpenSmpp](https://github.com/OpenSmpp/opensmpp)：OpenSmpp是一个成熟的Java库，可实现SMPP协议，并允许开发外部短信实体(ESME)等。

## DSL

* [Eclipse Xtend](https://github.com/eclipse/xtext-xtend)：Xtext是一个用于开发编程语言和DSL的框架。
* [MontiCore](https://github.com/MontiCore/monticore)：MontiCore是一个用于高效开发DSL的语言工作台，它处理定义DSL的扩展语法格式并生成用于处理DSL文档的Java组件，由亚琛工业大学开发。
* [Spoofax](https://github.com/metaborg/spoofax)：Spoofax是一个帮助开发者快速开发领域特定语言(DSL)的平台。

## JMX

* [Simple JMX](https://github.com/j256/simplejmx)：JMX Java库可帮助使用JMX和Web发布对象。
* [JMXUtils](https://github.com/martint/jmxutils)：让导出JMX mbean变得容易。

## RMI

* [ARMI](https://github.com/AugurSystems/ARMI)：ARMI是Java内置RMI的替代方案，最初是为了跨NAT工作而开发的。
* [Dirmi](https://github.com/cojen/Dirmi)：Dirmi是Java RMI的替代品，支持双向远程对象。
* [JrPip](https://github.com/goldmansachs/jrpip)：JrPip使用Java二进制序列化协议提供远程方法调用，由高盛银行开源。
* [SerializationDumper](https://github.com/NickstaDB/SerializationDumper)：一种以更易于理解的形式转储和重建Java序列化流和Java RMI数据包内容的工具。

## gRPC

* [Wire](https://github.com/square/wire)：Wire是适用于Android、Kotlin、Swift和Java的gRPC和协议缓冲区，Square开源。
* [Spring Grpc](https://github.com/spring-projects-experimental/spring-grpc)：Spring Grpc项目为开发Grpc应用程序提供了Spring友好的API和抽象。
* [gRPC Spring Boot](https://github.com/grpc-ecosystem/grpc-spring)：gRPC框架的Spring Boot Starter库。
* [gRPC Spring Boot](https://github.com/LogNet/grpc-spring-boot-starter)：gRPC的Spring Boot Starter模块。
* [gRPC Starter](https://github.com/DanielLiu1123/grpc-starter)：该项目为gRPC生态系统提供了开箱即用、高度可扩展的Spring Boot Starter。
* [JProtobuf](https://github.com/jhunters/jprotobuf)：JProtobuf是针对Java程序开发的一套简易类库，目的是简化Java语言对protobuf类库的使用，百度开源。
* [Polyglot](https://github.com/grpc-ecosystem/polyglot)：Polyglot是一个gRPC客户端，可以与任何gRPC服务器通信。
* [gRPC Spring Boot Starter](https://github.com/AnoyiX/grpc-spring-boot-starter)：Spring Boot快速集成gRPC，轻松实现远程方法调用。
* [gRPC Swagger](https://github.com/grpc-swagger/grpc-swagger)：使用Swagger-UI调试gRPC应用程序。
* [Mediator](https://github.com/ButterCam/Mediator)：gRPC调试代理跨平台GUI。
* [Google API Extensions Java](https://github.com/googleapis/gax-java)：适用于Java的Google API扩展。
* [gRPC Java Contrib](https://github.com/salesforce/grpc-java-contrib)：grpc-java库的有用扩展。
* [QuickBuffers](https://github.com/HebiRobotics/QuickBuffers)：QuickBuffers是Google Protocol Buffers的Java实现，专为零分配环境中的低延迟用例而开发。
* [Sisyphus](https://github.com/ButterCam/sisyphus)：基于JVM的现代gRPC后端开发框架。
* [nrtSearch](https://github.com/Yelp/nrtsearch)：基于Lucene的高性能gRPC服务器。

## 对象池

* [Stormpot](https://github.com/chrisvest/stormpot)：Stormpot是一个Java对象池库。
* [Apache Commons Pool](https://github.com/apache/commons-pool)：Commons Pool库提供了对象池API和许多对象池实现。
* [Fast Object Pool](https://github.com/DanielYWoo/fast-object-pool)：FOP是一个针对并发访问进行优化的轻量级高性能对象池。
* [PooledJMS](https://github.com/messaginghub/pooled-jms)：用于消息传递应用程序的JMS连接池，为JMS连接、会话和消息生产者提供池化。
* [Vibur Object Pool](https://github.com/vibur/vibur-object-pool)：Vibur对象池是一个通用并发Java对象池，完全使用标准Java并发实用程序构建，不使用任何同步块或方法，并且没有任何外部依赖项。
* [LitePool](https://github.com/nextopcn/lite-pool)：由Java编写的精简版快速对象池。
* [CoralPool](https://github.com/coralblocks/CoralPool)：CoralPool是一种高性能、轻量级且无垃圾的Java对象池实现。
* [Reactor Pool](https://github.com/reactor/reactor-pool)：Reactor Pool项目旨在为响应式应用程序提供一个通用对象池。

## CQRS框架

* [JdonFramework](https://github.com/banq/jdonframework)：JdonFramework是一个支持Pub-Sub异步编程模型的领域事件框架。
* [Reveno](https://github.com/dmart28/reveno)：Reveno是一款彻底的、快如闪电、耐用且简单的基于JVM的异步事务处理框架。
* [Splitet](https://github.com/Splitet/SplitetFramework)：Splitet是一个基于Java的事件溯源框架，计划以最小的学习曲线和易于适应的方式进行CQRS转换的团队可以从中受益。
* [Loom](https://github.com/loom/loom-java)：Loom是一组用于实现分布式消息传递和事件源模式的框架。
* [Sourcerer](https://github.com/elder-oss/sourcerer)：Sourcerer是一个固执己见、功能性且与存储无关的框架，用于使用事件源在Java 8中实现CQRS架构。
* [Apache Polygene](https://github.com/apache/polygene-java)：Java平台上的面向组合编程。
* [Dewdrop](https://github.com/matsientst/dewdrop)：Dewdrop是一个固执己见、简单而强大的框架，用于在Java中实现事件源。
* [ES4j](https://github.com/eventsourcing/es4j)：Java的事件捕获和查询框架。
* [PipelinR](https://github.com/sizovs/pipelinr)：用于通过管道使用处理程序和命令的小型实用程序库。
* [Assembler](https://github.com/pellse/assembler)：Assembler是一个响应式、函数式、类型安全和无状态的数据聚合框架，用于查询和合并来自多个数据源/服务的数据。
* [Occurrent](https://github.com/johanhaleby/occurrent)：Occurrent是一个基于云事件规范的事件溯源库。
* [Concursus](https://github.com/opencredo/concursus)：Concursus是一个Java 8框架，用于构建使用CQRS和事件源模式以及Cassandra事件日志实现的应用程序。
* [Thoth](https://github.com/MAIF/thoth)：Thoth是一个Java库，它提供了在应用程序中实现事件源的工具包。
* [Fluxtion](https://github.com/v12technology/fluxtion)：Fluxtion是一种Java开发生产力工具，使编写和维护事件驱动的业务逻辑变得更便宜、更快捷。
* [Synapse](https://github.com/otto-de/synapse)：实现事件溯源微服务的库。
* [Wow](https://github.com/Ahoo-Wang/Wow)：现代响应式CQRS架构，基于DDD和事件溯源的微服务开发框架。
* [Deezpatch](https://github.com/joel-jeremy/deezpatch)：简单、轻量级且高性能的调度库，用于解耦消息(请求和事件)和消息处理程序。
* [FactCast](https://github.com/factcast/factcast)：基于PostgreSQL的简单EventStore。

## DDD框架

* [COLA](https://github.com/alibaba/COLA)：COLA代表整洁面向对象分层架构，由阿里开源。
* [Axon](https://github.com/AxonFramework/AxonFramework)：Axon是一个基于DDD、CQRS和事件溯源原则构建渐进式事件驱动微服务系统的框架。
* [PICASO](https://zhuanlan.zhihu.com/p/7561767079)：PICASO是一套以DDD作为思想内核，专门为集成式复杂业务系统设计的通用基础框架，由京东开发。
* [jMolecules](https://github.com/xmolecules/jmolecules)：可帮助开发人员以无干扰、简单的旧式Java实现域模型的一组库。
* [Library](https://github.com/ddd-by-examples/library)：全面的领域驱动设计示例，包含问题空间战略分析和各种战术模式。
* [DDDplus](https://github.com/funkygao/cp-ddd-framework)：DDDplus是一个轻量级的DDD正向/逆向业务建模增强框架，支持复杂的系统架构演进。
* [Dante Cloud](https://gitee.com/dromara/dante-cloud)：Dante Cloud是企业级微服务架构和服务能力开发平台。
* [ContextMapper](https://github.com/ContextMapper/context-mapper-dsl)：ContextMapper是一个开源项目，为战略领域驱动设计、上下文映射、有界上下文建模和服务分解提供领域特定语言和工具。
* [Apache Causeway](https://github.com/apache/causeway)：Causeway是一个用Java快速开发领域驱动应用程序的框架。
* [DDDLib](https://github.com/dayatang/dddlib)：DDDLib是一个领域驱动设计类库。
* [DDD Base](https://github.com/linux-china/ddd-base)：Java领域驱动设计基础包。
* [Spine Event Engine](https://github.com/SpineEventEngine/core-java)：Spine Event Engine是一个Java框架，用于构建事件源和CQRS应用程序。
* [Aggregate Framework](https://github.com/changmingxie/aggregate-framework)：Aggregate Framework是一款基于DDD和CQRS思想开发的领域驱动框架。
* [Jexxa](https://github.com/jexxa-projects/Jexxa)：Jexxa是一个轻量级框架，可使用DDD来简化持久业务应用程序的实现。
* [SpringBoot-DDD](https://github.com/codingapi/springboot-framework)：基于Spring Boot为提供领域驱动设计与事件风暴开发落地，提供的范式开源框架。
* [Cheddar](https://github.com/travel-cloud/Cheddar)：Cheddar是一个Java框架，适用于AWS上的企业应用程序，使用域驱动设计(DDD)。
* [ZenWave SDK](https://github.com/ZenWave360/zenwave-sdk)：ZenWave SDK是一个用于DDD和API优先的可配置且可扩展的工具包，可以从不同模型的组合生成代码。

## 软件工程

* [Rosie](https://github.com/Karumi/Rosie)：Rosie是一个Android框架，用于创建遵循清洁架构原则的应用程序。
* [Eclipse Capella](https://github.com/eclipse/capella)：Capella是一款全面、可扩展且经过现场验证的MBSE工具和方法，可用于成功设计系统架构，由Thales开源。
* [Structurizr](https://c4model.com/)：Structurizr打破了架构图编辑器(例如UML)的传统拖放方法，并允许我们使用我们最了解的工具Java来描述我们的架构工件。
* [Spring Modulith](https://github.com/spring-projects/spring-modulith)：Spring Modulith允许开发人员构建结构良好的Spring Boot应用程序，并指导开发人员查找和使用由领域驱动的应用程序模块。
* [Moduliths](https://github.com/moduliths/moduliths)：Moduliths是用于构建模块化、整体式Spring Boot应用程序的框架。
* [ArchGuard](https://github.com/archguard/archguard)：ArchGuard是一个针对于微服务下的架构工作台/治理工具，它可以帮助架构师、开发人员进行架构自助，自定义架构的洞察、分析系统间的远程服务依赖情况、数据库依赖、API依赖等。
* [Structurizr Java](https://github.com/structurizr/java)：Structurizr基于“图即代码”构建，允许你从单个模型创建多个软件架构图。
* [JRugged](https://github.com/Comcast/jrugged)：JRugged库实现了用Java构建健壮的、可用于生产的服务器代码所需的一些常见模式，由Comcast开源。
* [RR](https://github.com/GuntherRademacher/rr)：RR是语法图生成器(也称为铁路图)，它是一个独立的工具，具有基于浏览器的GUI和批处理模式。
* [JIG](https://github.com/dddjava/jig)：JIG是一个支持代码设计的工具。
* [RRDiagram](https://github.com/Chrriis/RRDiagram)：RR图是一个Java库，可从代码或BNF表示法生成铁路图。
* [VMF](https://github.com/miho/VMF)：VMF是一个轻量级的建模框架，它可以方便地将带注解的Java接口转换为功能强大的实现。
* [Umple](https://github.com/umple/umple)：Umple是一种面向模型的编程技术，允许开发人员在传统代码中嵌入建模概念(例如UML 关联、状态机)、模式、生成模板和其他抽象，反之亦然，由渥太华大学开源。
* [Lowfer](https://github.com/mbouchenoire/lowfer)：Lowfer是一个简单的工具，可以帮助软件工程师和架构师记录、讨论和分析软件设计和架构。
* [FJage](https://github.com/org-arl/fjage)：FJage为Java和Groovy中面向代理的软件开发提供了一个轻量级且易于学习的平台，由新加坡国立大学开源。
* [JSyntrax](https://github.com/atp-mipt/jsyntrax)：JSyntrax是一个铁路图生成器，它创建了用于编程语言的语法的直观说明，由莫斯科物理技术学院开源。

## 设计模式

* [Apache Commons Chain](https://github.com/apache/commons-chain)：GoF责任链模式的实现。
* [Apache Commons Proxy](https://github.com/apache/commons-proxy)：用于动态代理的Java库。
* [Decorator](https://github.com/eyeem/decorator)：动态继承库，装饰器模式的实现。
* [AutoProxy](https://github.com/OleksandrKucherenko/autoproxy)：在接口/抽象类之上生成代理类，允许拦截调用。
* [Dynamic Proxy](https://github.com/neoremind/dynamic-proxy)：Dynamic Proxy是用于Java生成代理对象的有用库。
* [Pie](https://github.com/feiniaojin/pie)：Pie是一个可快速上手的责任链框架。

## 幂等处理

* [Idempotent](https://github.com/it4alla/idempotent)：幂等处理方案。
* [Idempotent Spring Boot Starter](https://github.com/pig-mesh/idempotent-spring-boot-starter)：对原有idempotent代码重构和功能增强。
* [Jdempotent](https://github.com/Trendyol/Jdempotent)：轻松使你的端点幂等，由Trendyol开源。
* [Idempotence4j](https://github.com/transferwise/idempotence4j)：Idempotence4j是一个轻量级库，为处理幂等操作提供支持。
* [Quidem](https://github.com/julianhyde/quidem)：Quidem是一个幂等查询执行器。
* [Tomato](https://github.com/lxchinesszz/tomato)：Tomato是一款专门为Spring Boot项目设计的幂等组件。

## 数据字典

* [Dict Trans](https://gitee.com/aizuda/dict-trans)：由爱组搭开源的简单字典翻译组件。
* [Transformer](https://github.com/luo-zhan/Transformer)：Transformer是一款功能全面的字段转换工具，只需要几个简单的注解，让开发更简单。

## 迁移&重构

* [EMT4J](https://github.com/adoptium/emt4j)：EMT4J是一个旨在简化Java版本迁移的项目，由阿里开源。
* [Rewrite](https://github.com/openrewrite/rewrite)：OpenRewrite项目是一个源代码自动重构生态系统，使开发人员能够有效消除其仓库中的技术债务，由Netflix开源。
* [Spring Boot Migrator](https://github.com/spring-projects-experimental/spring-boot-migrator)：Spring Boot Migrator旨在通过提供自动迁移的方法来帮助开发人员升级或迁移到Spring Boot。
* [Windup](https://github.com/windup/windup)：Windup是一个工具集，支持跨广泛转换和用例的大规模Java应用程序现代化和迁移项目，由RedHat开源。
* [Jakarta Migration](https://github.com/apache/tomcat-jakartaee-migration)：该工具的目的是自动对为Java EE 8编写并在Tomcat 9上运行的Web应用程序进行转换，以便可以在实现Jakarta EE 9的Tomcat 10上运行。
* [Scientist4J](https://github.com/rawls238/Scientist4J)：Github重构工具Scientist的Java移植。
* [RefactoringMiner](https://github.com/tsantalis/RefactoringMiner)：RefactoringMiner是一个用Java编写的库/API，可以检测Java项目历史中应用的重构。
* [Eclipse Transformer](https://github.com/eclipse/transformer)：Transformer提供了转换Java二进制文件的工具和运行时组件，将更改映射到Java包、类名称和相关资源名称，由Open Liberty团队开源。
* [Butterfly](https://github.com/paypal/butterfly)：Butterfly是一种应用程序代码转换工具，通常用于执行自动化应用程序迁移、升级以及源代码和配置更改，由Paypal开源。
* [MigrationMiner](https://github.com/hussien89aa/MigrationMiner)：MigrationMiner是检测两个Java第三方库之间迁移代码的工具。
* [Astra](https://github.com/alfasoftware/astra)：Astra是一个用于分析和重构Java源代码的Java工具。

## Bot

* [ChopperBot](https://github.com/Geniusay/ChopperBot)：ChopperBot一款全自动的主播切片机器人。
* [BenTen](https://github.com/intuit/benten)：BenTen是一个CUI聊天机器人框架，它提供了在几分钟内构建有用的对话聊天机器人所需的所有集成，由Intuit开发。
* [Tock](https://github.com/theopenconversationkit/tock)：Tock是一个完整且开放的平台，用于构建对话代理(也称为机器人)。
* [R-Bot](https://github.com/semicons/java_oci_manage)：本系统目前应用于甲骨文云/Azure云的一些快捷操作。
* [PhantomBot](https://github.com/PhantomBot/PhantomBot)：PhantomBot是一款积极开发的开源交互式Twitch机器人。
* [Line Message SDK](https://github.com/line/line-bot-sdk-java)：适用于Java的LINE Messaging API SDK可以轻松使用LINE Messaging API开发机器人，并且可以在几分钟内创建示例机器人。
* [BotLibre](https://github.com/BotLibre/BotLibre)：适用于人工智能、聊天机器人、虚拟代理、社交媒体自动化和实时聊天自动化的开放平台。
* [Repairnator](https://github.com/eclipse/repairnator)：Repairnator是Github上的一个软件机器人开源项目，特别用于自动化程序修复：构建失败修复、静态警告修复(SoraldBot)等。
* [EDDI](https://github.com/labsai/EDDI)：E.D.D.I是一种中间件，用于连接和管理LLM API机器人。
* [TradeBot](https://github.com/markusaksli/TradeBot)：使用Binance API的加密货币交易机器人。
* [春松客服](https://github.com/cskefu/cskefu)：春松客服是开源的智能客服系统。
* [Microsoft Bot Framework Java SDK](https://github.com/microsoft/botbuilder-java)：Microsoft Bot Framework提供了构建和连接智能机器人所需的功能，无论用户在哪里交谈，这些机器人都可以自然地交互，从文本/短信到Skype、Slack、Office 365邮件和其他流行服务。
* [Mutters](https://github.com/rabidgremlin/Mutters)：构建机器人大脑的框架。
* [FeishuBot](https://github.com/rawchen/FeishuBot)：飞书群聊/私聊ChatGPT机器人。
* [PokuBot](https://github.com/norecha/pokubot)：部落冲突机器人。
* [Command-Flow](https://github.com/FixedDev/command-flow)：适用于Java 8+的灵活且与平台无关的命令框架。
* [PircBotX](https://github.com/pircbotx/pircbotx)：PircBotX是一个强大的Java IRC客户端库，适用于机器人和用户客户端。
* [Kitteh IRC Client Lib](https://github.com/KittehOrg/KittehIRCClientLib)：KICL是一个功能强大的现代Java IRC库，使用Netty库构建，以最大限度地提高性能和可扩展性。

#### Discord机器人

* [JMusicBot](https://github.com/jagrosh/MusicBot)：JMusicBot是一个可以轻松设置和运行的Discord音乐机器人。
* [JDA](https://github.com/discord-jda/JDA)：该开源库旨在使用实时网关和REST API在Discord上实现机器人。
* [Kord](https://github.com/kordlib/kord)：Kord是一个基于协程、模块化的Discord API实现。
* [Discord4J](https://github.com/Discord4J/Discord4J)：Discord4J是一个快速、强大、无偏见的响应式库，可使用官方Discord Bot API快速轻松地开发适用于Java、Kotlin和其他JVM语言的Discord机器人。
* [CatNip](https://github.com/mewna/catnip)：Java中的Discord API包装器，完全异步/响应式，构建在RxJava之上。
* [Javacord](https://github.com/Javacord/Javacord)：一个易于使用的多线程库，用于在Java中创建Discord机器人。
* [DiscordJar](https://github.com/discord-jar/discord.jar)：DiscordJar是一个正在进行中的Discord API Java包装器。
* [GiveawayBot](https://github.com/jagrosh/GiveawayBot)：在Discord服务器上快速轻松地保存赠品。
* [Vortex](https://github.com/jagrosh/Vortex)：Discord审核机器人。
* [MantaroBot](https://github.com/Mantaro/MantaroBot)：使用JDA用Java制作的多用途Discord机器人。
* [Sokobot](https://github.com/PolyMarsDev/Sokobot)：一个可以玩推箱子的Discord机器人。
* [AIODE](https://github.com/robinfriedli/aiode)：可播放Spotify曲目和YouTube视频或任何URL(包括Soundcloud链接和Twitch流)的Discord机器人。
* [JDA Utilities](https://github.com/JDA-Applications/JDA-Utilities)：JDA Utilities是一系列与JDA一起使用来协助创建机器人的工具和实用程序。
* [Chuu](https://github.com/ishwi/Chuu)：一个Discord机器人，将Last.fm与Discord集成。
* [Ree6](https://github.com/Ree6-Applications/Ree6)：Ree6是一款由Presti维护的一体化Discord机器人。
* [Spectra](https://github.com/jagrosh/Spectra)：Spectra是一款私人、多用途、娱乐性和实用性的Discord机器人。

#### Telegram机器人

* [TelegramBots](https://github.com/rubenlagus/TelegramBots)：使用Telegram Bots API创建机器人的Java库。
* [Java Telegram Bot API](https://github.com/pengrad/java-telegram-bot-api)：用于Java的Telegram Bot API。
* [Kotlin Telegram Bot](https://github.com/kotlin-telegram-bot/kotlin-telegram-bot)：用Kotlin编写的Telegram Bot API的包装器。
* [TeleightBots](https://github.com/Teleight/TeleightBots)：TeleightBots是一个轻量级、高性能、易于使用的Java Telegram Bot API包装器。
* [TDLight Java](https://github.com/tdlight-team/tdlight-java)：基于TDLib的完整Bot和Userbot Telegram库。

#### Facebook机器人

* [JBot](https://github.com/rampatra/jbot)：JBot是一个Java框架，可在几分钟内创建Slack和Facebook机器人。
* [Messenger4j](https://github.com/messenger4j/messenger4j)：用于在Facebook Messenger平台上构建聊天机器人的Java库。

#### QQ机器人

* [Mirai](https://github.com/mamoe/mirai)：Mirai是一个在全平台下运行，提供QQ Android协议支持的高效率机器人库。
* [OpenShamrock](https://github.com/whitechi73/OpenShamrock)：OpenShamrock是基于Xposed实现OneBot标准的QQ机器人框架。
* [Shiro](https://github.com/MisakaTAT/Shiro)：基于OneBot协议的QQ机器人快速开发框架。
* [Smart QQ](https://github.com/ScienJus/smartqq)：SmartQQ API，可以用它实现自己的QQ机器人。
* [QQPD Bot Java](https://github.com/Kloping/qqpd-bot-java)：Java SDK主要基于基础API封装，提供给用户一种简单、高效的使用方式。
* [Simple Robot](https://github.com/simple-robot/simpler-robot)：Simple Robot是一个基于Kotlin协程的多平台Bot风格高性能异步事件调度框架，异步高效、Java友好。

#### 微信机器人

* [WeChat Robot](https://gitee.com/hellokaton/wechat-robot)：Java版微信普通号机器人。
* [Jeeves](https://github.com/kanjielu/jeeves)：一个智能微信机器人。
* [WeChatBotEngine](https://github.com/moontide/WeChatBotEngine)：基于微信网页版HTTP协议的机器人引擎。

## 安卓库

* [MPAndroidChart](https://github.com/PhilJay/MPAndroidChart)：强大的Android图表视图/图形视图库，支持折线图、饼图、雷达图、气泡图和烛台图以及缩放、平移和动画。
* [Lottie](https://github.com/airbnb/lottie-android)：适用于Android和iOS的移动库，它可以解析使用Bodymovin导出为JSON的Adobe After Effects动画，并在移动设备上本地渲染它们，由Airbnb开源。
* [VasDolly](https://github.com/Tencent/VasDolly)：VasDolly是一种快速多渠道打包工具，同时支持基于V1签名和V2、V3签名进行多渠道打包，由腾讯开源。
* [Glide](https://github.com/bumptech/glide)：快速高效的Android开源媒体管理和图像加载框架，它将媒体解码、内存和磁盘缓存以及资源池封装到一个简单易用的界面中。
* [Butter Knife](https://github.com/JakeWharton/butterknife)：将Android视图和回调绑定到字段和方法。
* [MIT App Inventor](https://github.com/mit-cml/appinventor-sources)：App Inventor是一款Google开发的手机编程软件，现由MIT开发维护。
* [NewPipe](https://github.com/TeamNewPipe/NewPipe)：适用于Android的自由轻量级流媒体前端。
* [Termux](https://github.com/termux/termux-app)：Android终端应用程序和Linux环境。
* [SmartRefreshLayout](https://github.com/scwang90/SmartRefreshLayout)：SmartRefreshLayout以打造一个强大、稳定、成熟的下拉刷新框架为目标，并集成各种的炫酷、多样、实用、美观的Header和Footer。
* [Freeline](https://github.com/alibaba/freeline)：Android超快速构建工具，Instant Run的替代品，由阿里开源。
* [Signal Android](https://github.com/signalapp/Signal-Android)：简单、强大且安全的信使，由Open Whisper Systems开发。
* [Telegram](https://github.com/DrKLO/Telegram)：一款注重速度和安全性的消息应用程序。
* [ExoPlayer](https://github.com/google/ExoPlayer)：适用于Android的可扩展媒体播放器，由Google开源。
* [RxAndroid](https://github.com/ReactiveX/RxAndroid)：适用于Android的RxJava绑定。
* [DoKit](https://github.com/didi/DoKit)：一款面向泛前端产品研发全生命周期的效率平台，由滴滴开源。
* [Walle](https://github.com/Meituan-Dianping/walle)：Android Signature V2 Scheme签名下的新一代渠道包打包神器，由美团开源。
* [Telescope](https://github.com/mattprecious/telescope)：一个简单的工具，可以在你的应用程序中轻松捕获错误报告。
* [PhotoView](https://github.com/Baseflow/PhotoView)：Android版ImageView的实现，支持通过各种触摸手势进行缩放。
* [ApkAnalyser](https://github.com/sonyxperiadev/ApkAnalyser)：ApkAnalyser是一种静态虚拟分析工具，用于检查和验证Android应用程序的开发工作，由索尼开源。
* [Fresco](https://github.com/facebook/fresco)：用于管理图像及其使用的内存的Android库，由Facebook开源。
* [Tinker](https://github.com/Tencent/tinker)：Android的热修复解决方案库，它支持dex、库和资源更新，无需重新安装apk，由腾讯开源。
* [Material Components Android](https://github.com/material-components/material-components-android)：适用于Android的模块化和可定制的Material Design UI组件。
* [VirtualXposed](https://github.com/android-hacker/VirtualXposed)：无需Root即可使用Xpose，解锁引导加载程序或修改系统镜像等。
* [CircleImageView](https://github.com/hdodenhof/CircleImageView)：快速的圆形ImageView，非常适合个人资料图像。
* [ARouter](https://github.com/alibaba/ARouter)：帮助Android APP进行组件化改造的路由框架，由阿里开源。
* [QMUI_Android](https://github.com/Tencent/QMUI_Android)：提高Android UI开发效率的UI库，由腾讯开源。
* [StringFog](https://github.com/MegatronKing/StringFog)：一款自动对字节码中的字符串进行加密Android插件工具。
* [JsBridge](https://github.com/lzyzsd/JsBridge)：Android Java和JavaScript的桥梁。
* [Stetho](https://github.com/facebookarchive/stetho)：Android应用程序的调试桥，支持强大的Chrome开发者工具等，由Facebook开源。
* [SmartTube](https://github.com/yuliskov/SmartTubeNext)：适用于运行Android操作系统的机顶盒和电视的高级播放器。
* [Matisse](https://github.com/zhihu/Matisse)：精心设计的Android本地图像和视频选择器，由知乎开源。
* [QtScrcpy](https://github.com/barry-ran/QtScrcpy)：Android实时显示控制软件。
* [LSPosed](https://github.com/LSPosed/LSPosed)：Xpose是一个模块框架，可以在不接触任何APK的情况下更改系统和应用程序的行为。
* [VasSonic](https://github.com/Tencent/VasSonic)：腾讯VAS团队开发的一款轻量级、高性能的Hybrid框架，旨在加速Android和iOS平台上的网站首屏速度。
* [uCrop](https://github.com/Yalantis/uCrop)：Android图像裁剪库。
* [Dex2Jar](https://github.com/pxb1988/dex2jar)：用于处理Android .dex和Java .class文件的工具。
* [PermissionsDispatcher](https://github.com/permissions-dispatcher/PermissionsDispatcher)：用于处理Android运行时权限的声明式API。
* [DanmakuFlameMaster](https://github.com/bilibili/DanmakuFlameMaster)：Android上开源弹幕解析绘制引擎项目，由B站开发。
* [APK Parser](https://github.com/hsiafan/apk-parser)：APK解析库，用于解码二进制XML文件，获取APK元信息。
* [Epoxy](https://github.com/airbnb/epoxy)：用于在RecyclerView中构建复杂的屏幕，由Airbnb开源。
* [Atlas](https://github.com/alibaba/atlas)：一个强大的Android动态组件框架，由阿里开源。
* [Litho](https://github.com/facebook/litho)：用于在Android上构建高效UI的声明式框架，由Facebook开源。
* [RePlugin](https://github.com/Qihoo360/RePlugin)：一个灵活、稳定、易用的Android插件框架，由360开源。
* [Shadow](https://github.com/Tencent/Shadow)：零反射全动态Android插件框架，由腾讯开源。
* [Robust](https://github.com/Meituan-Dianping/Robust)：一款高兼容性、高稳定性的Android HotFix解决方案，由美团开源。
* [MagicaSakura](https://github.com/bilibili/MagicaSakura)：Android多主题框架，由B站开源。
* [Bundletool](https://github.com/google/bundletool)：操作Android App Bundle和Android SDK Bundle的工具，由Google开源。
* [Andromeda](https://github.com/iqiyi/Andromeda)：Andromeda为本地和远程服务提供模块之间的通信，由爱奇艺开发。

## GUI开发/程序

这里列出了Java中常用的Swing、JavaFX开发库，以及一些Java开发的GUI工具和游戏引擎。

#### GUI框架

* [Swing](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/swing/package-summary.html)：Swing提供一组轻量级组件，可以在所有平台上以相同的方式工作。
* [JavaFX](https://github.com/openjdk/jfx)：JavaFX是一个开源的客户端应用程序平台，适用于基于Java SE的桌面、移动和嵌入式系统，Oracle开源。
* [Jetpack Compose](https://github.com/JetBrains/compose-multiplatform)：Compose Multiplatform是一个声明式框架，用于使用Kotlin跨多个平台共享UI，由JetBrains开发。
* [Eclipse SWT](https://github.com/eclipse-platform/eclipse.platform.swt)：SWT是一个用于Java的开源小部件工具包，由IBM开源。
* [AWT](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/java/awt/package-summary.html)：AWT是Java最初的依赖于平台的窗口、图形和用户界面小部件工具包，早于Swing。
* [TotalCross](https://github.com/TotalCross/totalcross)：TotalCross是一款帮助跨平台应用程序开发的工具包。
* [NetBeans Platform](https://netbeans.apache.org/tutorial/main/kb/docs/platform/)：NetBeans Platform是一个广泛的Java框架，可以在其上构建大型桌面应用程序。
* [JGoodies](https://www.jgoodies.com/)：JGoodies提供免费软件、产品、设计和服务，帮助你高效地使用Java桌面。
* [QtJambi](https://github.com/OmixVisualization/qtjambi)：QtJambi是Java编程语言的Qt绑定，最初由Qt公司的前身Trolltech开发。
* [Dragome](https://www.dragome.com/)：Dragome是一款开源工具，用于以纯Java语言创建客户端Web应用程序。
* [Apache Pivot](https://github.com/apache/pivot)：Pivot是一个用Java构建丰富互联网应用程序的平台，其中WTK工具包是它的图形组件，由VMware开源。
* [SwingME](http://swingme.org/)：类似于Java Swing的跨平台GUI框架，具有布局、边框、渲染器和许多组件，包括文本字段、按钮、窗口、选项卡窗格等。

#### Swing

* [Lanterna](https://github.com/mabe02/lanterna)：Lanterna是一个Java库，允许你在纯文本环境中编写简单的半图形用户界面，与C库curses非常相似，但具有更多功能。
* [JFormDesigner](https://www.formdev.com/)：JFormDesigner是一款专业的Java Swing用户界面GUI设计器。
* [RSyntaxTextArea](https://github.com/bobbylight/RSyntaxTextArea)：RSyntaxTextArea是一个用于Java Swing应用程序的可定制的语法突出显示文本组件。
* [Radiance](https://github.com/kirill-grouchnikov/radiance)：Radiance是一个库集合，用于基于Ephemeral设计系统编写现代、优雅且快速的Swing应用程序。
* [JxCapture](https://teamdev.com/jxcapture/)：JxCapture是一种跨平台、能Java应用程序提供综合屏幕抓取API的库程序。
* [SystemTray](https://github.com/dorkbox/SystemTray)：对Java 8+上的Swing/AWT、GtkStatusIcon和AppIndicator提供专业、跨平台的SystemTray支持。
* [DJ Native Swing](https://github.com/Chrriis/DJ-Native-Swing)：DJ Native Swing库允许将一些原生组件轻松集成到Swing应用程序中，并提供一些本机实用程序来增强Swing的API。
* [AutoComplete](https://github.com/bobbylight/AutoComplete)：AutoComplete是Swing JTextComponents的代码完成库，具有可用于RSyntaxTextArea实例的增强功能。
* [Chromium](https://github.com/equodev/chromium)：Equo Chromium Community小部件是一个跨平台浏览器，允许用户在Java应用程序内创建和呈现基于Web的现代UI。
* [jSystemThemeDetector](https://github.com/Dansoftowner/jSystemThemeDetector)：用于检测(桌面)操作系统是否使用深色UI主题的Java库。
* [Sierra](https://github.com/HTTP-RPC/Sierra)：Sierra是一个用于简化Java Swing应用程序开发的开源框架。
* [Cinch](https://github.com/palantir/Cinch)：管理MVC模式的组件操作/事件绑定的Java库，由Palantir开源。
* [jGAF](https://github.com/pgdurand/jGAF)：jGAF是一个旨在促进为Java平台创建跨平台图形应用程序的库。
* [Projector Server](https://github.com/JetBrains/projector-server)：用于远程运行Swing应用程序的服务器端库，由JetBrains开源。
* [ReflectionUI](https://github.com/dotxyteam/ReflectionUI)：基于Java反射的GUI构建器/生成器。
* [Compositor](https://compositor.sourceforge.net/)：Compositor让你可以将应用程序的UI与代码分开描述。
* [MDIUtilities](https://mdiutilities.sourceforge.io/)：MDIUtilities是一个提供各种实用程序类的Java库，主要用于Swing或JavaFX开发。

#### Swing主题库

* [FlatLaf](https://github.com/JFormDesigner/FlatLaf)：FlatLaf是Java Swing桌面应用程序的现代开源跨平台LaF。
* [Material Color Utilities](https://github.com/material-foundation/material-color-utilities)：Material Color Utilities通过一组包含算法和实用程序的颜色库为动态颜色提供支持，这些算法和实用程序可让你更轻松地在应用中开发颜色主题和方案。
* [BeautyEye](https://github.com/JackJiang2011/beautyeye)：BeautyEye是一款Java Swing跨平台外观实现。
* [Jewel](https://github.com/JetBrains/jewel)：Jewel旨在在Compose Desktop中重新创建IntelliJ平台的新UI Swing LaF，提供桌面优化的主题和组件集，由JetBrains开源。
* [WebLaf](https://github.com/mgarin/weblaf)：WebLaf是一个完全开源的外观和组件库，用纯Java编写，用于跨平台桌面Swing应用程序。
* [Darklaf](https://github.com/weisJ/darklaf)：Darklaf是基于Darcula-Laf的主题LaF。
* [VAqua](https://github.com/violetlib/vaqua)：VAqua是一种Java Swing外观，在macOS上运行，模拟标准macOS UI组件的外观和行为。
* [KControls](https://github.com/k33ptoo/KControls)：用于美化用户界面并赋予UI现代化的外观。
* [VTerminal](https://github.com/Valkryst/VTerminal)：用于Java的新LaF，允许基于网格显示具有自定义前景色/背景色、字体大小和伪着色器的Unicode字符。
* [Darcula](https://github.com/bulenkov/Darcula)：Darcula是Java桌面应用程序的LaF和代码编辑器主题。
* [Synthetica](https://www.jyloo.com/)：Synthetica是Swing的LaF，基于Synth，通过主题为Swing的核心组件提供了许多不同的外观，包括圆形边框、阴影弹出菜单和漂亮的图标。

#### Swing UI库

* [Material UI Swing](https://github.com/atarw/material-ui-swing)：适用于Java Swing的现代Material Design UI。
* [Swing9patch](https://github.com/JackJiang2011/Swing9patch)：Swing9patch工程是一组很酷的Java Swing可重用组件或UI效果。
* [SnapKit](https://github.com/reportmill/SnapKit)：SnapKit是一个现代Java UI库和工具，用于创建丰富的Java客户端应用程序。
* [ImGui](https://github.com/kotlin-graphics/imgui)：这是ImGui的Kotlin重写，一个用于C++的无膨胀图形用户界面库。
* [Lemur](https://github.com/jMonkeyEngine-Contributions/Lemur)：Lemur是用于在jMonkeyEngine应用程序中制作用户界面的GUI工具包。
* [Griffon](https://github.com/griffon/griffon)：Griffon是JVM的桌面应用程序开发平台，受Grails的启发，Griffon利用了约定优于配置、模块化和选择自由等概念。
* [JGoodies](https://www.jgoodies.com/downloads/libraries/)：JGoodies可帮助美化桌面应用程序并更快地构建设计良好、一致的视图。
* [Eclipse Nebula](https://github.com/eclipse/nebula)：该项目包含大量UI元素，可在基于Java和SWT的胖或瘦客户端应用程序中使用。
* [JIDE](https://github.com/jidesoft/jide-oss)：JIDE提供专业的Java UI组件，专注于为软件开发人员提供富客户端应用程序。

#### Swing组件库

* [SwingBits](https://github.com/eugener/oxbow)：SwingBits是Java Swing Toolkit的有用组件和实用程序的集合。
* [UiBooster](https://github.com/Milchreis/UiBooster)：UiBooster是一个精益库，用于为实用工具创建快速且简单的对话框。
* [JXMapViewer2](https://github.com/msteiger/jxmapviewer2)：该项目基于SwingX-WS的JXMapViewer组件。
* [HMeter](https://sourceforge.net/projects/hasnatmeter/)：HMeter是一个高度可定制的Java Swing小部件，旨在直观地显示水平进度。
* [Notify](https://github.com/dorkbox/Notify)：适用于Java 8+的桌面和应用程序的Linux、MacOS或Windows弹出窗口。
* [AsciiPanel](https://github.com/trystan/AsciiPanel)：AsciiPanel模拟Code page 437 ASCII终端显示，它支持Code page 437的全部256个字符、任意前景色、任意背景色和任意终端大小。
* [Two Slices](https://github.com/sshtools/two-slices)：用于Windows、Mac OS X和Linux上的Java桌面通知的简单库。
* [Swing Console](https://github.com/mikera/swing-console)：Swing文本控制台组件，可用于提供类似于Java中的终端的功能。
* [JTouchBar](https://github.com/Thizzer/jtouchbar)：用于在受支持的MacBook上使用touchbar API的Java库。
* [JSplitButton](https://github.com/rhwood/jsplitbutton)：Java Swing的分割按钮控件的简单实现。
* [JTreeTable](https://github.com/javagl/JTreeTable)：JTreeTable实现。
* [File Manager](https://github.com/javadev/file-manager)：Java/Swing基本文件管理器。
* [Fireplace](https://github.com/bric3/fireplace)：Java Swing火焰图组件。
* [JInputValidator](https://github.com/rhwood/jinputvalidator)：Java Swing的InputVerifier，在验证组件的右侧显示验证状态。
* [ModernDocking](https://github.com/andrewauclair/ModernDocking)：Modern Docking是一个简单的框架，旨在为Java Swing应用程序添加程序坞功能。

#### Swing布局库

* [Yoga](https://github.com/facebook/yoga)：Yoga是一个可嵌入的高性能Flexbox布局引擎，具有多种语言的绑定，由Facebook开源。
* [TableLayout](https://github.com/EsotericSoftware/tablelayout)：TableLayout是一个轻量级Java库，用于使用逻辑表(类似于HTML表格)设置UI小部件的位置和大小。
* [SlidingLayout](https://github.com/AurelienRibon/sliding-layout)：功能强大的Java Swing面板/布局，具有涉及滑动子面板的炫酷过渡。
* [Swing Library](https://github.com/oliverwatkins/swing_library)：该库包含许多Java Swing框架所缺少的高级组件和布局管理器。
* [PanelMatic](https://github.com/codeworth-gh/PanelMatic)：PanelMatic是一个成熟的Swing工具库，旨在轻松创建复杂的高质量布局。

#### Swing选择器

* [LGoodDatePicker](https://github.com/LGoodDatePicker/LGoodDatePicker)：Java Swing日期选择器，易于使用、美观、功能强大且本地化。
* [Swing Datetime Picker](https://github.com/DJ-Raven/swing-datetime-picker)：使用Java Swing的简单日期时间选择器实现。
* [JDatePicker](https://github.com/JDatePicker/JDatePicker)：JDatePicker和JDatePanel是一组用于Java Swing应用程序的高级DatePicker控件。
* [ColorPick](https://github.com/dheid/colorpicker)：一个使用Java Swing的漂亮颜色选择器，包含可视颜色选择和用于手动输入RGB和HSB值的输入框。
* [JnaFileChooser](https://github.com/steos/jnafilechooser)：使用Windows原生对话框的Java文件选择器API。

#### Swing图表库

* [JFreeChart](https://github.com/jfree/jfreechart)：JFreeChart是一个适用于Java平台的综合免费图表库，可在客户端(JavaFX和Swing)或服务器端使用。
* [Orson Charts](https://github.com/jfree/orson-charts)：Orson Charts是一个用于Java平台的3D图表库，可以生成各种3D图表，用于客户端应用程序(JavaFX和Swing)和服务器端应用程序(导出为PDF、SVG、PNG和JPEG)。
* [Matplotlib4j](https://github.com/sh0nk/matplotlib4j)：Matplotlib4j是一个用于Java、Scala和Kotlin的简单绘图库。
* [Ardor3D](https://github.com/Renanse/Ardor3D)：Ardor3D是一个基于Java的免费、面向专业的开源3D图形引擎。
* [JGraphX](https://github.com/jgraph/jgraphx)：JGraphX是一个Java Swing图表库。
* [Three Kt](https://github.com/markaren/three.kt)：流行的Three.js 3D库的Kotlin/JVM端口。
* [Java-GTK](https://github.com/bailuk/java-gtk)：Java的GTK 4绑定。
* [Mil Sym Java](https://github.com/missioncommand/mil-sym-java)：Mil Sym Java是一组陈旧的Java库，已在美国陆军任务指挥软件中使用多年。

#### Swing测试库

* [SwingLibrary](https://github.com/robotframework/SwingLibrary)：SwingLibrary是一个用于测试Java Swing应用程序的机器人框架库。
* [AssertJ Swing](https://github.com/assertj/assertj-swing)：该项目提供了一个简单直观的API，用于Swing用户界面的功能测试。
* [FEST-Swing](https://github.com/alexruiz/fest-swing-1.x)：该项目为Swing用户界面的功能测试提供了一个简单直观的API，从而使测试变得紧凑、易于编写和阅读，就像规范一样。
* [UISpec4J](https://github.com/UISpec4J/UISpec4J)：UISpec4J是一个开源功能和/或单元测试库，用于基于Swing的Java应用程序。

#### JavaFX

* [JFoenix](https://github.com/sshahine/JFoenix)：JFoenix是一个开源Java库，它使用Java组件实现Google Material Design。
* [Tray Notification](https://github.com/PlusHaze/TrayNotification)：Tray Notification是JavaFX的一个库，为台式计算机添加了易于使用的托盘通知。
* [MaterialFX](https://github.com/palexdev/MaterialFX)：MaterialFX是一个开源Java库，为JavaFX提供材料设计组件。
* [FXLauncher](https://github.com/edvin/fxlauncher)：JavaFX应用程序的自动更新启动器。
* [Scene Builder](https://github.com/gluonhq/scenebuilder)：Scene Builder是一款拖放式UI设计工具，可实现快速桌面和移动应用程序开发。
* [RxJavaFX](https://github.com/ReactiveX/RxJavaFX)：RxJavaFX是一个轻量级库，用于将JavaFX事件转换为RxJava Observables/Flowables，反之亦然。
* [WorkbenchFX](https://github.com/dlsc-software-consulting-gmbh/WorkbenchFX)：用于JavaFX应用程序的轻量级RCP框架。
* [MVVMFX](https://github.com/sialcasa/mvvmFX)：MVVMFX是一个应用程序框架，它为你提供使用JavaFX实现MVVM模式所需的组件。
* [JWM](https://github.com/HumbleUI/JWM)：JWM是一个跨平台的Java窗口管理和操作系统集成库。
* [ReactFX](https://github.com/TomasMikula/ReactFX)：ReactFX是对JavaFX响应式编程技术的补充。
* [Substrate](https://github.com/gluonhq/substrate)：Gluon Substrate是一款将JavaFX客户端应用程序转换为桌面、移动和嵌入式设备的本机可执行文件的工具。
* [WebFX](https://github.com/webfx-project/webfx)：WebFX是一个由GWT提供支持的JavaFX应用程序转译器，它可以将JavaFX应用程序转换为传统的独立纯JavaScript Web应用程序(在浏览器中执行不需要插件或服务器)。
* [TestFX](https://github.com/TestFX/TestFX)：TestFX是一个用于简单、干净地测试JavaFX应用程序和组件的库。
* [TornadoFX](https://github.com/edvin/tornadofx)：TornadoFX是一个适用于Kotlin编程语言的小型轻量级JavaFX框架。
* [ValidatorFX](https://github.com/effad/ValidatorFX)：ValidatorFX是JavaFX的表单验证库。
* [EasyBind](https://github.com/TomasMikula/EasyBind)：EasyBind在创建自定义绑定时利用Lambda来减少样板文件，为Bindings.select*方法提供类型安全的替代方案，并向ObservableValue添加单子操作。
* [Gluon Maps](https://github.com/gluonhq/maps)：Gluon Maps提供了一种将OpenStreetMaps集成到JavaFX应用程序中的简单方法，它速度极快，并提供图层叠加、多个图块集等等。
* [LibRawFX](https://github.com/lanthale/LibRawFX)：集成适用于所有主要操作系统(Linux、Windows、OSX)的JavaFX的LibRaw库。
* [UpdateFX](https://github.com/vinumeris/updatefx)：UpdateFX是一个小型、简单的JavaFX应用程序自动在线更新框架。
* [ReduxFX](https://github.com/netopyr/reduxfx)：ReduxFX是一组库，使你能够在JavaFX应用程序中使用函数式响应式编程。
* [EasyFXML](https://github.com/Tristan971/EasyFXML)：EasyFXML是一组固执己见的工具，旨在简化健壮且模块化的JavaFX应用程序的开发。
* [JRebirth](https://github.com/JRebirth/JRebirth)：JRebirth是一个JavaFX应用程序框架。
* [JPro](https://www.jpro.one/)：用于在浏览器中运行Java的工具平台。
* [JavaFX-Plus](https://gitee.com/Biubiuyuyu/JavaFX-Plus)：简化开发步骤和提高开发效率的JavaFX框架。
* [SupernautFX](https://github.com/SupernautApp/SupernautFX)：Supernaut.FX是一个用于JavaFX应用程序的轻量级依赖注入框架。
* [JavaFX Weaver](https://github.com/rgielen/javafx-weaver)：JavaFX和FXML的依赖注入支持库。
* [AfterburnerFX](https://github.com/AdamBien/afterburner.fx)：AfterburnerFX是一个基于约定优于配置和依赖注入的简约JavaFX MVP框架。
* [Spring Boot JavaFX Support](https://github.com/roskenet/springboot-javafx-support)：Spring Boot与JavaFX集成。
* [LiveDirsFX](https://github.com/TomasMikula/LiveDirsFX)：LiveDirsFX是目录监视程序、目录树模型(用于TreeView)和简单的异步文件I/O工具的组合。
* [APX](https://github.com/othreecodes/APX)：用于制作MVC类型桌面应用程序的JavaFX库。
* [Basilisk](https://github.com/basilisk-fw/basilisk)：Basilisk是JVM的桌面/移动应用程序开发平台。
* [DataFX](https://github.com/guigarage/DataFX)：DataFX是一个旨在使JavaFX UI控件中检索、处理、填充、查看和编辑数据更加容易的项目。
* [JacpFX](https://github.com/JacpFX/JacpFX)：JacpFX项目是一个API，用于使用JavaFX、Spring和类似Actor的组件方法以MVC风格创建富客户端。
* [WellBehavedFX](https://github.com/FXMisc/WellBehavedFX)：该项目为JavaFX定义和覆盖事件处理程序(例如键盘快捷键)提供了更好的机制。
* [JavaFX WebView Debugger](https://github.com/vsch/Javafx-WebView-Debugger)：用于JavaFX WebView浏览器调试的全功能Google Chrome Dev Tools。
* [SynchronizeFX](https://github.com/zeiss-digital-innovation/SynchronizeFX)：SynchronizeFX支持在本地计算机和网络上的不同JVM之间进行属性绑定。
* [JideFX](https://github.com/jidesoft/jidefx-oss)：JideFX通用层是JavaFX平台各种扩展和实用程序的集合。

#### JavaFX主题库

* [AtlantaFX](https://github.com/mkpaz/atlantafx)：现代JavaFX CSS主题集合，带有附加控件。
* [NSMenuFX](https://github.com/0x4a616e/NSMenuFX)：一个用于自定义macOS菜单栏的简单库，为你的JavaFX应用程序提供更原生的LaF。
* [JFXtras Styles](https://github.com/JFXtras/jfxtras-styles)：Java、JavaFX主题或LaF，目前包含JMetro主题。
* [JBootX](https://github.com/dicolar/jbootx)：JavaFX Bootstrap主题库。
* [BootstrapFX](https://github.com/kordamp/bootstrapfx)：BootstrapFX是Twitter Bootstrap的部分移植，它主要提供与原始样式非常相似的CSS样式表，同时针对JavaFX独特的CSS风格进行定制。
* [Transit](https://github.com/dukke/Transit)：Transit是一个现代JavaFX主题/风格，可用于为应用程序提供不同的LaF。
* [SyntheticaFX](https://www.jyloo.com/syntheticafx/)：SyntheticaFX提供主要为桌面上的专业商业应用程序制作的主题和组件。

#### JavaFX样式库

* [ControlsFX](https://github.com/controlsfx/controlsfx)：ControlsFX是JavaFX的一个开源项目，旨在提供真正高质量的UI控件和其他工具来补充核心JavaFX发行版。
* [AnimateFX](https://github.com/Typhon0/AnimateFX)：包含70多个即用型JavaFX动画的库。
* [PreferencesFX](https://github.com/dlsc-software-consulting-gmbh/PreferencesFX)：用于轻松创建应用程序设置/首选项UI的框架。
* [CustomStage](https://github.com/Oshan96/CustomStage)：CustomStage是一个JavaFX Undecorated Stage，它包含默认JavaFX Decorated Stage的原生行为，并且完全可设计样式。
* [CSSFX](https://github.com/McFoggy/cssfx)：CSSFX通过在运行的应用程序中提供CSS重新加载功能来提高开发人员的工作效率。
* [Animated](https://github.com/iamgio/animated)：Animated引入了隐式动画，这是JavaFX中的一个全新概念，受到Flutter动画和运动小部件的强烈启发。
* [FXSkins](https://github.com/dukke/FXSkins)：现有JavaFX控件的新外观集合，这些皮肤将为应用程序中使用的控件添加更多功能，几乎不需要更改代码。
* [JSilhouette](https://github.com/kordamp/jsilhouette)：JSilhouette为Java应用程序提供了额外的形状，目前支持JavaFX。
* [MDFX](https://github.com/JPro-one/markdown-javafx-renderer)：MDFX是一个简单的JavaFX Markdown渲染器。
* [Actlist](https://github.com/actlist/actlist)：Actlist是一个实用平台，可以轻松简单地执行你自己的行为列表。
* [Tornado FXControls](https://github.com/edvin/tornadofx-controls)：JavaFX的CSS样式控件库。
* [FX-BorderlessScene](https://github.com/goxr3plus/FX-BorderlessScene)：未修饰的JavaFX场景，实现了移动、调整大小、最小化、最大化、关闭和Windows Aero Snap控件。
* [LitFX](https://github.com/Birdasaur/LitFX)：LitFX可以将其效果作为透明覆盖层应用到你的JavaFX GUI，这样效果就可以与你的各种节点进行交互，而无需先验了解你的布局。
* [TUIOFX](http://tuiofx.org/)：TUIOFX工具包旨在支持想要开发共享交互表面(如支持TUIO协议的多用户、多点触控桌面和墙壁)的Java开发人员。
* [Undecorator](https://github.com/in-sideFX/Undecorator)：使用自定义皮肤来装饰未装饰的JavaFX阶段。

#### JavaFX组件库

* [RichTextFX](https://github.com/FXMisc/RichTextFX)：RichTextFX为JavaFX提供了一个节省内存的TextArea，允许开发人员设置文本范围的样式、内联显示自定义对象(不再需要HTMLEditor)，并仅在必要时覆盖特定的默认行为。
* [JFXtras](https://github.com/JFXtras/jfxtras)：JavaFX的支持库，包含工具程序类、扩展布局、控件和其他有趣的小部件。
* [DashboardFx](https://github.com/gleidsonmt/DashboardFx)：该项目是为JavaFX创建的自定义组件集的一部分。
* [GemsFX](https://github.com/dlsc-software-consulting-gmbh/GemsFX)：用于渲染SVG图像文件的控件，利用jsvg库，即使在缩放时也能确保高清质量。
* [CalendarFX](https://github.com/dlsc-software-consulting-gmbh/CalendarFX)：用于创建基于JavaFX的复杂日历视图的Java框架。
* [RXControls](https://github.com/leewyatt/rxcontrols)：RXControls是一个JavaFX自定义组件库，密码可见组件、轮播图组件、动态按钮组件等。
* [PDFViewFX](https://github.com/dlsc-software-consulting-gmbh/PDFViewFX)：允许应用程序显示PDF文件的自定义控件。
* [UnitFX](https://github.com/dlsc-software-consulting-gmbh/UnitFX)：UnitFX是一个轻量级框架，用于创建基于UOM的文本字段输入控件。
* [AnchorFX](https://github.com/alexbodogit/AnchorFX)：AnchorFX是JavaFX的免费开源库，用于创建具有Dock功能的图形界面。
* [PhoneNumberFX](https://github.com/dlsc-software-consulting-gmbh/PhoneNumberFX)：该仓库包含一个控件，用于输入世界上任何国家/地区的有效电话号码。
* [Monocle](https://github.com/TestFX/Monocle)：Monocle是针对嵌入式系统的JavaFX Glass窗口组件的实现。
* [JavaFXSVG](https://github.com/codecentric/javafxsvg)：用于向JavaFX添加SVG支持，允许像任何其他图像类型一样使用SVG图形。
* [FormsFX](https://github.com/dlsc-software-consulting-gmbh/FormsFX)：用于轻松创建JavaFX UI表单的框架。
* [FXParallax](https://github.com/dukke/FXParallax)：用于为Java添加视差效果的控件。
* [DockFX](https://github.com/RobertBColton/DockFX)：该库是为了填补JavaFX RIA平台中可用的Dock框架的空白，其目的是为你提供一个功能齐全的Dock库。
* [FxDock](https://github.com/andy-goryachev/FxDock)：JavaFX Dock框架。
* [UndoFX](https://github.com/FXMisc/UndoFX)：UndoFX是JavaFX的通用撤消管理器。
* [FXRibbon](https://github.com/dukke/FXRibbon)：Java的Ribbon控件，使用JavaFX框架，基于Microsoft Ribbon。
* [DesktopPaneFX](https://github.com/kordamp/desktoppanefx)：DesktopPaneFX是Swing JDesktopPane的JavaFX版本，它可以用作类似于JInternalFrames的单个“子”容器。
* [KeyboardFX](https://github.com/dlsc-software-consulting-gmbh/KeyboardFX)：JavaFX应用程序的屏幕键盘。
* [TiwulFX](https://github.com/panemu/tiwulfx-dock)：TiwulFX-Dock提供增强的JavaFX TabPane，支持选项卡重新排序、分离和对接。
* [FroXty](https://github.com/iAmGio/froxty)：FroXty是一个JavaFX库，可以轻松复制著名的iOS半透明效果。
* [GestureFX](https://github.com/tom91136/GestureFX)：JavaFX的轻量级捏合缩放窗格。
* [FXFileChooser](https://github.com/Oliver-Loeffler/FXFileChooser)：自定义JavaFX文件选择器，允许快速手动过滤，允许添加路径谓词作为过滤器。
* [FXTaskbarProgressBar](https://github.com/Dansoftowner/FXTaskbarProgressBar)：JavaFX的一个库，使你能够在Windows任务栏上显示进度。
* [Grid](https://github.com/manuel-mauky/Grid)：Grid是一个JavaFX(8)组件，用于基于方格网格的不同类型的小游戏，例如国际象棋或数独。
* [RichTextArea](https://github.com/gluonhq/rich-text-area)：RichTextArea是一种文本输入控件，它提供富文本功能以及表情符号和非文本对象(如图像、表格和超链接)。
* [BentoFX](https://github.com/Col-E/BentoFX)：JavaFX的Dock系统。
* [FXUI](https://gitee.com/lichenfei_fei/chenfei-fxui)：JavaFX常见自定义组件。

#### JavaFX图表库

* [JFreeChart](https://github.com/jfree/jfreechart)：JFreeChart是一个适用于Java平台的综合免费图表库，可在客户端(JavaFX和Swing)或服务器端使用。
* [TilesFX](https://github.com/HanSolo/tilesfx)：包含可用于仪表板的图块的JavaFX库。
* [Medusa](https://github.com/HanSolo/medusa)：用于仪表的JavaFX库，该项目的主要重点是提供可以多种方式配置的仪表。
* [Charts](https://github.com/HanSolo/charts)：JavaFX中的科学图表库。
* [ChartFX](https://github.com/fair-acc/chart-fx)：ChartFX是GSI为FAIR开发的一个科学图表库，专注于以25Hz更新速率对数字信号处理应用中常见的具有数万到500万个数据点的数据集进行性能优化的实时数据可视化。
* [GMapsFX](https://github.com/dlsc-software-consulting-gmbh/GMapsFX)：纯JavaFX API，允许你将Google地图添加到JavaFX应用程序，而无需与底层Google地图JavaScript API交互。
* [FXForm2](https://github.com/dooApp/FXForm2)：FXForm2是一个提供自动JavaFX表单生成的库。
* [Piccolo2D](https://github.com/piccolo2d/piccolo2d.java)：Piccolo2D是一种在Java创建强大、功能齐全的图形应用程序的方式，具有诸如缩放和多重表示等引人注目的功能。
* [VWorkflows](https://github.com/miho/VWorkflows)：用于构建特定领域的可视化编程环境的交互式流/图形可视化。
* [FXGraphics2D](https://github.com/jfree/fxgraphics2d)：FXGraphics2D是针对JavaFX Canvas的Java Graphics2D API的实现。
* [FlexGanttFX](https://dlsc.com/products/flexganttfx)：FlexGanttFX是目前可用于Java的最先进的基于JavaFX的甘特图框架。
* [FXyz](https://github.com/FXyz/FXyz)：JavaFX 3D可视化和组件库。
* [yFiles](https://www.yworks.com/products/yfiles-for-java)：yFiles是业界领先的图形可视化、编辑和分析软件库。
* [JView](https://www.perforce.com/products/visualization)：JViews提供先进的行业特定可视化。
* [Countries](https://github.com/HanSolo/countries)：Countries是一个JavaFX库，其中包含有关国家/地区的信息，例如形状、GPS坐标、城市和机场。
* [JFreePDF](https://github.com/jfree/jfreepdf)：JFreePDF是Java平台的一个库模块，允许你使用标准Java2D绘图API(Graphics2D)以Adobe的可移植文档格式(PDF)创建内容。
* [Graph Editor](https://github.com/eckig/graph-editor)：用于在JavaFX中创建和编辑类似图形的图表的库。
* [Eclipse SWTChart](https://github.com/eclipse/swtchart)：SWTChart允许创建不同类型的图表。
* [SmartGraph](https://github.com/brunomnsilva/JavaFXSmartGraph)：该项目提供了一个通用JavaFX图形可视化库，可以通过力导向算法实时自动排列顶点的位置。
* [JavaFX D3](https://github.com/stefaneidelloth/javafx-d3)：提供用于将JavaScript库d3.js与JavaFX应用程序使用的Java API。
* [Orson Charts](https://github.com/jfree/orson-charts)：Orson Charts是一个用于Java平台的3D图表库，可以生成各种3D图表，用于客户端应用程序(JavaFX和Swing)和服务器端应用程序(导出为PDF、SVG、PNG和JPEG)。
* [JavaFxDataviewer](https://github.com/jasrodis/javafx-dataviewer-wrapper)：JavaFxDataviewer是JavaFX的开源数据可视化工具。
* [JMathAnim](https://github.com/davidgutierrezrubio/jmathanim)：JMathAnim是一个用Java编写的库，旨在简化数学动画的制作过程。

#### JavaFX图标库

* [Ikonli](https://github.com/kordamp/ikonli)：Ikonli提供可在Java应用程序中使用的图标包，目前支持Swing和JavaFX UI工具包。
* [FXTrayIcon](https://github.com/dustinkredmond/FXTrayIcon)：用于JavaFX应用程序的库，可以更轻松地添加系统托盘图标。
* [jIconFont-Swing](https://github.com/jIconFont/jiconfont-swing)：jIconFont-Swing是一个API，用于提供由任何IconFont生成的图标。
* [Icon Generator](https://github.com/sshtools/icon-generator)：一个简单的库，用于在Java中生成Java2D(Swing/AWT)、JavaFX和SWT图标。

#### JavaFX布局库

* [MiGLayout](https://github.com/mikaelgrev/miglayout)：MigLayout是一个功能极其丰富的JavaFX/SWT/Swing布局管理器，它使布局问题变得微不足道。
* [Flowless](https://github.com/FXMisc/Flowless)：JavaFX的高效VirtualFlow，VirtualFlow是一个布局容器，可在垂直或水平流中布局单元格。
* [FlexBoxFX](https://github.com/onexip/FlexBoxFX)：FlexBoxFX是CSS3弹性框布局管理器的JavaFX实现。

#### JavaFX渲染库

* [DriftFX](https://github.com/eclipse-efx/efxclipse-drift)：DriftFX允许你将任何OpenGL内容直接渲染到JavaFX节点中。
* [OpenGLFX](https://github.com/husker-dev/openglfx)：该库为JavaFX添加了新元素，用于使用LWJGL、JOGL、LWJGL2或LibGDX渲染OpenGL图形。
* [NativeFX](https://github.com/miho/NativeFX)：JavaFX的原生渲染集成。
* [JFXShader](https://github.com/Teragam/JFXShader)：允许在JavaFX中使用OpenGL(GLSL)或DirectX(HLSL)自定义效果着色器。

#### 浏览器

* [WebBrowser](https://github.com/goxr3plus/JavaFX-Web-Browser)：用Java和JavaFX制作的Web浏览器。
* [Wolvic XR Browser](https://github.com/Igalia/wolvic)：Wolvic是一个开源浏览器，包括XR构建的沉浸式游戏、视频和环境。
* [JCEF](https://github.com/chromiumembedded/java-cef)：JCEF是一个简单的框架，用于使用Java编程语言将基于Chromium的浏览器嵌入到其他应用程序中。
* [GNGR](https://github.com/gngrOrg/gngr)：这是一款支持隐私的新型跨平台浏览器，GNGR是纯Java的Web标准的独立实现。
* [JxBrowser](https://teamdev.com/jxbrowser/)：将Chromium Web浏览器添加到你的Java应用程序中。
* [Gophie](https://github.com/jankammerath/gophie)：Gophie是“The Internet Gopher”的现代图形跨平台客户端或浏览器。
* [Lobo Evolution](https://github.com/LoboEvolution/LoboEvolution)：Lobo Evolution是一个可扩展的全Java Web浏览器和RIA平台。
* [Easy Browser](https://gitee.com/fhs-opensource/easy-browser)：Java开源浏览器，基于JxBrowser实现，已经包含了绝大多数的浏览器基础功能。
* [Webicity](https://github.com/WebicityBrowser/Webicity)：Webicity是一个简单的Web浏览器。
* [JFX-Browser](https://github.com/badarshahzad/Jfx-Browser)：JFX Browser是一个多选项卡浏览器。

#### JavaFX小工具

* [XPipe](https://github.com/xpipe-io/xpipe)：XPipe是一种新型的shell连接集线器和远程文件管理器，允许你从本地计算机访问整个服务器基础架构。
* [JabRef](https://github.com/JabRef/jabref)：JabRef是一个开源、跨平台的引文和引用管理工具。
* [AsciidocFX](https://github.com/asciidocfx/AsciidocFX)：AsciidocFX是一款书籍/文档编辑工具，用户可以创建PDF、Epub、Mobi和HTML等形式的书籍/文档。
* [PDF4Teachers](https://github.com/ClementGre/PDF4Teachers)：PDF4Teachers专为教师设计，它允许你以出色的工作效率对PDF返回的评估进行注释。
* [FoFa Viewer](https://github.com/wgpsec/fofa_viewer)：FoFa Viewer是一个用JavaFX编写的用户友好的FOFA客户端，由狼组安全团队开源。
* [Markdown Writer FX](https://github.com/JFormDesigner/markdown-writer-fx)：用JavaFX编写的开源Markdown编辑器。
* [ToolsFx](https://github.com/Leon406/ToolsFx)：基于Kotlin、TornadoFX的跨平台密码学工具箱。
* [Phoenicis](https://github.com/PhoenicisOrg/phoenicis)：Phoenicis PlayOnLinux是一款软件，可让你轻松安装和使用大量专为Windows运行的游戏和应用程序。
* [XR3Player](https://github.com/goxr3plus/XR3Player)：XR3Player是一个先进的JavaFX媒体播放器。
* [Artillery](https://github.com/Weik1/Artillery)：Artillery是一个插件化Java漏洞扫描器。
* [DrawingBotV3](https://github.com/SonarSonic/DrawingBotV3)：DrawingBotV3是一款用于将图像转换为风格化绘图的软件。
* [LaTeXDraw](https://github.com/latexdraw/latexdraw)：LaTeXDraw是LaTeX的图形绘图编辑器。
* [JDKMon](https://github.com/HanSolo/JDKMon)：JDKMon是一个用JavaFX编写的小工具，它会尝试检测你计算机上安装的所有JDK，并通知你发现的每个OpenJDK发行版的新更新和漏洞。
* [MQTT.fx](https://www.softblade.de/)：MQTT.fx是开发和生产中测试物联网路由的工具。
* [CorreoMQTT](https://github.com/EXXETA/correomqtt)：CorreoMQTT是使用HiveMQ客户端库的现代图形MQTT客户端。
* [TerminalFX](https://github.com/javaterminal/TerminalFX)：TerminalFX是一个用JavaFX 18编写的终端模拟器应用程序。
* [VocabHunter](https://github.com/VocabHunter/VocabHunter)：VocabHunter是一个帮助外语学习者的系统。
* [Quelea](https://github.com/quelea-projection/Quelea)：Quelea是适用于教堂的开源投影软件。
* [Image2LaTeX](https://github.com/blaisewang/img2latex-mathpix)：Image2LaTeX提供将图像转换为某些LaTeX方程格式和OCR的核心功能。
* [TRex Stateless GUI](https://github.com/cisco-system-traffic-generator/trex-stateless-gui)：TRex Stateless GUI应用程序为TRex提供图形用户界面，由Cisco开发。
* [FakeImageDetection](https://github.com/afsalashyana/FakeImageDetection)：FakeImageDetection的目标是识别假图像(经过数字更改的图像)。
* [OwlPlug](https://github.com/DropSnorz/OwlPlug)：OwlPlug是一个音频插件管理器，用于在Windows、MacOS和Linux上管理VST/AU/LV2插件的小工具。
* [LogFX](https://github.com/renatoathaydes/LogFX)：LogFX是一个多平台、免费和开源的日志查看器，旨在处理非常大的文件而不影响性能。
* [Object Graph Visualizer](https://github.com/Nurtak/ObjectGraphVisualization)：Object Graph Visualizer是一个主要用于计算机科学课程的工具，旨在帮助新学生理解面向对象的范式和模式。
  。
* [Flow](https://github.com/eclab/flow)：Flow是一款完全模块化的多音色和复调加法软件合成器，由乔治梅森大学开源。
* [Scenic View](https://github.com/JonathanGiles/scenic-view)：Scenic View是一个JavaFX应用程序，旨在让你轻松了解应用程序场景图的当前状态，并且还可以轻松操作场景图的属性，而无需继续编辑代码。
* [Usagi](https://github.com/OHDSI/Usagi)：Usagi是一个帮助创建编码系统和词汇标准概念之间映射的应用程序，由OHDSI开源。
* [Trinity](https://github.com/Birdasaur/Trinity)：Trinity提供性能分析和XAI工具，非常适合深度学习系统或执行复杂分类或解码的其他模型。
* [Board Instrumentation Framework](https://github.com/intel/Board-Instrumentation-Framework)：这个项目允许你灵活地对几乎任何你想要的内容进行仪表化和图形化显示，由Intel开发。

#### GUI程序

* [LanguageTool](https://github.com/languagetool-org/languagetool)：LanguageTool是一款开源校对软件，适用于英语、西班牙语、法语、德语、葡萄牙语、波兰语、荷兰语和其他20多种语言，它可以发现许多简单的拼写检查器无法检测到的错误。
* [FinalShell](https://www.hostbuf.com/)：FinalShell是一款一体化的服务器，网络管理软件。
* [SoapUI](https://github.com/SmartBear/soapui)：SoapUI是一个免费、开源的跨平台API和Web Service功能测试解决方案。
* [Protege](https://github.com/protegeproject/protege)：Protege是一个免费的开源本体编辑器，支持最新的OWL 2.0标准，由斯坦福开发。
* [Processing](https://github.com/processing/processing)：Processing是一本灵活的软件速写本，也是一种用于学习编码的语言。
* [JISA](https://github.com/OE-FET/JISA)：JISA是一个用Java编写的库，旨在用作为常见实验室仪器创建实验控制系统的方法，由剑桥大学开源。
* [MIST](https://github.com/usnistgov/MIST)：MIST是美国国家标准与技术研究所开发的显微图像拼接应用程序。
* [Piped](https://github.com/TeamPiped/Piped)：Piped是另一种隐私友好的YouTube前端，设计高效。
* [SageTV](https://github.com/google/sagetv)：SageTV是一个跨平台联网DVR和媒体管理系统，由Google开发。
* [Sonarqube](https://github.com/SonarSource/sonarqube)：SonarQube是一个开源的代码质量管理系统。
* [TempestSDR](https://github.com/martinmarinov/TempestSDR)：该项目是一个软件工具包，用于使用软件定义无线电(SDR)接收器远程窃听视频监视器。
* [SmartGit](https://www.syntevo.com/smartgit/)：SmartGit是一个Git GUI客户端，支持GitHub、BitBucket、GitLab拉取请求和评论。
* [Burp Suite](https://portswigger.net/burp)：Burp Suite是一个用于测试网络应用程序安全性的图形化工具，由PortSwigger开发。
* [AnyLogic](https://www.anylogic.com/)：AnyLogic是一套结合多种模拟(仿真)理论的建模开发工具。
* [yEd](https://www.yworks.com/products/yed)：yEd是一款功能强大的桌面应用程序，可用于快速有效地生成高质量图表，由yWorks开发。
* [Uppaal](https://uppaal.org/)：Uppaal是一个集成工具环境，用于对实时系统进行建模、验证和确认，由乌普萨拉大学和奥尔堡大学开源。
* [NetLogo](https://github.com/NetLogo/NetLogo)：NetLogo是一个多智能体可编程建模环境，由西北大学开源。
* [MooTool](https://github.com/rememberber/MooTool)：开发者常备小工具。
* [SDRTrunk](https://github.com/DSheirer/sdrtrunk)：SDRTrunk是一个跨平台Java应用程序，用于使用软件定义无线电(SDR)解码、监控、记录和流式传输集群移动和相关无线电协议。
* [Proxyee Down](https://github.com/proxyee-down-org/proxyee-down)：Proxyee Down是一款开源的免费HTTP高速下载器，底层使用Netty开发，支持自定义HTTP请求下载且支持扩展功能，可以通过安装扩展实现特殊的下载需求。
* [Cyberduck](https://github.com/iterate-ch/cyberduck)：Cyberduck是一款适用于Mac和Windows的自由FTP、SFTP、WebDAV、Amazon S3、Backblaze B2、Microsoft Azure和OneDrive以及OpenStack Swift文件传输客户端。
* [STM32CubeMX](https://www.st.com/en/development-tools/stm32cubemx.html)：ST公司推出的一种自动创建单片机工程及初始化代码的工具。
* [BIMserver](https://github.com/opensourceBIM/BIMserver)：BIMserver使你能够存储和管理建筑(或其他建筑相关)项目的信息，由荷兰国家应用科学院和埃因霍芬理工大学开发。
* [ImageJ](https://imagej.net/ij/index.html)：ImageJ是一个基于Java的公共图像处理软件，由美国国立卫生研究院开发。
* [EtherPad](https://github.com/ether/pad)：Etherpad Lite是一种更简单、更快、更轻的协作编辑解决方案。
* [Uncle Novel](https://github.com/uncle-novel/uncle-novel)：一个桌面端应用，支持 MacOS/Windows，提供了全网小说的转码阅读功能。
* [OpenSim](https://github.com/opensim-org/opensim-gui)：OpenSim是一款软件，可让用户开发肌肉骨骼结构模型并创建运动动态模拟，由斯坦福开源。
* [OpenAPC](http://www.openapc.com/)：OpenAPC是一种开源APC(高级过程控制)解决方案，它具有高度灵活性和可配置性，涵盖了从家庭控制到工业自动化的各种自动化、可视化和过程控制任务。
* [Wordfast](https://www.wordfast.com/)：Wordfast是一款翻译记忆软件，它为自由译者、语言服务供应者与跨国公司提供了翻译记忆独立平台的解决方案。
* [OpenRocket](https://github.com/openrocket/openrocket)：OpenRocket是一款免费、功能齐全的模型火箭模拟器，可让你在实际建造和飞行火箭之前设计和模拟火箭。
* [Vivado](https://www.xilinx.com/products/design-tools/vivado.html)：Vivado是赛灵思开发的用于HDL设计的合成和分析的软件套件，具有用于片上系统开发和高级综合的附加功能。
* [Citespace](https://citespace.podia.com/)：CiteSpace是一个免费的Java应用程序，用于可视化和分析科学文献中的趋势和模式。
* [OpenPnP](https://github.com/openpnp/openpnp)：OpenPnP是一种开源SMT拾放系统，包括可立即运行的软件以及可以构建和修改的硬件设计。
* [ArcTime](https://arctime.org/)：ArcTime是一个易用、强大、高效的字幕制作软件，由南京亿铭科技开发。
* [RapidWright](https://github.com/Xilinx/RapidWright)：RapidWright是AMD的一个开源项目，它通过DCP文件为Vivado提供了一个新的桥梁。
* [Unidata AWIPS](https://github.com/Unidata/awips2)：AWIPS是一个气象软件包，它用于解码、显示和分析数据，最初由雷神公司为国家气象局(NWS)开发。
* [ThinkPHP](https://github.com/Lotus6/ThinkphpGUI)：Thinkphp漏洞利用工具，支持各版本TP漏洞检测、命令执行、getshell。
* [JMRI](https://github.com/JMRI/JMRI)：JMRI是一个模型铁路数字指挥与控制软件。
* [IPED](https://github.com/sepinf-inc/IPED)：IPED是一款开源软件，可用于处理和分析数字证据，由巴西联邦警察局的数字取证专家开发。
* [NodeBox](https://github.com/nodebox/nodebox)：NodeBox系列工具让你能够按照自己想要的方式创建生成设计，由布鲁塞尔圣卢卡斯大学学院开发。
* [FigTree](https://github.com/rambaut/figtree)：FigTree被设计为系统发育树的图形查看器和用于生成可发表图表的程序。
* [ArtiSynth](https://github.com/artisynth/artisynth_core)：ArtiSynth是一个3D机械建模系统，支持多体和有限元(FEM)模型以及接触和约束的组合模拟，由不列颠哥伦比亚大学开源。
* [PEmbroider](https://github.com/CreativeInquiry/PEmbroider)：PEmbroider是一个使用Processing进行计算刺绣的开源库，由CMU开源。
* [Particle Life](https://github.com/tom-mohr/particle-life-app)：Particle Life的GUI，这是一个显示逼真行为的粒子系统。
* [jExifToolGUI](https://github.com/hvdwolf/jExifToolGUI)：jExifToolGUI是一个多平台Java/Swing图形前端，由Phil Harvey开发，用于优秀的命令行ExifTool应用程序。
* [WePush](https://github.com/rememberber/WePush)：专注批量推送的小而美的工具，目前支持：模板消息-公众号、模板消息-小程序、微信客服消息等。
* [BlobSaver](https://github.com/airsquared/blobsaver)：用于自动保存SHSH blob的跨平台GUI和CLI应用程序。
* [BiglyBT](https://github.com/BiglySoftware/BiglyBT)：基于Azureus开源项目的功能齐全的Bittorrent客户端。
* [Gephi](https://github.com/gephi/gephi/)：Gephi是适用于各种图形和网络的领先可视化和探索软件。
* [Holer](https://github.com/wisdom-projects/holer)：Holer是一个将局域网中的应用映射到公网访问的端口映射软件，支持转发基于TCP协议的报文。
* [MooInfo](https://github.com/rememberber/MooInfo)：OSHI的可视化实现，用于查看有关系统和硬件的信息。
* [Freenet](https://github.com/hyphanet/fred)：Freenet是一个抗审查通信和发布平台，它是一种点对点软件，提供分布式、加密、去中心化的数据存储，起源于爱丁堡大学。
* [RuneLite](https://github.com/runelite/runelite)：RuneLite是一个免费、开源的OldSchool RuneScape客户端。
* [Chatty](https://github.com/chatty/chatty)：Chatty是一款用Java编写的Twitch桌面聊天客户端，具有许多Twitch特定功能。
* [Zettelkasten](https://github.com/Zettelkasten-Team/Zettelkasten)：Zettelkasten是一款知识管理工具。
* [Moneydance](https://infinitekind.com/moneydance)：Moneydance是一款易于使用且功能齐全的个人理财应用程序。
* [Bits N Picas](https://github.com/kreativekorp/bitsnpicas)：Bits N Picas是一组用于创建和转换位图和表情符号字体的工具。
* [Bad Peggy](https://www.malavida.com/en/soft/bad-peggy/)：Bad Peggy会分析你的文件夹以查找损坏的JPEG图像。
* [TeXtidote](https://github.com/sylvainhalle/textidote)：LaTeX文档和其他格式的修正工具。
* [Vuze](https://www.vuze.com/)：Vuze是一个用Java编写的BitTorrent客户端，且支持I2P和Tor匿名网络协议。
* [JDemetra+](https://github.com/jdemetra/jdemetra-app)：JDemetra+是比利时国家银行(NBB)与德意志联邦银行和欧盟统计局根据欧洲统计系统(ESS)指南合作开发的季节性调整(SA)新工具。
* [Bitwig Studio](https://www.bitwig.com/)：Bitwig Studio是由Bitwig公司开发的专有数字音频工作站。
* [Archi](https://github.com/archimatetool/archi)：Archi是一款免费、开源、跨平台的工具和编辑器，用于创建ArchiMate模型。
* [JLearnIt](https://www.jlearnit.com/)：JLearnIt是一款免费的多语言词典，按类别分类，帮助你循序渐进地学习其他语言的词汇。
* [TuxGuitar](https://github.com/helge17/tuxguitar)：TuxGuitar是一个用Java编写的开源多轨指法谱编辑器和播放器。
* [DocFetcher](https://github.com/docfetcher/DocFetcher)：DocFetcher是一个开源桌面搜索应用程序，它允许你搜索计算机上的文件内容。
* [PIPE](https://github.com/sarahtattersall/PIPE)：平台独立的Petri网编辑器。
* [MuCommander](https://github.com/mucommander/mucommander)：MuCommander是一款轻量级、跨平台的文件管理器，具有双窗格界面。
* [Open Visual Traceroute](https://github.com/leolewis/openvisualtraceroute)：Open Visual Traceroute是一款提供视觉化的路由追踪工具。
* [Plot Digitizer](https://plotdigitizer.sourceforge.net/)：Plot Digitizer是一个Java程序，用于对功能数据的扫描图进行数字化。
* [RipMe](https://github.com/RipMeApp/ripme)：RipMe是一个适用于各种网站的专辑翻录工具。
* [FileBot](https://www.filebot.net/)：FileBot是重命名和组织电影、电视节目和动漫的终极工具。
* [KSar](https://github.com/vlsi/ksar)：KSar是一个sar图形工具，可以绘制Linux、Mac和Solaris sar输出的图形。
* [Pixelitor](https://github.com/lbalazscs/Pixelitor)：Pixelitor是一个高级Java图像编辑器，具有图层、图层蒙版、文本图层、110多个图像滤镜和颜色调整、多重撤消等。
* [Archimedes](https://github.com/ArchimedesCAD/Archimedes)：Archimedes是一款免费开源CAD软件。
* [BT747](https://www.bt747.org/)：BT747是一款基于MTK芯片组控制GPS数据记录器的应用程序。
* [B4J](https://github.com/AnywhereSoftware/B4J)：B4J是一款100%免费的桌面、服务器和物联网解决方案开发工具，由Anywhere软件公司开发。
* [JaamSim](https://github.com/jaamsim/jaamsim)：JaamSim是自2002年开发的基于Java的离散事件仿真环境，由Ausenco开源。
* [TreeForm](https://github.com/frekky/TreeForm)：TreeForm语法树绘图软件是一个语言语法/语义树绘图编辑器。
* [FullSync](https://github.com/fullsync/fullsync)：FullSync是一个功能强大的工具，可帮助你保持各种数据的多个副本同步。
* [Shutter Encoder](https://github.com/paulpacifico/shutter-encoder)：Shutter Encoder是最好的视频转换器软件之一，它还可以处理图像和音频。
* [Spectrum Analyzer](https://github.com/pavsa/hackrf-spectrum-analyzer)：适用于Windows/Linux的hackrf_sweep频谱分析仪GUI。
* [Janelia Workstation](https://github.com/JaneliaSciComp/workstation)：Janelia Workstation是一个神经科学发现平台，用于处理、分析、注释和共享大规模3D显微镜数据，由霍华德休斯医学研究所开源。
* [Java Modelling Tools](https://jmt.sourceforge.net/)：JMT是由米兰理工大学和伦敦帝国理工学院开发的一套应用程序，旨在为性能评估、使用分析和模拟技术的系统建模、容量规划和工作负载特征研究提供全面的框架。
* [Whole Platform](https://github.com/wholeplatform/whole)：Whole Platform是一种用于工程软件生产的开源技术。
* [MyTourbook](https://github.com/mytourbook/mytourbook)：MyTourbook是一款免费软件，用于可视化和分析由GPS设备、自行车或运动电脑和测力计记录的行程。
* [Virtual Satellite](https://github.com/virtualsatellite/VirtualSatellite4-Core)：Virtual Satellite是一款DLR开源软件，用于基于模型的系统工程MBSE。
* [Sweet Home 3D](http://www.sweethome3d.com/)：Sweet Home 3D是一款免费的室内设计应用程序，它可以帮助你绘制房屋平面图、在其上布置家具并以3D形式查看结果。
* [Autopsy](https://github.com/sleuthkit/autopsy)：Autopsy是Sleuth Kit和其他开源数字取证工具的图形界面。
* [Rachota](https://rachota.sourceforge.net/en/index.html)：Rachota是一款用于跟踪不同项目时间的便携式应用程序。
* [JMSToolBox](https://github.com/jmstoolbox/jmstoolbox)：JMSToolBox是一个“通用”JMS客户端，能够以一致的方式与市场上数量最多的队列管理器/队列提供程序进行交互。
* [MSPaintIDE](https://github.com/MSPaintIDE/MSPaintIDE)：这个应用程序给MS Paint带来了提升，可以让MS Paint突出显示、编译和执行代码。
* [JDiskReport](https://www.jgoodies.com/freeware/jdiskreport/)：JDiskReport使你能够了解文件和目录在磁盘驱动器上占用了多少空间，并帮助你找到过时的文件和文件夹。
* [Freerouting](https://github.com/freerouting/freerouting)：Freerouting是一款先进的自动布线器，适用于所有支持标准Specctra或Electra DSN接口的PCB程序。
* [Paintera](https://github.com/saalfeldlab/paintera)：Paintera是一种通用可视化工具，用于3D体积数据和分割/重建中的校对，主要关注连接组学中电子显微照片的神经元重建，由Saalfeld实验室开源。
* [Mars Simulation](https://github.com/mars-sim/mars-sim)：Mars Simulation是一个基于Java的开源项目，它模拟火星上最初定居者的任务操作和活动，其建模和模拟细节的保真度比大多数经典模拟游戏更高。
* [ELamX2](https://github.com/AndiMb/eLamX2)：ELamX²是一款用Java编写的开源复合计算器，由德累斯顿工业大学航空航天工程学院飞机工程系主任开发。
* [Dayon](https://github.com/retgal/dayon)：Dayon是一种易于使用、跨平台的远程桌面协助解决方案。
* [Autoplot](https://autoplot.org/)：Autoplot是一款用于网络数据的交互式浏览器，由NASA等组织开发。
* [Underscore Backup](https://underscorebackup.com/)：Underscore Backup是一款数据备份和恢复解决方案，主要用于Windows和Mac操作系统，提供了一种安全、高效的方式来备份用户的数据。

#### IDE

* [IntelliJ IDEA](https://github.com/JetBrains/intellij-community)：IntelliJ IDEA是领先的Java和Kotlin IDE，由JetBrains开发。
* [Eclipse](https://github.com/eclipse-platform)：Eclipse是一个开源、基于Java的可扩展开发平台，由IBM开发。
* [Visual Studio Code](https://code.visualstudio.com)：Visual Studio Code是Microsoft开源的一个轻量级但功能强大的源代码编辑器，也支持作为IDE开发Java。
* [Android Studio](https://developer.android.com/studio)：Android Studio是用于开发Android应用的Google官方IDE，基于Intellij引擎。
* [Apache NetBeans](https://github.com/apache/netbeans)：NetBeans是一个开源开发环境、工具平台和应用程序框架，最初由Oracle开发。
* [JetBrains Fleet](https://www.jetbrains.com/fleet)：Fleet是JetBrains公司推出的一款下一代集成开发环境，使用Kotlin开发。
* [MyEclipse](https://www.genuitec.com/products/myeclipse)：MyEclipse是一个基于Eclipse平台构建的专有Java IDE，由Genuitec软件公司提供。
* [Spring Tools](https://github.com/spring-projects/sts4)：Spring官方出品的基于Eclipse的Java IDE。
* [Light Table](https://github.com/LightTable/LightTable)：Light Table是一款新一代代码编辑器，它能通过即时反馈将你与你的创作紧密联系起来。
* [JDeveloper](https://www.oracle.com/application-development/technologies/jdeveloper.html)：Oracle JDeveloper是一个免费的集成开发环境，可简化基于Java的应用程序的开发。
* [DevEco Studio](https://developer.huawei.com/consumer/cn/deveco-studio/)：DevEco Studio基于IntelliJ IDEA开源版本打造，为运行在HarmonyOS系统上的应用和元服务提供一站式的开发平台，由华为开发。
* [JBoss Developer Studio](https://devstudio.jboss.com/updates/)：JBoss Developer Studio是由RedHat和Exadel创建并开发的IDE。
* [HBuilder](https://www.dcloud.io/)：HBuilder是DCloud推出的一款支持HTML 5的Web开发IDE，本身由Java编写。
* [BlueJ](https://github.com/k-pet-group/BlueJ-Greenfoot)：BlueJ是Java编程语言的IDE，主要用于教育目的，但也适合小规模软件开发，最初由莫纳什大学开源，目前由伦敦国王学院维护。
* [GigaIDE](https://gitverse.ru/features/gigaide/)：GigaIDE是由俄罗斯公司研发的免费Spring IDE，旨在为开发者提供高效的Spring应用开发体验。
* [JBuilder](https://borland-jbuilder.software.informer.com/)：JBuilder是Borland公司出品的Java集成编程环境，有不同功能程度的多个版本。
* [Consulo](https://github.com/consulo/consulo)：Consulo是一个多语言IDE，基于Intellij引擎。
* [Codenvy](https://github.com/codenvy/codenvy)：Codenvy是一个基于云的IDE，支持Java和许多其他语言，由RedHat维护。
* [RapidClipse](https://rapidclipse.com/)：RapidClipse是一个免费的Eclipse发行版，用于使用Java进行快速跨平台开发，由XDEV软件公司提供。
* [Greenfoot](https://www.greenfoot.org/home)：Greenfoot是使用Java或Stride的集成开发环境，主要用于高中和本科生的教育目的，由坎特伯雷肯特大学开源。
* [DrJava](http://www.drjava.org/)：DrJava是一款轻量级的Java编程语言IDE，主要面向初学者设计，由莱斯大学开源。
* [AIDE](https://www.android-ide.com/)：AIDE是一个Android/Java IDE，可以在Android手机上进行Android软件和游戏的开发，由Appfour提供。
* [TIDE](https://sourceforge.net/projects/tide)：TIDE是一个小巧但功能强大且齐全的开源Java IDE。
* [FDT](https://fdt.powerflasher.com/)：FDT是一个基于Eclipse的IDE，适用于交互式开发人员、自由职业者和机构，由Powerflasher开发。
* [FeatureIDE](https://github.com/FeatureIDE/FeatureIDE)：FeatureIDE是一个基于Eclipse的IDE，支持面向特性的软件开发的所有阶段，主要由德国马格德堡大学、帕绍大学开发。
* [Nightcode](https://github.com/oakes/Nightcode)：Nightcode是一款适用于Clojure和ClojureScript的简洁IDE。
* [Flash Builder](https://en.wikipedia.org/wiki/Adobe_Flash_Builder)：Adobe Flash Builder是基于Eclipse平台构建的IDE，可加速富互联网应用程序和跨平台桌面应用程序的开发。
* [RStudio](https://github.com/rstudio/rstudio)：RStudio是R编程语言的集成开发环境，由Posit软件公司开源。
* [Aptana Studio 3](https://github.com/aptana/studio3)：Aptana Studio 3是一个Web开发IDE，由Axway开源。
* [AndroidIDE](https://github.com/AndroidIDEOfficial/AndroidIDE)：AndroidIDE是一款适用于Android开发的IDE。
* [CodeAssist](https://github.com/tyron12233/CodeAssist)：用于在Android上构建Android应用程序的实验性IDE。
* [Cosmic IDE](https://github.com/Cosmic-Ide/Cosmic-IDE)：Cosmic是一款功能强大、丰富的IDE，适用于Android上的JVM开发。
* [JCreator](https://jcreator.en.softonic.com/)：JCreator是一款专有软件的Java编辑器，由Xinox公司制作。
* [PraxisLive](https://github.com/praxis-live/praxis-live)：PraxisLive是一种混合视觉实时编程Java IDE。
* [JGRASP](https://www.jgrasp.org/)：JGRASP是一个轻量级开发环境，专门用于提供软件可视化的自动生成，以提高软件的可理解性，由奥本大学开源。
* [FlowV](https://gitee.com/feisuanyz/java)：Java全自动开发工具，能快速创建微服务项目，由飞算云智开源。
* [Halcyon IDE](https://github.com/s4n7h0/Halcyon-IDE)：Halcyon IDE是第一个专为Nmap Script开发人员开发的IDE。
* [Portugol Studio](https://github.com/UNIVALI-LITE/Portugol-Studio)：Portugol Studio是一个学习编程的环境，针对讲葡萄牙语的编程初学者，由UNVALI教育技术创新实验室开发。

#### 文本编辑器

* [JEdit](https://www.jedit.org/)：JEdit是一个用Java语言开发的文本编辑器。
* [JEditor](https://jeditor.sourceforge.io/)：jEditor是一款具有语法高亮功能的简单Java文本编辑器，旨在作为其他应用程序的库使用。
* [Jext](https://github.com/romainguy/jext)：Jext是适用于Windows、Linux和macOS的源代码编辑器。
* [λiquid](https://github.com/mogenslund/liquid)：Clojure文本编辑器，用于编辑Clojure代码和Markdown代码。
* [JNotepad](https://gitee.com/jcnc-org/JNotepad)：JNotepad是一款简约而强大的跨平台文本编辑器，旨在提供用户友好的界面和丰富的功能以及插件化使用。
* [Pure Writer](https://github.com/PureWriter/desktop)：Pure Writer是一个开源编辑器。

#### 在线编辑器

* [Online Java](https://www.online-java.com)：Online Java是一个基于Web的工具，它是快速、健壮、强大的Java语言在线编译器之一。
* [JDoodle](https://www.jdoodle.com/)：JDoodle是一款在线Java编译器IDE，它能够让你在浏览器中编写、运行和调试Java代码，无需在本地安装任何开发环境。
* [SnapCode](https://github.com/reportmill/SnapCode)：SnapCode使你可以在任何现代浏览器中快速轻松地开始编写Java代码。
* [Compilejava](https://www.compilejava.net/)：Compilejava是运行Java代码的简单在线工具。

#### 数学软件

* [MATLAB](https://www.mathworks.com/products/matlab.html)：MATLAB是一种用于算法开发、数据可视化、数据分析以及数值计算的高级技术计算语言和交互式环境，其GUI部分由Java开发，这是MathWorks公司的商业软件。
* [Maple](https://www.maplesoft.com/products/Maple/)：Maple是一个符号计算和数值计算软件平台。
* [Mathematica](https://www.wolfram.com/mathematica/)：Mathematica是一款科学计算软件，有时候也被称为计算机代数系统，广泛使用于科学、工程、数学、计算等领域。
* [GeoGebra](https://github.com/geogebra/geogebra)：GeoGebra是一款动态数学软件，于2001年由Markus Hohenwarter在奥地利萨尔茨堡大学制作。
* [Scilab](https://gitlab.com/scilab/scilab)：Scilab是用于数值计算的免费开源软件，为工程和科学应用提供强大的计算环境，由法国国立计算机及自动化研究院和法国国立桥梁学院开源。
* [MagicPlot](https://www.magicplot.com/)：MagicPlot是一款技术绘图、曲线拟合和数据分析应用程序，这是一个商业软件。
* [SmartPLS](https://www.smartpls.com/)：SmartPLS是用于使用偏最小二乘(PLS)路径建模方法进行基于方差的结构方程建模(SEM)的软件。
* [CaR](http://car.rene-grothmann.de/doc_en/index.html)：CaR是动态几何程序，它模拟计算机上的指南针和尺子结构，由艾希施泰特天主教大学开源。
* [JGEX](https://github.com/kovzol/Java-Geometry-Expert)：JGEX是一款结合了动态几何软件(DGS)、自动几何定理证明器(GTP)和视觉动态证明呈现方法的软件，由奥地利林茨教区私立大学教育学院开源。
* [CARMetal](https://carmetal.en.uptodown.com/windows)：CARMetal是动态几何程序C.a.R.的改编版本。
* [Hodoku](https://github.com/PseudoFish/Hodoku)：HoDoKu是一个用Java编写的数独助手，有英语和德语版本。
* [JLaTeXMath](https://github.com/opencollab/jlatexmath)：JLaTeXMath是显示LaTeX代码的最佳Java库。
* [Gaalop](https://github.com/CallForSanity/Gaalop)：Gaalop是一款将几何代数(GA)表达式编译并优化为高级编程语言代码的软件。
* [JPlotter](https://jplotter.sourceforge.io/)：JPlotter是一个开源数学绘图仪，可以绘制任意数学函数的图形。

#### UML工具

* [StarUML](https://staruml.io/)：StarUML是一个开源的UML工具列表软件，由韩国MKLabs公司开源。
* [ArgoUML](https://github.com/argouml-tigris-org/argouml)：ArgoUML是领先的开源UML建模工具，支持所有标准UML 1.4图。
* [PlantUML](https://github.com/plantuml/plantuml)：PlantUML是一个允许你通过简单的文本描述创建各种UML图的组件。
* [IBM Rational Rose](https://www.ibm.com/docs/en/rational-soft-arch/9.7.0?topic=migration-rational-rose-model)：Rational Rose是统一建模语言的开发环境，这是IBM的商业产品。
* [Visual Paradigm](https://www.visual-paradigm.com/cn/)：Visual Paradigm是一款功能强大的UML工具，适用于不同层次的用户，从初学者到专家。
* [Astah](https://astah.net/)：Astah是由日本公司Change Vision创建的UML建模工具。
* [Eclipse Papyrus](https://eclipse.dev/papyrus/)：Papyrus是一个工业级的开源基于模型的工程工具。
* [Modelio](https://github.com/ModelioOpenSource/Modelio)：Modelio是一种建模解决方案，提供基于企业架构、软件开发和系统工程常用标准的广泛功能。
* [MagicDraw](https://www.3ds.com/products/catia/no-magic/magicdraw)：MagicDraw是一款出色的建模工具，这是达索的商业工具。
* [Violet](https://github.com/violetumleditor/violetumleditor)：Violet是一个UML编辑器，易于学习和使用、可以画出漂亮的图表、完全免费跨平台。
* [JetUML](https://github.com/prmr/JetUML)：JetUML支持以最少的麻烦来绘制软件设计思想的草图，由麦吉尔大学教授开源。
* [UMLet](https://github.com/umlet/umlet)：UMLet是一个开源UML工具，具有简单的用户界面。
* [TinyUML](https://sourceforge.net/projects/tinyuml/)：TinyUML是一款免费软件工具，可轻松快速地创建UML 2图。

#### 办公软件

* [LibreOffice](https://github.com/LibreOffice/core)：LibreOffice是一款免费且功能强大的办公套件，是OpenOffice的继承者。
* [ThinkFree Office](https://thinkfree.com/)：ThinkFree Office是一款廉价却高效的Microsoft Office替代品。
* [OpenOffice](https://www.openoffice.org/)：OpenOffice是一个开源的办公包软件，起源于Sun公司从StarDivision收购的StarOffice。
* [永中Office](https://www.yozosoft.com/index.html)：永中Office是由永中科技公司用Java语言开发的一个可以在Windows、Linux等多个不同操作系统上运行的办公软件，与微软Microsoft Office相似。
* [泰山Office](http://web.ts-it.cn/index.html)：泰山Office采用Java开发，深度优化JVM，全面兼容国产CPU(龙芯、飞腾、鲲鹏、兆芯、申威)、国产操作系统(UOS、中标麒麟、银河麒麟、深度等)，是一款完全自主可控、安全可靠的国产基础办公软件。
* [Hancom Office](https://www.hancom.com/main/main.do)：Hancom Office是一款专有办公套件，其中包括文字处理器、电子表格软件、演示软件和PDF编辑器以及可通过互联网浏览器访问的在线版本，由韩国Hancom公司开发。
* [HCL Notes](https://www.hcl-software.com/domino)：HCL Notes是由HCLTech销售的适用于Unix、IBM i、Windows、Linux和macOS的专有协作软件平台。

#### 思维导图

* [Freeplane](https://github.com/freeplane/freeplane)：Freeplane是一款免费的开源软件应用程序，支持在工作、学校和家庭中思考、共享信息、完成工作。
* [Xmind](https://xmind.app/)：XMind是一个由香港XMind公司开发的脑力激荡法和心智图的软件工具，其主要用途为帮助用户捕捉想法，组织各类报表。
* [FreeMind](https://freemind.sourceforge.io/)：FreeMind是一款跨平台、用Java编写的绘制思维导图的软件。
* [VUE](https://github.com/VUE/VUE)：VUE是一个用Java编写的免费开源概念图应用程序，由塔夫茨大学学术技术小组开发。
* [CompendiumNG](https://github.com/compendiumng/compendiumng)：CompendiumNG是一个功能强大的应用程序，可让你使用节点和链接结构创建广泛的地图，由英国开放大学开源。
* [MindRaider](https://mindraider.sourceforge.net/)：MindRaider是一款个人笔记本和大纲编辑器。

#### 数据库软件

* [DBeaver](https://github.com/dbeaver/dbeaver)：DBeaver是一款免费的跨平台数据库工具。
* [Datagrip](https://www.jetbrains.com/datagrip/)：Datagrip是JetBrains公司开发的适用于关系数据库和NoSQL数据库的强大跨平台工具。
* [Chat2DB](https://github.com/CodePhiliaX/Chat2DB)：Chat2DB是一款AI first的数据管理、开发、分析工具，由阿里开源。
* [CloudDM](https://www.clougence.com/clouddm-personal)：CloudDM是开云集致公司推出的一款一站式多数据源开发管理工具。
* [MDUT](https://github.com/SafeGroceryStore/MDUT)：MDUT是一款集成多种主流数据库类型的中文数据库跨平台使用工具。
* [Jailer](https://github.com/Wisser/Jailer)：Jailer是一个用于数据库子集、模式和数据浏览的工具。
* [SQLeo](https://sqleo.sourceforge.io/)：SQLeo是一个强大的SQL工具，可将复杂查询(由OBIEE、Microstrategy、Cognos、Hyperion、Pentaho...生成)转换或反转为图表，以简化可视化和分析。
* [Aqua Data Studio](https://aquadatastudio.com/products/)：业界领先的数据库IDE，具有数据管理和可视化功能。
* [ChartSQL](https://docs.chartsql.com/)：ChartSQL Studio是一个源代码编辑器，用于创建和执行SQL脚本、图表和可视化，由Sales Insights开源。
* [DbVisualizer](https://www.dbvis.com/)：DbVisualizer是一个功能丰富的SQL工具，适用于Windows、Mac和Linux系统。
* [SQL Workbench](https://codeberg.org/sql-workbench/workbench)：SQL Workbench/J是一个独立于DBMS的跨平台SQL查询工具。
* [Oracle SQL Developer](https://www.oracle.com/database/sqldeveloper/)：Oracle SQL Developer是一个免费的集成开发环境，可简化传统部署和云部署中Oracle数据库的开发和管理。
* [Airpal](https://github.com/airbnb/airpal)：Airpal是一种基于Web的查询执行工具，它利用Facebook的PrestoDB使用户可以轻松编写查询和检索结果，由Airbnb开源。
* [SchemaSpy](https://github.com/schemaspy/schemaspy)：SchemaSpy是一个数据库元数据分析器，它可以帮助你的数据库管理员和开发人员可视化、导航和理解你的数据模型。
* [SQuirreL](https://github.com/squirrel-sql-client/squirrel-sql-code)：SQuirrel SQL是一个用Java写的数据库管理工具。
* [BrModelo](https://github.com/chcandido/brModelo)：BrModelo是用于数据库ER模型的工具，由圣卡塔琳娜联邦大学和安第列斯-圭亚那大学联合开发。
* [DBEdit 2](https://dbedit2.sourceforge.net/)：DBEdit 2是一个数据库编辑器，适合作为所有关系型数据库的前端。
* [QStudio](https://github.com/timeseries/qstudio)：QStudio是一个免费的SQL GUI，它允许运行SQL脚本、轻松浏览表格、绘制图表和导出结果，由TimeStored开源。
* [Rel](https://reldb.org/c/)：Rel是一个主要用于教育目的的DBMS，但也适合用作桌面DBMS或轻量级服务器。
* [ODC](https://github.com/oceanbase/odc)：ODC是一款开源、全能的跨场景数据库协同开发和数据管理工具，阿里开源。
* [CloudBeaver](https://github.com/dbeaver/cloudbeaver)：CloudBeaver是一个云数据库管理器，提供丰富的Web界面。
* [EXperDB Management](https://github.com/experdb/eXperDB-Management)：EXperDB Management是一款PostgreSQL管理工具。
* [Binjr](https://github.com/binjr/binjr)：Binjr是一个独立的时序浏览器，它将其他应用程序生成的时序数据呈现为动态可编辑视图，并提供高级功能以顺畅高效地导航数据。
* [SQLucky](https://github.com/tenie/SQLucky)：SQLucky是一个跨平台数据库可视化操作工具。
* [DatabaseFX](https://gitee.com/databasefx/dbfx)：DatabaseFX是一个免费、跨平台、基于JavaFX和Vert.X SQL客户端的开源数据库管理工具。
* [ASH Viewer](https://github.com/akardapolov/ASH-Viewer)：ASH Viewer提供数据库中活动会话历史记录数据的图形视图。

#### 数据库建模

* [Open ModelSphere](http://www.modelsphere.com/org/index.html)：Open ModelSphere是一个强大的数据、流程和UML建模工具。
* [PDManer](https://gitee.com/robergroup/pdmaner)：PDManer元数建模，是一款多操作系统开源免费的桌面版关系数据库模型建模工具，相对于PowerDesigner，他具备界面简洁美观，操作简单，上手容易等特点。
* [CHINER](https://gitee.com/robergroup/chiner)：CHINER元数建模，一款丰富数据库生态、独立于具体数据库之外的、数据库关系模型设计平台。
* [SchemaCrawler](https://github.com/schemacrawler/SchemaCrawler)：SchemaCrawler是一个免费的数据库模式发现和理解工具。
* [FML](https://github.com/alibaba/fast-modeling-language)：FML是一种专为维度建模而设计的类似SQL的语言，由阿里开源。

#### 反编译

* [JADX](https://github.com/skylot/jadx)：JADX是一个Dex到Java反编译器。
* [JD GUI](https://github.com/java-decompiler/jd-gui)：JD GUI是一个独立的图形实用程序，可从class文件中显示Java源代码。
* [Recaf](https://github.com/Col-E/Recaf)：Recaf是一个易于使用的现代Java字节码编辑器，可以抽象出Java程序的复杂性。
* [ClassyShark](https://github.com/google/android-classyshark)：ClassyShark是一款面向Android开发人员的独立二进制检查工具，由Google开源。
* [JClasslib](https://github.com/ingokegel/jclasslib)：JClasslib是一个工具，可以可视化已编译的Java类文件和所包含的字节码的各个方面。
* [GDA](https://github.com/charles2gan/GDA-android-reversing-Tool)：GDA是一款强大的Dalvik字节码反编译器。
* [Luyten](https://github.com/deathmarine/Luyten)：Procyon的开源Java反编译器GUI。
* [Classpy](https://github.com/zxh0/classpy)：Classpy是一个GUI工具，用于研究Java类文件、Lua二进制块、Wasm二进制代码和其他二进制文件格式。
* [Jar Analyzer](https://github.com/jar-analyzer/jar-analyzer)：Jar Analyzer是一个分析Jar文件的GUI工具。
* [ClassViewer](https://github.com/ClassViewer/ClassViewer)：ClassViewer是一个轻量级的Java类文件查看器。
* [JADXecute](https://github.com/LaurieWired/JADXecute)：JADXecute是JADX的一个插件，通过添加动态代码执行功能来增强其功能。
* [Decompiler](https://github.com/sotasan/decompiler)：Decompiler是一个GUI应用程序，允许你使用各种反编译器浏览Java包。
* [Vineflower](https://github.com/Vineflower/vineflower)：Vineflower是一种现代通用JVM语言反编译器，专注于提供最佳的质量、速度和可用性。
* [Fernflower](https://github.com/fesh0r/fernflower)：Fernflower是第一个实际工作的Java分析反编译器，也可能是一般高级编程语言的分析反编译器，由JetBrains开源。
* [Friday](https://github.com/zifeihan/friday)：Friday是一个Java实时反编译工具。
* [Class Visualizer](https://github.com/jonatan-kazmierczak/class-visualizer)：Class Visualizer是一个免费交互式类图生成器。
* [JODE](https://jode.sourceforge.io/)：JODE是一个包含Java反编译器和优化器的Java包。
* [Malimite](https://github.com/LaurieWired/Malimite)：Malimite是一款iOS反编译器，旨在帮助研究人员分析和解码IPA文件。
* [ABC Decompiler](https://github.com/ohos-decompiler/abc-decompiler)：ABC Decompiler是基于JADX和ABCDE实现的鸿蒙abc/方舟字节码反编译工具。
* [Jar Explorer](http://dst.in.ua/jarexp/index.html?l=en)：Jar Explorer是一个用于浏览Java库(称为JAR、WAR、EAR、APK、AAR和ZIP文件)的GUI工具。

#### 代码混淆

* [ProGuard](https://github.com/Guardsquare/proguard)：ProGuard是一个免费的Java字节码收缩器、优化器、混淆器和预验证器。
* [Simplify](https://github.com/CalebFenton/simplify)：Simplify是一个Android虚拟机和反混淆器。
* [ClassFinal](https://gitee.com/roseboy/classfinal)：ClassFinal是一款Java class文件安全加密工具，支持直接加密jar包或war包，无需修改任何项目代码，兼容Spring框架；可避免源码泄漏或字节码被反编译。
* [Deobfuscator](https://github.com/java-deobfuscator/deobfuscator)：Deobfuscator旨在对大多数商用Java混淆器进行反混淆。
* [DashO](https://www.preemptive.com/products/dasho/)：DashO的目标是让混淆和应用程序强化变得比以往更容易。
* [Zelix](https://zelix.com/)：Zelix可保护你的Java代码免遭反编译和逆向工程。
* [JBCO](http://www.sable.mcgill.ca/JBCO/)：JBCO是一款Java字节码混淆器，由麦吉尔大学开源。
* [Black Obfuscator](https://github.com/CodingGay/BlackObfuscator)：Black Obfuscator是一款针对Android APK Dex文件的混淆器，它可以帮助开发者通过控制流扁平化来保护源代码，并使分析实际程序控制流变得困难。
* [Skidfuscator](https://github.com/skidfuscatordev/skidfuscator-java-obfuscator)：Skidfuscator是一个概念验证混淆工具，旨在利用SSA形式来优化和混淆Java字节码代码流。
* [Radon](https://github.com/ItzSomebody/radon)：Radon是一个磨损的Java字节码混淆器。
* [Caesium](https://github.com/sim0n/Caesium)：Cesium是一个强大的Java字节码混淆器。
* [DProtect](https://github.com/open-obfuscator/dProtect)：DProtect是Proguard的扩展，具有增强的代码混淆功能。
* [Bozar](https://github.com/vimasig/Bozar)：Bozar是一个带GUI的Java字节码混淆器。
* [Threadtear](https://github.com/GraxCode/threadtear)：Threadtear是一个多功能的Java反混淆工具。
* [QProtect](https://mdma.dev/)：QProtect是第一个提供自定义变压器支持的混淆器。
* [BisGuard](https://www.bisguard.com/)：BisGuard是一个商业的Java反混淆工具。
* [ClassGuard](https://zenofx.com/classguard/)：ClassGuard是一个防止Java反编译的工具，由Zenofx开发。
* [SandMark](http://sandmark.cs.arizona.edu/)：SandMark是亚利桑那大学开发的一款工具，用于Java字节码的软件水印、防篡改和代码混淆。
* [Stringer](https://jfxstore.com/stringer/)：Stringer可保护Java应用程序二进制文件(JAR、Java 9模块、OSGI、WAR、EAR、Eclipse RCP)免遭逆向工程和修改。
* [Obfuscator](https://github.com/superblaubeere27/obfuscator)：Obfuscator是一个GUI Java混淆器。
* [JObfuscator](https://www.pelock.com/products/jobfuscator)：JObfuscator是Java语言的源代码混淆器。
* [Enigma](https://github.com/FabricMC/Enigma)：Enigma是用于Java字节码反混淆的工具。
* [JMD](https://github.com/yocontra/JMD)：JMD是一个通用的Java字节码反混淆工具。
* [Gruntpocalypse](https://github.com/SpartanB312/Grunt)：Gruntpocalypse是一个用Kotlin编写的JVM字节码混淆器，具有30多种功能。
* [Java Obfuscator](https://github.com/alpheratzteam/obfuscator)：用于保护Java应用程序的混淆器。
* [Deobfuscator](https://github.com/narumii/Deobfuscator)：Java反混淆器。
* [Jar Obfuscator](https://github.com/jar-analyzer/jar-obfuscator)：Jar Obfuscator是一个Jar/Class文件混淆工具。
* [JNIC](https://jnic.dev/)：强大的Java原生混淆器。
* [JNT2](https://exile.club/product/jnt)：JNT2是一个功能强大的软件，它使用Java本机接口将JVM字节码转换为C。
* [Masxinlingvonta](https://github.com/superblaubeere27/masxinlingvonta)：将Java字节码编译为LLVM IR和本机代码。
* [Protector4J](https://protector4j.com)：保护Java代码免于反编译，这是VLINX公司的商业产品。
* [qProtect](https://qtechnologies.dev/)：提供自定义转换器支持的Java混淆器。

#### 逆向工程

* [Ghidra](https://github.com/NationalSecurityAgency/ghidra)：Ghidra是一个由美国国家安全局研究局创建和维护的软件逆向工程框架。
* [JEB Community Edition](https://www.pnfsoftware.com/jeb/community-edition)：JEB是一款针对Android应用程序和本机机器代码的反汇编和反编译软件。
* [Apktool](https://github.com/iBotPeaches/Apktool)：Apktool是一款用于对第三方、封闭式、二进制Android应用程序进行逆向工程的工具。
* [Bytecode Viewer](https://github.com/Konloch/bytecode-viewer)：Bytecode Viewer是一个轻量级用户友好的Java/Android字节码查看器、反编译器等。
* [BinAbsInspector](https://github.com/KeenSecurityLab/BinAbsInspector)：BinAbsInspector是一款用于自动化逆向工程和扫描二进制文件漏洞的静态分析器，是腾讯科恩实验室孵化的长期研究项目。
* [JByteMod](https://github.com/GraxCode/JByteMod-Beta)：JByteMod是一个多功能字节码编辑器，具有语法突出显示、实时反编译和方法绘图功能。
* [BinDiff](https://github.com/google/bindiff)：BinDiff是一款开源的二进制文件比较工具，可以帮助漏洞研究人员和工程师快速找到反汇编代码中的差异和相似之处，由Google开源。
* [Super JADX](https://github.com/pkilller/super-jadx)：Super JADX是一个JADX插件，添加了逆向工程的新功能。
* [BinNavi](https://github.com/google/binnavi)：BinNavi是一个二进制分析IDE，允许检查、导航、编辑和注释控制流图以及反汇编代码的调用图，由Google开源。
* [Helios](https://github.com/helios-decompiler/standalone-app)：Helios是一款一体化Java逆向工程工具，它具有与最新反编译器集成的功能。
* [Kaiju](https://github.com/cmu-sei/kaiju)：Kaiju是Ghidra软件逆向工程套件的二进制分析框架扩展，由CMU开源。
* [ABCDE](https://github.com/Yricky/abcde)：ABCDE是一个使用Kotlin编写的OpenHarmony逆向工具包。
* [SkidSuite](https://github.com/GenericException/SkidSuite)：SkidSuite 3是与Java应用程序逆向工程相关的有用工具的集合。

## 游戏开发

这里列出来Java中用于开发游戏的引擎、库、工具，以及Java开发的一些免费游戏

#### 游戏引擎

* [LibGDX](https://github.com/libgdx/libgdx)：LibGDX是一个基于OpenGL的跨平台Java游戏开发框架，专为Windows、Linux、macOS、Android、Web浏览器和iOS设计。
* [LWJGL](https://github.com/LWJGL/lwjgl3)：LWJGL是一个Java库，支持跨平台访问流行的原生API，可用于图形(OpenGL、Vulkan、bgfx)、音频(OpenAL、Opus)、并行计算(OpenCL、CUDA)和XR(OpenVR、LibOVR、OpenXR)应用程序。
* [FXGL](https://github.com/AlmasB/FXGL)：Java/JavaFX/Kotlin游戏引擎库。
* [Godot Kotlin/JVM](https://github.com/utopia-rise/godot-kotlin-jvm)：这是Godot游戏引擎的Kotlin语言绑定。
* [KTX](https://github.com/libktx/ktx)：KTX是一个扩展LibGDX的Kotlin游戏框架。
* [jMonkeyEngine](https://github.com/jMonkeyEngine/jmonkeyengine)：jMonkeyEngine是一款适合富有冒险精神的Java开发人员的3D游戏引擎。
* [Defold](https://github.com/defold/defold)：Defold是一款完全免费的游戏引擎，可用于开发桌面、移动和Web游戏。
* [KorGE](https://github.com/korlibs/korge)：KorGE是Kotlin的现代多平台游戏引擎。
* [Ludii](https://github.com/Ludeme/Ludii)：Ludii是一个通用游戏系统，旨在玩、评估和设计各种游戏，包括棋盘游戏、纸牌游戏、骰子游戏、数学游戏等，由马斯特里赫特大学开源。
* [SGDK](https://github.com/Stephane-D/SGDK)：适用于Sega Mega Drive的免费开源开发套件。
* [OpenRTS](https://github.com/methusalah/OpenRTS)：OpenRTS是一个3D即时战略游戏引擎。
* [JBox2d](https://github.com/jbox2d/jbox2d)：JBox2d是C++物理引擎LiquidFun和Box2d的Java端口。
* [Terasology](https://github.com/MovingBlocks/Terasology)：Terasology项目诞生于受Minecraft启发的技术演示，并正在成为体素世界中各种类型游戏设置的稳定平台。
* [AndEngine](https://github.com/nicolasgramlich/AndEngine)：免费Android 2D OpenGL游戏引擎。
* [XMage](https://github.com/magefree/mage)：XMage允许你与一名或多名在线玩家或电脑对手玩万智牌。
* [JForGame](https://github.com/kingston-csj/jforgame)：JForGame是一个用Java编写的轻量级高性能手游服务端框架，包含游戏服、跨服、匹配服、后台管理系统等模块。
* [FriceEngine](https://github.com/icela/FriceEngine)：FriceEngine是一个简单、轻量级的原生游戏引擎，主要运行在JVM上。
* [TripleA](https://github.com/triplea-game/triplea)：TripleA是一款回合制策略游戏和棋盘游戏引擎，类似于Axis & Allies或Risk。
* [Delver](https://github.com/Interrupt/delverengine)：Delver游戏引擎和编辑器。
* [Litiengine](https://github.com/gurkenlabs/litiengine)：LITIENGINE是一个免费且开源的Java 2D游戏引擎，它提供了一个全面的Java库和一个专用的地图编辑器来创建基于图块的2D游戏。
* [Mini2Dx](https://github.com/mini2Dx/mini2Dx)：Mini2Dx的主要目标是提供一个初学者友好、精通的框架，用于用Java快速原型设计和构建2D游戏。
* [LGame](https://github.com/cping/LGame)：一个跨平台的Java游戏引擎，支持JavaFX/Android/IOS/HTML5/Linux/MAC/Windows。
* [Jake2](https://bytonic.de/html/jake2.html)：Quake II游戏引擎的Java端口。
* [Forge](https://github.com/Card-Forge/forge)：世界上最伟大的纸牌游戏的非官方规则引擎。
* [LionEngine](https://github.com/b3dgs/lionengine)：LionEngine是专为Lionheart Remake项目开发的游戏引擎，可轻松与Java一起使用。
* [SilenceEngine](https://github.com/sriharshachilakapati/SilenceEngine)：SilenceEngine是一款2D/3D游戏引擎，可以为你处理游戏开发的低级方面，如图形、输入处理、资源加载和碰撞检测。
* [DimensioneX](https://www.dimensionex.net/)：DimensioneX是一款简单、免费的多人(MMORPG)游戏引擎。
* [VASSAL](https://github.com/vassalengine/vassal)：VASSAL是一个游戏引擎，用于构建和玩在线改编的棋盘游戏和纸牌游戏。
* [Env3D](https://sourceforge.net/projects/env3d/)：用Java编写的3D引擎，面向计算机科学专业的学生。
* [FastJ](https://github.com/fastjengine/FastJ)：FastJ是一个免费开源的基于Java的2D游戏引擎和框架，它旨在使用Java(和JVM语言)提供最佳的2D游戏制作体验。
* [Kool](https://github.com/fabmax/kool)：Kool是一个多平台OpenGL/WebGPU/Vulkan游戏引擎，适用于桌面Java、Android和浏览器。
* [OpenRSC](https://github.com/Open-RSC/Core-Framework)：该仓库包含Open RuneScape Classic游戏框架。
* [Yaeger](https://github.com/han-yaeger/yaeger)：Yaeger是一个教育游戏引擎运行时，也是一个功能齐全的2D游戏引擎，只需要传统的面向对象编程风格，由HAN应用科技大学开源。
* [Arc](https://github.com/Anuken/Arc)：基于LibGDX的Java游戏开发框架。
* [VatraLib](https://vatrasoft.de/vatralib/)：VatraLib是一个用Java编写的2D游戏引擎。

#### 游戏服务器

* [NettyGameServer](https://github.com/jwpttcg66/NettyGameServer)：NettyGameServer是基于Netty 4.X实现的手机游戏分布式服务器，支持TCP、UDP、HTTP、WebSocket链接。
* [JetServer](https://github.com/menacher/java-game-server)：JetServer是一个基于高速NIO套接字的多人Java游戏服务器，使用Netty和Jetlang编写。
* [Game Server](https://github.com/jzyong/game-server)：Game Server是一个基于棋牌、MMORPG游戏的分布式java游戏服务器。
* [Summer](https://github.com/SwingFrog/Summer)：Summer是一个轻量级、一站式的Java游戏服务器框架，也可用于开发简单的Web服务。
* [Mmorpg](https://github.com/kingston-csj/mmorpg)：Mmorpg是一个用Java编写的分布式高性能mmorpg手游服务端框架。
* [GameServer4j](https://github.com/jzyong/GameServer4j)：分布式Java游戏服务器，包括登录、网关、游戏演示。
* [ZFoo](https://github.com/zfoo-project/zfoo)：ZFoo是一个极快的企业服务器框架，可用于RPC、游戏服务器、Web服务器。
* [IoGame](https://gitee.com/game-town/ioGame)：IoGame是一个轻量级的网络编程框架，适用于网络游戏服务器、物联网、内部系统及各种需要长连接的场景。
* [Apollo](https://github.com/apollo-rsps/apollo)：Apollo是一个高性能、模块化的RuneScape模拟器，具有一系列用于管理数据文件和插件的实用程序。
* [Noark](https://gitee.com/xiaoe/noark3)：Noark是一个游戏服务器端框架，可快速开发出一个易维护、易扩展且稳定高能的游戏服务器。
* [Carmelo](https://github.com/needmorecode/carmelo)：Carmelo是一个快速、可扩展的Java服务器框架，专为在线游戏而设计。
* [Okra](https://github.com/ogcs/Okra)：Okra是一个简单的使用Java开发的高性能、高扩展、高并发、低延迟的服务器框架。
* [Gamioo](https://github.com/jiangguilong2000/gamioo)：游戏服务器框架，基于此框架，可以快速实现一个高可用、易维护、稳定、高性能的游戏服务器。
* [TenIO](https://github.com/congcoi123/tenio)：TenIO是一个用于创建多人在线游戏的开源项目，其中包括专门为多人游戏设计的基于Java NIO的服务器。
* [Avalon](https://gitee.com/codeborker/Avalon)：基于Akka的高性能可伸缩的Java网络游戏服务器，简单的单服务器开发与集群开发的切换。
* [Lila](https://github.com/lichess-org/lila)：Lila是一款免费的在线国际象棋游戏服务器，专注于实时游戏玩法和易用性。
* [Grasscutter](https://github.com/Grasscutters/Grasscutter)：Grasscutter是一个实验性游戏服务器，旨在模拟玩某个动漫游戏的体验。
* [PretendYoureXyzzy](https://github.com/ajanata/PretendYoureXyzzy)：反人类卡牌克隆、服务器和Web客户端。
* [LunarCore](https://github.com/Melledy/LunarCore)：某回合制动漫游戏的游戏服务器重新实现。
* [L2J Server](https://bitbucket.org/l2jserver/l2j-server-game)：L2J Server是一款完全用Java编写的开源服务器模拟器，适用于著名的韩国MMORPG。

#### 游戏开发库

* [PlayN](https://github.com/playn/playn)：PlayN是一个用Java编写的跨平台Java游戏开发库，面向HTML5浏览器、桌面JVM、Android和iOS设备。
* [Zircon](https://github.com/Hexworks/zircon)：Zircon是一个可扩展且用户友好的多平台图块引擎。
* [Recast4j](https://github.com/ppiastucki/recast4j)：Recast和Detour导航网格工具集的Java端口。
* [Slick2D](https://slick.ninjacave.com/)：Slick2D是一组易于使用的工具和实用程序，围绕LWJGL、OpenGL绑定，使2D Java游戏开发变得更加容易。
* [ODE4j](https://github.com/tzaeschke/ode4j)：ODE是一个用于模拟刚体动力学的开源高性能库。
* [JInput](https://github.com/jinput/jinput)：用于访问输入设备的库，由Sun公司游戏技术小组发起。
* [Dyn4j](https://github.com/dyn4j/dyn4j)：纯Java 2D碰撞检测和物理引擎，旨在快速、稳定、可扩展且易于使用。
* [SquidLib](https://github.com/yellowstonegames/SquidLib)：SquidLib是一个功能非常齐全的库，部分目标是制作传统Roguelike和类似类型的游戏。
* [Libbulletjme](https://github.com/stephengold/Libbulletjme)：BulletPhysics和V-HACD的JNI接口。

#### 寻路算法

* [Baritone](https://github.com/cabaletta/baritone)：Baritone是Impact从4.4版本开始使用的寻路系统。
* [Pathetic](https://github.com/bsommerfeld/pathetic)：Pathetic是一个高度可配置的Java A*寻路库，旨在通过自定义节点验证和成本处理实现可扩展性。
* [Recast4j](https://github.com/recast4j/recast4j)：Recast和Detour导航网格工具集的Java端口。

#### 实体框架

* [Artemis ODB](https://github.com/junkdog/artemis-odb)：Artemis ODB是一个基于Java的高性能实体组件系统框架。
* [Ashley](https://github.com/libgdx/ashley)：用Java编写的小型实体框架，它的灵感来自于Ash和Artemis等框架。
* [Dominion](https://github.com/dominion-dev/dominion-ecs-java)：Dominion是Java的一个实体组件系统库。
* [Zay ES](https://github.com/jMonkeyEngine-Contributions/zay-es)：Zay-ES是一种基于Java的高性能实体组件系统，它可以避免ES架构的大多数/所有典型缺点，同时又不牺牲ES的“纯度”。

#### 游戏编辑器

* [Spine](https://github.com/EsotericSoftware/spine-runtimes)：Spine是一款针对游戏开发的2D骨骼动画编辑工具，旨在提供更高效和简洁的工作流程，以创建游戏所需的动画。
* [HyperLap2D](https://github.com/rednblackgames/HyperLap2D)：HyperLap2D是一个功能强大、独立于平台的可视化编辑器，适用于复杂的2D世界和场景。
* [Overlap2D](https://github.com/UnderwaterApps/overlap2d)：Overlap2D是一款2D关卡和UI编辑器，具有与引擎无关的游戏开发理念。
* [Bladecoder Adventure](https://github.com/bladecoder/bladecoder-adventure-engine)：Bladecoder Adventure引擎是一组用于创建交互式图形冒险(经典点击游戏)的工具。
* [Talos](https://github.com/rockbite/talos)：基于节点的开源VFX编辑器，具有强大的界面和随时可用的LibGDX运行时。

#### 开源游戏

* [Unciv](https://github.com/yairm210/Unciv)：Civ V的开源、注重可修改性的Android和桌面重制版，使用LibGDX制作。
* [BGBlitz](https://www.bgblitz.com/)：BGBlitz是一款专门用于玩西洋双陆棋的计算机程序。
* [Mindustry](https://github.com/Anuken/Mindustry)：Mindustry是一款2D游戏，其玩法融合塔防、工厂自动化、沙盒与即时战略。
* [Shattered Pixel Dungeon](https://github.com/00-Evan/shattered-pixel-dungeon)：Shattered Pixel Dungeon是一款传统的Roguelike地下城探索角色扮演游戏。
* [Pixel Dungeon](https://github.com/watabou/pixel-dungeon)：Pixel Dungeon是一款传统的Roguelike游戏，具有像素艺术图形和简单的界面，适用于Android、iOS、Windows、Mac和Linux。
* [ByteLegend](https://github.com/ByteLegend/ByteLegend)：ByteLegend是一款免费、开源的MMORPG游戏，你可以在其中获得现实世界的高薪编程技能。

#### 游戏开发工具

* [Live2D](https://www.live2d.com/)：Live2D是一种应用于电子游戏的绘图渲染技术，由日本Cybernoids公司开发。
* [DisUnity](https://github.com/ata4/disunity)：用Java编写的Unity资源和资源包文件的实验性命令行工具集，主要用于提取。
* [Alice](https://github.com/TheAliceProject/alice3)：Alice是一个基于块的创新编程环境，可以轻松创建动画、构建交互式叙述或以3D方式编写简单游戏，由CMU开源。

## 2D/3D渲染

* [JMathPlot](https://github.com/yannrichet/jmathplot)：Java交互式2D和3D绘图。
* [Lets Plot](https://github.com/JetBrains/lets-plot)：Lets-Plot是一个基于图形语法原理构建的多平台绘图库，由JetBrains开源。
* [VTM](https://github.com/mapsforge/vtm)：OpenGL矢量地图库-在Android、iOS、桌面和浏览器上运行。
* [Constellation](https://github.com/constellation-app/constellation)：Constellation是一款以图形为中心的数据可视化和交互式分析应用程序，支持跨大型复杂数据集的数据访问、联合和操作功能。
* [Skija](https://github.com/JetBrains/skija)：Skia是一个开源2D图形库，提供可跨各种硬件和软件平台工作的通用API，Skija是Skia的高质量Java绑定，由JetBrains开源。
* [Jzy3d](https://github.com/jzy3d/jzy3d-api)：Jzy3d是一个用Java轻松绘制3D和2D图表的框架，使用快速原生GPU渲染或基于CPU的渲染来增强跨OS/JVM/GPU组合的可移植性。
* [Marlin](https://github.com/bourgesl/marlin-renderer)：Marlin是一个开源Java 2D渲染引擎，基于OpenJDK的Pisces实现，针对性能进行了优化(改进了内存使用和占用空间、更好的多线程)和更好的视觉质量。
* [Oreon](https://github.com/fynnfluegge/oreon-engine)：OpenGL/Vulkan Java 3D引擎。
* [LEGUI](https://github.com/SpinyOwl/legui)：Java OpenGL GUI库，专为与最新的LWJGL(LWJGL 3)一起使用而创建。
* [ImGui-Java](https://github.com/SpaiR/imgui-java)：ImGui基于JNI的绑定。
* [GraphicsFuzz](https://github.com/google/graphicsfuzz)：GraphicsFuzz提供了自动查找和简化图形驱动程序中的错误的工具，由Google开源。
* [ModernUI](https://github.com/BloCamLimb/ModernUI)：从低级3D图形API到高级视图模型的现代桌面框架，用于开发2D/3D渲染软件或游戏引擎，具有国际化支持和许多新技术。
* [Art of Illusion](https://github.com/ArtOfIllusion/ArtOfIllusion)：Art of Illusion是一个免费、开源的3D建模和渲染软件。
* [Scenery](https://github.com/scenerygraphics/scenery)：由Kotlin和Vulkan提供支持的JVM上体积和几何数据的灵活VR可视化。
* [Grafx](https://grafx.sourceforge.net/)：Grafx是一个开源Java库，用于使用浮点端点坐标绘制线条和填充多边形。
* [ClearVolume](https://github.com/ClearVolume/clearvolume)：ClearVolume是一个实时3D可视化库，专为SPIM和DLSM显微镜等高端体积显微镜而设计，由马克斯普朗克分子细胞生物学和遗传学研究所开源。
* [Vulkan4j](https://github.com/chuigda/vulkan4j)：使用Project Panama API的Java Vulkan绑定。

## 移动开发框架

* [Compose Multiplatform](https://github.com/JetBrains/compose-multiplatform)：Compose Multiplatform是一个声明式框架，用于使用Kotlin跨多个平台共享UI，由JetBrains开发。
* [CodenameOne](https://github.com/codenameone/CodenameOne)：Codename One是面向Java和Kotlin开发人员的移动优先跨平台环境。
* [Multi-OS Engine](https://github.com/multi-os-engine/multi-os-engine)：Multi-OS Engine为iOS平台API提供了Java运行时和Java接口，用于开发具有原生外观、原生性能以及可从Android应用程序移植常见Java逻辑模块的原生iOS应用程序。
* [MobileUI](https://mobileui.dev/)：MobileUI是第一个适用于iOS和Android的基于Java的原生UI框架。

## JVM代理

* [BlockHound](https://github.com/reactor/BlockHound)：用于检测来自非阻塞线程的阻塞调用的Java代理。
* [One Java Agent](https://github.com/alibaba/one-java-agent)：One Java Agent提供插件化支持，统一管理众多的Java Agent，由阿里开源。
* [Dongtai-agent-java](https://github.com/HXSecurity/DongTai-agent-java)：DongTai Agent是针对Java应用程序的数据采集工具，由火线安全开源。
* [KnowAgent](https://github.com/didi/KnowAgent)：基于日志模板构建，采集任务动态管控、数据质量精确度量，一站式日志采集平台，由滴滴开源。
* [JADE](https://jade.tilab.com/)：JADE是一个用Java实现的软件代理开发框架，由意大利电信开发。
* [RR4J](https://github.com/Kartikvk1996/RR4J)：RR4J是一个记录Java字节码执行情况并允许开发人员在本地重放的工具。
* [FlowTracker](https://github.com/coekie/flowtracker)：FlowTracker是一个Java代理，用于跟踪程序如何读取、操作和写入数据。
* [Zorka](https://github.com/jitlogic/zorka)：Zorka是一个用于Java应用程序的可编程通用监控代理。
* [Disco](https://github.com/awslabs/disco)：Disco包括用于创建Java代理的框架，用于分布式系统的面向切面的工具，由AWS开源。
* [JVMQuake](https://github.com/Netflix-Skunkworks/jvmquake)：一个JVMTI代理，它附加到你的JVM，并在程序变得不稳定时自动发出信号并终止它，由Netflix开源。
* [Java Debug Tool](https://github.com/pandening/Java-debug-tool)：Java Debug Tool是一个动态调试工具，它提供了一些调试命令来在运行时调试你的代码。
* [inspectIT Ocelot](https://github.com/inspectIT/inspectit-ocelot)：inspectIT Ocelot是一个零配置Java代理，用于基于OpenCensus库动态收集应用程序性能、跟踪和行为数据。
* [LinkAgent](https://github.com/shulieTech/LinkAgent)：LinkAgent是一个基于Java的开源代理，旨在通过JVM字节码收集Java应用程序的数据和控制功能，而无需修改应用程序代码。
* [File Leak Detector](https://github.com/jenkinsci/lib-file-leak-detector)：File Leak Detector是一个小型Java代理，可以跟踪在JVM中打开文件的位置/时间/人员。
* [EA Agent Loader](https://github.com/electronicarts/ea-agent-loader)：EA Agent Loader是一套面向Java Agent开发者的实用工具，由艺电开源。

## 热加载

* [JRebel](http://zeroturnaround.com/software/jrebel/)：JRebel是一款JVM插件，它使得Java代码修改后不用重启系统，立即生效。
* [Spring Loaded](https://github.com/spring-projects/spring-loaded)：Spring Loaded是一个JVM代理，用于在JVM运行时重新加载class文件更改。
* [Hotswap Agent](https://github.com/HotswapProjects/HotswapAgent)：Java无限运行时类和资源重定义。
* [Fakereplace](https://github.com/fakereplace/fakereplace)：该项目提供了一个JavaAgent和一个客户端，用于在标准JDK热交换提供的基础上热替换JVM中的类。
* [RelProxy](https://github.com/jmarranz/relproxy)：RelProxy是一个简单的Java和Groovy热类重加载器，提供透明的动态编译和类重新加载，以及纯Java代码的脚本支持和Shell。

## 类加载

* [SOFAArk](https://github.com/sofastack/sofa-ark)：SOFAArk是一款基于Java实现的动态热部署和轻量级类隔离框架，由蚂蚁集团开源贡献，主要提供应用模块的动态热部署和类隔离能力。
* [Pandora](https://www.infoq.cn/article/kgxytjb2cr7hgukmjg0p)：Pandora是由淘宝团队打造的基于隔离技术而构建出的新一代的隔离容器
* [JCL](https://github.com/kamranzafar/JCL)：JCL是一个可配置、动态且可扩展的自定义类加载器，可以直接从Jar文件和其他源加载Java类。
* [Java Dynamic Load Jar](https://github.com/Trinea/java-dynamic-load-jar)：解决在不同JAR中加载相同类时的类加载器隔离问题。
* [ModRun](https://github.com/nanosai/modrun)：ModRun可以直接从Maven仓库加载和运行类，并在运行时解决依赖关系。
* [Land](https://github.com/oldratlee/land)：Land是一个通过类加载器实现的简单Java依赖隔离容器。
* [JuShaTa](https://github.com/didi/JuShaTa)：JuShaTa是一个Java容器，提供模块隔离及模块热加载能力，由滴滴开源。
* [JBoss Modules](https://github.com/jboss-modules/jboss-modules)：JBoss Modules是Java模块化(非分层)类加载和执行环境的独立实现。
* [Classloader Leak Prevention](https://github.com/mjiderhamn/classloader-leak-prevention)：类加载器泄漏预防库。
* [Plexus Classworlds](https://github.com/codehaus-plexus/plexus-classworlds)：Plexus Classworlds是一个为需要复杂操作Java类加载器的容器开发人员提供的框架。
* [Cytodynamics](https://github.com/linkedin/Cytodynamics)：Cytodynamics是一个使JVM上的动态JAR加载和类加载器隔离变得简单的Java库，由LinkedIn开源。
* [XJar](https://github.com/core-lib/xjar)：Spring Boot JAR安全加密运行工具，同时支持的原生JAR。