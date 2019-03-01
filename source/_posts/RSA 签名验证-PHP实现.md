---
title: RSA 签名验证-PHP实现
date: 2019-03-01 14:37:10
categories:
    - programming
tags: 
    - RSA
    - php
---

```php
    /**
     * 数据签名
     * @param $data
     * @param $privateCertPath
     * @return string
     */
    public static function sign(string $data, $privateCertPath)
    {
        $data       = is_array($data) ? base64_encode(json_encode($data, JSON_UNESCAPED_SLASHES | JSON_UNESCAPED_UNICODE)) : $data;
        $privateKey = file_get_contents($privateCertPath);
        $privateKey = openssl_pkey_get_private($privateKey);
        $crypted    = '';
        openssl_sign($data, $crypted, $privateKey);
        openssl_free_key($privateKey);

        $signedData = base64_encode($crypted);

        return $signedData;
    }

    /**
     * 签名验证
     * @param        $data
     * @param string $sign
     * @param string $publicCertPath
     * @return bool
     */
    public static function verify($data, string $sign, $publicCertPath)
    {
        $data    = is_array($data) ? base64_encode(json_encode($data, JSON_UNESCAPED_SLASHES | JSON_UNESCAPED_UNICODE)) : $data;
        $pubKey  = file_get_contents($publicCertPath);
        $pubKey  = openssl_pkey_get_public($pubKey); // 可用返回资源 id
        $isValid = (bool)openssl_verify($data, base64_decode($sign), $pubKey);
        openssl_free_key($pubKey);

        return $isValid;
    }


```