---
layout:     post
title:      識別
subtitle:   百度-圖像識別
date:       2019-04-15
author:     Francis
header-img: image/bg-desk.jpeg
catalog: true
tags:
    - 識別


---

# 百度圖像識別

## 參考資料

* [百度圖像識別SDK資源](http://ai.baidu.com/sdk#vis)

* [百度圖像識別PHP SDK文檔](http://ai.baidu.com/sdk#vis)



## 下載

* 下載SDK

* 通過[控制台](http://ai.baidu.com/tech/imagecensoring)註冊賬號，創建應用，獲取 `APPID AK SK`



## 說明

~~~php
  const APP_ID = "xxx"; //百度 APP ID
  const API_KEY = 'xxx'; //百度API KEY
  const SECRET_KEY = 'xxx'; //百度 Secret Key

  public static function actionTest()
  {
    
    $client = new AipImageClassify(self::APP_ID, self::API_KEY, self::SECRET_KEY); //新建AipImageClassify。
    /**
    * 打印示例如下： 
    * class common\components\lib\AipHttpClient#32 (4) {
    	public $headers =>
   	  array(0) {
    	}
    	public $connectTimeout =>  //連接的超時時間，可以通過setConnectionTimeoutInMillis 配置
    	int(60000)
    	public $socketTimeout => //傳輸數據的超時時間，可通過setSocketTimeoutInMillis配置
    	int(60000)
    	public $conf =>
    	array(0) {
    }
    *
    */
   
  }
~~~

## 案例(可運行)

~~~php

    public static function actionTest1()
    {
        $token = self::getToken(); //獲取鑒權的token ，有效期為一個月，可存儲在數據庫中
        $image_url = Yii::getAlias('@common').'/img/lotos.jpg'; //要識別的圖片地址
        $type = 'general'; //指定圖片類型， 如植物、動物……
        switch ($type) {
            case 'general': //通用物品識別
                $url = 'https://aip.baidubce.com/rest/2.0/image-classify/v2/advanced_general';
                break;
            case 'dish': //菜品識別
                $url = 'https://aip.baidubce.com/rest/2.0/image-classify/v2/dish';
                break;
            case 'car': //車輛識別
                $url = 'https://aip.baidubce.com/rest/2.0/image-classify/v1/car';
                break;
            case 'animal': //動物識別
                $url = 'https://aip.baidubce.com/rest/2.0/image-classify/v1/animal';
                break;
            case 'plant': //植物識別
                $url = 'https://aip.baidubce.com/rest/2.0/image-classify/v1/plant';
                break;
            case 'landmark': //地標識別
                $url = 'https://aip.baidubce.com/rest/2.0/image-classify/v1/landmark';
                break;
        }
        $url .= '?access_token=' . $token;
        $img = file_get_contents($image_url);
        $img = base64_encode($img); 
        $bodys = array(
            'image' => $img
        );
        $res = self::request_post1($url, $bodys); //獲取返回結果
        $result = json_decode($res);
        var_dump($result); //處理返回數據
    }

/**
     * 調用鑒權接口 獲取token
     *
     * @return string token （有效期為1個月）
     */
    public static function getToken()
    {
        $url = 'https://aip.baidubce.com/oauth/2.0/token';
        $post_data['grant_type']       = 'client_credentials';
        $post_data['client_id']      = self::API_KEY;
        $post_data['client_secret'] = self::SECRET_KEY;
        $o = "";
        foreach ( $post_data as $k => $v )
        {
            $o.= "$k=" . urlencode( $v ). "&" ;
        }
        $post_data = substr($o,0,-1);

        $res = self::request_post($url, $post_data);
        $res_json = json_decode($res);
        $token = $res_json->access_token;
        return $token;
    }


 /**
     * @param string $url
     * @param string $param
     * @return bool|mixed
     */
    public static function request_post1($url = '', $param = '')
    {
        if (empty($url) || empty($param)) {
            return false;
        }

        $postUrl = $url;
        $curlPost = $param;
        // 初始化curl
        $curl = curl_init();
        curl_setopt($curl, CURLOPT_URL, $postUrl);
        curl_setopt($curl, CURLOPT_HEADER, 0);
        // 要求结果为字符串且输出到屏幕上
        curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
        curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, false);
        // post提交方式
        curl_setopt($curl, CURLOPT_POST, 1);
        curl_setopt($curl, CURLOPT_POSTFIELDS, $curlPost);
        // 运行curl
        $data = curl_exec($curl);
        curl_close($curl);

        return $data;
    }

    public static function request_post($url = '', $param = '') {
        if (empty($url) || empty($param)) {
            return false;
        }

        $postUrl = $url;
        $curlPost = $param;
        $curl = curl_init();//初始化curl
        curl_setopt($curl, CURLOPT_URL,$postUrl);//抓取指定网页
        curl_setopt($curl, CURLOPT_HEADER, 0);//设置header
        curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);//要求结果为字符串且输出到屏幕上
        curl_setopt($curl, CURLOPT_POST, 1);//post提交方式
        curl_setopt($curl, CURLOPT_POSTFIELDS, $curlPost);
        $data = curl_exec($curl);//运行curl
        curl_close($curl);

        return $data;
    }
~~~

### 結果

~~~
object(stdClass)[31]
  public 'log_id' => int 1883750361532269775
  public 'result_num' => int 5
  public 'result' => 
    array (size=5)
      0 => 
        object(stdClass)[36]
          public 'score' => float 0.997094
          public 'root' => string '植物-其它' (length=13)
          public 'baike_info' => 
            object(stdClass)[37]
              ...
          public 'keyword' => string '荷花' (length=6)
      1 => 
        object(stdClass)[38]
          public 'score' => float 0.785377
          public 'root' => string '植物-其它' (length=13)
          public 'baike_info' => 
            object(stdClass)[39]
              ...
          public 'keyword' => string '映日荷花' (length=12)
      2 => 
        object(stdClass)[40]
          public 'score' => float 0.577897
          public 'root' => string '植物-其它' (length=13)
          public 'baike_info' => 
            object(stdClass)[41]
              ...
          public 'keyword' => string '莲花' (length=6)
      3 => 
        object(stdClass)[42]
          public 'score' => float 0.249397
          public 'root' => string '植物-花' (length=10)
          public 'baike_info' => 
            object(stdClass)[43]
              ...
          public 'keyword' => string '夏日荷花' (length=12)
      4 => 
        object(stdClass)[44]
          public 'score' => float 0.035726
          public 'root' => string '植物-其它' (length=13)
          public 'baike_info' => 
            object(stdClass)[45]
              ...
          public 'keyword' => string '并蒂莲' (length=9)

~~~



