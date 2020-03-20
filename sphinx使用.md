### sphinx功能

* 主要功能：

  * 高索引和搜索性能；
  * 先进的索引和查询工具（灵活且功能丰富的文本标记器，查询语言，集中不同的排名模式等）；
  * 先进的结果集后处理（对文本搜索结果进行表达式选择，where，order by ，group by ，having 等）；
  * 通过验证的可扩展性，每秒可扩展至数十亿个文档，数TB数据和数千个查询；
  * 易于与SQL和XML数据源以及SphinxQL，SphinxAPI或SphinxSE搜索接口集成；
  * 通过分布式搜索轻松扩展；

* 扩展点：

  * 具有很高的索引速度（根据内部基准，每个内核的速度高达10-15MB/秒）

  * 具有很高的搜索速度（针对上千万个文档，内部基准上有1.2GB数据，每个内核最高可达150-250个查询/秒）；

  * 具有高扩展性（已知的最大群集索引超过3,000,000,000个文档，最繁忙的一个高峰每天超过50,000,000个查询）；

  * 通过组合词组接近度排名和统计（BM25）排名提供良好的相关性排名；

  * 提供分布式搜索功能；

  * 提供文档摘要生成；

  * 提供具有SphinxQL或SphinAPI接口的应用程序中进行搜索，以及从具有可插拔的SphinxSE存储引擎的MySQL中进行搜索的功能；

  * 支持布尔，短语，单词接近度和其他类型的查询；

  * 每个文档支持多个全文本字段（默认情况下为32个）；

  * 每个文档支持多个其他属性（组，时间戳等）；

  * 支持停用词；

  * 支持形态词形式字典；

  * 支持标记化异常；

  * 支持UTF-8编码；

  * 支持词干（libstemmer库）

    [libstemmer库]: http://sphinxsearch.com/wiki/doku.php?id=charset_tables

  * 支持MYSQL（支持所有类型的表，包括MyISAM，InniDB，NDB，Archive等）

  * 本机支持符合 ADBC的数据库（MS  SQL，Oracle等）

### sphinx官网

* http://sphinxsearch.com/
* 压缩包包括：
  * indexer:创建全文索引的实用程序
  * searchd:一个守护程序，他使外部软件（web 程序）能够搜索全文索引
  * sphinxapi:一组搜索的客户端API库，用于流行的web脚本语言
  * spelldump：一个简单的命令行工具，可从ispellorMySpell格式字典（与OpenOffice捆绑在一起）中提取项目，以帮助自定义索引，以与wordforms一起使用
  * indextool:一个实用程序，用于转储有关索引的各种调试信息；
  * wordbreaker：一个在2.1.1版本中添加的将复合词分解为单独词的实用程序

### sphinx使用指南

假定sphinx安装在/usr/local/sphinx 因此searchd可以找到/usr/local/sphinx/bin/searchd

要使用sphinx，需要：

1. 创建一个配置文件，
   1. 默认配置文件名为sphinx.conf 默认情况下，所有sphinx程序都在当前工作目录中查找此文件。
   2. 由配置文件sphinx.conf.dist创建了示例配置文件，其中记录了所有选项configure。复制并编辑该示例文件以进行自己的配置：（假设sphinx已安装到/usr/local/sphinx）
      1. `cd /usr/local/sphinx/etc`
      2. `cp sphinx.conf.dist  sphinx.conf`
      3. `vi sphinx.conf`
      4. documents 从Mysql数据库设置示例配置文件到索引表；因此，有一个example.sql  示例数据文件可使用一些文档填充该表以进行测试；
      5. `mysql -u test < /usr/local/sphinx/etc/example.sql`
2. 运行索引器从您的数据创建全文索引：
   1. `cd /usr/local/sphinx/etc`
   2. `/usr/local/sphinx/bin/indexer --all`
3. 查询新创建的索引
   1. 链接服务器
      1. `mysql -h 0 -P 9306`
      2. `SELECT  * FROM test1 WHERE MACTH('my document')`
      3. `INSERT INTO rt VALUES (1,'this is ', 'a  sample text ', 11)`
      4. `SELECT gid/11 FROM rt WHERE MACTH(text) GROUP BY gid`
      5. `SELECT * FROM rt ORDER BY gid DESC`
      6. `SHOW TABLES`
      7. `SELECT * , WEIGHT() FROM test1 WHERE MATCH('"document  one "/1'); SHOW META` 
      8. `SET profiling= 1;SELECT * FROM test1 WHEE id IN (1,2,4);SHOW PROFILE`
      9. `SELECt id ,id % 3 idd FROM test1 WHERE MATCH('this is | nothing ') GROUP BY idd; SHOW PROFILE`
      10. `CALL KEYWORDS('one  two  THREE ', 'test1')`

### sphinx匹配模式

* SPH_MATCH_ALL 匹配所有查询分词（默认模式）
  * 如"苹果手机"，不匹配"我有一个手机"，但可以匹配"我有一个手机，它是苹果品牌"。
  * 因为"苹果手机"被分成了"苹果"，"手机" 两个单词，匹配的条件是必须同时包含这两个词，所以"我有一个手机"不符合匹配要求
* SPH_MATCH_ANY 匹配查询词中的任意一个分词
  * 如"苹果手机"，会匹配"我有一个苹果"，因为索引库中包含任一个分词即可被搜到，当然也能匹配"我有一个手机，它是苹果品牌"
* SPH_MATCH_PHRASE 将整个查询看作一个词组，要求按顺序完整匹配；
  * 这个与MYSQL中的"select * from tab where keyword like '%苹果手机%'  "类似，如"华为手机"，不匹配"我有一个手机是华为"，可以匹配"我有一个华为手机"
* SPH_MATCH_BOOLEAN,将查询看作一个布尔表达式，可以简单的与或非运算
  * 如(apple !banana)|(apple !pear)
  * 意思是搜索出所有匹配apple，但不匹配banana和pear的查询分词
* SPH_MATCH_EXTENDED 2 扩展匹配模式
  * 将查询看作一个SPHINX内部语言的表达式
  * 或(OR)操作符
    * hello | word
  * 非(NOT)操作符
    * hello -world
    * hello !world
  * 字段(field)搜索
    * @title hello @body world
    * title 中包含hello ; body字段中包含world
  * 字段限位修饰符
    * @body[50] hello
      * body字段位数限制在50以内
  * 多字段搜索符
    * @(title,body) hello world
      * title 或 body 字段中包含hello world
  * 全字段搜索
    * @* hello
      * 只要其中一列包含hello
  * 近似搜索符
    * "hello world"~10
      * hello world 之间最多有10个词
  * 阀值匹配符
    * "the world is wonderful place"/3
    * 至少匹配3个词
  * 严格有序搜索
    * aaa<<bbb<<ccc
    * aaa bbb ccc 必须按先后顺序出现
  * 字段开始和字段结束修饰符
    * ^hello....world$
    * 限定字符以hello开头，以world结尾

