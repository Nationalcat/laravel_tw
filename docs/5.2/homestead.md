---
layout: post
title: homestead
tag: 5.2
---
# Laravel Homestead

- [簡介](#introduction)
- [安裝與設定](#installation-and-setup)
    - [前置動作](#first-steps)
    - [設定 Homestead](#configuring-homestead)
    - [啟動 Vagrant box](#launching-the-vagrant-box)
    - [根據專案分別安裝](#per-project-installation)
- [常見用法](#daily-usage)
    - [Accessing Homestead Globally](#accessing-homestead-globally)
    - [透過 SSH 連接](#connecting-via-ssh)
    - [連接資料庫](#connecting-to-databases)
    - [增加更多網站](#adding-additional-sites)
    - [設定 Cron 排程器](#configuring-cron-schedules)
    - [連接埠](#ports)
- [Blackfire 分析器](#blackfire-profiler)

<a name="introduction"></a>
## 簡介

Laravel 致力於讓 PHP 開發體驗更愉快，也包含你的本地開發環境。[Vagrant](http://vagrantup.com) 提供了一個簡單、優雅的方式來管理與供應虛擬機器。

Laravel Homestead 是一個官方預載的 Vagrant box，提供你一個美好的開發環境，你不需要在你的本機電腦安裝 PHP、HHVM、網頁伺服器或任何伺服器軟體。不用擔心搞亂你的系統！Vagrant box 可以搞定一切。如果有什麼地方爛掉了，你可以在幾分鐘內快速的砍掉並重建虛擬機器！

Homestead 可以在任何 Windows、Mac 或 Linux 系統上面執行，裡面包含了 Nginx 網頁伺服器、PHP 7.0、MySQL、Postgres、Redis、Memcached、Node，以及所有你在使用 Laravel 開發各種精彩的應用程式時所需要的軟體。

> **附註：**如果您是 Windows 的使用者，您可能需要啟用硬體虛擬化（VT-x）。這通常需要透過 BIOS 來啟用它。

<a name="included-software"></a>
### 內建軟體

- Ubuntu 14.04
- Git
- PHP 7.0
- HHVM
- Xdebug
- Nginx
- MySQL
- Sqlite3
- Postgres
- Composer
- Node（附帶了 PM2、Bower、Grunt 與 Gulp）
- Redis
- Memcached
- Beanstalkd
- [Blackfire 分析器](#blackfire-profiler)

<a name="installation-and-setup"></a>
## 安裝與設定

<a name="first-steps"></a>
### 前置動作

在啟動你的 Homestead 環境之前，你必須先安裝 [VirtualBox 5.x](https://www.virtualbox.org/wiki/Downloads) 或 [VMWare](http://www.vmware.com) 以及 [Vagrant](http://www.vagrantup.com/downloads.html)。這些軟體在各個常用的平台都有提供易用的視覺化安裝程式。

若要使用 VMware provider，你需要同時購買 VMware Fusion / Workstation 及 [VMware Vagrant plug-in](http://www.vagrantup.com/vmware)。雖然他不是免費的，但 VMware 可以在共享資料夾上獲得較快的性能。

#### 安裝 Homestead Vagrant box

當 VirtualBox / VMware 以及 Vagrant 安裝完成後，你可以在終端機以下列指令將 'laravel/homestead' 這個 box 安裝進你的 Vagrant 程式中。下載 box 會花你一點時間，時間長短將依據你的網路速度決定：

    vagrant box add laravel/homestead

如果此指令執行失敗了，請確保你安裝的 Vargrant 是最新版。

#### 安裝 Homestead

你可以簡單地透過手動 clone 資源庫的方式來安裝 Homestead。建議可將資源庫克隆至你的「home」目錄中的 `Homestead` 資料夾，如此一來 Homestead box 將能提供主機服務給你所有的 Laravel 專案：

    cd ~

    git clone https://github.com/laravel/homestead.git Homestead

一旦你克隆完 Homestead 資源庫，即可在 Homestead 目錄中執行 `bash init.sh` 指令來建立 `Homestead.yaml` 設定檔。`Homestead.yaml` 檔案將會被放置在 `~/.homestead` 隱藏目錄中：

    bash init.sh

<a name="configuring-homestead"></a>
### 設定 Homestead

#### 設定 Vagrant 提供者

在 `Homestead.yaml` 檔案中的 `provider` 是用來設定你想要使用哪一個 Vagrant 提供者，像是：`virtualbox`、`vmware_fusion` 或 `vmware_workstation`。你可以根據喜好來決定提供者：

    provider: virtualbox

#### 設定共享目錄

你可以在 `Homestead.yaml` 檔案的 `folders` 屬性裡列出所有你想與 Homestead 環境共享的目錄。這些目錄中的檔案若有更動，它們將會同步更動在你的本機電腦與 Homestead 環境。你可以將多個共享目錄都設定於此：

    folders:
        - map: ~/Code
          to: /home/vagrant/Code

若要啟用 [NFS](http://docs.vagrantup.com/v2/synced-folders/nfs.html)，你只需要在共享目錄的設定值中加入一個簡單的參數：

    folders:
        - map: ~/Code
          to: /home/vagrant/Code
          type: "nfs"

#### 設定 Nginx 網站

對 Nginx 不熟悉嗎？沒關係。`sites` 屬性幫助你可以輕易的指定 `網域` 對應至 homestead 環境中的目錄。在 `Homestead.yaml` 檔案中已包含一個網站設定範例。同樣的，你可以增加數個網站到 Homestead 環境中。Homestead 可以為每個你正在開發中的 Laravel 專案提供方便的虛擬化環境：

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public

你可以將 `hhvm` 屬性設定為 `true` 讓 Homestead 裡面的任一個網站改用 [HHVM](http://hhvm.com)：

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          hhvm: true

#### Hosts 檔案

你必須為 Nginx 網站在你機器中的 `hosts` 檔案增加「網域」。`hosts` 檔案會將你對 Homestead 網站的請求重導至 Homestead 機器。在 Mac 或 Linux 上，該檔案通常會存放在 `/etc/hosts`。在 Windows 上，則存放於 `C:\Windows\System32\drivers\etc\hosts`。你增加至該檔案的內容看起來會像這樣：

    192.168.10.10  homestead.app

務必確認 IP 位置與你的 `~/.homestead/Homestead.yaml` 檔案中設定相同。一旦將網域設定在 `hosts` 檔案之後，你就可以透過網頁瀏覽器造訪網站！

    http://homestead.app

<a name="launching-the-vagrant-box"></a>
### 啟動 Vagrant box

當你編輯完 `Homestead.yaml` 後，在終端機裡進入你的 Homestead 目錄並執行 `vagrant up` 指令。Vagrant 就會自將虛擬主機啟動並自動設定你的共享目錄和 Nginx 網站。

如果要移除虛擬機器，可以使用 `vagrant destroy --force` 指令。

<a name="per-project-installation"></a>
### 根據專案分別安裝

有別於將 Homestead 安裝成全域環境且讓所有的專案共用同一個 Homestead box，你可以各別為每一個專案獨立配置一個 Homstead。如果你希望直接在專案裡傳遞 `Vagrantfile`，那麼替每個專案安裝 Homestead 即是你可以考慮的方式，這將會允許其他人可以簡單地執行 `vagrant up` 即能開始工作於此專案。

你可以使用 Composer 將 Homestead 直接安裝至你的專案中：

    composer require laravel/homestead

一旦 Homestead 安裝完畢，你可以使用 `make` 指令產生 `Vagrantfile` 與 `Homestead.yaml` 存放於專案的根目錄。這個 `make` 指令將會自動配置 `sites` 及 `folders` 於 `Homestead.yaml` ：

Mac / Linux:

    php vendor/bin/homestead make

Windows:

        vendor\bin\homestead make

接著，執行在終端機中執行 `vagrant up` 並透過網頁瀏覽器造訪 `http://homestead.app`。再次提醒，你仍然需要在 `/etc/hosts` 裡設定 `homestead.app` 或其他想要使用的網域。

<a name="daily-usage"></a>
## 常見用法

<a name="accessing-homestead-globally"></a>
### 全域存取 Homestead

有時你可能想你檔案系統任何地方 `vagrant up` 你的 Homestead 機器。你可以增加簡單的 Bash 別名至你的 Bash 設定檔來做到。此別名能讓你在系統的任何位置執行所有的 Vagrant 指令，並自動在你的 Homestead 安裝位置執行：

    alias homestead='function __homestead() { (cd ~/Homestead && vagrant $*); unset -f __homestead; }; __homestead'

確保調整別名中的 `~/Homestead` 路徑為你實際 Homestead 的安裝位置。一旦別名被設定後，你可以在系統的任何位置執行像是 `homestead up` 或 `homestead ssh` 的指令。

<a name="connecting-via-ssh"></a>
### 透過 SSH 連接

你可以在終端機裡進入你的 Homestead 目錄並執行 `vagrant ssh` 指令藉此以 SSH 連上你的虛擬主機。

但是，你可能會經常需要透過 SSH 連上你的 Homestead 主機，因此你可以考慮在你的本機電腦上創建一個上述的「別名」來快速 SSH 至 Homestead box。

<a name="connecting-to-databases"></a>
### 連接資料庫

`Homestead` 的資料庫設定了 MySQL 與 Postgres 兩種資料庫，並可立即使用。為了方便使用，Laravel 的 `env` 檔案預設會設定框架會使用此資料庫。

如果想要從本機電腦上透過 Navicat 或者是 Sequel Pro 連接你的 MySQL 或 Postgres 資料庫，你應該連接 `127.0.0.1` 的連接埠 `33060`（MySQL）或 `54320`（Postgres）。而資料庫的帳號及密碼分別是 `homestead` 與 `secret`。

> **注意：**在本機電腦你應該只使用這些非標準的連接埠來連接資料庫。因為當 Laravel 執行於虛擬主機_中_時，你會在 Laravel 的資料庫設定檔使用預設的 3306 及 5432 連接埠。

<a name="adding-additional-sites"></a>
### 增加更多網站

一旦 Homestead 環境設定完畢且直行後，你可能會想要為 Laravel 應用程式增加更多的 Nginx 網站。你可以在單一個 Homestead 環境中執行多個 Laravel 安裝程式。若要增加額外的網站，只要增加網站至你的 `~/.homesteadHomestead.yaml` 檔案中，接著在你的 Homestead 目錄執行 `vagrant provision` 終端機指令。

<a name="configuring-cron-schedules"></a>
### 設定 Cron 排程器

Laravel 提供了便利的方式來[排程 Cron 任務](/laravel_tw/docs/5.2/scheduling)，透過 `schedule:run` Artisan 指令，排程便會在每分鐘被執行。`schedule:run` 指令會檢查定義在你 `App\Console\Kernel` 類別中排程的任務，判斷哪個任務該被執行。

如果你想為 Homestead 網站使用 `schedule:run` 指令，你可以在定義網站時設置 `schedule` 選項為 `true`：

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          schedule: true

該網站的 Cron 任務會被定義於虛擬機器的 `/etc/cron.d` 資料夾中。

<a name="ports"></a>
### 連接埠

以下的連接埠將會被轉發至 Homestead 環境：

- **SSH：**2222 &rarr; 轉發至 22
- **HTTP：**8000 &rarr; 轉發至 80
- **HTTPS：**44300 &rarr; 轉發至 443
- **MySQL：**33060 &rarr; 轉發至 3306
- **Postgres：**54320 &rarr; 轉發至 5432

#### 轉發更多連接埠

如果你希望，你也可以藉著指定連接埠的通訊協定來轉發更多額外的連接埠給 Vagrant box：

    ports:
        - send: 93000
          to: 9300
        - send: 7777
          to: 777
          protocol: udp

<a name="blackfire-profiler"></a>
## Blackfire 分析器

由 SensioLabs 推出的 [Blackfire 分析器](https://blackfire.io) 能協助你自動收集程式運行時的相關數據，像是 RAM、CPU time 及 disk I/O。若想在 Homestead 中替你的應用程式使用這個分析器是非常容易的。

在 Homestead box 中同樣已經預裝好所有需要的套件，很簡單地你只需要在 `Homestead.yaml` 檔案中設定你的 Blackfire **Server ID 及 token 即可：

    blackfire:
        - id: your-server-id
          token: your-server-token
          client-id: your-client-id
          client-token: your-client-token

一旦你設定完 Blackfire 的認證，在你的 Homestead 目錄下執行 `vagrant provision` 來重新配置 box。當然，請務必閱讀 [Blackfire 使用文件](https://blackfire.io/getting-started) 來學習如何在你的網頁瀏覽器上安裝 Blackfire 擴充套件。
