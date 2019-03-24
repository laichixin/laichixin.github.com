---
layout:     post
title:      微信小程序-應用篇
subtitle:   微信小程序2-應用篇
date:       2019-03-24
author:     Francis
header-img: image/bg-desk.jpeg
catalog: true
tags:
    - wechat
---

# 微信小程序2-應用篇

## 配置環境

### 域名問題

* 由於小程序只可與指定域名進行網絡通訊，所以是需要配置域名的，請[參考網絡域名文檔](<https://developers.weixin.qq.com/miniprogram/dev/framework/ability/network.html>)

* <font color="red">有一個情景，想讓本地小程序與連接本地（http://127.0.0.1:9402/）服務器通訊，而不使用合法域名，怎麼辦呢？</font>

  * 猜想，你應該會遇到這個問題

    ![illegal_domain_name](/image/wechat/2019-03-24-wechat_miniprogram/illegal_domain_name.png)

  * 別急，你只需要在微信小程序，「設置」->「項目設置」->把“不校验合法域名、web-view（业务域名）、TLS版本以及HTTPS证书”選項勾選上，就可以與本地服務器通訊了

    ![skip_demain_name_inspection](/image/wechat/2019-03-24-wechat_miniprogram/skip_demain_name_inspection.png)

## 登錄

### 登錄邏輯

在開發登錄前，先了解一下「微信登錄流程吧」，請[參考文檔](<https://open.wechat.com/cgi-bin/newreadtemplate?t=overseas_open/docs/mini-programs/development/api/api-login>)。

雖然文檔已介紹的非常清楚了，但……時序圖真的對於理解login模塊太重要了，於是將微信登錄的時序圖截取如下：

![login_time_sequence_diagram](/image/wechat/2019-03-24-wechat_miniprogram/login_time_sequence_diagram.png)

* <font color="blue">嗯，簡單的說：</font>
  1. 小程序調用`wx.login()`獲取臨時憑證code，并傳給後台服務器<font color="blue">（小程序處理）</font>
  2. 後台服務器根據code獲取用戶openid、session_key（密鑰）<font color="blue">（後台服務器處理）</font>
  3. 後台服務器返回自定義的登錄態（如token等），以標識用戶身份在當前項目中的合法性<font color="blue">（後台服務器處理）</font>
  4. 小程序獲取token，并將token放到請求的header中 <font color="blue">（小程序處理）</font>



### 應用

通過時序圖，想必我們對於「微信登錄」的流程有了一定的了解，然後，就是應用了……

####小程序端（處理步驟一、步驟四）

#####步驟一、

小程序調用`wx.login()`獲取臨時憑證code，并傳給後台服務器，[wx.login()文檔](<https://developers.weixin.qq.com/miniprogram/dev/api/wx.login.html>)

~~~js
//wx.login 獲取臨時憑證code
wx.login({  
  success(res) {  //步驟一：成功獲取code的回調函數，則將code傳給後台服務器
    if (res.code) {
       let code = res.code; //微信語法，將獲取的code賦值給 變量code
       let params = { //定義需要傳輸的數據
          code: code
        } 
        // 給後台服務器發起網絡請求
        wx.request({
        url: 'https://test.com/login', //後台服務器登錄接口
        data: params,  //需要傳給後台服務器的數據
        header: {//指定數據格式為json
           'content-type':'application/json'
        },
        method: "POST", //指定請求方式 
        success: function (res) {
            //步驟四：  成功獲取 [後台服務器返回自定義的登錄態（如token等）]
          }    
      })
    } 
  }
})
~~~



##### 步驟四、

賦值給網絡請求的header



#### 後台服務器端（處理步驟二、步驟三）

##### 步驟二、

後台服務器根據code調用`code2Session` 獲取用戶openid、session_key（密鑰），[請參考code2Session文檔](<https://developers.weixin.qq.com/miniprogram/dev/api-backend/code2Session.html>)

* 以php的easywechat插件為例

* ~~~php
  //調用easywechat函數
  Yii::$app->miniProgram->auth->session($js_code); //js_code是步驟一獲取的code
  
  //session函數在easywechat中的定義
   public function session(string $code)
   {
       $params = [
           'appid' => $this->app['config']['app_id'], //服務器端配置的微信appid
           'secret' => $this->app['config']['secret'], //服務器端配置的微信secret
           'js_code' => $code, //小程序端傳遞code(步驟一)
           'grant_type' => 'authorization_code',
       ];
       return $this->httpGet('sns/jscode2session', $params);
  }
  ~~~

* 根據上面的調用，有如下返回值：

  | 屬性        | 類型   | 說明                                                         |
  | ----------- | ------ | ------------------------------------------------------------ |
  | openid      | string | 用戶唯一標識                                                 |
  | session_key | string | 會話密鑰                                                     |
  | unionid     | string | 用戶在開放平台的唯一標識符（微信开放平台下的不同应用，unionid是相同的） |
  | errcode     | number | 錯誤碼 0：成功 ， -1：系統繁忙， 40029：code無效 ， 45011：頻率限制，每個用戶每分鐘100次 |
  | errmsg      | string | 錯誤信息                                                     |

##### 步驟三、

後台服務器返回自定義的登錄態（如token等,請參考[博客JSON-Web-Token](https://laichixin.github.io/2019/03/24/JSON-Web-Token/)）

* 登錄態的方案是多樣的， 可以是傳統的session等，本後台採取的方案是`JSON WEB TOKEN`，[請參考Json WEB TOKEN](https://jwt.io/introduction/)

* 後台可以根據，步驟二獲取的用戶openid，查詢數據庫用戶表的用戶信息， 獲得用戶表user_id，根據user_id 生成 `json web token` 登錄態返回給小程序



##頁面

~~~js
const app = getApp(); //獲取
~~~



## 全局app文件

### 全局app.json文件

* 參考資料

  [全局app.json文件](https://developers.weixin.qq.com/miniprogram/dev/framework/config.html#%E5%85%A8%E5%B1%80%E9%85%8D%E7%BD%AE)

* 示例

* ~~~js
  {
    "pages": [  //指定目錄路徑
      "pages/index/index",
      "pages/page1/page1",
    ],
    "tabBar": { //底部tab欄表現
      "color": "#ccc", //tab 上的文字默認顏色，僅支持十六進制顏色
      "selectedColor": "#fecb00",//tab 上的文字選中時的顏色，僅支持十六進制顏色
      "backgroundColor": "#fff", //tab的背景色，僅支持十六進制顏色
      "list": [ //tab列表 最少2個，最多5個
        {
          "pagePath": "pages/index/index", //頁面路徑
          "text": "主頁", //tab按鈕文字
          "iconPath": "images/img1.png", //圖標
          "selectedIconPath": "images/img1_1.png" //選中圖標
        },
        {
          "pagePath": "pages/page1/page1", //頁面路徑
          "text": "頁面1",//tab按鈕文字
          "iconPath": "images/img2.png", //圖標
          "selectedIconPath": "images/img2_1.png"//選中圖標
        },
      
      ]
    },
    "window": { //全局默認窗口表先
      "backgroundTextStyle": "light",//下拉loading樣式
      "navigationBarBackgroundColor": "#fff", //導航欄背景顏色
      "navigationBarTitleText": "導航欄標題文字內容", //導航欄標題文字內容
      "navigationBarTextStyle": "black" //導航欄標題顏色
    },
    "debug": true, //是否開啟debug模式
    "permission": { //小程序接口權限相關配置
      "scope.userLocation": {
        "desc": "你的位置信息将用于小程序位置接口的效果展示"
      }
    }
  }
  ~~~

* 

### 全局app.js文件

~~~js
var http = require('utils/http.js'); //封裝全局函數，如獲取token，header等
let { //WeToast插件
  WeToast 
} = require('wetoast/wetoast.js'); 
App({ 
  WeToast, //后面可以通过app.WeToast访问
  http, //上面引入的全局函數
  onLaunch: function (options) { //小程序初始化時，觸發一次（全局只觸發一次）
    this.globalData.scene_source_id = options.scene || ''; //獲取小程序的場景
    wx.getSystemInfo({ //獲取系統信息
      success: function (res) {
        this.globalData.width = res.windowWidth; //窗口寬度
        this.globalData.height = res.windowHeight  //窗口高度
        this.globalData.screenHeight = res.screenHeight
        this.globalData.dpr = res.windowWidth / 750;
      }
    })
    let token = this.globalData.token;  //獲取全局的token
    http.header.Authorization = token; //給封裝全局函數header賦值(token)
    wx.onNetworkStatusChange(function (res) { //獲取網絡狀態
      if (!res.isConnected) { //若沒有連網絡
        let pages = getCurrentPages(); //函數用於獲取當前頁面的棧的實例， 例如：以數組的形式按棧的順序給出，第一個元素為首頁，最後一個元素為當前頁面 
        let currentPage = pages[pages.length - 1]; //當前頁面
        currentPage.setData({
          pageStatus: 'noNetwork' //當前頁面傳遞網絡狀態為 ‘沒有網絡’
        })
      }
    })
  },
  globalData: { //定義全局變量
    token: '', //全局token
    webHost: 'http://127.0.01:9402/', //服務器的url //域名
    userInfo: {  //用戶信息
      nickname: '', //用戶名稱
      avatar: '' //用戶頭像
    },
    version: '0.0.2', //版本號
    width: '', //寬度
    height: '', //高度
    screenHeight: '',
    dpr: '',
  }
})
~~~







## wxml

### view視圖容器

* 示例

  ~~~xml
  <!--常常作為，樣式控制，以及邏輯if-elif-else控制-->
  <view class="container" wx:if="{{pageStatus=='onLine'}}">
  <!--若bannerList不為空-->
    <view class="swiper-block" wx:if="{{bannerList.length>0}}">
     </view>
  </view>      
  ~~~

  * 控制樣式： class

  * 控制邏輯

  * 可以做渲染

    ~~~xml
    <view class='goBindText' wx:if="{{a == 1}}">1</view>
    <view class='goBindText' wx:elif="{{a == 2}}">2</view>
    <view class='goBindText' wx:else>{{a}}</view>
    ~~~

* 參考資料

  [view視圖容器文檔](<https://developers.weixin.qq.com/miniprogram/dev/component/view.html>)



### block控制屬性

- 示例

  ```xml
  <!-- 只作為邏輯判斷if-elif-else for, 控制多個元素-->
  <block wx:if="{{true}}">
    <view>view1</view>
    <view>view2</view>
  </block>
  
  <!--for 循環, 默認value為item-->
   <block wx:for="{{bannerList}}" wx:key="{{index}}">
       <swiper-item class="swiper-item" bindtap="handleTapSwiper" data-object="{{item}}">
           <image src="{{item.cover_map}}" class="slide-image"/>
       </swiper-item>
  </block>
      
  ```

  

- 參考資料

  [block控制屬性](https://developers.weixin.qq.com/miniprogram/dev/framework/view/wxml/conditional.html)



### swiper（滑塊視圖容器)

* 示例

* ~~~xml
  <!--若bannerList不為空-->
    <view class="swiper-block" wx:if="{{arr.length>0}}">
      <!--
        * swiper(滑塊視圖容器)
        * indicator-dots Boolean false	是否显示面板指示点 
        * autoplay Boolean false 是否自動切換
        * interval Number 5000 自動切換時間
        * duration Number 500 滑動動畫時長
        * circular Boolean false 是否採用銜接滑動
      -->
      <swiper class="swiper-wrap" indicator-dots="{{arr.length>1 ? true:false}}" autoplay="{{true}}" interval="4000" duration="500" circular="{{true}}">
        <block wx:for="{{arr}}" wx:key="{{index}}">
          <swiper-item class="swiper-item" bindtap="點擊觸發函數" data-object="{{item}}">
            <image src="{{item.cover_map}}" class="slide-image"/>
          </swiper-item>
        </block>
      </swiper>
    </view>
  ~~~

* 參考資料

  [swiper（滑塊視圖容器)](https://developers.weixin.qq.com/miniprogram/dev/component/swiper.html) 



### Text文本

* 示例

  ~~~xml
  <!--常常作為，文字顯示（可邏輯if-elif-else控制文字輸出）-->
  <text class="vote-text">{{vode ?'投票开播':'已投票'}}</text>
  
  <!--可作為文字按鈕-->
   <view class='goBindText'><text bindtap='點擊觸發函數'>立即支付</text></view>
  ~~~

  

* 參考資料

  [text（文本）](<https://developers.weixin.qq.com/miniprogram/dev/component/text.html>)





## 插件

### WeToast

1. 下載插件：[WeToast地址](https://github.com/kiinlam/wetoast/tree/master)

2. 解壓后，將「src目錄」下的三個文件(wetoast.js、wetoast.wxml、wetoast.wxss)導入項目(wetoast中）

3. 使用

   * 在項目app.js中引入wetoast.js，其他頁面都可以使用了

   * ~~~js
     let { WeToast } = require('wetoast/wetoast.js')    // 返回構造函數，變量可自定義
     App({
       WeToast,   // 後面可以通過app.WeToast訪問
       onLaunch: function () {
     ....
     ~~~

   * 在項目app.wxss引入wetoast.wxss

     ~~~js
     @import "wetoast/wetoast.wxss";
     ~~~

   * 引入WeToast模板結構

     ~~~js
     <import src="/wetoast/wetoast.wxml"/>
     <template is="wetoast" data="{{...__wetoast__}}"/>
     ~~~

   * 在xx.js文件中

     ~~~js
     const app = getApp();
       // 僅執行一次，用於獲取、設置數據
         onLoad () {
             //創建可重複使用的WeToast實例，并附加到this上，通過this.wetoast訪問
             new app.WeToast()
         },
     ~~~

   * ~~~js
     this.wetoast.toast({
        title: 'msg',
        duration: 1500
     })
     ~~~

   4. 參數說明

      | 參數           | 類型     | 必填  | 說明                           |
      | -------------- | -------- | ----- | ------------------------------ |
      | img            | string   | *可選 | 提示圖片，網絡地址或base64     |
      | imgClassName   | string   | 否    | 自定義圖片樣式時使用class      |
      | imgMode        | string   | 否    | 參考小程序image組件mode屬性    |
      | title          | string   | *可選 | 提示內容                       |
      | titleClassName | string   | 否    | 自定義內容樣式時使用的class    |
      | duration       | Nunber   | 否    | 提示的持續時間，默認1500毫秒   |
      | success        | Function | 否    | 提示即將隱藏時                 |
      | fail           | Function | 否    | 調用過程拋出錯誤時的會帶哦函數 |
      | complete       | Function | 否    | 調用結束時的回調函數           |

      

   

   

   

   

   













