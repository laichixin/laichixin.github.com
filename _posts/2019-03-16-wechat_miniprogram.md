---
layout:     post
title:      微信小程序
subtitle:   微信小程序
date:       2019-03-16
author:     Francis
header-img: image/bg-desk.jpeg
catalog: true
tags:
    - wechat
---

# 微信小程序


## 雲開發

### 簡介 & 初始化環境

#### 參考資料

- [雲開發文檔](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/basis/getting-started.html)
- [云數據庫文檔](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/guide/database.html)

#### 簡介

* 開發者無需搭建服務器，即可開發微信小程序，小遊戲。

* 目前提供三大基礎能力：

  * 數據庫：文檔型數據庫（json格式）

    * 參考資料

      * [雲開發數據庫官網文檔](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/guide/database.html)

    * 文檔型云數據庫與關係型數據庫對比關係

      | 關係型             | 文檔型云數據庫     |
      | ------------------ | ------------------ |
      | 數據庫（database） | 數據庫database     |
      | 表（table）        | 集合（collection） |
      | 行（row）          | 記錄（record/doc） |
      | 列（column）       | 字段（field）      |

    *  文檔型云數據庫簡介

      * 字段（filed）值：可以是數字、字符串、數組或對象

      * 每條記錄（record/doc）
        * `_id` 字段： 唯一標識（相當於關係數據庫的主鍵ID）
          * `_id`：可以被自定義
        * `_openid`字段： 標識記錄的創建者(即：小程序的用戶)
          * `_openid`是默認創建的，不可自定義或修改（注意在控制台或云函數創建的記錄不會有`open_id`字段）
        * 

    * 數據庫API

      * 由兩部分組成
        * 小程序端
          * 小程序端API擁有嚴格的調用權限控制
            * 非敏感數據，開發者可以在小程序內直接調用
        * 服務端
          *  高度安全的數據，可以云函數內通過服務端API進行操作

    *  使用API操作數據庫

      1. 獲取數據庫引用

         ~~~
         const db = wx.cloud.database()
         ~~~

      2. 構造查詢語句  &  觸發數據庫操作

         * 案例

           ~~~js
           /**官網例子
            * 
            * 查詢'book'集合中 ，發表於'United States'的文檔(圖書)
            **/
           
           db.collection('books').where({
             publishInfo: {
               country: 'United States'
             }
           }).get({
             success(res) {
             // 输出 [{ "title": "The Catcher in the Rye", ... }]
               console.log(res)
             }
           })
           ~~~

         * 解析

           * collection方法獲取一個集合的引用
           * where方法，傳入一個對象，數據庫返回集合中字段等於指定值的json文檔。(支持複雜查詢)
           * get方法會觸發網絡請求，往數據庫增刪改查操作 

  * 存儲：

    * 參考資料

      [官網存儲文檔](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/guide/storage.html)

    * 提供了帶權限的雲端上傳/下載能力，開發者可以在小程序端和云函數端通過API使用云存儲功能

    * 小程序端

      * 上傳： 調用 `wx.cloud.uploadFile`

        * 官網案例 

          ~~~js
          // 讓用戶選擇一張圖片
          wx.chooseImage({
            success: chooseResult => {
              // 將圖片上傳至云存儲空間
              wx.cloud.uploadFile({
                // 指定上傳到的云路徑
                cloudPath: 'my-photo.png',
                // 指定要上傳的文件的小程序臨時文件路徑
                filePath: chooseResult.tempFilePaths[0],
                // 成成功回調
                success: res => {
                  console.log('上传成功', res)
                },
              })
            },
          })
          ~~~

          

      * 下載： 調用`wx.cloud.downloadFile`

    * 

  * 云函數：

    * 參考資料

      [官網云函數指引](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/guide/functions.html)

    *  云函數的環境是與客戶端完全隔離的，在云函數上可以私密且安全的操作數據庫（高度安全的數據）

    * 小程序內提供了專門用於云函數調用的API，開發者可以在云函數內使用 [`wx-server-sdk`](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/guide/functions/wx-server-sdk.html)提供的  [`getWXContext`](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/reference-server-api/utils/getWXContext.html) 方法獲取到每次調用的上下文（`appid`、`openid`等）

    * 官網例子

      * 比如定義一個名為`add`的云函數，功能是將傳入的兩個參數與a和b相加

        ~~~js
        //index.js 是入口文件，云函數被調用時會執行該文件導出的main方法
        //event包含了調用端(小程序端)調用該函數時傳過來的參數，同時還包含了可以通過 getWXContext方法獲取的用戶登錄`openId`和 小程序 `appId`的信息
        const cloud = require('wx-server-sdk')
        exports.main = (event, context) => {
          const {userInfo, a, b} = event
          const {OPENID, APPID} = cloud.getWXContext() // 這裡獲取到的 openId 和 appId 是可信的
          const sum = a + b
        
          return {
            OPENID,
            APPID,
            sum
          }
        }
        ~~~

      * 在開發者工具中上傳部署add云函數后，可以在小程序中調用add云函數

        ~~~js
        wx.cloud.callFunction({
          // 需調用的云函數名
          name: 'add',
          // 傳給云函數的參數
          data: {
            a: 12,
            b: 19,
          },
          // 成功回調
          complete: console.log
        })
        // 當然 promise 方式也是支持的
        wx.cloud.callFunction({
          name: 'add',
          data: {
            a: 12,
            b: 19
          }
        }).then(console.log)
        ~~~

* 資源環境
  * 一個環境配套獨立的云開發資源 （包括：數據庫、存儲空間、云函數等）
  * 開通后默認兩個環境（建議用於測試環境、正式環境） 

#### 初始化環境

##### 云環境的創建 

* 新建項目選擇一個空目錄

* 需要填入AppID（注意：不可使用測試號） [AppID申請地址](https://mp.weixin.qq.com/wxopen/waregister?action=step1)

* 勾選 創建’雲開發‘

* 如下圖

  ![init_project](/image/wechat/miniprogram/init_project.png)

##### 與普通小程序的區別

* 無遊客模式、也不可使用測試號
* `project.config.json`中增加了`cloudfunctionRoot`字段，用於指定存放云函數目錄
* `cloudfunctionRoot` 指定的目錄有特殊的圖標

##### 注意 

*   [^版本問題]
*  <font color="red">此文檔只做初步的簡介，沒能實時更新，請以官網為準</font>



### 開發指引

#### 雲開發控制台

##### 簡介

* 「雲開發控制台」，用以可視化管理云資源，包含的模塊如下

  * 概覽：查看云資源總體使用情況
  * 用戶管理： 查看小程序的用戶訪問記錄
  * 數據庫：管理數據庫集合、記錄、權限設置、索引設置
  * 存儲管理：管理云文件、權限設置
  * 云函數：管理云函數、查看調用日誌、監控記錄
  * 統計分析：查看云資料詳細使用統計

* `app.js`頁面`wx.cloud.init`方法，設置`traceUser: true`

  * 當用戶訪問有云能力的小程序時， 會記錄用戶訪問時間； 在「雲開發控制台」的「用戶管理」中，會顯示訪問用戶列表（默認：按時間倒敘）

    ![trance_user](/image/wechat/miniprogram/trance_user.png)

  * 







[^版本問題]: 雲開發能力從基礎庫2.2.3開始支持，有部分用戶沒能被覆蓋， 如果使上傳的代碼能夠覆蓋全量用戶，請在 `app.json/game.json`中增加`cloud:true`的字段