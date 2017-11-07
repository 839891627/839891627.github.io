---
title: PHP实现打包下载
date: 2017-11-07 14:45:48
categories: 编程
tags: php
---

```php
    $files   = [
        'https://farm6.staticflickr.com/5584/14985868676_b51baa4071_h.jpg',
        'https://farm6.staticflickr.com/5591/15008867125_68a8ed88cc_m.jpg'
    ];
    $zipname = time(); // 打包后的文件名

    $tmpFile = tempnam(storage_path(), ''); // 临时文件目录

    $zip = new \ZipArchive();
    $zip->open($tmpFile, \ZipArchive::CREATE);

    foreach ($files as $file) {
        $fileContent = file_get_contents($file);
        $zip->addFromString(basename($file), $fileContent);
    }
    $zip->close();

    header('Content-Type: application/zip');
    header('Content-disposition: attachment; filename=' . $zipname . '.zip');
    header('Content-Length: ' . filesize($tmpFile));
    readfile($tmpFile);

    unlink($tmpFile); // 删除临时文件
```
