---
title: PHP微信支付开发实战-下单与支付
date: 2023-01-30 00:15:00 +0800
categories: [PHP]
tags: []
pin: false
---

> 撰写时间：2020-09-17，整理时间：2023-01-30

## 一、概述

本系列文章将讨论基于微信支付的项目开发中，涉及到的下单与支付、退款、以及订单查询的PHP后端代码实现。在本系列文章中，将以代码片段作为示例，来讨论PHP后端接口实现的过程。

在本系列的接口示例中，返回的状态码标识如下

```yaml
0: 业务成功
-1: 业务失败
```

开发环境

- ThinkPHP 6
- PHP 7 运行环境

本文是第一篇，我们先讨论下单与支付。

## 二、定义数据库

在项目中，我们通常需要在业务数据库中生成订单数据，同时需要在微信中台生成对应的订单。并且业务订单状态需要与微信订单状态一致。

```sql
-- 建立表结构
CREATE TABLE `orders`
(
    `id`              INT(11) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '表ID',
    `fee`             DECIMAL(15, 2)   NOT NULL DEFAULT 0.00 COMMENT '订单费用',
    `status`          TINYINT(1)       NOT NULL DEFAULT 0 COMMENT '状态:0-未支付,1-已支付,2-退款中,3-已退款',
    `time_start`      int(11) UNSIGNED          DEFAULT NULL COMMENT '交易起始时间',
    `time_expire`     int(11) UNSIGNED          DEFAULT NULL COMMENT '交易结束时间',
    `order_time`      int(11) UNSIGNED          DEFAULT NULL COMMENT '支付完成时间',
    `out_trade_no`    varchar(32)      NOT NULL COMMENT '商户订单号',
    `out_refund_no`   varchar(32)               DEFAULT NULL COMMENT '商户退款订单号',
    `transaction_id`  varchar(32)               DEFAULT NULL COMMENT '微信支付流水号',
    `refund_desc`     varchar(64)               DEFAULT NULL COMMENT '退款原因',
    `refund_fee`      DECIMAL(15, 2)   NOT NULL DEFAULT 0.00 COMMENT '退款金额',
    `refund_time`     int(11) UNSIGNED          DEFAULT NULL COMMENT '退款完成时间',
    `create_time`     INT(11) UNSIGNED          DEFAULT NULL COMMENT '创建时间',
    `update_time`     INT(11) UNSIGNED          DEFAULT NULL COMMENT '更新时间',
    `delete_time`     INT(11) UNSIGNED          DEFAULT NULL COMMENT '删除时间',
    CONSTRAINT `pk_id_orders` PRIMARY KEY (`id`)
) ENGINE = InnoDB
  AUTO_INCREMENT = 100
  DEFAULT CHARSET = utf8mb4
  COLLATE utf8mb4_unicode_ci COMMENT ='订单信息表';
```

在本例中我们只考虑基本数据字段，但在实际项目开发中你应当考虑实际业务需求。本例订单状态只考虑四种：未支付、已支付、退款中、已退款。数据库中，订单费用单位记录为元。

## 三、下单

```php
$order = new Order();
$order['fee'] = 0.1;
$order['out_trade_no'] = date('YmdHis').mt_rand(1000, 9999);
$order['time_start'] = time();
$order['time_expire'] = strtotime('+10 minutes');
$order->save();
```

在上面代码中，Order类代表订单模型。我们将订单费用固定设置为0.1元（实际项目中应当根据商品费用、折扣、运费等来计算），订单号我们根据时间生成，并拼接上一个随机数。要注意的是，在实际开发中，订单号要避免重复。接下来我们在common.php文件中定义一个请求微信下单接口的方法

```php
function order_request($url, $data = null)
{
    $curl = curl_init();
    curl_setopt($curl, CURLOPT_URL, $url);
    if (!empty($data)) {
        curl_setopt($curl, CURLOPT_POST, 1);
        curl_setopt($curl, CURLOPT_POSTFIELDS, $data);
    }
    curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
    $output = curl_exec($curl);
    curl_close($curl);
    return $output;
}
```

接着，定义下单的参数

```php
$params = [
    'appid' => config('wx.app_id'), // APP ID
    'mch_id' => config('wx.mch_id'), // 商户号
    'nonce_str' => md5(time()), // 随机字符串
    'sign_type' => 'MD5', // 加密方式
    'body' => '订单费用', // 商品描述
    'attach' => '微信小程序支付', // 附加信息
    'out_trade_no' => $order['out_trade_no'], // 商户订单号
    'total_fee' => $order['fee'] * 100, // 标价金额,单位分
    'spbill_create_ip' => request()->ip(), // 终端IP
    'time_start' => date('YmdHis', $order['time_start']), // 交易起始时间
    'time_expire' => date('YmdHis', $order['time_expire']), // 交易结束时间
    'notify_url' => 'https://test.com/order/callback', // 通知地址
    'trade_type' => 'JSAPI', // 交易类型
    'openid' => $user['openid'], // 用户openid
];
```

以上代码中，列出的参数是最基本的，实际项目中你可以根据业务需求设置更多的参数。此外，我们把`APPID`和商户号写在配置文件中。要注意的是，微信下单接口中，接收的金额单位为分。

微信订单接口只支持xml，接着我们从数组参数中构造xml字符串

```php
$stringA = '';
// 根据键名对参数进行字典排序
ksort($params);
// 创建xml
$xml = '';
$xml .= '<xml>';
// 遍历参数
foreach ($params as $key => $value) {
    // 拼装参数
    $stringA .= $key . '=' . $value . '&';
    //拼装xml
    $xml .= '<' . $key . '>' . $value . '</' . $key . '>';
}
// 与商户API秘钥进行拼接
$signTmp = $stringA . 'key=' . config('wx.mch_key');
$sign = strtoupper(md5($signTmp));// 签名后的32位字符

// 将签名添加到请求参数中
$xml .= '<sign>' . $sign . '</sign>';
$xml .= '</xml>';
```

下单成功后，将返回结果结构好支付参数，返回给前端调起微信支付

```php
$url = 'https://api.mch.weixin.qq.com/pay/unifiedorder'; //下单接口地址
$res_xml = order_request($url, $xml);//下单
$simpleXMLElement = simplexml_load_string($res_xml, 'SimpleXMLElement', LIBXML_NOCDATA);
// 将返回的xml转为对象
$jsonStr = json_encode($simpleXMLElement);// 将对象转为json字符串
$jsonArray = json_decode($jsonStr, true);// 将json字符串转为数组
if ($jsonArray['return_code'] != 'SUCCESS') {
    return json(['code'=>-1,'msg'=>'下单失败']);
}
// 构造支付参数
$data['appId'] = $jsonArray['appid'];
$data['timeStamp'] = strval(time());
$data['nonceStr'] = md5(time());
$data['package'] = 'prepay_id=' . $jsonArray['prepay_id'];
$data['signType'] = 'MD5';
ksort($data);

// 构造支付签名
$data['key'] = config('wx.mch_key');//商户API秘钥
$stringTmp = '';
foreach ($data as $key => $value) {
    $stringTmp .= $key . '=' . $value . '&';
}
$stringTmp = rtrim($stringTmp, '&');
$data['paySign'] = strtoupper(md5($stringTmp));
unset($data['key']);

// 将支付参数返回给前端
return json($data);
```

在以上代码中，我们把商户API秘钥写在配置文件中。

## 四、支付通知回调

当前端用户支付成功之后，微信会将支付结果发送一个GET请求回调到业务服务器，我们需要设置好对应的路由并匹配上回调方法，回调地址就是下单时设置的通知地址，通知内容是XML。在回调方法中，我们先获取参数。

```php
$xml = file_get_contents('php://input');
if (!$xml) {
    exit(0);// xml数据为空，终止程序
}
// 将xml解析为对象
$simpleXMLElement = simplexml_load_string($xml, 'SimpleXMLElement', LIBXML_NOCDATA);
// 将对象转为数组
$jsonStr = json_encode($simpleXMLElement);
$jsonArray = json_decode($jsonStr, true);
```

接下来验证签名

```php
$stringA = '';
foreach ($jsonArray as $key => $value) {
    if ($key == 'sign') {
        continue;
    }
    // 拼装参数
    $stringA .= $key . '=' . $value . '&';
}
$signTmp = $stringA . 'key=' . config('wx.mch_key');// 与商户API秘钥进行拼接
$sign = strtoupper(md5($signTmp));// 签名后的32位字符
if ($sign != $jsonArray['sign']) {
    exit(0);// 签名错误，终止程序
}
```

验证签名通过之后，更新订单状态，并且将结果返回给微信服务器即可

```php
$order = Order::where('out_trade_no',$jsonArray['out_trade_no'])->find();
if (!$order) {
    exit(0);// 查询不到订单，终止程序
}
// 更新订单状态
$order['status'] = 1;
$order['order_time'] = time();
$order['transaction_id'] = $jsonArray['transaction_id'];
$order->save();
// 给微信返回数据
$xml = '<xml>';
$xml .= '<return_code>' . '<![CDATA[SUCCESS]]>' . '</return_code>';
$xml .= '<return_msg>' . '<![CDATA[OK]]>' . '</return_msg>';
$xml .= '</xml>';
return response($xml, 200, [], 'xml');
```

> 本文原创首发自微信订阅号**极客开发者**，禁止转载
