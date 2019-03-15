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


# Logstash    



