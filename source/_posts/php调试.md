---
title: php调试
date: 2018-08-29 12:54:23
categories: php
---

配置php xdebug.

<!-- more-->

- ### Xdebug 安装

  - 在 `phpinfo()`中确定版本

  - 在 <https://xdebug.org/download.php> 寻找合适版本(64位和32位可以都试一试)

    需要注意 TS,VC11部分

  <img src="/img/post/php/debug/1.png">

  - 将下载后的内容放到 <font size="3" color="red">xampp/php/ext</font> 中

  - 在php.ini中添加如下配置

      ```ini
            [Xdebug]
            zend_extension="E:\code\language\php\xampp5\php\ext\php_xdebug-2.5.5-5.6-vc11.dll"
            xdebug.remote_enable = on
            xdebug.remote_handler = dbgp
            xdebug.remote_mode = req
            xdebug.remote_host = localhost
            xdebug.remote_port = 9000
            xdebug.idekey = PHPSTORM
      ```

  - 重启apache服务.检查phpinfo()中是否有 xdebug

- ### 配置phpstorm
  - File->Settings->Languages & Frameworks->PHP->Servers

    <img src="/img/post/php/debug/2.png">

  - File->Settings->Languages & Frameworks->PHP->Debug

    <img src="/img/post/php/debug/3.png">

- ### 配置 XDebug helper

  - 在Chrome中搜索并安装 XDebug helper 扩展。
  - 安装成功后，在 Chrome 扩展程序列表中找到 XDebug helper，点击选项，将 IDE key 选项选为 PhpStorm

- ### 开启

  - 在 PHPStorm 中开启 Debug 监听，点击右上角像电话一样的图标，图标变绿表示成功；
  - 在 Chrome 中开启 XDebug helper 插件：