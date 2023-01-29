---
title: PHP微信支付开发实战-订单查询
date: 2023-01-30 00:18:00 +0800
categories: [PHP]
tags: []
pin: false
---

> 撰写时间：2020-10-23，整理时间：2023-01-30

## 一、概述

本系列文章将讨论基于微信支付的项目开发中，涉及到的下单与支付、退款、以及订单查询的后端代码实现。在本系列文章中，将以代码片段作为示例，来讨论PHP后端接口实现的过程。

在本系列的接口示例中，返回的状态码标识如下：

```yaml
0: 业务成功
-1: 业务失败
```

开发环境

- ThinkPHP 6
- PHP 7 运行环境

本文是第三篇，我们先讨论订单查询。

订单查询接口是用于查询订单状态的，当用户支付成功之后，或者退款成功之后，微信服务器可能没有及时完成通知（这种概率很小）。我们可以设置订单查询接口给前段调用来更新业务的订单状态，及时完成订单状态的更新。

## 二、请求查询接口

```php
// 根据客户端传过来的业务订单id，获取到业务订单对象
$order = Orders::where('id',request()->params('id'))->find();
if (!$order) {
    return json(['code':-1,'msg':'订单不存在']);
}
// 构造请求微信接口的参数
$params = [
    'appid' => config('wx.app_id'),// APP ID
    'mch_id' => config('wx.mch_id'),// 商户号
    'out_trade_no' => $order['out_trade_no'],
    'nonce_str' => md5(time()),
    'sign_type' => 'MD5',
];
// 构造xml
$xml = '<?xml version="1.0" encoding="utf-8"?>';
$xml .= '<xml>';
$stringA = '';
ksort($params);
foreach ($params as $key => $value) {
    $stringA .= $key . '=' . $value . '&';
    $xml .= '<' . $key . '>' . $value . '</' . $key . '>';
}
$signTmp = $stringA . 'key=' . config('wx.mch_key');// 拼接商户号
$sign = strtoupper(md5($signTmp));// 签名后的32位字符
// 将签名添加到xml中
$xml .= '<sign>' . $sign . '</sign>';
$xml .= '</xml>';
```

构造好参数之后，发送请求

```php
// 发起https请求
$url = 'https://api.mch.weixin.qq.com/pay/orderquery';
$res_xml = order_request($url, $xml);
$simpleXMLElement = simplexml_load_string($res_xml, 'SimpleXMLElement', LIBXML_NOCDATA);
// 将SimpleXMLElement对象转为数组
$jsonStr = json_encode($simpleXMLElement);
$jsonArray = json_decode($jsonStr, true);
```

如果订单已经支付，更新为已支付状态，如果订单已退款，更新为退款状态

```php
// 更新订单状态
if (strval($jsonArray['return_code']) === 'SUCCESS') {
    if (strval($jsonArray['trade_state']) === 'SUCCESS') {
        if ($order['status'] !== 1) {
            // 支付成功
            $order['status'] = 1;
            $order['transaction_id'] = $jsonArray['transaction_id'];
        }
    } elseif (strval($jsonArray['trade_state']) === 'REFUND') {
        if ($order['status'] !== 3) {
            // 退款成功
            $order['status'] = 3;
            $order['transaction_id'] = $jsonArray['transaction_id'];
        }
    }
    $order->save();
}
```
