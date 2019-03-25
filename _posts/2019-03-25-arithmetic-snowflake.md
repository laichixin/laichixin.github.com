---
layout:     post
title:      Snowflake算法
subtitle:   解決生成唯一性ID的問題
date:       2019-03-25
author:     Francis
header-img: image/bg-desk.jpeg
catalog: true
tags:
    - arithmetic

---

# Snowflake算法

## 引入問題

<font color="green">怎樣生成全局唯一ID？</font>

* 我可以用UUID生成
  * 缺點：
    * 長度太長（32位+4位），
    * 無序的，影響入庫性能
* 數據庫自增
  * 缺點
    * 容易根據數據，猜到數據量
    * ID生成太過依賴數據庫

* 時間戳+隨機數
  * 缺點
    * 並發時重複率高

* Php  uniqid()函數生成不重複的唯一標識符，基於微妙級當前時間戳。在高並發或者間隔時間極短的情況下，會出現重複數據,哪怕使用了`uniqid('',true);`在並發情況下也會出現重複
  * 解決方案，uniqid() 結合 md5() 、時間戳、用戶表ID等
* 

## 參考資料

### UUID

* [請參考UUID維基百科](https://zh.wikipedia.org/wiki/%E9%80%9A%E7%94%A8%E5%94%AF%E4%B8%80%E8%AF%86%E5%88%AB%E7%A0%81)
* 簡單的說
  * UUID重複的概率接近零，可以忽略不計。
  * UUID共36字符，由32個的十六進制，4個連字符組成（8長度-4長度-4長度-4長度-12長度 ）----理論上，`每納秒`總數是`16的32次方`個
* 缺點
  * UUID太長（32位 + 4位連字符）
  * UUID是「無序的」，入庫性能比較差
    * 原因，數據庫涉及「B+樹索引」，主要考慮的因素「列表次數多少」和「每個節點空間利用」
      * 比如主鍵ID自增，每個索引節點都存儲著N個ID， 當ID按遞增順序插入時，新的ID會插入到最後一個節點中，但節點滿了，才裂變出新的節點 ，這種是性能比較高的插入
    * 若是無序的插入，「裂變次數增加」和 「節點空間利用低」



###  SnowFlake

*  [請參考Twitter SnowFlake文檔](https://developer.twitter.com/en/docs/basics/twitter-ids.html)
* [請參考snowflake ID 生成算法](https://github.com/downgoon/snowflake/blob/master/docs/SnowflakeTutorial_zh_CN.md)

### php

* 移位運算符： 在二進制的基礎上對數字進行平移， 按移動方向和填充數字可分為如下三種：

  1. 「<<」 左移  

     * 數學意義：在沒有溢出的情況下，左移一位相等於乘以2的1次方，左移n位相當於乘以2的n次方

     * 需要移位的數字 << 移動的位數 

       ~~~php
       /*#例子 32位機 4 << 2 得到的結果是16
        *1)首先把3轉為二進制數 
            0000 0000 0000 0000 0000 0000 0000 0100
         *2) 然後把該數字高位(左側)的兩個零移除，其他數字都朝左平移2位，最後在低位(右側)的兩個空位補零。 得到結果
            0000 0000 0000 0000 0000 0000 0001 0000
         *3）得到的十進制數位 16   
       */      
       
       ~~~

  2. 「>>」 帶符號右移

     * 數學意義 ：右移一位相等於處於2，右移n位，相當於除於2的n次方

     *  需要移位的數字 << 移動的位數  

       ~~~php
       /*#例子 32位機 16 >> 2 得到的結果是4
        *1)首先把12轉為二進制數 
            0000 0000 0000 0000 0000 0000 0001 0000
         *2) 然後把地位(最右)的兩個數字移出，因為該數字是正數，所以在高位補零， 得到結果
           0000 0000 0000 0000 0000 0000 0000 0100
         *3）得到的十進制數位 4 
       */    
       ~~~

  3.  「>>>」無法號右移

* 位運算

  * `$a & $b`  And(按位與)   將把結果`$a`和`$b`中都為1的位設為1

    ~~~php
    // 如： 9 & 5 
    // 9的二進制
      0000 0000 0000 0000 0000 0000 0000 1001
    // 5的二進制
      0000 0000 0000 0000 0000 0000 0000 0101     
    // 9和5的二進制按位與 =  所以結果是1
       0000 0000 0000 0000 0000 0000 0000 0001           
    ~~~

  * `$a | $b`  Or(按位或)   將把結果`$a`和`$b`中任何一個為1的位設為1

  * `$a ^ $b`  Xor(按位異或)   將把結果`$a`和`$b`中一個1，另一個為0的位設為1

  * `~$a`  Not(按位取反)   將`$a`中為0的位設為1，反之亦然

## SnowFlake算法

### SnowFlake結構

![snowFlake](/image/arithmetic/snowFlake/snowFlake.png)

除了最高位bit標記為不可用以外，其餘三組bit，均可根據業務浮動 。默認三組位如上圖所示。

* SnowFlake共64位（1位 41位 10位 12位），即8個字節
  * 1位 ：是符號位， 固定位0 
    * 正數為0，負數為1，因為生成的數是正數，所以 該值固定為0
  * 41位： 相對時間戳(毫秒級)，請注意（該值是時間戳的差值：當前時間戳-開始時間戳）
    * 41位：大約可以使用69年
      * 計算方法：  2^41 / (3600時 * 24天 * 365年 ) /1000毫秒  約等於69年
  * 10位： 數據機器位
    * 10位：大約可以支持2^10  = 1023臺機器
    * 但，可能很多業務只涉及一、二台機器，所以這個機器碼我們可以設置為0號、1號……
  * 12位： 序列號
    * 支持一毫秒產生2^12 =4095個自增序列id



### PHP之SnowFlake

下面代碼來源於網絡

~~~php

class Snowflake
{
    static $workerId; //機器ID

    static $twepoch = 1361775855078; //起始時間戳（可以設為項目開始時間）

    static $sequence = 0; //序號從0開始
    static $sequenceMask = 1023; //序號自增最大值

    static $maxWorkerId = 15;  //workerID 最大值

    static $workerIdShift = 10; //10位

    static $timestampLeftShift = 14; //(時間要偏移的位數)

    private  static $lastTimestamp = -1; //上一個時間戳

    /**
     *
     * @param int $work_id workid
     * @throws \Exception
     */
    function __construct(int $work_id = 0){
        //workid 大小校驗
        if($work_id > self::$maxWorkerId || $work_id< 0 )
        {
            throw new \Exception("workid 不能小於{".self::$maxWorkerId."} 或小於 0");
        }
        self::$workerId=$work_id;
    }

    /**
     * 獲取當前時間戳（毫秒）
     *
     * @return float
     */
    function timeGen(){
        return  (int)sprintf("%.0f", microtime(true) * 1000);
    }

    /**
     * 得到下一毫秒時間戳
     *
     * @param $lastTimestamp  int 毫秒
     * @return int
     */
    function tilNextMillis(int $lastTimestamp) {
        //獲取當前時間戳（毫秒）
        $timestamp = $this->timeGen();
        //直到生成下一毫秒
        while ($timestamp <= $lastTimestamp) {
            $timestamp = $this->timeGen();
        }
        return $timestamp;
    }

    /**
     * 生成唯一ID
     * @return int
     */
    function  nextId()
    {
        //獲取當前當前時間戳
        $timestamp = $this->timeGen();

        //$lastTimestamp == $timestamp：表示本次調用與上一次調用落在統一毫秒內，ID生成依靠三個部分（相對毫秒數 +機器ID + 自增序號），現在兩次都落在同一毫秒上，說明前兩部分不足以區分ID，必須靠第三部分自增序列來區分
        if (self::$lastTimestamp == $timestamp) {
            //表示序号自增1，后面的按位与后，再判断if (sequence == 0) ,相當於代碼 "sequence++; if (sequence > maxSequence)"，只是按位與操作性能更高
            self::$sequence = (self::$sequence + 1) & self::$sequenceMask;
            if (self::$sequence == 0) {

                //同一機器，同一毫秒，自增序列還越界了，只能等待下一毫秒
                $timestamp = $this->tilNextMillis(self::$lastTimestamp);
            }
        } else {
            //本次調用與上次調用不在同一毫秒，這是序號可以歸零0 ，因為每一毫米的序列都是從0開始增長到
            self::$sequence = 0;
        }
        //當前時間比上一次時間還要小，這是不允許的，因為時間只能前進，否則會導致ID可能重複
        if ($timestamp < self::$lastTimestamp) {
            throw new Excwption("Clock moved backwards. Refusing to generate id for " . (self::$lastTimestamp - $timestamp) . " milliseconds");
        }
        self::$lastTimestamp = $timestamp; //記錄這一次時間
        $nextId = ((sprintf('%.0f', $timestamp) - sprintf('%.0f', self::$twepoch)) << self::$timestampLeftShift) | (self::$workerId << self::$workerIdShift) | self::$sequence;
        return $nextId;

    }

}
~~~

~~~php
 $Snowflake = new Snowflake();
$id = $Snowflake->nextId();  //生成的ID

~~~



### SnowFlake 優缺點

* 缺點
  * 依賴系統時鐘，如果時光回流，會造成ID衝突，以及，擾亂本來有順序的ID

* 優點
  *  在內存內生成ID，高性能高可用
  *  ID呈遞自增順序， 如果插入關係型數據庫，能有效利用性能（控制B-Tree裂變次數，更充分利用節點空間） 

 

