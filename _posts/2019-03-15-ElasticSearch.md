---
layout:     post
title:      ElasticSearch
subtitle:   搜索
date:       2019-03-15
author:     Francis
header-img: image/bg-desk.jpeg
catalog: true
tags:
    - ElasticSearch
---

# ElasticSearch

## 參考文檔

* [手冊](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)

* [guide](https://www.elastic.co/guide/index.html)

  

## 基礎應用

* ElasticSearch是開源的、分佈式的、可擴展的、實時的搜索與數據分析引擎。

* ElasticSearch是面向文檔的（存儲整個對象或文檔），每個文檔的內容可以被檢索。  json作為文檔序列化格式



### 安裝與運行

1. 安裝Java JDK（建議：較新版本）

   [官方提供](https://blog.csdn.net/vvv_110/article/details/72897142)    [mac可參考](https://blog.csdn.net/vvv_110/article/details/72897142)

2. 安裝Elasticsearch

   * 從[官網](https://www.elastic.co/downloads/elasticsearch) 安裝適合你操作系統的ElasticSearch

   * 我用mac安裝

   * ~~~
     #brew 安裝
     brew install elasticsearch
     
     #安裝完成后，可以輸入下面命令查看信息：
     brew info elasticsearch
      
     #可在瀏覽器中查看：
     http://localhost:9200
     #在終端查看
     curl 'http://localhost:9200/?pretty'
     
     #現在已經啟動并運行了一個ElasticSearch節點了
     ~~~

3. 安裝kibana

   * 用mac安裝

   * ~~~
     brew install kibana
     
     #安裝完成后，可以輸入下面命令查看信息：
      brew info kibana
      
     #可在瀏覽器中查看：
      http://localhost:5601
     ~~~

   * kibana常用功能

     * DevTools開發者工具

     * Discover數據搜索查看
     * Visualize圖表製作
     * Dashboard儀錶盤製作
     * Managerment 配置管理
     * Timelion 時序數據的高級可視化分析

### 目錄

~~~
安装目录：/usr/local/Cellar/elasticsearch/{elasticsearch-version}/
日志目录：/usr/local/var/log/elasticsearch/
插件目录：/usr/local/var/elasticsearch/plugins/
配置目录：/usr/local/etc/elasticsearch
~~~



### 與ElasticSearch通訊

* 語法 

  `curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'`

  * `VERB`: 適當的HTTP方法或者 `GET`、`POST`、`PUT`、`HEAD`、`DELETE`
  * `PROTOCOL`: ` http` 或`https`
  * `HOST`: ElasticSearch 集群中任意節點的主機名，或者用`localhost`代表本地機器節點
  * `PORT`: ElasticSearch HTTP服務端口號，默認`9200`
  * `PATH`: API的終端路徑 如`_cluster/stats`
  * `QUERY_STRING`: 任意可選的查詢字符串參數， 如： `?pretty`將格式化地輸出JSON返回值，更容易閱讀
  * `BODY`: 一個json格式的請求體

* 例子：

  ~~~
  //查看本地集群 文檔 數
  
  curl -i GET "localhost:9200/bank/_count?pretty" -H 'Content-Type: application/json' -d'
  {
     "query": { "match_all": {} }
    }
   '
  
  ~~~



###  用案例解釋概念

#### 有一個關於‘大學生信息’需求

* 檢索任意一大學生的完整信息
* 允許結構化搜索，比如查詢25歲以上的大學生
* 允許簡單的全文索引以及較複雜的短語索引
* 支持在匹配文檔內容中高亮顯示搜索片段
* 支持基於數據創建和管理分析儀錶盤

#### 存儲學生信息

*  概念

  <font color="blue">一個ElasticSearch集群《包含》多個索引； 每個索引《包含》多個類型； 不同類型存儲多個文檔； 每個文檔有多個屬性；</font>

  * <font color="Red">索引（名詞）</font>:

    一個索引，類似`一個傳統數據庫`，是一個存儲關係型文檔的地方

  * <font color="red">索引（動詞）</font>

     `索引一個文檔`，就是存儲一個文檔到一個<font color="red">索引（名詞）</font> 中，以便它可以被檢索和查詢到。    (<font color="blue">類似傳統數據庫的Insert</font>)

  * <font color="red">倒排索引</font>: ——提升數據檢索速度

    <font color="red">倒排索引的原理</font>

* <font color="grey">創建Index索引</font>

  ~~~
  #創建名為 university的索引
  curl -X PUT "localhost:9200/university?pretty"  
  # 查看所有索引
  curl -X GET "localhost:9200/_cat/indices?v"
  ~~~

  

*  <font color="grey">索引一個文檔</font>

  ~~~
  # 一個文檔代表一個學生
  #university 索引名稱
  #student 類型(學生)
  #1  特定學生的ID
  # 增加第一位學生
  curl -X POST "localhost:9200/university/student/1/?pretty" -H 'Content-Type: application/json' -d' {
      "name" : "李莉",
      "ID"  :        "330000000000000004",
      "sex" :        "female",
      "age" :        25,
      "about" :      "我喜歡閱讀，喜歡跳舞",
      "interests": [ "跳舞", "音樂", "閱讀" ]
  }
  '
  # 查看新增結果
  curl -X GET "localhost:9200/university/student/1?pretty"
  
  #增加第二位學生
  curl -X POST "localhost:9200/university/student/2/?pretty" -H 'Content-Type: application/json' -d' {
      "name" : "張三",
      "ID"  :        "330000000000000001",
      "sex" :        "male",
      "age" :        26,
      "about" :      "我喜歡唱歌，喜歡街舞",
      "interests": [ "街舞", "唱歌" ]
  }
  '
  
  #增加第三位學生
  curl -X POST "localhost:9200/university/student/3/?pretty" -H 'Content-Type: application/json' -d' {
      "name" : "張小東",
      "ID"  :        "330000000000000002",
      "sex" :        "male",
      "age" :        25,
      "about" :      "我喜歡交朋友，我是程序員，我喜歡足球",
      "interests": [ "足球" ]
  }
  '
  
  #增加第四位學生
  curl -X POST "localhost:9200/university/student/4/?pretty" -H 'Content-Type: application/json' -d' {
      "name" : "李明",
      "ID"  :        "330000000000000003",
      "sex" :        "male",
      "age" :        25,
      "about" :      "我喜歡籃球，我喜歡跳街舞",
      "interests": [ "籃球","街舞" ]
  }
  '
  
  #增加第五位學生
  curl -X POST "localhost:9200/university/student/5/?pretty" -H 'Content-Type: application/json' -d' {
      "name" : "王可可",
      "ID"  :        "330000000000000005",
      "sex" :        "female",
      "age" :        23,
      "about" :      "我喜歡英語，我的英語閱讀能力很好，大家可以請教我",
      "interests": [ "學習" ]
  }
  '
  
  #查詢所有學生信息
  curl -X GET "localhost:9200/university/student/_search"
  #分页查询 每頁10條
  curl -X GET "localhost:9200/university/student/_search"?size=10
  
  #查詢性別為'female'同學的信息
  # 用的是輕量的查询字符串 （Query-string_）搜索
  curl -X GET "localhost:9200/university/student/_search?q=sex:female"
  
  # 查詢 包含"male"的所有文檔
  curl -X GET "localhost:9200/university/student/_search?q=male"
  
  ##原理： ElasticSearch 會類似增量一個名叫_all的額外字段，字段內容是所有字段的值拼接而成的字符串
  
  # 查詢喜歡about屬性喜歡”街舞“的文檔
  ##發現 about屬性 '跳舞' 和 '舞'的文檔的找到了，  
  ##原因： ElasticSearch 默認按相關性得分排序，即每個文檔跟查詢的匹配程度。
  curl -X GET "localhost:9200/university/_search" -H 'Content-Type: application/json' -d'
  {
    "query": {
      "match": {
         "about":"街舞"
      }
    }
  }
  '
  
  # 查詢about 屬性 含有”街舞“短語的文檔
  ## match_phrase 匹配短語
  curl -X GET "localhost:9200/university/_search" -H 'Content-Type: application/json' -d'
  {
    "query": {
      "match_phrase": {
         "about":"街舞"
      }
    }
  }
  '
  
  #在每個搜索結果中，‘高亮’部分文本片段， 以便讓更多用戶知道為何該文檔符合查詢條件
  curl -X GET "localhost:9200/university/_search" -H 'Content-Type: application/json' -d'
  {
    "query": {
      "match": {
         "about":"街舞"
      }
    },
    "highlight": {
          "fields" : {
              "about" : {}
          }
      }
  }
  '
  
  # 分析，25歲學生的興趣愛好
  curl -X GET "localhost:9200/university/student/_search" -H 'Content-Type: application/json' -d'
  {
  "query": {
      "match": {
         "age":"25"
      }
    },
    "aggs": {
      "all_interests": {
        "terms": { "field": "interests.keyword" }
      }
    }
  }
  '
  
  # 分析，興趣愛好數量分佈
  curl -X GET "localhost:9200/university/student/_search" -H 'Content-Type: application/json' -d'
  {
    "aggs": {
      "all_interests": {
        "terms": { "field": "interests.keyword" }
      }
    }
  }
  '
  
  # 查詢特定興趣愛好 ，學生的平均年齡
  curl -X GET "localhost:9200/university/student/_search" -H 'Content-Type: application/json' -d'
  {
      "aggs" : {
          "all_interests" : {
              "terms" : { "field" : "interests.keyword" },
              "aggs" : {
                  "avg_age" : {
                      "avg" : { "field" : "age" }
                  }
              }
          }
      }
  }
  '
  
  
  
  #查詢性別為'male',齡處於[10，40]的所有文檔
  curl -X GET "localhost:9200/university/_search" -H 'Content-Type: application/json' -d'
  {
    "query": {
      "bool": {
        "must": { "match": {
            "sex":"male"
        } },
        "filter": {
          "range": {
            "age": {
              "gte": 10,
              "lte": 40
            }
          }
        }
      }
    }
  }
  '
  
  # 查詢年齡處於[10，40]的所有文檔
  curl -X GET "localhost:9200/university/_search" -H 'Content-Type: application/json' -d'
  {
    "query": {
      "bool": {
        "must": { "match_all": {} },
        "filter": {
          "range": {
            "age": {
              "gte": 10,
              "lte": 40
            }
          }
        }
      }
    }
  }
  '
  ~~~

  更多用法，請查看文檔

  

### 索引原理

#### ElasticSearch 對照 傳統數據庫

* 關係數據庫 —> 數據庫 -> 表 -> 行 -> 列
* ElasticSearch -> 索引（Index） ->類型(type) ->文檔(Documents) ->字段(Field) 

####  ElasticSearch怎樣做到快速索引？

”倒排索引“

 ElasticSearch 為每個field都建立一個倒排索引。

ElasticSearch為了快速找到某個項，將所有項 進行排序， 二分法查找項（Log N的查找效率）---類似B-Tree。

直接通過內存查找項，但如果項太多，放在內存中也不現實，所以有有了項的index， 類似字典的索引頁，A開頭的有哪些項，分別在哪一頁

#### 倒排索引

繼上面的例子

| ID   | name   | age  | sex    |
| ---- | ------ | ---- | ------ |
| 1    | 李莉   | 25   | female |
| 2    | 張三   | 26   | male   |
| 3    | 張小東 | 25   | male   |
| 4    | 李明   | 25   | male   |
| 5    | 王可可 | 23   | female |

##### ElasticSearch建立所以如下

* name

  | 項     | ID   |
  | ------ | ---- |
  | 李莉   | [1]  |
  | 張三   | [2]  |
  | 張小東 | [3]  |
  | 李明   | [4]  |
  | 王可可 | [5]  |

* age

  | 項   | ID      |
  | ---- | ------- |
  | 23   | [5]     |
  | 25   | [1,3,4] |
  | 26   | [2]     |

* Sex

  | 項     | ID      |
  | ------ | ------- |
  | female | [1,5]   |
  | male   | [2,3,4] |

  
  未完待續
# kibana

## 安裝與運行

* 啟動（以mac为例子）

  ~~~
  brew services start kibana //啟動
  brew services list  //查看啟動的
  ~~~

* 修改Kibana配置（以mac为例子）

  * 修改配置文件：`sudo vi /usr/local/etc/kibana/kibana.yml`

  * ~~~
    取消注释
    server.port: 5601 // Kibana端口
    elasticsearch.url: "http://localhost:9200” 
    elasticsearch.username: "user"
    elasticsearch.password: "123456"
    ~~~

  * 访问

    ~~~
    localhost:5601/status
    ~~~


## 目錄

~~~
/usr/local/Cellar/kibana/{kibana-version}/
/usr/local/etc/kibana/
~~~



# Logstash    

## 參考資料

* [官網文檔](https://www.elastic.co/guide/en/logstash/master/installing-logstash.html)
* [Logstash Reference](https://www.elastic.co/guide/en/logstash/current/index.html)

## 簡介與工作原理

### 簡介

* 動態收集不同源的數據，過濾，并指定按不同格式輸出數據。

  如：通過Logstash，可以輕鬆獲取大量運行日誌，并結合ElasticSearch、Kibana進行分析和統計運行日誌

### 工作原理

* 三個步驟： inputs(必須) —> filters(選項) —> outputs(必須)
  1. Inputs
     * 抓取數據到logstash
     * 經常遇到的數據源，文件、系統日誌、redis、beats，更多請參考[Inputs Plugins](https://www.elastic.co/guide/en/logstash/6.5/input-plugins.html)
  2. Filters
     *  過濾/處理數據
     * 常用的Filters ： gork(將結構數據 過濾為 有結構的數據)、mutate(重命名、替代、移除、修改fileds)、drop(刪除)、clone(克隆)、geoip(增加ip信息)
  3. outputs
     * 輸出源
     * 常用的輸出源： elasticsearch(保存數據高效、方便、查詢簡單)、file(輸出數據到某個磁盤文件)、graphite(開源，請參考[graphite文檔](https://graphite.readthedocs.io/en/latest/))、statsd()， 更多請參考[Outputs Plugins](https://www.elastic.co/guide/en/logstash/6.5/output-plugins.html)
  4. Codecs 
     * 流過濾器，Codecs能讓容易的區分不同進程的輸出/輸入、  常用的codecs ： json. plain(text), msgpack，  更多請參考[codec Plugins](https://www.elastic.co/guide/en/logstash/6.5/codec-plugins.html)
* 每個input ，Logstash管道都會用獨立線程運行、主要運行在內存或硬盤上。



## 安裝與運行

* [安裝方法，請查看官網Installing Logstash](https://www.elastic.co/guide/en/logstash/current/installing-logstash.html)

* mac 安裝

* ~~~
  brew install logstash
  logstash --version
  ~~~

* 啟動/重啟/關閉

  ~~~
  brew services start logstash
  brew services stop logstash
  brew services restart logstash
  ~~~

* 用logStash基本的管道测试一下(mac環境)

  * 在目錄`/usr/local/Cellar/logstash/6.7.0` 
  * 輸入命令`bin/logstash -e 'input { stdin { } } output { stdout {} }'`
    * `-e` : 直接從命令行修改配置 
    * `input`：指定輸入；
    * `stdin`:從控制台輸入
    *  `output`:指定輸出
    * `stdout`:輸出到控制台
  * 輸入`MY name is Francis` 會打印如下結果

  * 打印結果如下

    ~~~
    {
        "@timestamp" => 2019-04-16T03:43:19.910Z,
          "@version" => "1",
           "message" => "MY name is Francis",
              "host" => "adminde-MacBook-Pro.local"
    }
    ~~~

* 更多運行相關請查看[Running Logstash From the Command Line](https://www.elastic.co/guide/en/logstash/5.4/running-logstash-command-line.html#command-line-flags)

## 目錄

~~~
安裝目錄： /usr/local/Cellar/logstash/{logstash-version}/
配置文件：/usr/local/etc/logstash/
~~~

* 提示  mac環境 可以通過`brew info logstash `查看安裝相關信息

## 插件

* 參考資料
  * [Input plugins](https://www.elastic.co/guide/en/logstash/current/input-plugins.html)
  * [output plugins](https://www.elastic.co/guide/en/logstash/current/output-plugins.html)
  * [filter plugins](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html)

* Logstash有豐富的插件，處理 `input`、`filter`、`output`

* 通過命令`bin/logstash-plugin`腳本管理Logstash的插件

* 查看插件

  提示：以下命令，請在安裝目錄下執行 (mac環境：/usr/local/Cellar/logstash/版本號)

  * 列出安裝的插件

    ~~~
    bin/logstash-plugin list
    ~~~

  * 列出安裝插件的版本信息

    ~~~
    bin/logstash-plugin list --verbose
    ~~~

  * 查看在`input`、`filter`、`output`場景中適用的插件

  *  ~~~
    //查看適合在input中使用的插件
    bin/logstash-plugin list --group input
    //查看適合在filter中使用的插件
    bin/logstash-plugin list --group filter
    //查看適合在output中使用的插件
    bin/logstash-plugin list --group output
    
    ~~~

* 安裝插件

  * 一般情況下，插件在`RubyGems.org`中，可通過下面命令可安裝插件

    * 如：想要安裝`filter`場景的`geoip`插件，可執行下面命令。可以從`RubyGems.org`中檢索到插件，并進行安裝，安裝成功後，即可在配置文件中使用

    * ```shell
      bin/logstash-plugin install logstash-filter-geoip
      
      //成功會有類似如下的提示
      //Validating logstash-filter-geoip
      //Installing logstash-filter-geoip
      //Installation successful
      ```

  * 某些情況下，插件不在`RubyGems.org`中，可指定本地插件進行安裝，命令如下

  * ~~~
    bin/logstash-plugin install 本地路徑/插件名.gem
    ~~~

* 更新插件

  * 更新所有插件

    ~~~
    bin/logstash-plugin update
    ~~~

  * 更新指定插件，

    ~~~
    //如，想要更新`filter`場景的`geoip`插件
    bin/logstash-plugin update  logstash-filter-geoip
    ~~~

* 移除插件

* ~~~
  //如，想要更新`filter`場景的`geoip`插件
  bin/logstash-plugin remove logstash-filter-geoip
  ~~~

* 注意

  一般情況下，安裝插件，需要訪問`RubyGems.org`， 若不能訪問，請參考[Proxy Support](https://www.elastic.co/guide/en/logstash/current/working-with-plugins.html#removing-plugins)

* 可創建自己的`插件`, 請參考[Generating Plugins](https://www.elastic.co/guide/en/logstash/current/plugin-generator.html)

* 更多，請參考官網

  

# Filebeat

## 參考資料

* [Filebeat文檔](https://www.elastic.co/guide/en/beats/filebeat/7.0/filebeat-getting-started.html)

## 簡介、安裝、配置、運行

### 簡介

Fliebeat是輕量級的，資源友好的工具，佔用資源少、可靠、低延遲。

常常用來收集服務端文件日誌，并將收集的文件 發送到Logstash示例中處理

## 安裝

* mac 環境

  ~~~
  curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.0.0-darwin-x86_64.tar.gz
  tar xzvf filebeat-7.0.0-darwin-x86_64.tar.gz
  ~~~

* 其他請參考[Filebeat安裝文檔](https://www.elastic.co/guide/en/beats/filebeat/7.0/filebeat-installation.html)

## 配置filebeat.yml文件

* `filebeat.yml`文件在在`/usr/local/etc/filebeat/`目錄中(mac環境) 

##  目錄

~~~
安装目录：/usr/local/Cellar/filebeat/{filebeat-version}/
配置目录：/usr/local/etc/filebeat/
缓存目录：/usr/local/var/lib/filebeat/
~~~





# 案例

## 1、通過Filebeat 獲取指定服務器日誌， 輸出logstash ，并打印在控制台中

注意：<font color="red">Make sure you have the latest compatible version of the Beats input plugin for Logstash installed,請參考[logstash-setup](https://www.elastic.co/guide/en/beats/libbeat/1.2/logstash-installation.html#logstash-setup)</font>

1. 配置`filebeat.yml`(mac環境，該文件在/usr/local/etc/filebeat/中)

   * 將日誌文件通過`filebeat`收集到`logstash`中
   * `filebeat.yml`配置如下

   ~~~
   filebeat.inputs:
   - type: log
     paths:
       -/Users/admin/Documents/code/pro/k5/common/runtime/logs/2019/04/17/mqtt_data_rctl.log //日誌文件路徑 ,也可以指定某個文件 如：/var/log/*.log
   
   output.logstash:
     hosts: ["localhost:5044"] //logstash的url
   ~~~

2. 配置logstash

   * 在 安裝目錄(mac環境：/usr/local/Cellar/logstash/6.7.0) 創建一個配置文件`test-pipeline.conf`

   *  `test-pipeline.conf`文件內容如下：

     ~~~
     input { //輸入
         beats {  //這部分表示  指定logstash 的input 來自beats
            port => "5044"
         }
     }
     filter {
         grok {
            match => { "message" => "%{COMBINEDAPACHELOG}" } //過濾器插件，將非結構化數據解析為結構化和可查詢數據  注意安裝插件
         }
         geoip {
           source => "clientip" //客戶端IP地理信息 注意安裝插件
         }
     }
     output { //輸出 
         //stdout {codec => rubydebug} //codec => rubydebug輸出到控制台
     }
     ~~~

3. 檢查并啟用logstash 

   * 在`/usr/local/Cellar/logstash/6.7.0`目錄執行

   ~~~
   bin/logstash -f test-pipeline.conf --config.reload.automatic //config.reload.automatic的意思是啟動配置加載(就不需在修改配置后 ，重啟logstash)  
   
   //也可以配置  --config.test_and_exit   如：bin/logstash -f test-pipeline.conf --config.test_and_exit //config.test_and_exit的意思是解析配置文件并報告任何錯誤
   ~~~

4. 啟動filebeat

   * 在`/usr/local/etc/filebeat`目錄中執行

   ~~~
   filebeat -e -c filebeat.yml -d "publish"
   
   ~~~

   * 因為有緩存，若修改了配置，想要filebeat重新加載日誌，請先在緩存目錄(mac環境`/usr/local/var/lib/filebeat/`)執行 `rm registry`刪除Filebeat註冊文件

5. 查看结果(logstash控制台輸出)

   ![logstash_stdout_rubydebug](/image/elasticsearch/logstash_stdout_rubydebug.png)

   

   

   









# Graphite 

## 參考資料

[Graphite 文檔](https://graphite.readthedocs.io/en/latest/)











































