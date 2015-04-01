# 加密

* 介绍
* 基本用法

## 介绍

Laravel 通过 Mcrypt PHP 扩充扩展包提供功能强大的 AES 加密功能。

## 基本用法

#### 加密
```
    $encrypted = Crypt::encrypt('secret');
```
> **注意：** 请确保 `config/app.php` 文件中的 `key` 选项配置了 16, 24, 或 32 字符的随机字串，否则加密的数值不会安全。

#### 解密
```
    $decrypted = Crypt::decrypt($encryptedValue);
```
#### 配置密码与模式

您也可以使用加密器来配置密码和模式：

```
    Crypt::setMode('ctr');

    Crypt::setCipher($cipher);
```