---
title: 怎样使用JWT进行授权验证？
date: 2023-01-31 00:56:00 +0800
categories: [理论基础]
tags: []
pin: false
---

## 一、概述

JWT可以取代以往的基于 COOKIE/SESSION 的鉴权体系，是目前最热门跨域鉴权的解决方案，接下来从 JWT 的原理，到 PHP 示例代码，简单说明业务怎样使用 JWT 进行授权验证。

## 二、JWT的原理是什么？

JWT定制了一个标准，实际上就是将合法用户（一般指的是 通过 账号密码验证、短信验证，以及小程序code，或者通过其他验证逻辑  验证为合法的用户）的授权信息，加密起来，然后颁发给客户端。客户端请求需要鉴权的接口的时候，通过 HTTP报文 头部的 Authorization回传。

首先，需要指定一个密钥（secret）。这个密钥只有服务器才知道，不能泄露给用户。然后，使用 Header 里面指定的签名算法（默认是 HMAC SHA256），按照下面的公式产生签名：

```text
HMACSHA256(
 base64UrlEncode(header) + "." +
 base64UrlEncode(payload),
 secret)
```

下面是 JWT包含的数据：

- Header（头部）

Header 部分是一个 JSON 对象，描述 JWT 的元数据，通常是下面的样子。

```json
{
 "alg": "HS256",
 "typ": "JWT"
}
```

上面的JSON对象中，alg属性表示签名的算法，默认是 HMAC SHA256；typ属性表示这个令牌（token）的类型。

- Payload（负载）

此部分主要用用于存放数据，其中有官方指定的默认字段如下

```yaml
iss: 签发人
exp: 过期时间
sub: 主题
aud: 受众
nbf: 生效时间
iat: 签发时间
jti: JWT编号
```

我们还可以添加自己的字段，但是不要我加密的信息放在这里，因为Paypload数据是谁都能解析出来的。我们一般把uid（用户id）、用户名等 开放信息存在这里

- Signature（签名）

Signature是JWT最重要的部分，是对前两部分的签名，防止数据篡改。

## 三、怎样使用JWT？

基于JWT制定的标准，衍生出来了实现 JWT 的多个开源库 [https://jwt.io/libraries](https://jwt.io/libraries) ，当然我们也可以自己实现 JWT 的逻辑，但是也没有必要重复去造轮子，开发者应该利用更多的时间 去思考 有意义的事情。我们可以使用由 Google Firebase 开发的 [firebase/php-jwt 库](https://github.com/firebase/php-jwt)， 这个库也是目前最热门的 PHP JWT 库。下面介绍基于该库，实现常用的两种 JWT 验证方式。

- HS256加密 ：生成与验证JWT

使用 HS256 算法生成 JWT，这是一种对称加密，使用同一个密钥串进行加密和解密。

加密过程：

```php
// 加密密钥，要尽可能复杂点
$key = 'kol.mama.com.cn.12334556';
$payload = [
  // 签发者
  'iss' => 'kol',
  // 主题
  'sub' => 'kol user authorization',
  // 签发时间
  'iat' => time(),
  // 生效时间
  'nbf' => time(),
  // 截止时间
  'exp' => time() + 3600 * 24 * 3,
  // 自定义字段：uid
  'uid' => 123456,
  // 自定义字段：用户名
  'user_name' => '用户1'
];
$token = JWT::encode($payload, $key);
```

解密过程：

```php
$token = str_replace('Bearer ', '', $request->header('Authorization'));
$payload = JWT::decode($token, $key, ['HS256']);
```

- RS256加密 ：生成与验证JWT

这是一种非对称加密，加密和解密使用 一个 密钥对

```shell
# 生成私钥
ssh-keygen -t rsa -b 2048 -f private.key
# 使用私钥生成公钥
openssl rsa -in private.key -pubout -outform PEM -out public.key
```

加密过程

```php
$priKey = file_get_contents('./private.key');
$payload = [
  // 签发者
  'iss' => 'kol',
  // 主题
  'sub' => 'kol user authorization',
  // 签发时间
  'iat' => time(),
  // 生效时间
  'nbf' => time(),
  // 截止时间
  'exp' => time() + 3600 * 24 * 3,
  // 自定义字段：uid
  'uid' => 123456,
  // 自定义字段：用户名
  'user_name' => '用户1'
];
$token = JWT::encode($payload, $priKey, 'RS256');
```

解密过程

```php
$pubKey = file_get_contents('./public.key');
$token = str_replace('Bearer ', '', $request->header('Authorization'));
$payload = JWT::decode($token, $pubKey, ['RS256']);
```

- JWT 解密（验证）

如果正常通过验证，将解析出 payload 在加密前的原数据，我们可以基础处理业务逻辑；

如果 token 已经过期，或者 token 是非法 token，这时候我们通常认为用户的操作是 非法请求，系统也将会抛出对应的异常，我们只需进行捕获并 处理相关拦截的 逻辑即可。如下示例代码：

```php
try {
    // 验证JWT
    $token = str_replace('Bearer ', '', $request->header('Authorization'));
    $payload = JWT::decode($token, config('jwt.key'), ['HS256']);
} catch (Exception $exception) {
    // 终止业务逻辑，向客户端返回错误信息
    $data = [
        'code' => $exception->getCode() ?: 401,
        'msg' => $exception->getMessage(),
        'data' => null,
    ];
    return json($data);
}
```

## 四、客户端怎么回传JWT？

JWT 官网的标准是将 JWT 凭证放在 HTTP 报文 头部的 Authorization 中进行请求，如向服务器请求 用户的 个人信息，HTTP报文 如下示例

```text
GET https://api.example.com/user
Accept: application/json
Authorization: Bearer $TOKEN
```

## 五、使用JWT要注意什么？

JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。

JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，JWT 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证（如通过手机 验证码 再次验证，或者再次输入用户密码进行验证）。
