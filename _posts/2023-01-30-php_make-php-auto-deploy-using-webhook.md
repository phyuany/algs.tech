---
title: 怎样利用钩子实现git仓库的PHP代码自动部署？
date: 2023-01-30 00:12:00 +0800
categories: [PHP]
tags: []
pin: false
---

> 撰写时间：2019-03-04，整理时间：2023-01-30

## 一、什么是自动部署钩子？

项目地址

- 码云：[https://gitee.com/jikerdev/PHPWebHook](https://gitee.com/jikerdev/PHPWebHook)
- GitHub：[https://github.com/jikerdev/PHPWebHook](https://github.com/jikerdev/PHPWebHook)

简单地说自动部署钩子就是实现代码同步的一个程序，程序会在特定的情况会被触发，比如开发者将代码推送到git服务器时。本文使用PHP语言来编写一个能实现PHP项目自动部署的程序。

## 二、目标需求

本文使用的是[码云](https://gitee.com)作为示例，在我们的业务服务器上部署钩子程序，当我们推送代码到[码云](https://gitee.com)仓库之后，使[码云](https://gitee.com)触发网络钩子功能，实现代码同步到业务服务器，达到项目自动部署的目的。首先需要实现代码同步功能即可，同时，代码同步到业务服务器之后，发送通知邮件给代码推送者。

## 二、实现过程

### 2.1 初始化项目

创建一个空的项目目录，在目录之下使用composer安装一个phpmailer邮件发送依赖库，composer指令如下

```shell
composer require phpmailer/phpmailer
```

### 2.2 定义邮件发送者对象

在项目根目录创建MailSender.php文件，首先在头部引入在1中安装的phpmailer依赖，如下

```php
<?php
require_once 'vendor/autoload.php';

// 引入phpmailer依赖
use PHPMailer\PHPMailer\Exception;
use PHPMailer\PHPMailer\PHPMailer;
use PHPMailer\PHPMailer\SMTP;
```

在MailSender.php文件中添加MailSender类，并设置SMTP邮件发送的相关参数，如下代码

```php
class MailSender
{
    /*
     * SMTP配置
     * */
    private $smtp_host = 'smtp.exmail.qq.com';//SMTP服务器地址
    private $smtp_from = '极客开发者-管理员';//发送者
    private $smtp_username = 'admin@jkdev.cn';//邮箱账号
    private $smtp_password = '******';//邮箱密码
    private $smtp_port = '465';//端口号
}
```

再创建是实例化邮件发送者的方法obtainEmailSender，第一个参数是邮件发送的目标邮箱数组（也就是说，可以同时将通知邮件发到多个目标邮箱），第二个参数代表发送主题，第三代表邮件内容，如下代码：

```php
public function obtainEmailSender(array $addresses, $subject, $body)
    {
        $mailSender = new PHPMailer(true);
        $mailSender->CharSet = 'UTF-8';

        //Server settings
        $mailSender->SMTPDebug = SMTP::DEBUG_SERVER;                       // Enable verbose debug output
        $mailSender->isSMTP();                                             // Send using SMTP
        $mailSender->Host = $this->smtp_host;                              // Set the SMTP server to send through
        $mailSender->SMTPAuth = true;                                      // Enable SMTP authentication
        $mailSender->Username = $this->smtp_username;                      // SMTP username
        $mailSender->Password = $this->smtp_password;                      // SMTP password
        $mailSender->SMTPSecure = PHPMailer::ENCRYPTION_SMTPS;             // Enable TLS encryption; `PHPMailer::ENCRYPTION_SMTPS` also accepted
        $mailSender->Port = $this->smtp_port;                              // TCP port to connect to

        //Recipients
        $mailSender->setFrom($this->smtp_username, $this->smtp_from);
        foreach ($addresses as $index => $address) {
            $mailSender->addAddress($address);                             // Name is optional
        }

        // Content
        $mailSender->isHTML(true);                                  // Set email format to HTML
        $mailSender->Subject = $subject;
        $mailSender->Body = $body;

        //返回邮件对象
        return $mailSender;
    }
```

### 2.3 定义钩子

创建钩子入口文件index.php，并引入MailSender.php文件，如下代码：

```php
<?php
include 'MailSender.php';
```

首先验证提交者是否将代码提交到master分支，其次验证密码是否正确。如果验证不通过时，直接退出程序，如下代码：

```php
// 检测IP
if (!in_array($_SERVER['REMOTE_ADDR'], $allowIpArr)) {
    echo '非法IP:' . $_SERVER['REMOTE_ADDR'];
    exit(0);
}

// 获取请求参数
$headers = getallheaders();
$body = json_decode(file_get_contents("php://input"), true);
// 请求密码
$password = 'www.jkdev.cn';

// 验证提交分支是否为master
if (!isset($body['ref']) || $body['ref'] !== 'refs/heads/master') {
    echo '非主分支' . $body;
    exit(0);
}

// 验证提交密码是否正确
if (!isset($body['password']) || $body['password'] !== $password) {
    echo '密码错误';
    exit(0);
}
```

通过验证之后，在服务器拉取git服务器上的最新代码

```php
// 验证成功，拉取代码
$path = $body['project']['path'];
$command = 'cd /var/www/html/' . $path . ' && git pull 2>&1';
$res = shell_exec($command);
```

在以上代码中，先使用cd命令进入服务器上的项目目录，这里要注意，项目后缀路径必须和git服务器上的项目路径是一致的。再使用git pull命令拉取代码，使用2>&1指令会返回git执行结果。最后使用shell_exec执行命令并使用$res变量来接收执行结果。
最后，定义通知邮件发送内容，代码如下：

```php
// 发送邮件
$addresses = [
    $body['sender']['email'],// 将邮件发送给发送者
    $body['repository']['owner']['email']// 将邮件发送给仓库所有者
];
// 去除重复的内容
$addresses = array_unique($addresses);

try {
    // 更新说明
    $title = '部署成功通知';
    // 构造邮件内容
    $message = $body['head_commit']['message'];// 提交信息
    $datetime = date('Y-m-d H:i:s', $body['timestamp'] / 1000);// 时间
    $pusher = $body['pusher']['name'];// 提交人
    $name = $body['project']['name'];// 项目名
    $path = $body['project']['path'];// 路径
    $content = <<<HTML
<html>
<body>
    <h2>{$body['project']['name']}已部署成功</h2>
    <p>
    描述：<span style="font-size: 16px; color: cadetblue">$message</span> <br>
    时间：<span style="font-size: 16px; color: red">$datetime</span> <br>
    提交人：<span style="font-size: 16px; color: cadetblue">$pusher</span> <br>
    项目名称：<span style="font-size: 16px; color: cadetblue">$name</span> <br> 
    项目路径：<span style="font-size: 16px; color: cadetblue">$path</span>
    </p>
</body>
</html>
HTML;

    // 发送邮件
    $emailSender = (new MailSender())->obtainEmailSender($addresses, $title, $content);
    $emailSender->send();
    // 返回结果
    echo '邮件发送成功，git pull执行结果：' . $res;
} catch (\PHPMailer\PHPMailer\Exception $e) {
    echo '邮件发送失败，git pull执行结果：' . $res . '，邮件日志：' . $e;
}
```

在以上代码中，我们使用代码推送者和仓库所有者作为目标邮件通知对象。如果两个目标是同一个邮箱，将只取一个。其次构造邮件发送内容，使用邮件发送者的send方法进行邮件发送。最终，将git拉取结果和邮件发送结果响应给请求者。

## 三、总结

本文结合[码云](https://gitee.com)的网络钩子（web hook）功能，使用PHP代码编写了一个HTTP接口，当开发者往[码云](https://gitee.com)上提交代码时，将触发钩子携带相关信息去调用业务服务器接口，从而我们可以在业务服务器上触发shell命令去同步git服务器上的代码，达到自动部署的目的。你还可以参考码云网络钩子的文档，进而进行代码改进，实现其他的网络钩子相关的业务需求！

参考链接：[https://gitee.com/help](https://gitee.com/help)
本文原创首发自微信公众号：[极客开发者](https://blog.jkdev.cn/usr/uploads/2017/11/612527812.jpg)，
禁止转载!
