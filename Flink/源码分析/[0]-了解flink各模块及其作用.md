# Apache Flink 项目结构和模块职责

Apache Flink 是一个用于分布式流处理和批处理的开源平台。以下是 Flink 1.14 项目的整体结构和各模块的职责：

## 项目结构

```
flink
├── flink-annotations              # Flink 的注释模块，提供稳定性和公共 API 注释
├── flink-clients                  # Flink 客户端，包括 CLI 和 Java API 客户端
├── flink-connector-*              # 各种 Flink 连接器，用于与外部系统集成（如 Kafka, JDBC, HBase 等）
├── flink-container                # Flink 容器模块，用于在 Kubernetes 和 Docker 上运行 Flink
├── flink-contrib                  # 贡献模块，包含社区贡献的扩展和实验性功能
├── flink-core                     # 核心数据结构和算法
├── flink-dist                     # Flink 分发包，包含二进制和脚本
├── flink-docs                     # Flink 文档，包含项目的用户指南和参考手册
├── flink-dstl                     # 分布式流数据库，提供分布式流数据查询和管理功能
├── flink-end-to-end-tests         # 端到端测试，用于验证 Flink 各组件和功能的集成
├── flink-examples                 # 示例程序，包括批处理和流处理示例
├── flink-external-resources       # 外部资源模块，用于管理和使用外部资源
├── flink-filesystems              # 文件系统模块，提供对不同文件系统的支持（如 HDFS、S3 等）
├── flink-formats                  # 数据格式模块，用于定义不同的数据格式（如 Avro, Parquet 等）
│   ├── flink-avro                    # Avro 格式支持
│   ├── flink-avro-confluent-registry # Avro 格式与 Confluent Schema Registry 集成
│   ├── flink-avro-glue-schema-registry  # Avro 格式与 AWS Glue Schema Registry 集成
│   ├── flink-compress                # 压缩支持，提供对不同压缩格式的支持
│   ├── flink-csv                     # CSV 格式支持
│   ├── flink-format-common           # 通用格式支持，提供各种格式的通用功能
│   ├── flink-hadoop-bulk             # Hadoop Bulk 格式支持
│   ├── flink-json                    # JSON 格式支持
│   ├── flink-orc                     # ORC 格式支持
│   ├── flink-orc-nohive              # ORC 格式支持（无需 Hive）
│   ├── flink-parquet                 # Parquet 格式支持
│   ├── flink-sequence-file           # SequenceFile 格式支持
│   ├── flink-sql-avro                # Flink SQL 中 Avro 格式支持
│   ├── flink-sql-avro-confluent-registry  # Flink SQL 中 Avro 格式与 Confluent Schema Registry 集成
│   ├── flink-sql-orc                 # Flink SQL 中 ORC 格式支持
│   └── flink-sql-parquet             # Flink SQL 中 Parquet 格式支持
├── flink-fs-tests                 # 文件系统的测试模块
├── flink-java                     # Flink 的 Java API 模块
├── flink-jepsen                   # Jepsen 测试模块，用于分布式系统一致性测试
├── flink-kubernetes               # Kubernetes 集成模块
├── flink-libraries                # Flink 库，包括 Flink CEP（复杂事件处理）和 Flink SQL
│   ├── flink-cep                  		# 复杂事件处理库，用于检测流中的模式
│   ├── flink-cep-scala            		# 复杂事件处理库的 Scala API
│   ├── flink-gelly                		# 图处理 API
│   ├── flink-gelly-examples      		# 图处理 API 的示例
│   ├── flink-gelly-scala         		# 图处理 API 的 Scala 实现
│   └── flink-state-processing-api 		# 状态处理 API
├── flink-metrics
│		├── flink-metrics-core          	# 核心度量模块，定义度量器和度量报告接口
│		├── flink-metrics-datadog       	# 提供将 Flink 度量数据发送到 DataDog 监控平台的集成支持
│		├── flink-metrics-dropwizard    	# 提供与 Dropwizard Metrics 库的集成支持，用于收集、报告和显示度量数据
│		├── flink-metrics-graphite      	# 提供将 Flink 度量数据发送到 Graphite 监控系统的集成支持
│		├── flink-metrics-influxdb      	# 提供将 Flink 度量数据发送到 InfluxDB 时间序列数据库的集成支持
│		├── flink-metrics-jmx           	# 提供 JMX（Java Management Extensions）的集成支持，用于导出和管理 Flink 度量数据
│		├── flink-metrics-prometheus    	# 提供将 Flink 度量数据暴露给 Prometheus 监控系统的集成支持
│		└── flink-metrics-slf4j         	# 提供将 Flink 度量数据记录到 SLF4J 日志系统的集成支持
├── flink-optimizer                # 优化器，用于批处理作业的优化
├── flink-python                   # Flink 的 Python API
├── flink-queryable-state          # 查询状态模块，允许查询运行作业的状态
├── flink-quickstart               # 快速入门模块，提供示例项目和模板
├── flink-rpc                      		# RPC 模块，用于远程过程调用
│		├── flink-rpc-akka         		 		# 基于 Akka 的 RPC 实现模块
│		├── flink-rpc-akka-loader   			# 加载 Akka RPC 的辅助模块
│		└── flink-rpc-core         			 # RPC 核心模块，定义了 RPC 框架的核心接口和功能
├── flink-runtime                  # 运行时核心功能
├── flink-runtime-web              # 运行时的 Web 界面
├── flink-scala                    # Flink 的 Scala API
├── flink-scala-shell              # Scala Shell，提供交互式 Scala 环境
├── flink-state-backends           # 状态后端模块，用于存储和管理状态
│		├── flink-statebackend-changelog      # 基于变更日志的状态后端实现模块
│		├── flink-statebackend-heap-spillable # 支持溢写的堆内存状态后端实现模块
│		└── flink-statebackend-rocksdb        # 使用 RocksDB 作为状态后端的实现模块
├── flink-streaming-java           # 流处理核心实现
├── flink-streaming-scala          # 流处理的 Scala 实现
├── flink-table                    # Flink SQL 和 Table API
│ 	├── flink-sql-client                # SQL 客户端模块，提供交互式 SQL 查询功能
│ 	├── flink-sql-parser                # SQL 解析器模块，用于解析 SQL 查询语句
│ 	├── flink-sql-parser-hive           # Hive 兼容的 SQL 解析器模块
│ 	├── flink-table-api-java            # Java 版本的表 API 模块
│ 	├── flink-table-api-java-bridge     # Java 版本的表 API 桥接模块
│ 	├── flink-table-api-scala           # Scala 版本的表 API 模块
│ 	├── flink-table-api-scala-bridge    # Scala 版本的表 API 桥接模块
│ 	├── flink-table-code-splitter       # 表代码分割器模块，用于将表查询计划拆分成多个部分
│ 	├── flink-table-common              # 通用表 API 模块，提供表 API 的通用功能和工具类
│ 	├── flink-table-planner             # 表计划器模块，负责将逻辑查询计划优化为物理执行计划
│ 	├── flink-table-runtime             # 表运行时模块，包含表运行时的核心逻辑和执行引擎
│ 	└── flink-table-uber                # 综合表模块，整合了所有表相关的模块和功能
├── flink-test-utils               # 测试工具
├── flink-test-utils-parent        # 测试工具的父模块
├── flink-tests                    # 各种集成测试
├── flink-walkthroughs             # 教程示例模块
├── flink-yarn                     # YARN 集成模块
└── flink-yarn-tests               # YARN 集成的测试模块
```

**flink-annotations**

- 提供了 Flink 的注释，用于稳定性和公共 API 的注释。这些注释有助于开发人员理解和使用 Flink 的 API，并提供了文档和提示。

**flink-clients**

- 包括 Flink 的 CLI（命令行界面）和 Java API 客户端。CLI 允许用户通过命令行管理 Flink 集群和提交作业，Java API 客户端允许开发人员编写和提交 Flink 作业。

**flink-connector-\***

- 各种 Flink 连接器模块，用于与外部系统集成，例如 Kafka、JDBC、HBase、Elasticsearch 等。每个连接器模块提供相应系统的源（Source）和接收器（Sink），实现与外部系统的数据交互。

**flink-container**

- 提供了在 Kubernetes 和 Docker 等容器环境中运行 Flink 集群的支持。该模块包括容器化部署所需的配置和脚本。

**flink-contrib**

- 社区贡献的扩展模块，包含实验性功能和非核心模块。这些模块通常由社区成员贡献，用于实验新的功能或扩展 Flink 的能力。

**flink-core**

- 包含 Flink 的核心数据结构和算法实现。这些包括数据流处理引擎、作业调度、状态管理等核心功能的实现。

**flink-dist**

- 包含了 Flink 的二进制发布版和启动脚本，用于分发和部署 Flink。

**flink-docs**

- Flink 的官方文档模块，包含用户指南、参考手册和开发文档，帮助用户和开发人员了解和使用 Flink。

**flink-dstl**

- 分布式流数据库，提供分布式流数据查询和管理功能，支持复杂的数据流处理和实时查询。

**flink-end-to-end-tests**

- 端到端测试模块，用于验证 Flink 各组件和功能的集成和正确性。

**flink-examples**

- 包含各种批处理和流处理的示例程序，帮助用户学习和理解如何使用 Flink 解决实际问题。

**flink-external-resources**

- 外部资源模块，用于管理和使用外部资源，如文件、配置等。

**flink-filesystems**

- 文件系统模块，提供对不同文件系统（如 HDFS、S3 等）的支持，使 Flink 可以直接读写这些文件系统中的数据。

**flink-formats**

- 数据格式模块，用于定义和实现不同的数据格式（如 Avro、Parquet 等）的支持，支持 Flink 读写不同格式的数据。

**flink-fs-tests**

- 文件系统模块的测试模块，用于测试文件系统模块的功能和兼容性。

**flink-java**

- Flink 的 Java API 模块，提供了用于开发和执行 Java 版本的 Flink 作业的 API 和工具类。

**flink-jepsen**

- Jepsen 测试模块，用于分布式系统的一致性测试，验证 Flink 在各种故障和并发条件下的行为。

**flink-kubernetes**

- Kubernetes 集成模块，提供了在 Kubernetes 上部署和管理 Flink 集群的支持。

**flink-libraries**

- Flink 库模块，包括复杂事件处理（CEP）、图处理（Gelly）和 Flink SQL 等高级库的实现和集成。

**flink-metrics**

- 提供度量和监控支持的模块，包括与各种监控系统（如 DataDog、Graphite、Prometheus 等）集成的度量报告。

**flink-optimizer**

- 作业优化器模块，用于优化 Flink 批处理作业的执行计划，提高作业的性能和效率。

**flink-python**

- Flink 的 Python API 模块，允许用户使用 Python 编写和提交 Flink 作业，提供了与 Java API 类似的功能和接口。

**flink-queryable-state**

- 查询状态模块，允许用户查询运行中的 Flink 作业的状态信息，便于实时监控和分析。

**flink-quickstart**

- 包含快速入门示例项目和模板，帮助用户快速上手并体验 Flink 的基本功能和特性。

**flink-rpc**

- 远程过程调用（RPC）模块，提供了 Flink 内部组件和服务之间通信的基础设施，支持各种 RPC 协议和实现。

**flink-runtime**

- 运行时核心功能模块，包括作业管理、任务调度、数据交换和状态管理等核心执行引擎的实现。

**flink-runtime-web**

- 运行时 Web 界面模块，提供了 Flink 集群的 Web 界面，用于监控和管理 Flink 应用程序和集群的状态。

**flink-scala**

- Flink 的 Scala API 模块，提供了用于开发和执行 Scala 版本的 Flink 作业的 API 和工具类。

**flink-scala-shell**

- Scala Shell 模块，提供了一个交互式 Scala 环境，使用户可以在命令行中进行 Flink Scala API 的交互式开发和测试。

**flink-state-backends**

- 状态后端模块，用于管理和存储 Flink 作业的状态信息，支持不同的状态存储后端（如 RocksDB、内存、文件系统等）。

**flink-streaming-java**

- 流处理核心实现的 Java 版本模块，包括流数据处理引擎、窗口操作、流数据转换等核心功能的实现。

**flink-streaming-scala**

- 流处理核心实现的 Scala 版本模块，提供了对流处理核心功能的 Scala API 支持。

**flink-table**

- Flink SQL 和 Table API 的核心模块，提供了在 Flink 中使用 SQL 查询和操作数据表的功能和接口。

**flink-test-utils-parent**

- 测试工具模块的父模块，定义了测试工具模块的基本配置和依赖管理。

**flink-tests**

- 各种集成测试模块，用于验证 Flink 不同功能和组件的集成情况和正确性。

**flink-walkthroughs**

- 教程示例模块，提供了一系列用于学习和探索 Flink 特性和使用方法的实用示例和教程。

**flink-yarn**

- YARN 集成模块，提供了在 Apache YARN 上部署和运行 Flink 应用程序的支持和功能。

**flink-yarn-tests**

- YARN 集成测试模块，用于测试在 YARN 上部署