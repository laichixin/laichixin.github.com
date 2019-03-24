---
layout:     post
title:      微信小程序
subtitle:   微信小程序2-應用篇
date:       2019-03-16
author:     Francis
header-img: image/bg-desk.jpeg
catalog: true
tags:
    - wechat
---

# 微信小程序2-應用篇

##  配置環境

### 域名問題

* 由於小程序只可與指定域名進行網絡通訊，所以是需要配置域名的，請[參考網絡域名文檔](<https://developers.weixin.qq.com/miniprogram/dev/framework/ability/network.html>)

* <font color="red">有一個情景，想讓本地小程序與連接本地（http://127.0.0.1:9402）服務器通訊，而不使用合法域名，怎麼辦呢？</font>

  * 猜想，你應該會遇到這個問題

    ![cloudfunction](/image/wechat/2019-03-24-wechat_miniprogram/illegal_domain_name.png)

  * 別急，你只需要在微信小程序，「設置」->「項目設置」->把“不校验合法域名、web-view（业务域名）、TLS版本以及HTTPS证书”選項勾選上，就可以與本地服務器通訊了

    ![cloudfunction](/image/wechat/2019-03-24-wechat_miniprogram/skip_demain_name_inspection.png)

  * 

* 

## 登錄











