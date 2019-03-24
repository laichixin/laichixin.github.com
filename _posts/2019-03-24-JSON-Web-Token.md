---
layout:     post
title:      JSON Web Toekn
subtitle:   JSON Web Toekn
date:       2019-03-24
author:     Francis
header-img: image/bg-desk.jpeg
catalog: true
tags:
    - 認證

---

# JSON Web Token

## 參考資料

[JSON Web Token](https://jwt.io/introduction/)



## 初始“認證”

* 通常，我們訪問「網站、APP等」，都會被要求輸入“用戶名和密碼”。 只要我們輸入的“用戶名和密碼”正確，就可以合法的訪問「網站、軟件」的某些內容/頁面。但對於不合法的訪問（如我們嘗試著訪問我們沒有權限的頁面），「網站、軟件」都會阻止我們的訪問。
  * <font color="red">我在想……</font>
    * 既然「網站、APP」的每個頁面都會驗證我們身份的合法性，那麼，他們一定知道我們的信息，但……我記得……，我只在登錄時輸入了一次用戶名和密碼，他們是怎樣記住我們的信息的？
    * <font color="green">我記得，在上週三，我辦了一張健身卡，只在辦卡的時候填寫了自己的資料（姓名等），哦，還交了錢……然後他們給了我一張健身卡（卡上有卡號），之後每次我去健身房的時候，只要我刷一下卡就能進入了，是一樣的原理嗎？</font>
    * <font color="red">遇到問題了：健身房我進不去了，這讓我很憤怒……</font>
      * 昨天我在健身房看到了一個心儀之人，於是我今天可以換了一套很漂亮的健身服（想引起那位美女的注意），我滿懷期待，來到健身房門口，透過玻璃門，我看到了心儀之人……緊接著我跟往常一樣……嗯？（好像沒聽到滴滴的一聲……我重複了幾次……）
      * 我找到工作人員，工作人員告訴我， “不好意思，先生，門卡器剛剛壞了，獲取不了你會員卡的信息……”， 跟工作人員balabala一大堆， 就是不給進，”……好氣“。
      * <font color="green">換一種信息存儲方式，不就可以解決這個問題嗎？  只要每次我每次都帶著自己的信息（姓名，有效期，金額），工作人員或機器都可以驗證我提供的信息，不就可以解決這個問題嗎？</font>
* session驗證方式
  1. 用戶填寫了“用戶名和密碼”，用戶點擊「登錄」，服務器能拿到用戶填寫的“用戶名和密碼”（<font color="green">相當於辦了健身卡</font>）
  2. 服務器驗證用戶填寫的“用戶名和密碼”，如果驗證通過， 會將用戶信息（用戶名，用戶角色、權限、登錄時間等）存儲在當前對話（session）
  3. 服務器會返回一個session_id，存儲在用戶端的Cookie（<font color="green">相當於用戶拿到健身卡，健身卡上有卡號</font>）
  4. 用戶之後每一次對「網站、軟件」的訪問，都會通過Cookie，將session_id傳到服務器（<font color="green">相當於用戶拿著健身卡去健身房</font>）
  5. 服務器收到session_id，找到前提保存的用戶信息（用戶姓名、角色、權限等）(<font color="green">相當於健身房讀取健身卡號，根據健身卡號，找到用戶的信息</font>)
*   <font color="red">session 問題</font>
  * 單點登錄問題， 一家公司有 A、B兩個網站，能不能用戶登錄了A網站，B網站就自動登錄呢？
  * 一種方案，session_id保存在服務端，但需要將session_id持久化
  * 另一種方案，直接將信息保存在客戶端（如：JSON Web Token）

## JSON Web Toeken 簡介

### JWT原理

* 服務器驗證用戶登錄信息后，給用戶返還一份JSON格式的數據，之後，用戶只需要攜帶JSON格式數據，服務器就可以隨時認證用戶身份了 

  ~~~json
  {
    "姓名": "Francis",
    "角色": "VIP會員",
    "到期时间": "2019年12月31日23点59分59秒"
  }
  ~~~

* <font  color="red">有一個問題</font>

  * <font color="green">如果用戶偷偷將「到期時間」修改為「2020年12月31日23点59分59秒」呢？ 這樣公司豈不是虧了？</font>
  * 嗯，好問題，為了防止用戶篡改數據，決定 在返給用戶的基礎上，加上”簽名“（不了解沒關係，下面會說喲）

  

### JWT數據結構

* 最終的JWT是這樣的

* ~~~json
  eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImp0aSI6IjVjOTYyYmY4NDEyZGY4NDQ3MCJ9.eyJqdGkiOiI1Yzk2MmJmODQxMmRmODQ0NzAiLCJpYXQiOjE1NTMzNDU1MjgsIm5iZiI6MTU1MzM0NTUzMSwiZXhwIjoxNTUzMzQ5MTI4LCJkYXRhIjp7InVpZCI6MX19.fsjRUoThP4M6kYKTmBZWyyj_RFqSUz1dsz6xChMObiU
  ~~~

  * JWT是一個字符串

  * JWT裡面有兩個點（.）， 分成三個部分，這三部分分別是

    * 頭部（Header）

      * 指定簽名的算法，指定Token的類型等描述JWT的元數據

      * 頭部實際的模樣

      * ~~~json
        {
          "alg": "HMAC SHA256",
          "typ": "JWT"
        }
        ~~~

      * 然後使用`Base64URL算法`，將JSON轉為我們看到的字符串

    * 負載（Payload）

      * 就是內容啦， 注意喲，這一部分信息是公開的，不能放私密信息喲

      * 負載實際的模樣

        ~~~json
        {
          "iss":"簽發人",
          "exp" : "過期時間",
          "sub": "主題",
          "name": "受眾",
          "nbf" : "生效時間",
           "iat":"簽發時間",
           "jti": "編號",  
           //…… 當然你也可以定義其他自定義字段啦
        }
        ~~~

      * 然後使用`Base64URL算法`，將JSON轉為我們看到的字符串

    * 簽名（Signature）

      * 簽名，就是對 頭部（header）和負載（payload）進行簽名，防止用戶篡改數據

      * 簽名說明

        * 服務器指定一個密鑰（secret）：這個密鑰需要保密喲

        * 使用剛才頭部(header)指定的簽名算法（如：HMAC SHA256）

          ~~~
          HMACSHA256(
            base64UrlEncode(頭部header) + "." +
            base64UrlEncode(負載payload),
            密鑰secret)
          ~~~

    * 然後將，頭部.負載.簽名 用點（.）相連就可以返給用戶啦

### JWT注意點

* 由於服務器端不再保存session，而JWT又是按特定規則簽發的，並且保存在客戶端。所以，JWT簽發后在生命週期內一直有效。
  * 不能與保存在服務端的session_id一樣，可以隨時指定廢棄某個token
  * 如要需要廢棄某個JWT ，請添加額外的邏輯判斷
* JWT存放於客戶端，驗證信息都包含在其中，所以存在洩露風險，建議不要將JWT的有效期設置的太長， 在傳輸過程中，可以採用"HTTPS"協議



## JWT的應用

### 服務器端

以php語言為例，用的是`https://github.com/lcobucci/jwt`插件

~~~php
$token = Yii::$app->jwt->getBuilder()
    ->setId(uniqid() . rand(10000, 99999), true)// 設置ID
    ->setIssuedAt(time())// 設置token生存時間
    ->setNotBefore(time() + 3)// 設置在x秒內該token無法使用
    ->setExpiration(time() + 過期時間單位s)// 設置過期時間
    ->set('data', $data) //數組格式的$data 如 ["open_id" => "openid", "user_id" => "用戶id"]
    ->sign(new Sha384(), 密鑰)// 對上面的信息使用某種（Sha384）算法签名
    ->getToken(); // 获取生成的token
~~~



### 客戶端

以小程序為例：

* 小程序收到服務端返回的jwt， 可以存儲在Cookie、localStorage等，
* 為了能跨域，比較好的做法是放在請求的頭部Authorization中











