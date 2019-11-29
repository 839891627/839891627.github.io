---
title: guzzle http 上传文件
date: 2019-11-29 18:28:25
categories: 编程
tags: php
---

```php
    $params = \Yii::$app->request->post();
    foreach ($_FILES as $name => $FILE) {
        $data[] = [
            'name' => $name,
            'filename' => $FILE['name'],
            'contents' => fopen($FILE['tmp_name'], 'r'),
        ];
    }
    foreach ($params as $key => $value) {
        $data[] = [
            'name' => $key,
            'contents' => $value,
        ];
    }

    // xxxx

    $ret = self::$client->request('post', url, [
        'multipart' => $params, // 使用 multipart
        'headers'   => $headers
    ]);
```
