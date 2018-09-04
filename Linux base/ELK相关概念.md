# ELK相关概念

[TOC]



# lucene

lucene是apache基金会旗下的,是一个全文搜索引擎.

http://baike.baidu.com/link?url=beARWCbOSrvNCddEQ65j10KKVogstGyW_5S6BZ48Ls_mO9-htGnoPCW6ifqD1abA_-vwM2uTAunMXb3wWjJNb_

### document 

包含一个或者多个域的容器

​	域

​		field1:value1,field2:value2,field3:value3, .....



### 域

有许多选项

- 索引选项

  *索引选项用通过倒排索引来控制文本是否可被搜索*

  ```
  Index:ANYLYZED  //分析(切词)并单独作为索引项
  Index:Not_ANYLYZED //不切词,把整个内容当作一个索引项
  Index:ANYLYZED_NORMS //类似于Index:ANYLYZED,但是不存储token的Norms(加权)信息
  Index:Not_ANYLYZED_NORMS //类似于Index:ANYLYZED,但是不存储值的Norms(加权)信息
  Index:Not   //不对此域进行索引,因此不能被索引
  ```

  ​

- 存储选项

  *是否需要存储域的真实值*

  ```
  store.YES //存储真实值
  store.NO  //不存储真实值
  ```

  ​

- 项向量使用选项

  *用于在搜索期间该文档所有的唯一项都能完全从文档中检索时使用*



### 搜索

查询Lucene索引时,它返回一个有序的scoreDoc对象,查询时,Lucene会为每个文档计算其score

API:

IndexSearcher:搜索索引入口

- IndexSearcher中的search方法
    ```
     1. TermQuery : 可以对特定的域进行搜索,Term是索引中的最小索引片段, Term包含:field:value
     2. TermRangQuery:可以在多个特定项进行搜索
     3. NumericRangeQuery:数值范围搜索
     4. PrefixQuery:搜索指定字符串开头的项
     5. BooleanQuery:组合查询 AND , OR , NOT
     6. PhraseQuery:
     7. WildcardQuery
     8. FuzzyQuery:模糊查询
    ```

- query

- # QueryPharse

- TopDocs

- ScoreDoc



# ElasticSearch

引自百度百科

> ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。



## 基本概念

**索引(index)**:在Elasticsearch中存储数据的行为就叫做**索引(indexing)**,是个文档容器,索引名必须用小写字母

在Elasticsearch中，文档归属于一种**类型(type)**,而这些类型存在于**索引(index)**中，我们可以画一些简单的对比图来类比传统关系型数据库：

+

```
Relational DB -> Databases -> Tables -> Rows -> Columns
Elasticsearch -> Indices   -> Types  -> Documents -> Fields

```

Elasticsearch集群可以包含多个**索引(indices)**（数据库），每一个索引可以包含多个**类型(types)**（表），每一个类型包含多个**文档(documents)**（行），然后每个文档包含多个**字段(Fields)**（列）

**映射(mapping)**:定义如何分析和切词的方法

## ES集群

集群Cluster:节点的集合是集群,集群的默认名称是elasticsearch,节点是靠集群名称来识别,加入集群和删除集群也靠集群名称

**node**:一个**节点(node)**就是一个Elasticsearch实例

**share**:将索引切割成为物理存储组件,每一个shard都是一个独立完整的索引,一个**分片(shard)**是一个最小级别**“工作单元(worker unit)”**,它只是保存了索引中所有数据的一部分

- primary shard
- replica 用于冗余和备份,shard的副本数量可以自定义

### ES Cluster启动

启动的时候,集群中的节点通过多播或者单播的方式在9300/tcp查找集群的名称.以此来确定同一网络中的其他节点,启动之后,节点会选举出一个主节点(master),临时管理集群级别的一些变更，例如新建或删除索引、增加或移除节点等。主节点不参与文档级别的变更或搜索，这意味着在流量增长的时候，该主节点不会成为集群的瓶颈。任何节点都可以成为主节点



### 集群的状态

**集群健康(cluster health)**。集群健康有三种状态：`green`、`yellow`或`red`

| 颜色       | 意义                    |
| -------- | --------------------- |
| `green`  | 所有主要分片和复制分片都可用        |
| `yellow` | 所有主要分片可用，但不是所有复制分片都可用 |
| `red`    | 不是所有的主要分片都可用          |



ES端口

参与集群事物:9300/tcp

接受请求:9200/tcp



Restful API:

四类API

- 检查集群,节点.索引等是否健康,以及获取其相应的状态
- 管理集群,节点,索引和元数据
- 执行crud操作
- 执行高级操作,例如paging filtering等



cluster APIs

```
health
	curl -XGET 'http://127.0.0.1:9200/_cluster/health?pretty'
state
	curl -XGET 'http://127.0.0.1:9200/_cluster/state/<metrics>?pretty'
stats
	curl -XGET 'http://127.0.0.1:9200/_cluster/state'
	curl -XGET 'http://127.0.0.1:9200/_nodes/stats'
```



## 访问

```
curl -x<VERB> '<PROTOCOL>://HOST:PORT/<PATH>?<QUERY_STRING>' -d'<BODY>'
	VERB:GET PUT DELETE等
	PROTOCOL:http https
	PATH:API 的终端路径（例如 _count 将返回集群中文档数量）。Path 可能包含多个组件，例如：	          _cluster/stats 和 _nodes/stats/jvm 。
	QUERY_STING:查询参数,例如?pretty表示用易读格式输出
	BODY:请求的主体
```



## 插件

安装

- 直接放在plugin目录中

- 使用plugin脚本安装

  ```
  /usr/share/elasticsearch/bin/plugin -h  //-h来获取帮助
  	-l ,list列出
  	-i ,--install 安装
  ```

站点插件

```
http://HOST:9200/_plugin/plugin_name
```



## 查询数据

Query API

Query DSL:Query Domain Search Language,是一种JSON格式的,可以实现多种查询,比如simple term query, phrase,range boolean fuzzy

查询的方式

- 通过Restful request API查询

  ```
  curl -XGET 'http://localhost:9200/_search?pretty'
  ```

- 通过发送rest request body进行

  ```
  curl -XGET 'http://localhost:9200/_search?pretty' -d'
  {
  	"query":{"match_all": {}}
  }'
  ```



多索引,多类型查询

- /_search:所有索引
- /INDEX_NAME/_search:单索引
- /INDEX_NAME1,INDEX_NAME2/_search:多索引
- /INDEX_NAME/TYPE/_search单类型搜索
- INDEX_NAME/TYPE1,TYPE2/_search,多类型搜索



### mapping和analysis

```
GET /_search?q='elk'  //全文_all域搜索elk
GET /_search?q=store:'elk' //在store中搜索elk
```

分析需要分析器进行 analyzer

组件

- 字符过滤器
- 分词器
- 分词过滤器

ES内置的分析器

- standard analyzer
- simple analyzer
- whitespace analyzer
- language analyzer



query DSL

分成2类

- query dsl:执行full-text查询,基于相关度匹配结果

  查询执行过程复杂 且不会被缓存

- filter dsl 执行exact查询时,基于结果yes和no进行评估

  速度快,可缓存



查询语句语法检查

```
GET /INDEX/_validate/query?pretty
{
	.....
}

GET /INDEX/_validate/query?explain&pretty
{
	.....
}

```



# Logstash

支持多数据获取

```
配置框架
input {
	...
}

filter {
	...
}

output {
	...
}
```

四种类型的插件

input , filter , codec , output

数据类型

- array: [item1,item2,...]
- boolean : true , false
- bytes
- codec:编码器
- hash:key => value
- number
- password
- path
- string

字段引用: []

条件判断:== ,!= ,> ,>= ,<=,=~,



Logstash的工作流程

input \==>filter\==> output (如果没有需要处理数据,filter可以省略)



## Logstash的插件

配置好以后测试和运行配置文件的语法

```
logstash -f PATH --configtest //PATH是指.conf的配置文件  这个配置文件全部在conf.d文件夹下面
logstash -f PATH //运行配置好的配置文件
```



### input plugin

- *File:从指定的文件读取事务,使用filewatch监听文件变化(官方文档https://www.elastic.co/guide/en/logstash/current/plugins-inputs-file.html)*

  下面是个例子

  ```
  input {
  	file {
  		path =>["var/log/messages"]
  		type =>"system"
  		start_position =>"beginning"
  		}
  }
  output {
  	stdout {
  		codec =>rubydebug
  		}
  }
  ```

- udp

  *通过udp协议从网络监听messages,必须的参数是`prot` 和`host` ,`port` 是监听自己的端口,`host` 指明自己监听的地址*

  *官方学习文档https://www.elastic.co/guide/en/logstash/current/plugins-inputs-udp.html*

  ​

  collectd:性能监控工具

  ```
  yum -y install collectd //安装collectd
  vim /etc/collectd.conf
  	启用Hostname
  	启用<Plugin network> 修改其中的server和端口就好,server指的是logstash的主机地址,端口指的是监听的udp端口
  		
  ```

  下面是个例子

  ```
  input {
  	udp {
  		port =>25826
  		codec =>collectd {}
  		type =>"collectd"
  		}
  }
  output {
  	stdout {
  		codec =>rubydebug
  		}
  }
  ```

- redis插件

  *从redis读取数据,支持redis channel和lists*

### filter plugin

*将event通过output发出之前进行处理*



grok :分析并结构化文本数据,是logstash将非结构化数据转化为结构化的可查询数据的不二之选

官方文档地址https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html

​	预定义模式文件位置:/opt/logstash/vendor/bundle/jruby/1.9/gems/logstash-patterns-core-	   0.3.0/patterns/grok-patterns(使用命令`rpm -ql logstash |grep "patterns$"`查找)

语法格式

```
%{SYNTAX:SEMANTIC}
	SYNTAX:预定义模式名称
	SEMANTIC:匹配到文本的自定义标识符
	
比如
55.3.244.1 GET /index.html 15824 0.043

写成pattern就成了
%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}

写配置文件
------------------------------------------------------------------------
input {
  file {
    path => "/var/log/http.log"
  }
}
filter {
  grok {
    match => { "message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}" }
  }
}
------------------------------------------------------
After the grok filter, the event will have a few extra fields in it:
client: 55.3.244.1
method: GET
request: /index.html
bytes: 15824
duration: 0.043
```

自定义grok模式

语法格式

```
?<field_name>the pattern here)

引用PATTERN_NAME(?the pattern here)
```

# kibana

官网文档https://www.elastic.co/guide/en/kibana/current/upgrade.html





