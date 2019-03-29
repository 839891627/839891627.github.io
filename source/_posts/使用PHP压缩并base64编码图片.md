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

class Image
{

    /**
     * 图片 base64 压缩并编码
     * @param $image_file
     * @return string
     */
    public static function base64EncodeImage($image_file)
    {
        $tmpImg = __DIR__ . '/tmp_compressed_img.jpg'; // 图片压缩临时文件
        self::compress($image_file, $tmpImg); // 压缩图片

        $base64_image = '';
        $image_info = getimagesize($tmpImg);
        $image_data = fread(fopen($tmpImg, 'r'), filesize($tmpImg));
        $base64_image = 'data:' . $image_info['mime'] . ';base64,' . chunk_split(base64_encode($image_data));

        unlink($tmpImg); // 删除临时图片

        return $base64_image;
    }

    /**
     * 图片压缩
     * @param $source string 源文件地址
     * @param $destination string 存放压缩后文件地址
     * @param $quality int 压缩质量
     * @return mixed
     */
    public static function compress($source, $destination, $quality = 75)
    {
        $info = getimagesize($source);

        if ($info['mime'] == 'image/jpeg') {
            $image = imagecreatefromjpeg($source);
        } elseif ($info['mime'] == 'image/gif') {
            $image = imagecreatefromgif($source);
        } elseif ($info['mime'] == 'image/png') {
            $image = imagecreatefrompng($source);
        }

        imagejpeg($image, $destination, $quality);

        return $destination;
    }

}

$source = '/Users/caojinliang/Downloads/Abstract 2.jpg';
$img = Image::base64EncodeImage($source);
echo "<img src='$img' width='800' height='600'>";

```
