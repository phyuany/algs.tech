---
title: PHP微信支付开发实战-退款
date: 2023-01-30 00:16:00 +0800
categories: [PHP]
tags: []
pin: false
---

> 撰写时间：2020-09-17，整理时间：2023-01-30

## 一、概述

本系列文章将讨论基于微信支付的项目开发中，涉及到的下单与支付、退款、以及订单查询的后端代码实现。在本系列文章中，将以代码片段作为示例，来讨论 PHP 后端接口实现的过程。

在本系列的接口示例中，返回的状态码标识如下：

```yaml
0: 业务成功
-1: 业务失败
```

开发环境

- ThinkPHP 6
- PHP 7 运行环境

本文是第二篇，我们讨论退款问题

## 二、退款申请

订单支付成功之后即可退款，退款的金额可以小于或者等于订单的下单金额。请求参数相对下单接口略有变化，如下代码

```php
$order = Order::where('id',request()->params('id'))->find();
$order['out_refund_no'] = date('YmdHis').mt_rand(1000, 9999);
$params = [
    'appid' => config('wx.app_id'),
    'mch_id' => config('wx.mch_id'),
    'nonce_str' => md5(time()),
    'sign_type' => 'MD5',
    'transaction_id' => $order['transaction_id'],
    'out_trade_no' => $order['out_trade_no'],
    'out_refund_no' => $order['out_refund_no'],
    'total_fee' => $order['fee'] * 100,
    'refund_fee' => $order['fee'] * 100,
    'refund_desc' => $params['refund_desc'],
    'notify_url' => 'https://test.com/orders/callback_refund',// 通知地址
];
```

构造xml

```php
// 创建xml
$xml = '<?xml version="1.0" encoding="utf-8"?>';
$xml .= '<xml>';
// 遍历参数
$stringA = '';
// 根据键名对参数进行字典排序
ksort($params);
foreach ($params as $key => $value) {
    $stringA .= $key . '=' . $value . '&';
    $xml .= '<' . $key . '>' . $value . '</' . $key . '>';
}
$signTmp = $stringA . 'key=' . config('wx.mch_key');// 与商户API秘钥进行拼接
$sign = strtoupper(md5($signTmp));// 签名后的32位字符

// 将签名添加到请求参数中
$xml .= '<sign>' . $sign . '</sign>';
$xml .= '</xml>';
```

退款申请需要安全证书（在微信商户号里申请），我们重新定义在common.php一个用于请求的方法

```php
function order_refund_request($url, $data = null)
{
    $curl = curl_init();
    curl_setopt($curl, CURLOPT_URL, $url);
    if (!empty($data)) {
        curl_setopt($curl, CURLOPT_POST, 1);
        curl_setopt($curl, CURLOPT_POSTFIELDS, $data);
    }
    curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
    // 证书设置
    curl_setopt($curl, CURLOPT_SSLCERTTYPE, 'PEM');
    curl_setopt($curl, CURLOPT_SSLCERT, dirname(__FILE__) . '/cert/apiclient_cert.pem');// 客户端cert路径
    curl_setopt($curl, CURLOPT_SSLKEYTYPE, 'PEM');
    curl_setopt($curl, CURLOPT_SSLKEY, dirname(__FILE__) . '/cert/apiclient_key.pem');// 客户端key路径

    $output = curl_exec($curl);
    curl_close($curl);
    return $output;
}
```

这里要注意证书的路径，一定要匹配。接着发送退款请求

```php
$url = 'https://api.mch.weixin.qq.com/secapi/pay/refund';
$res_xml = order_refund_request($url, $xml);
trace('微信中台申请退款，返回信息');
trace($res_xml);
$simpleXMLElement = simplexml_load_string($res_xml, 'SimpleXMLElement', LIBXML_NOCDATA);
//将SimpleXMLElement转为数组
$jsonStr = json_encode($simpleXMLElement);
$jsonArray = json_decode($jsonStr, true);
if (isset($jsonArray['return_code']) && $jsonArray['return_code'] == 'SUCCESS') {
    // 退款申请成功，更新订单状态
    $order['status'] = 2;
    return json(['code'=>0,'msg'=>'成功']);
} else {
    //响应失败
    return json(['code'=>-1,'msg'=>'响应失败']);
}
```

## 三、退款通知回调

退款回调返回的数据是加密的，回调地址是退款申请中的通知地址。我们需要先解密返回数据，再根据返回数据去更新订单状态。首先在当前类里定义解密方法

```php
private function refundDecrypt($str, $key)
{
    $key = md5($key);
    $str = base64_decode($str);
    return openssl_decrypt($str, 'aes-256-ecb', $key, OPENSSL_RAW_DATA);
}
```

接下来，获取参数并解密

```php
$xml = file_get_contents('php://input');
if (!$xml) {
    exit(0);
}
$simpleXMLElement = simplexml_load_string($xml, 'SimpleXMLElement', LIBXML_NOCDATA);
// 将SimpleXMLElement转为数组
$jsonStr = json_encode($simpleXMLElement);
$jsonArray = json_decode($jsonStr, true);

// 中台返回数据解密
if (!isset($jsonArray['req_info'])) {
    exit(0);
}
$decryptStr = $this->refundDecrypt($jsonArray['req_info'], 
// 将xml解析为array
$simpleXMLElement = simplexml_load_string($decryptStr, 'SimpleXMLElement', LIBXML_NOCDATA);
// 将SimpleXMLElement对象转为数组
$jsonStr = json_encode($simpleXMLElement);
// 解析字段
$jsonResArr = json_decode($jsonStr, true);
// 验证参数
if (!(isset($jsonResArr['out_refund_no']) && isset($jsonResArr['refund_status']) && (strval($jsonResArr['refund_status']) === 'SUCCESS'))) {
    exit(0);//参数错误，终止程序
}
// 通过退款订单号查询订单
$order = Order::where('out_refund_no',$jsonResArr['out_refund_no'])->find();
if (!$order) {
    exit(0);//查询不到相应订单,终止程序
}
// 更新订单状态
if ($order['status'] !== 3) {
    // 更新订单状态
    trace('微信退款回调,正在更新订单状态');
    $order['status'] = 3;
    $order['transaction_id'] = $jsonResArr['transaction_id'];
    $order['refund_fee'] = round($jsonResArr['refund_fee'] / 100, 2);
    $order['refund_time'] = time();
    $saveOrder = $order->save();
}
// 给微信返回数据
$xml = '<xml>';
$xml .= '<return_code>' . '<![CDATA[SUCCESS]]>' . '</return_code>';
$xml .= '<return_msg>' . '<![CDATA[OK]]>' . '</return_msg>';
$xml .= '</xml>';
return response($xml, 200, [], 'xml');
```
