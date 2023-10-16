# ElasticSearch基本使用



## 1-准备以及安装

### 1.1 安装docker

```bash
#设置系统内核参数，否则会因为内存不足无法启动
# 查看内核max_map_count参数值，默认为65530
cat /proc/sys/vm/max_map_count
 
# 重新设置max_map_count的值
sysctl -w vm.max_map_count=262144
# 立即生效
sysctl -p
```

### 1.2 修改配置文件

```bash
mkdir -p /mydata/elasticsearch/config
mkdir -p /mydata/elasticsearch/data/
mkdir -p /mydata/elasticsearch/plugins
echo "http.host: 0.0.0.0" >> /mydata/elasticsearch/config/elasticsearch.yml
chmod -R 777 /mydata/elasticsearch/
```

### 1.3 运行

```bash
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 
-e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms512m -Xmx512m" 
-v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml 
-v /mydata/elasticsearch/data/:/usr/share/elasticsearch/data 
-v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins 
-d elasticsearch:7.7.0

# 复制下面这段可直接运行，由于格式原因复制上面的可能运行会被断开导致命令不完整
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /mydata/elasticsearch/data/:/usr/share/elasticsearch/data -v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins -d elasticsearch:7.7.0

#参数说明
--name表示镜像启动后的容器名称  

-d: 后台运行容器，并返回容器ID；

-e: 指定容器内的环境变量

-p: 指定端口映射，格式为：主机(宿主)端口:容器端口

discovery.type=single-node：单机运行

如果启动不了，可以加大内存设置：-e ES_JAVA_OPTS="-Xms512m -Xmx512m"
```

### 1.4 处理跨域

```bash
#进入容器
docker exec -it elasticsearch(或者容器id) /bin/bash

#找到配置文件
vim config/elasticsearch.yml

#把这两行写进配置文件中(注意是yml配置文件)：
http.cors.enabled: true 
http.cors.allow-origin: "*"

#配置修改完后需执行命令exit退出容器，接着执行docker restart 容器ID重启容器即可。
```

### 1.5 安装IK分词器

下载ik分词器压缩包：[ik分词器:7.7.0](https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.7.0/elasticsearch-analysis-ik-7.7.0.zip)

```bash
--第一种方法
#将压缩包移动到容器中
docker cp /tmp/elasticsearch-analysis-ik-7.7.0.zip elasticsearch:/usr/share/elasticsearch/plugins
#如果有xftp工具直接下载后，连接并移到远程服务器上

#进入容器
docker exec -it elasticsearch /bin/bash  

#创建目录
mkdir /usr/share/elasticsearch/plugins/ik

#将文件压缩包移动到ik中
mv /usr/share/elasticsearch/plugins/elasticsearch-analysis-ik-7.7.0.zip /usr/share/elasticsearch/plugins/ik

#进入目录
cd /usr/share/elasticsearch/plugins/ik

#解压
unzip elasticsearch-analysis-ik-7.7.0.zip

#删除压缩包
rm -rf elasticsearch-analysis-ik-7.7.0.zip

--第二种远程下载（不稳定）
wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.7.0/elasticsearch-analysis-ik-7.7.0.zip

```

使用第二种方式

执行cd /usr/share/elasticsearch/plugins来到插件目录创建一个IK目录。

将压缩包移动到IK目录中，执行解压指令elasticsearch-analysis-ik-7.7.0.zip

接着删除压缩包即可，此时你可以看到一个config包和几个jar包

最后退出容器，重启重启容器即可。



## 2-基本使用



### 2.1 **spring-boot-starter-data-elasticsearch 方式**

#### 2.1.1配置文件

```xml
#引入依赖
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>

#配置文件
spring:
  elasticsearch:
    rest:
      uris: 0.0.0.0:9200

```



#### 2.1.2 SpringBoot中使用

##### 2.1.2.1声明索引，自动创建，操作前的操作

@Document表示该为一个文档，@Id不用解释了，@Field里的type为类型，若要支持分词器查询需要设置为Text类型，analyzer指的是分词器类型这里的ik分词器支持中文的分词

```java
@Document(indexName = "user")
public class User implements Serializable {

    @Id
    private String uId;
	
    @Field(type = FieldType.Text,analyzer = "ik_max_word")
    private String name;
    
    @Field(type = FieldType.KeyWord)
    private Integer age;

    @Field(type = FieldType.Text,analyzer = "ik_max_word")
    private String address;
    
    //省略getter setter 
}
```



写一个接口UserRepository继承ElasticsearchRepository，ElasticsearchRepository包含了基本的增删改查的能力，并在接口类上加上@Repository注解，注入到spring [容器](https://cloud.tencent.com/product/tke?from_column=20065&from=20065)中。

```java
@Repository
public interface UserRepository extends ElasticsearchRepository<User, String> {

}
```

方法名支持的关键字如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020122112010389.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d5YnNoZW4=,size_16,color_FFFFFF,t_70)

##### 2.1.2.2一些基本操作

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserRepository userRepository;


    @Override
    public long count() {
        return userRepository.count();
    }

    @Override
    public User save(User user) {
        return userRepository.save(user);
    }

    @Override
    public void delete(User user) {
        userRepository.delete(user);
    }

    @Override
    public Iterable<User> getAll() {
        return userRepository.findAll();
    }

    @Override
    public List<User> getByName(String name) {
        List<User> list = new ArrayList<>();
        MatchQueryBuilder matchQueryBuilder = new MatchQueryBuilder("name", name);
        Iterable<User> iterable = userRepository.search(matchQueryBuilder);
        iterable.forEach(e->list.add(e));
        return list;
    }

    //分页快速查询模板
    @Override
    public Page<User> pageQuery(Integer pageNo, Integer pageSize, String kw) {
        NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
                .withQuery(QueryBuilders.matchPhraseQuery("name", kw))
                .withPageable(PageRequest.of(pageNo, pageSize))
                .build();
        return userRepository.search(searchQuery);
    }
}
```

##### 2.1.2.3NativeSearchQuery

```java
//下面是NativeSearchQuery的一些内部属性，基本上都是ES的一些内部对象
	//查询条件，查询的时候，会考虑关键词的匹配度，并按照分值进行排序
	private QueryBuilder query;
	//查询条件，查询的时候，不考虑匹配程度以及排序这些事情
	private QueryBuilder filter;
	//排序条件的builder
	private List<SortBuilder> sorts;
	private final List<ScriptField> scriptFields = new ArrayList<>();
	private CollapseBuilder collapseBuilder;
	private List<FacetRequest> facets;
	private List<AbstractAggregationBuilder> aggregations;
	//高亮显示的builder
	private HighlightBuilder highlightBuilder;
	private HighlightBuilder.Field[] highlightFields;
	private List<IndexBoost> indicesBoost;

```

###### QueryBuilders

之前按在**NativeSearchQuery**的**withquery(QueryBuilders.matchPhraseQuery("name", kw))**可以看到是以**QueryBuliders**来操作的

**精确查询**

指定字符串作为关键词查询，关键词支持分词

```java
//查询title字段中，包含 ”开发”、“开放" 这个字符串的document；相当于把"浦东开发开放"分词了，再查询；
QueryBuilders.queryStringQuery("开发开放").defaultField("title");
//不指定feild，查询范围为所有feild
QueryBuilders.queryStringQuery("青春");
//指定多个feild
QueryBuilders.queryStringQuery("青春").field("title").field("content");

```

以关键字“开发开放”，关键字不支持分词

```java
QueryBuilders.termQuery("title", "开发开放")
QueryBuilders.termsQuery("fieldName", "fieldlValue1","fieldlValue2...")
```

以关键字“开发开放”，关键字支持分词

```java
QueryBuilders.matchQuery("title", "开发开放")
QueryBuilders.multiMatchQuery("fieldlValue", "fieldName1", "fieldName2", "fieldName3")
```

**模糊查询**

左右模糊查询，其中fuzziness的参数作用是在查询时，es动态的将查询关键词前后增加或者删除一个词，然后进行匹配

```java
QueryBuilders.fuzzyQuery("title", "开发开放").fuzziness(Fuzziness.ONE)
```

前缀查询，查询title中以“开发开放”为前缀的document；

```java
QueryBuilders.prefixQuery("title", "开发开放")
```

通配符查询，支持*和？，？表示单个字符；注意不建议将通配符作为前缀，否则导致查询很慢

```java
QueryBuilders.wildcardQuery("title", "开*放")
QueryBuilders.wildcardQuery("title", "开？放")
```

**注意，**
在分词的情况下，针对fuzzyQuery、prefixQuery、wildcardQuery不支持分词查询，即使有这种doucment数据，也不一定能查出来，因为分词后，不一定有“开发开放”这个词；

**范围查询**

```java
//闭区间查询
QueryBuilders.rangeQuery("fieldName").from("fieldValue1").to("fieldValue2");
//开区间查询，默认是true，也就是包含
QueryBuilders.rangeQuery("fieldName").from("fieldValue1").to("fieldValue2").includeUpper(false).includeLower(false);
//大于
QueryBuilders.rangeQuery("fieldName").gt("fieldValue");
//大于等于
QueryBuilders.rangeQuery("fieldName").gte("fieldValue");
//小于
QueryBuilders.rangeQuery("fieldName").lt("fieldValue");
//小于等于
QueryBuilders.rangeQuery("fieldName").lte("fieldValue");

```

**多个关键字组合查询boolQuery()**

```java
QueryBuilders.boolQuery()
QueryBuilders.boolQuery().must();//文档必须完全匹配条件，相当于and
QueryBuilders.boolQuery().mustNot();//文档必须不匹配条件，相当于not
QueryBuilders.boolQuery().should();//至少满足一个条件，这个文档就符合should，相当于or
```

**完整demo展示**

```java
NativeSearchQuery nativeSearchQuery = new NativeSearchQueryBuilder()
            .withQuery(QueryBuilders.boolQuery()
                    .should(QueryBuilders.termQuery("title", "开发"))
                    .should(QueryBuilders.termQuery("title", "青春"))
                    .mustNot(QueryBuilders.termQuery("title", "潮头"))
            )
            .withSort(SortBuilders.fieldSort("id").order(SortOrder.DESC))
            .withPageable(PageRequest.of(0, 50))
            .build();
```

注意排序时，有个坑，就是在以id排序时，比如降序，结果可能并不是我们想要的。因为根据id排序，es实际上会根据_id进行排序，但是_id是string类型的，排序后的结果会与整型不一致。



### 2.2 使用RestHighLevelClient 

### **2.3高亮查询**

```java
//使用前
    @Autowired
    private ElasticsearchRestTemplate elasticsearchRestTemplate;
//接下来使用elasticsearchRestTemplate操作高亮
String preTag = "<font color='#dd4b39'>";
    String postTag = "</font>";
    NativeSearchQuery nativeSearchQuery = new NativeSearchQueryBuilder()
            .withQuery(QueryBuilders.matchQuery("title", "开发"))
            .withPageable(PageRequest.of(0, 50))
            .withSort(SortBuilders.fieldSort("id").order(SortOrder.DESC))
            .withHighlightFields(new HighlightBuilder.Field("title").preTags(preTag).postTags(postTag))
            .build();

    AggregatedPage<ArticleEntity> page = elasticsearchRestTemplate.queryForPage(nativeSearchQuery, ArticleEntity.class,
            new SearchResultMapper() {
                @Override
                public <T> AggregatedPage<T> mapResults(SearchResponse response, Class<T> clazz, Pageable pageable) {
                    List<ArticleEntity> chunk = new ArrayList<>();
                    for (SearchHit searchHit : response.getHits()) {
                        if (response.getHits().getHits().length <= 0) {
                            return null;
                        }
                        ArticleEntity article = new ArticleEntity();
                        article.setMyId(Long.valueOf(searchHit.getSourceAsMap().get("id").toString()));
                        article.setContent(searchHit.getSourceAsMap().get("content").toString());
                        HighlightField title = searchHit.getHighlightFields().get("title");
                        if (title != null) {
                            article.setTitle(title.fragments()[0].toString());
                        }
                        chunk.add(article);
                    }
                    if (chunk.size() > 0) {
                        return new AggregatedPageImpl<>((List<T>) chunk);
                    }
                    return null;
                }

                @Override
                public <T> T mapSearchHit(SearchHit searchHit, Class<T> type) {
                    return null;
                }
            });


    List<ArticleEntity> articleEntities = page.getContent();
    articleEntities.forEach(item -> System.out.println(item.toString()));
```

