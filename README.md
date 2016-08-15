# elasticsearch
elasticsearch分布式搜索配置及使用(一下均为2.3以后版本配置)
#配置文件说明：
    配置文件位于%ES_HOME%/config/elasticsearch.yml文件中，用Editplus打开它，你便可以进行配置。

    所有的配置都可以使用环境变量，例如：
    node.rack: ${RACK_ENV_VAR}
    表示环境变量中有一个RACK_ENV_VAR变量。
    下面列举一下elasticsearch的可配置项：
    1. 集群名称，默认为elasticsearch：
    cluster.name: elasticsearch
    2. 节点名称，es启动时会自动创建节点名称，但你也可进行配置：
    node.name: "Franz Kafka"
    3. 是否作为主节点，每个节点都可以被配置成为主节点，默认值为true：
    node.master: true
    4. 是否存储数据，即存储索引片段，默认值为true：
    node.data: true
    master和data同时配置会产生一些奇异的效果：
    1) 当master为false，而data为true时，会对该节点产生严重负荷；
    2) 当master为true，而data为false时，该节点作为一个协调者；
    3) 当master为false，data也为false时，该节点就变成了一个负载均衡器。
    你可以通过连接http://localhost:9200/_cluster/health或者http://localhost:9200/_cluster/nodes，或者使用插件http://github.com/lukas-vlcek/bigdesk或http://mobz.github.com/elasticsearch-head来查看集群状态。
    5. 每个节点都可以定义一些与之关联的通用属性，用于后期集群进行碎片分配时的过滤：
    node.rack: rack314
    6. 默认情况下，多个节点可以在同一个安装路径启动，如果你想让你的es只启动一个节点，可以进行如下设置：
    node.max_local_storage_nodes: 1
    7. 设置一个索引的碎片数量，默认值为5：
    index.number_of_shards: 5
    8. 设置一个索引可被复制的数量，默认值为1：
    index.number_of_replicas: 1
    当你想要禁用公布式时，你可以进行如下设置：
    index.number_of_shards: 1
    index.number_of_replicas: 0
    这两个属性的设置直接影响集群中索引和搜索操作的执行。假设你有足够的机器来持有碎片和复制品，那么可以按如下规则设置这两个值：
    1) 拥有更多的碎片可以提升索引执行能力，并允许通过机器分发一个大型的索引；
    2) 拥有更多的复制器能够提升搜索执行能力以及集群能力。
    对于一个索引来说，number_of_shards只能设置一次，而number_of_replicas可以使用索引更新设置API在任何时候被增加或者减少。
    ElasticSearch关注加载均衡、迁移、从节点聚集结果等等。可以尝试多种设计来完成这些功能。
    可以连接http://localhost:9200/A/_status来检测索引的状态。
    9. 配置文件所在的位置，即elasticsearch.yml和logging.yml所在的位置：
    path.conf: /path/to/conf
    10. 分配给当前节点的索引数据所在的位置：
    path.data: /path/to/data
    可以可选择的包含一个以上的位置，使得数据在文件级别跨越位置，这样在创建时就有更多的自由路径，如：
    path.data: /path/to/data1,/path/to/data2
    11. 临时文件位置：
    path.work: /path/to/work
    12. 日志文件所在位置：
    path.logs: /path/to/logs
    13. 插件安装位置：
    path.plugins: /path/to/plugins
    14. 插件托管位置，若列表中的某一个插件未安装，则节点无法启动：
    plugin.mandatory: mapper-attachments,lang-groovy
    15. JVM开始交换时，ElasticSearch表现并不好：你需要保障JVM不进行交换，可以将bootstrap.mlockall设置为true禁止交换：
    bootstrap.mlockall: true
    请确保ES_MIN_MEM和ES_MAX_MEM的值是一样的，并且能够为ElasticSearch分配足够的内在，并为系统操作保留足够的内存。
    16. 默认情况下，ElasticSearch使用0.0.0.0地址，并为http传输开启9200-9300端口，为节点到节点的通信开启9300-9400端口，也可以自行设置IP地址：
    network.bind_host: 192.168.0.1
    17. publish_host设置其他节点连接此节点的地址，如果不设置的话，则自动获取，publish_host的地址必须为真实地址：
    network.publish_host: 192.168.0.1
    18. bind_host和publish_host可以一起设置：
    network.host: 192.168.0.1
    19. 可以定制该节点与其他节点交互的端口：
    transport.tcp.port: 9300
    20. 节点间交互时，可以设置是否压缩，转为为不压缩：
    transport.tcp.compress: true
    21. 可以为Http传输监听定制端口：
    http.port: 9200
    22. 设置内容的最大长度：
    http.max_content_length: 100mb
    2 3 . 禁止HTTP
    http.enabled: false
    2 4 . 网关允许在所有集群重启后持有集群状态，集群状态的变更都会被保存下来，当第一次启用集群时，可以从网关中读取到状态，默认网关类型（也是推荐的）是local：
    gateway.type: local
    2 5 . 允许在N个节点启动后恢复过程：
    gateway.recover_after_nodes: 1
    2 6 . 设置初始化恢复过程的超时时间：
    gateway.recover_after_time: 5m
    2 7 . 设置该集群中可存在的节点上限：
    gateway.expected_nodes: 2
    2 8 . 设置一个节点的并发数量，有两种情况，一种是在初始复苏过程中：
    cluster.routing.allocation.node_initial_primaries_recoveries: 4
    另一种是在添加、删除节点及调整时：
    cluster.routing.allocation.node_concurrent_recoveries: 2
    2 9 . 设置复苏时的吞吐量，默认情况下是无限的：
    indices.recovery.max_size_per_sec: 0
    30 . 设置从对等节点恢复片段时打开的流的数量上限：
    indices.recovery.concurrent_streams: 5
    3 1 . 设置一个集群中主节点的数量，当多于三个节点时，该值可在2-4之间：
    discovery.zen.minimum_master_nodes: 1
    3 2 . 设置ping其他节点时的超时时间，网络比较慢时可将该值设大：
    discovery.zen.ping.timeout: 3s
    http://elasticsearch.org/guide/reference/modules/discovery/zen.html上有更多关于discovery的设置。
    3 3 . 禁止当前节点发现多个集群节点，默认值为true：
    discovery.zen.ping.multicast.enabled: false
    3 4 . 设置新节点被启动时能够发现的主节点列表（主要用于不同网段机器连接,不同IP之间需要设置，用于分布式配置）：
    discovery.zen.ping.unicast.hosts: ["host1", "host2:port", "host3[portX-portY]"]
    35.设置是否可以通过正则或者_all删除或者关闭索引
    action.destructive_requires_name 默认false 允许 可设置true不允许
    index.mapper.dynamic : false
    关闭动态创建索引
    
    
#中文分词elasticsearch-ik安装：
    Elasticsearch中，内置了很多分词器（analyzers），例如standard （标准分词器）、english（英文分词）和chinese （中文分词）。其中standard 就是无脑的一个一个词（汉字）切分，所以适用范围广，但是精准度低；english 对英文更加智能，可以识别单数负数，大小写，过滤stopwords（例如“the”这个词）等；chinese 效果很差，后面会演示。这次主要玩这几个内容：安装中文分词ik，对比不同分词器的效果，得出一个较佳的配置。关于Elasticsearch，之前还写过两篇文章：Elasticsearch的安装，运行和基本配置 和 备份和恢复，需要的可以看下: 
    Elasticsearch的中文分词很烂，所以我们需要安装ik。首先从github上下载项目，解压：
    cd /tmp
    wget https://github.com/medcl/elasticsearch-analysis-ik/archive/master.zip
    unzip master.zip
    cd elasticsearch-analysis-ik/
    然后使用mvn package 命令，编译出jar包 elasticsearch-analysis-ik-1.4.0.jar。
    需要下载对应的elasticsearch ik版本源码 1.9.0 对应版本2.3.1
    mvn package
    将target/releases/下所有文件包复制到Elasticsearch的plugins/analysis-ik 目录下，再把解压出的ik目录（配置和词典等），复制到Elasticsearch的config 目录下。然后编辑配置文件elasticsearch.yml ，在后面加一行：
    index.analysis.analyzer.ik.type : "ik"
    最后重启即可
    
#elasticsearch映射创建说明：
    curl -XPUT http://10.26.38.122:9200/member_msg
    {
    "mappings": {
        "vote_redpack_votelog": {
            "properties": {
                "unionId": {
                    "type": "string",
                    "index": "not_analyzed"
                },
                "toUnionId": {
                    "type": "string",
                    "index": "not_analyzed"
                },
                "nickName": {
                    "type": "string",
                    "index": "not_analyzed"
                },
                "headImgUrl": {
                    "type": "string",
                    "index": "no"
                },
                "ctime": {
                    "type": "date"
                },
                "toHeadImgUrl": {
                    "type": "string",
                    "index": "no"
                },
                "huodongId": {
                    "type": "integer"
                },
                "location": {
                    "type": "geo_point"
                },
                "toNickName": {
                    "type": "string",
                    "index": "not_analyzed"
                },
                "ip": {
                    "type": "string",
                    "index": "not_analyzed"
                },
                "locationCity": {
                    "type": "string",
                    "analyzer": "ik"
                },
                "ipCity": {
                    "type": "string",
                    "index": "not_analyzed"
                }
            }
        }
    }
    }
    
    2、简单的统计：
      //每分钟发出的红包个数
    {
        "query": {
            "range": {
                "ctime": {
                    "to": 1464796799000,
                    "from": 1464710400000
                }
            }
        },
        "aggs": {
            "moneys": {
                "date_histogram": {
                    "field": "ctime",
                    "interval": "hour",
                    "time_zone": "+08:00",
                    "format":"yyyy-MM-dd HH:mm:ss"
                },
                "aggs": {
                    "make_money": {
                        "terms": {"field": "money"}
                    }
                }
            }
        }
      }
  3、数据同步到elasticsearch
    1)采用mysql同步使用官方mysql-jdbc效率比较低，目前采用批量数据读取然后使用es的bluk批量导入功能效果还是不错的
    2)是否可以将elasticsearch作为数据库，答案是可以的，给予elasticsearch的分布式横向扩展功能，我们无需关心分片，只要将每个索引的分片
      建立合理的个数即可，建议单个机器只启动一个node
