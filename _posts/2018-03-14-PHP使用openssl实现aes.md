---
layout: post
title: 'PHP使用openssl实现aes'
date: 2018-03-14 11:15:32 +0800
categories: ['编程']
tags: ['php']
author: Alex
---

## 1 链接

个人博客: [alex-my.xyz](http://alex-my.xyz)

CSDN: [blog.csdn.net/alex_my](http://blog.csdn.net/alex_my)

# 2 代码与说明

- 说明

  - IV: 向量，在下面的模式下为 16 字长字符串，UTF8 编码，128 位
  - KEY: 密钥，16 字长字符串，UTF8 编码，128 位
  - BLOCK: 128
  - MODE: CBC
  - PADDING: PKCS5/PKCS7, 在验证网站上测试，没发现区别
  - 加密结果以 Base64 方式编码

- 代码

  ```php
  class AESHelp {
      private $iv;
      private $key;
      private $method = 'AES-128-CBC';

      function __construct($iv, $key) {
          $this->iv = $iv;

          $ivLength = openssl_cipher_iv_length($this->method);
          if (strlen($this->iv) < $ivLength) {
              throw new Exception("iv less than {$ivLength}");
          }

          $this->key = $key;
      }

      function encrypt($data) {
          return base64_encode(openssl_encrypt($data, $this->method, $this->key, $options = OPENSSL_RAW_DATA, $this->iv));
      }

      function decrypt($data) {
          return openssl_decrypt(base64_decode($data), $this->method, $this->key, $options = OPENSSL_RAW_DATA, $this->iv);
      }
  }
  ```

# 3 示例

```txt
$iv = 'a123456789012345';
$key = 'hello';
$aes = new AESHelp($iv, $key);

$str1 = $aes->encrypt('123456');
echo "str1: {$str1} <br>";  // 输出: kkO6XPGyof3DQLSunkeTuA==

$str2 = $aes->decrypt($str1);
echo "str2: {$str2} <br>";  // 输出：123456
```

- 可上以下网站进行检测
  - [http://www.seacha.com/tools/aes.html](http://www.seacha.com/tools/aes.html)
