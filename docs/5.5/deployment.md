---
layout: post
title: deployment
---
# 部署

- [介紹](#introduction)
- [伺服器設定](#server-configuration)
    - [Nginx](#nginx)
- [優化](#optimization)
    - [自動載入優化](#autoloader-optimization)
    - [優化設定檔的載入](#optimizing-configuration-loading)
    - [優化路由的載入](#optimizing-route-loading)
- [部署到 Forge](#deploying-with-forge)

<a name="introduction"></a>
## 介紹

當你準備部署 Laravel 應用程式到正式上線主機時，你能執行一些操作，用來確保你的應用程式能更有效的執行。在這個文件中，我們將介紹如何用很棒的方式來部署應用程式。

<a name="server-configuration"></a>
## 伺服器設定

<a name="nginx"></a>
### Nginx

如果你把應用程式部署到執行 Nginx 的伺服器，你可以使用下面的設定內容來開始設定你的網頁伺服器。此文件內容最好是根據你的伺服器設定需求來客製化。如果你需要託管你的伺服器，可以考慮 [Laravel Forge](https://forge.laravel.com) 等服務：

    server {
        listen 80;
        server_name example.com;
        root /example.com/public;

        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Content-Type-Options "nosniff";

        index index.html index.htm index.php;

        charset utf-8;

        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }

        location = /favicon.ico { access_log off; log_not_found off; }
        location = /robots.txt  { access_log off; log_not_found off; }

        error_page 404 /index.php;

        location ~ \.php$ {
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass unix:/var/run/php/php7.1-fpm.sock;
            fastcgi_index index.php;
            include fastcgi_params;
        }

        location ~ /\.(?!well-known).* {
            deny all;
        }
    }

<a name="optimization"></a>
## 優化

<a name="autoloader-optimization"></a>
### 自動載入優化

部署到正式上線主機的時候，確保你優化了 Composer 的類別自動載入器映射，以便 Conposer 能更快的找到正確的檔案並加載到給定的檔案：

    composer install --optimize-autoloader

> {tip} 除了優化自動載入器，你應該始終將 `composer` 放在你的專案版本控制系統。當 `composer.lock` 檔案存在於你的系統時，專案的依賴套件項目能被快速的安裝。

<a name="optimizing-configuration-loading"></a>
### 優化設定檔的載入

當你在部署應用程式到正式上線主機時，你應該記得在你部署過程中有執行 Artisan 指令的 `config:cache`：

    php artisan config:cache

這個指令可以將所有的 Laravel 設定檔快取到一個檔案，這會大大減少了載入設定值時框架對檔案系統的存取次數。

<a name="optimizing-route-loading"></a>
### 優化路由的載入

如果你正在建構一個很多路由的大型應用程式時，你應該確保在部署過程中執行 Artisan 的 `route：cache` 命令：

    php artisan route:cache

這個指令會將所有的路由註冊減少為快取檔案中的單個方法呼叫，有助於提升數擁有百個路由的系統的速度。

> {note} 因為這個功能使用到 PHP 序列化，所以你只能快取專門使用控制器類別的路由。PHP 目前無法將閉包給序列化。

<a name="deploying-with-forge"></a>
## 部署到 Forge

如果你還沒有準備好管理自己的伺服器設定，又或者不太會設定執行強大的 Laravel 應用程式所需的各種服務，[Laravel Forge](https://forge.laravel.com) 會是一個不錯的選擇。

Laravel Forge 能在各種伺服器服務的提供商（像是 DigitalOcean、Linode 或 AWS 等等）上建立伺服器。除此之外，Forge 也可以安裝和管理建構 Laravel 應用程式所需的所有工具，像是 Nginx、MySQL、Redis、Memcached 和 Beanstalk 等。
