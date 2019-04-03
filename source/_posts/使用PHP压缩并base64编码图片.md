---
title: 使用PHP压缩并base64编码图片
date: 2019-03-29 11:44:53
categories: 编程
tags: 
    - 图片
    - php
---

> - 图片 base64 的目的不是减小其大小，而是嵌入在网页中，以此避免浏览器再去对图片发送另一个单独的请求的开销
> - 图片 base64 后会变大为原来的 4/3

<!-- more -->

```php
<?php

/**
 * 图像处理公用类
 * Created by PhpStorm.
 * User: user
 * Date: 2019/1/16
 * Time: 下午4:22
 */
class Image
{

    /**
     * 图片 base64 压缩并编码
     * @param $image_file
     * @return string
     */
    public static function base64EncodeImage($image_file)
    {
        $base64_image = '';
        if (filesize($image_file) / 1024 / 1024 > 1) {
            // 源文件大于 1m 进行压缩
            $tmpImg = self::compress($image_file); // 压缩图片
            $base64_image = 'data:image/jpeg;base64,' . chunk_split(base64_encode($tmpImg));
        } else {
            $image_info = getimagesize($image_file);
            $image_data = fread(fopen($image_file, 'r'), filesize($image_file));
            $base64_image = 'data:' . $image_info['mime'] . ';base64,' . chunk_split(base64_encode($image_data));
        }

        return $base64_image;
    }

    /**
     * 图片压缩
     * @param $source string 源文件地址
     * @param $quality int 压缩质量
     * @param $destination string 存放压缩后文件地址
     * @return false|string
     */
    public static function compress($source, $quality = 50, $destination = '')
    {
        ini_set('memory_limit', -1); // 以防内存问题
        $info = getimagesize($source);

        if ($info['mime'] == 'image/jpeg') {
            $image = imagecreatefromjpeg($source);
        } elseif ($info['mime'] == 'image/gif') {
            $image = imagecreatefromgif($source);
        } elseif ($info['mime'] == 'image/png') {
            $image = imagecreatefrompng($source);
        }

        if ($destination) {
            // 如果指定保存文件路径
            imagejpeg($image, $destination, $quality);

            return $destination;
        } else {
            // 直接输出 image stream
            ob_start();
            imagejpeg($image, null, $quality);
            $data = ob_get_clean();

            return $data;
        }
    }
}

$source = '/Users/caojinliang/Downloads/Abstract 2.jpg';
$img = Image::base64EncodeImage($source);
echo "<img src='$img' width='800' height='600'>";

```
