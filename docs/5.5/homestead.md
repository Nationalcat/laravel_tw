---
layout: post
title: homestead
tag: 5.5
---
# Laravel Homestead

- [介紹](#introduction)
- [安裝與設定](#installation-and-setup)
    - [第一個步驟](#first-steps)
    - [設定 Homestead](#configuring-homestead)
    - [啟動 Vagrant Box](#launching-the-vagrant-box)
    - [根據專案分別安裝](#per-project-installation)
    - [安裝 MariaDB](#installing-mariadb)
    - [安裝 Elasticsearch](#installing-elasticsearch)
    - [別名](#aliases)
- [常見用法](#daily-usage)
    - [全域存取 Homestead](#accessing-homestead-globally)
    - [透過 SSH 連接](#connecting-via-ssh)
    - [連接 Databases](#connecting-to-databases)
    - [新增更多網站](#adding-additional-sites)
    - [環境變數](#environment-variables)
    - [設定 Cron 排程器](#configuring-cron-schedules)
    - [設定 Mailhog](#configuring-mailhog)
    - [連接埠](#ports)
    - [共享環境變數](#sharing-your-environment)
    - [多個 PHP 版本](#multiple-php-versions)
    - [網頁伺服器](#web-servers)
- [網路介面](#network-interfaces)
- [更新 Homestead](#updating-homestead)
- [特定虛擬機設定](#provider-specific-settings)
    - [VirtualBox](#provider-specific-virtualbox)

<a name="introduction"></a>
## 介紹

Laravel 致力於讓 PHP 開發體驗更愉快，也包含你的本地開發環境。[Vagrant](https://www.vagrantup.com)  提供了一個簡單、優雅的方式來管理與供應虛擬機器。

Laravel Homestead 是一個官方預載的 Vagrant box，提供你一個美好的開發環境，而不再需要你在本機電腦上安裝 PHP、網頁伺服器或任何伺服器軟體。完全不用擔心會搞亂你的系統！Vagrant box 可以搞定這一切。如果有什麼地方壞掉了，你也可以在幾分鐘內快速的砍掉並重建虛擬機器！

Homestead 可以在任何 Windows、MacOS 或 Linux 系統上執行，並內建了 Nginx 網頁伺服器、PHP 7.2、PHP 7.1、PHP 7.0、PHP 5.6、MySQL、PostgreSQL、Redis、Memcached、Node 以及所有你在使用 Laravel 開發各種精彩的應用程式時所需要的軟體。

> {note} 如果你是使用 Windows，你可能需要啟用硬體虛擬化（VT-x）。這通常需要透過 BIOS 來啟用它。如果你的 UEFI 系統上有使用 Hyper-V，還需停用 Hyper-V 才能夠存取 VT-x。

<a name="included-software"></a>
### 內建軟體

<div class="content-list" markdown="1">
- Ubuntu 16.04
- Git
- PHP 7.2
- PHP 7.1
- PHP 7.0
- PHP 5.6
- Nginx
- Apache（可選）
- MySQL
- MariaDB（可選）
- Sqlite3
- PostgreSQL
- Composer
- Node (附帶 Yarn、Bower、Grunt 和 Gulp)
- Redis
- Memcached
- Beanstalkd
- Mailhog
- Elasticsearch（可選）
- ngrok
</div>

<a name="installation-and-setup"></a>
## 安裝與設定

<a name="first-steps"></a>
### 第一個步驟

在啟動 Homestead 環境之前，你必須先安裝 [VirtualBox 5.2](https://www.virtualbox.org/wiki/Downloads)、[VMWare](https://www.vmware.com) 、 [Parallels](https://www.parallels.com/products/desktop/)，或是 [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v)，接著安裝 [Vagrant](https://www.vagrantup.com/downloads.html)。這些軟體在各個常用的平台都有提供易用的視覺化安裝程式。

若要使用 VMware provider，你將需要同時購買 VMware Fusion / Workstation 以及 [VMware Vagrant plug-in](https://www.vagrantup.com/vmware)。雖然這不是免費的，但是 VMware 能夠在共享資料夾上提供較快的性能。

若要使用 Parallels provider，你將需要安裝 [Parallels Vagrant plug-in](https://github.com/Parallels/vagrant-parallels)。這是免費的。

受限於 [Vagrant 的限制](https://www.vagrantup.com/docs/hyperv/limitations.html)，Hyper-V 可為你忽略所有的網路設定。

#### 安裝 Homestead Vagrant Box

一旦 VirtualBox / VMware 和 Vagrant 安裝完成了，你就能在終端機上使用以下指令將 `laravel/homestead` box 安裝到你的 Vagrant 中。下載 box 會花你一點時間，時間長短將依據你的網路頻寬來決定：

    vagrant box add laravel/homestead

如果這個指令失敗了，請確認一下你的 Vagrant 是否是最新版的。

#### 安裝 Homestead

你可以透過手動 clone 資源庫的方式來安裝 Homestead。建議將資源庫複製到你的「home」目錄中的 Homestead 資料夾，如此一來 Homestead box 就能提供主機服務給你所有的 Laravel 專案：

git clone https://github.com/laravel/homestead.git ~/Homestead

你應該檢查一下 Homestead 的標籤版本，因為 `master` 分支並非是穩定版本。你能在 [GitHub 發佈頁面](https://github.com/laravel/homestead/releases)上找到最新的穩定版本：

    cd ~/Homestead

    // 複製預期的版本...
    git checkout v7.1.2

如果你已經複製了 Homestead 資源庫，就可以從 Homestead 目錄中執行 `bash init.sh` 指令來建立 `Homestead.yaml` 設定檔。`Homestead.yaml` 檔案會被放置在 Homestead 目錄中：

    // Mac / Linux...
    bash init.sh

    // Windows...
    init.bat

<a name="configuring-homestead"></a>
### 設定 Homestead

#### 設定你的虛擬機

在 `Homestead.yaml` 檔案中的 `provider` 是用來設定你想要使用哪一個 Vagrant 提供者，像是：`virtualbox`、`vmware_fusion`、`vmware_workstation`、`parallels` 或 `hyperv`。你可以根據喜好來決定提供者：

    provider: virtualbox

#### 設定共用資料夾

你可以在 `Homestead.yaml` 檔案的 `folders` 屬性裡列出所有你想與 Homestead 環境共享的目錄。這些目錄中的檔案若有更動，它們會將你的本機電腦的變更給同步 Homestead 環境。因此，你可以將多個共享目錄都設定於此：

    folders:
        - map: ~/code
          to: /home/vagrant/code

如果你只有建立幾個網站，這種映射方式會很好使用。然而，隨著網站數量不斷的增加，你會發現效能開始變差。這問題會在低接設備或專案有大量檔案的狀況下格外有感。如果你遇到這問題，請試著將每個專案映射到各自的 Vagrant 資料夾中：

    folders:
        - map: ~/code/project1
          to: /home/vagrant/code/project1

        - map: ~/code/project2
          to: /home/vagrant/code/project2

如果要啟用 [NFS](https://www.vagrantup.com/docs/synced-folders/nfs.html)，你只需要在共享目錄的設定值中加入一個簡單的參數：

    folders:
        - map: ~/code
          to: /home/vagrant/code
          type: "nfs"

> {note} 在你使用 NFS 的時候，應該考慮安裝 [vagrant-bindfs](https://github.com/gael-ian/vagrant-bindfs) 插件。這個插件會保留 Homestead box 中的檔案與目錄的使用者或群組的權限。

你也可以透過 Vagrant 的 [Synced Folders](https://www.vagrantup.com/docs/synced-folders/basic_usage.html)，將想帶入的任何選項列在 options 關鍵字下方

    folders:
        - map: ~/code
          to: /home/vagrant/code
          type: "rsync"
          options:
              rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
              rsync__exclude: ["node_modules"]

#### 設定 Nginx 網站

對 Nginx `不熟悉嗎？沒關係。sites` 屬性幫助你可以輕易的指定「網域」對應至 homestead 環境中的目錄。在 `Homestead.yaml` 檔案中已包含一個網站設定範例。同樣的，你可以增加數個網站到 Homestead 環境中。Homestead 可以為每個你正在開發中的 Laravel 專案提供方便的虛擬化環境：

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public

在配置 Homestead box 之後，若有更改 `sites` 屬性，你應該重新執行配置指令 `vagrant reload --provision` 來更新虛擬機裡的 Nginx 設定。

#### Hosts 檔案

你必須為 Nginx 網站在你機器中的 `hosts` 檔案增加「網域」。`hosts` 檔案會將你對 Homestead 網站的請求重導至 Homestead 機器。在 Mac 或 Linux 上，該檔案通常會存放在 `/etc/hosts`。在 Windows 上，則存放於 `C:\Windows\System32\drivers\etc\hosts`。你增加至該檔案的內容看起來會像這樣：

    192.168.10.10  homestead.test

務必確認 IP 位置與你的 `Homestead.yaml` 檔案中設定相同。一旦將網域設定在 `hosts` 檔案之後，你就可以透過網頁瀏覽器造訪網站！

    http://homestead.test

<a name="launching-the-vagrant-box"></a>
### 啟動 Vagrant Box

當你編輯完 `Homestead.yaml` 後，開啟終端機，進入 Homestead 目錄，並執行 `vagrant up` 指令。Vagrant 就會自將虛擬主機啟動並自動設定共享目錄和 Nginx 網站。

如果要移除虛擬機器，可以使用 `vagrant destroy --force` 指令。

<a name="per-project-installation"></a>
### 根據專案分別安裝

有別於將 Homestead 安裝成全域環境且讓所有的專案共用同一個 Homestead box，你可以各別為每一個專案獨立配置一個 Homstead。如果你希望直接在專案裡傳遞 `Vagrantfile`，那麼替每個專案安裝 Homestead 即是你可以考慮的方式，這將會允許其他人可以簡單地執行 `vagrant up` 即能開始工作於此專案。

使用 Composer 將 Homestead 直接安裝至你的專案中：

    composer require laravel/homestead --dev

一旦 Homestead 安裝完畢，你可以使用 `make` 指令產生 `Vagrantfile` 與 `Homestead.yaml` 存放於專案的根目錄。這個 `make` 指令將會自動配置 `sites` 及 `folders` 於 `Homestead.yaml`。

Mac / Linux:

    php vendor/bin/homestead make

Windows:

    vendor\\bin\\homestead make

接著，在終端機中執行 vagrant up 指令，並透過網頁瀏覽器造訪 http://homestead.test 。再次提醒，你仍然需要在 `/etc/hosts` 裡設定 `homestead.test` 或其他想要使用的網域。

<a name="installing-mariadb"></a>
### 安裝 MariaDB

如果您偏好 MariaDB 而不是 MySQL，可以在 `Homestead.yaml` 檔案中加入 `mariadb` 選項。這選項會移除 MySQL 並同時安裝 MariaDB。 MariaDB 是 MySQL 的一個直接替代品，所以在你的環境設定中的資料庫驅動程式仍然需使用 `mysql`：

    box: laravel/homestead
    ip: "192.168.10.10"
    memory: 2048
    cpus: 4
    provider: virtualbox
    mariadb: true

<a name="installing-elasticsearch"></a>
### 安裝 Elasticsearch

若要安裝 Elasticsearch，請將 `elasticsearch` 選項新增到你的 `Homestead.yaml` 檔案。預設的安裝會建立一個名為「homestead」的集群。你絕對不行將作業系統一半的記憶體分配給 Elasticsearch，所以請確認你的 Homestead 機器是否至少有兩倍的記憶體：

    box: laravel/homestead
    ip: "192.168.10.10"
    memory: 4096
    cpus: 4
    provider: virtualbox
    elasticsearch: 6

> {tip} 可以到 [Elasticsearch 官方文件](https://www.elastic.co/guide/en/elasticsearch/reference/current)找到更多設定方法。

<a name="aliases"></a>
### 別名

透過修改 Homestead 目錄中的 `aliases` 檔案可以把 Bash 別名新增到你的 Homestead 機器中：

    alias c='clear'
    alias ..='cd ..'

更新 `aliases` 檔案內容之後，你應該使用  `vagrant reload --provision` 指令來重新設定 Homestead 機器。這會確保你的新別名可以在機器上使用。

<a name="daily-usage"></a>
## 常見用法

<a name="accessing-homestead-globally"></a>
### 全域存取 Homestead

有時你可能想從任何地方 `vagrant up` 你的 Homestead 機器。你可以在 Mac / Linux 上增加簡單的 Bash 函式至你的 Bash 設定檔來做到。 在 Windows 上，你可以添加一個「batch」檔案到你的 `PATH`。此函式會自動指到你的 Homestead 安裝位置，能讓你在系統的任何位置執行任意的 Vagrant 指令：

#### Mac / Linux

    function homestead() {
        ( cd ~/Homestead && vagrant $* )
    }

確保函式中 `~/Homestead` 路徑為你實際 Homestead 的安裝位置。一旦函式被設定後，你可以在系統的任何位置執行像是 `homestead up` 或 `homestead ssh` 的指令。

#### Windows

在你的機器上的任何地方建立一個 `homestead.bat` batch 檔案，並包含以下內容:

    @echo off

    set cwd=%cd%
    set homesteadVagrant=C:\Homestead

    cd /d %homesteadVagrant% && vagrant %*
    cd /d %cwd%

    set cwd=
    set homesteadVagrant=

確認你有將腳本中的路徑 `C:\Homestead` 調整成你 Homesteam 的安裝位置。建立完檔案之後，把此檔案位置加入你的 `PATH`。你就可以從系統上的任何地方執行像是 `homestead up` 或 `homestead ssh` 的指令。

<a name="connecting-via-ssh"></a>
### 透過 SSH 連接

你可以在終端機裡進入你的 Homestead 目錄，並執行 `vagrant ssh` 指令藉此以 SSH 連上你的虛擬主機。

但是，你可能會經常需要透過 SSH 連上你的 Homestead 主機，因此你可以考慮在你的本機電腦上創建一個上述的「Bash 函式」來快速 SSH 至 Homestead box。

<a name="connecting-to-databases"></a>
### 連接資料庫

homestead 的資料庫已經設定了 MySQL 與 Postgres 兩種資料庫。為了方便使用，Laravel 的 `.env` 檔案預設會設定框架會使用此資料庫。

如果要從本機資料庫的客戶端連接到 MySQL 或 PostgreSQL 資料庫，你應該連接到 `127.0.0.1` 和 port `33060`（MySQL）或 `54320`（PostgreSQL）。資料庫的帳號及密碼為 `homestead` / `secret`。

> {note} 在本機電腦你應該只使用這些非標準的連接埠來連接資料庫。因為當 Laravel 執行於虛擬主機中時，你會在 Laravel 的資料庫設定檔使用預設的 3306 及 5432 連接埠。

<a name="adding-additional-sites"></a>
### 新增更多網站

一旦完成了 Homestead 環境設定並成功執行，你可能會想要為你的 Laravel 應用程式新增更多的 Nginx 網站。你可以在單一個 Homestead 環境中執行許多的 Laravel 安裝。若要新增其他網站，只要新增該網站到你的 `Homestead.yaml` 檔案中：

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
        - map: another.test
          to: /home/vagrant/code/another/public

如果 Vagrant 不再自動管理你的「hosts」檔案，你可能需要將新的網站新增到這個檔案中：

    192.168.10.10  homestead.test
    192.168.10.10  another.test

一旦新增好網站，請從 Homestead 目錄中執行 `vagrant reload --provision` 指令。

<a name="site-types"></a>
#### 網站類型

Homestead 支援幾種類型的網站來讓你輕易的執行非 Laravel 的專案。例如，我們可以使用 `symfony2` 網站類型來輕易的將 Symfony 應用程式新增到 Homestead：

    sites:
        - map: symfony2.test
          to: /home/vagrant/code/Symfony/web
          type: "symfony2"

目前可用的網站類型有：`apache`、`laravel`（預設）、`proxy`、`silverstripe`、`statamic`、`symfony2` 和 `symfony4`。

<a name="site-parameters"></a>
#### 網站參數

你可以透過 `params` 網站指令來新增額外的 Nginx 的 `fastcgi_param` 值到你的網站。例如，我們將要新增一個 `BAR` 值到 `FOO` 參數：

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
          params:
              - key: FOO
                value: BAR

<a name="environment-variables"></a>
### 環境變數

你能新增它們到你的 `Homestead.yaml` 檔案來設定全域的環境變數：

    variables:
        - key: APP_ENV
          value: local
        - key: FOO
          value: bar

在更新 `Homestead.yaml` 之前，請務必執行過 `vagrant reload --provision` 指令來重新設定機器。這會更新所有安裝的 PHP 版本的 PHP-FPM 設定，且還會更新 `vagrant` 使用者的環境。

<a name="configuring-cron-schedules"></a>
### 設定 Cron 排程器

Laravel 提供了便利的方式來[排程 Cron 任務](/laravel_tw/docs/5.5/scheduling)，透過 Artisan 的 `schedule:run` 指令，排程便會在每分鐘被執行。`schedule:run` 指令會檢查你定義在 `App\Console\Kernel` 類別中排程的任務，判斷哪個任務該被執行。

如果你想為 Homestead 網站使用 `schedule:run`  指令，你可以在定義網站時設置 `schedule` 選項為 `true`：

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
          schedule: true

該網站的 Cron 任務會被定義於虛擬機器的 `/etc/cron.d` 資料夾中。

<a name="configuring-mailhog"></a>
### 設定 Mailhog

Mailhog 可以讓你輕易的擷取你送出的 Email 並檢查它，而不用真的將郵件發送給其他人。請使用以下郵件設定來更新 `.env` 檔案：

    MAIL_DRIVER=smtp
    MAIL_HOST=localhost
    MAIL_PORT=1025
    MAIL_USERNAME=null
    MAIL_PASSWORD=null
    MAIL_ENCRYPTION=null

<a name="ports"></a>
### 連接埠

以下的連接埠預設將會被轉發至 Homestead 環境：

- **SSH:** 2222 &rarr; Forwards To 22
- **ngrok UI:** 4040 &rarr; Forwards To 4040
- **HTTP:** 8000 &rarr; Forwards To 80
- **HTTPS:** 44300 &rarr; Forwards To 443
- **MySQL:** 33060 &rarr; Forwards To 3306
- **PostgreSQL:** 54320 &rarr; Forwards To 5432
- **Mailhog:** 8025 &rarr; Forwards To 8025

#### 轉發到其他連接埠

你可以轉發更多額外的連接埠給 Vagrant box，同時也可指定連接埠的通訊協定：

    ports:
        - send: 50000
          to: 5000
        - send: 7777
          to: 777
          protocol: udp

<a name="sharing-your-environment"></a>
### 共享環境變數

有時候你可能會希望和合作夥伴分享你現在的工作環境或者分享到一個client上。Vagrant有一個內建的方法，透過 `vagrant share` 支援這個功能;然而，如果你有多個網站同時使用你的 `Homestead.yaml` 檔案，這功能將無法使用。

要解決這個問題，Homestead 加入了自己的 `share` 指令。開始前，透過 `vagrant ssh` 連線到你的 Homestead 機器然後執行 `share homestead.test`。這會從你的 `Homestead.yaml` 分享 `homestead.test` 網站。當然，你可以將 `homestead.test`替換成任何其他網站。

    share homestead.test

執行這個指令之後，你將會看到一個 Ngrok 視窗，其中包含著活動記錄和共享網站的公開存取網址。如果你想指定一個自定的區域、子網域或者其他 Ngrok runtime 選項，你可以加入到 `share` 下:

    share homestead.test -region=eu -subdomain=laravel

> {note} 請記住，Vagrant 本質上還是不安全的，當你執行 `share` 指令，你會暴露你的虛擬機器位置到網路上。

<a name="multiple-php-versions"></a>
### 多個 PHP 版本

> {note} 這個功能只相容於 Nginx。

Homestead 6 在同一個虛擬機上支援了多個 PHP 版本的切換。你可以在 `Homestead.yaml` 檔案中指定給定網站要使用的 PHP 版本。目前可用的 PHP 版本有：`5.6`、`7.0`、`7.1` 和 `7.2`（預設）：

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
          php: "5.6"

另外，你可以透過 CLI 來使用任何有支援的 PHP 版本：

    php5.6 artisan list
    php7.0 artisan list
    php7.1 artisan list
    php7.2 artisan list

<a name="web-servers"></a>
### 網頁伺服器

Homestead 預設採用 Nginx 作為網頁伺服器。然而，如果指定 `apache` 作為網站類型，就會改安裝 Apache。雖然兩個網站伺服器都能安裝，但無法同時*運行*。`flip` 是簡易切換網頁伺服器的指令。`flip` 指令會自行確定運行中的伺服器已關閉，才會接著啟動另一台機器。你可以透過 SSH 進到 Homestead 機器上執行該指令：

    flip

<a name="network-interfaces"></a>
## 網路介面

`Homestead.yaml` 檔案中的 `networks` 屬性，用來配置 Homestead 環境中的網路介面卡。你可以依需求配置多張介面卡：

    networks:
        - type: "private_network"
          ip: "192.168.10.20"

啟用 [橋接模式](https://www.vagrantup.com/docs/networking/public_network.html) 介面卡，設定 `bridge` 屬性，並且更改 network type 屬性為 `public_network`:

    networks:
        - type: "public_network"
          ip: "192.168.10.20"
          bridge: "en1: Wi-Fi (AirPort)"

啟用 [DHCP模式](https://www.vagrantup.com/docs/networking/public_network.html)，只要從設定中移除 `ip` 屬性：

    networks:
        - type: "public_network"
          bridge: "en1: Wi-Fi (AirPort)"

<a name="updating-homestead"></a>
## 更新 Homestead

你可以用兩步驟更新 Homestead。首先，你應該先使用 `vagrant box update` 更新你的 Vagrant box:

    vagrant box update

接著，你需要更新 Homestead 原始碼。如果你已經複製了儲存庫，你可以在儲存庫的位置透過 `git pull origin master` 更新。

如果你已經透過專案的 `composer.json` 安裝 Homestead，你應該確保 `composer.json` 檔案包含 `"laravel/homestead": "^7"` 然後更新你的相依套件：

    composer update

<a name="provider-specific-settings"></a>
## 特定虛擬機設定

<a name="provider-specific-virtualbox"></a>
### VirtualBox

#### `natdnshostresolver`

預設的 Homestead 會將 `natdnshostresolver` 設定為 `on`。這可以讓 Homestead 去使用本機作業系統的 DNS 設定。如果你想要覆寫這個行為，請新增下面幾行到你的 `Homestead.yaml` 檔案：

    provider: virtualbox
    natdnshostresolver: off

#### Windows 的捷徑

如果無法再 Windows 機器上運作該捷徑，你可能需要新增以下內容到 `Vagrantfile` 檔案中：

    config.vm.provider "virtualbox" do |v|
        v.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
    end
