---
layout: post
title: homestead
---
# Laravel Homestead

- [導覽](#introduction)
- [內建軟體](#included-software)
- [安裝與設定](#installation-and-setup)
- [常見用法](#daily-usage)
- [連接埠](#ports)
- [Blackfire 分析器](#blackfire-profiler)

<a name="introduction"></a>
## 導覽

Laravel 致力於讓 PHP 開發體驗更愉快，也包含你的本地開發環境。[Vagrant](http://vagrantup.com) 提供了一個簡單、優雅的方式來管理與供應虛擬機器。

Laravel Homestead 是一個官方預載的 Vagrant「封裝包」，提供你一個美好的開發環境，你不需要在你的本機端安裝 PHP、HHVM、網頁伺服器或任何伺服器軟體。不用擔心搞亂你的系統！Vagrant 封裝包可以搞定一切。如果有什麼地方爛掉了，你可以在幾分鐘內快速的砍掉並重建虛擬機器。

Homestead 可以在任何 Windows、Mac 或 Linux 上面運行，裏面包含了 Nginx 網頁伺服器、PHP 5.6、MySQL、Postgres、Redis、Memcached 還有所有你要開發精彩的 Laravel 應用程式所需的軟體。

> **附註：** 如果您是 Windows 的使用者，您可能需要啟用硬體虛擬化（VT-x）。通常需要透過 BIOS 來啟用它。

Homestead 目前是建置且測試於 Vagrant 1.7。

<a name="included-software"></a>
## 內建軟體

- Ubuntu 14.04
- PHP 5.6
- HHVM
- Nginx
- MySQL
- Postgres
- Node (With Bower, Grunt, and Gulp)
- Redis
- Memcached
- Beanstalkd
- [Laravel Envoy](/laravel_tw/docs/5.0/envoy)
- [Blackfire Profiler](#blackfire-profiler)

<a name="installation-and-setup"></a>
## 安裝與設定

### 安裝 VirtualBox / VMware 與 Vagrant

在啟動你的 Homestead 環境之前，你必須先安裝 [VirtualBox](https://www.virtualbox.org/wiki/Downloads) 和 [Vagrant](http://www.vagrantup.com/downloads.html). 兩套軟體在各平台都有提供易用的視覺化安裝程式。

#### VMware

除了 VirtualBox 外， Homestead 也支援 VMware。如果要在 Homestead 使用 VMwate，你必須購買 VMware Fusion / Desktop 及 [VMware Vagrant 外掛](http://www.vagrantup.com/vmware)。VMware 在虛擬機器中的共享資料夾提供更快的效能。

### 增加 Vagrant 封裝包

當 VirtualBox / VMware 和 Vagrant 安裝完成後，你可以在終端機以下列命令將 `laravel/homestead` 封裝包安裝進你的 Vagrant 安裝程式中。下載封裝包會花你一點時間，時間長短將依據你的網路速度決定:

	vagrant box add laravel/homestead

如果上述指令失敗，你的 Vagrant 是舊版需要完整的鏈結：

	vagrant box add laravel/homestead https://atlas.hashicorp.com/laravel/boxes/homestead

### 安裝 Homestead

你可以透過手動複製資源庫的方式來安裝 Homestead。建議可將資源庫複製至你的 "home" 目錄中的 `Homestead` 資料夾，如此一來 Homestead 封裝包將能提供主機服務給你所有的 Laravel（及 PHP）專案:

	git clone https://github.com/laravel/homestead.git Homestead

一旦你安裝完 Homestead，即可在 Homestead 目錄執行 `bash init.sh` 指令來創建 `Homestead.yaml` 設定檔:

	bash init.sh

此 `Homestead.yaml` 檔，將會被放置在你的 `~/.homestead` 目錄中。

### 設定你的提供者

你能夠在 `Homestead.yaml` 檔案中設定 `provider`，指定 Vagrant 要使用哪個提供者：`virtualbox`、`vmware_fusion` (Mac OS X) 或 `vmware_workstation` (Windows)。你可以指定你偏好的提供者。

	provider: virtualbox

### 設定你的 SSH 金鑰

再來你要編輯 `Homestead.yaml`。可以在檔案中設定你的 SSH 公開金鑰路徑，以及主要機器與 Homestead 虛擬機器之間的共享目錄。

你沒有 SSH 金鑰？在 Mac 和 Linux 下，你可以利用下面的指令來創建一個 SSH 金鑰組:

	ssh-keygen -t rsa -C "you@homestead"

在 Windows 下，你需要安裝 [Git](http://git-scm.com/) 並且使用包含在 Git 裏的 `Git Bash` 來執行上述的指令。另外你也可以使用 [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) 和 [PuTTYgen](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)。

一旦你創建了一個 SSH 金鑰，記得在你的 `Homestead.yaml` 檔案中的 `authorize` 屬性指明金鑰路徑。

### 設定你的共享資料夾

`Homestead.yaml` 檔案中的 `folders` 屬性列出所有你想跟你的 Homestead 環境共享的資料夾列表。這些資料夾中的檔案若有更動，他們將會在你的本機與 Homestead 環境裡保持同步。你可以將你需要的共享資料夾都設定進去。

如果要開啟 [NFS](http://docs.vagrantup.com/v2/synced-folders/nfs.html)，只需要共享資料夾中增加簡單的一行：

	folders:
	    - map: ~/Code
	      to: /home/vagrant/Code
	      type: "nfs"

### 設定你的 Nginx 站台

對 Nginx 不熟悉？沒關係。`sites` 屬性允許你簡單的對應一個 `網域` 到一個 homestead 環境中的目錄。在 `Homestead.yaml` 檔案中有一個範例的站台設定。同樣的，你可以加任何你需要的站台到你的 Homestead 環境中。Homestead 可以為你每個進行中的 Laravel 專案提供方便的虛擬化環境。

你可以透過設定 `hhvm` 屬性為 `true` 來讓虛擬站台支援 [HHVM](http://hhvm.com):

	sites:
	    - map: homestead.app
	      to: /home/vagrant/Code/Laravel/public
	      hhvm: true

Each site will be accessible by HTTP via port 8000 and HTTPS via port 44300.

### Bash Aliases

如果要增加 Bash aliases 到你的 Homestead 封裝包中，只要將內容加到 `~/.homestead` 根目錄的 `aliases` 檔案中即可。

### 啟動 Vagrant 封裝包

當你根據你的喜好編輯完 `Homestead.yaml` 後，在終端機從你的 Homestead 資料夾下執行 `vagrant up` 指令。

Vagrant 會將虛擬機器開機，並且自動設定你的共享目錄和 Nginx 站台。如果要移除虛擬機器，可以使用 `vagrant destroy --force` 指令。

為了你的 Nginx 站台，別忘記在你的機器的 `hosts` 檔將「網域」加進去。`hosts` 檔會將你的本地網域的站台請求重導至你的 Homestead 環境中。在 Mac 和 Linux，該檔案放在 `/etc/hosts`。在 Windows 環境中，它被放置在 `C:\Windows\System32\drivers\etc\hosts`。你要加進去的內容類似如下：

	192.168.10.10  homestead.app

務必確認 IP 位置與你的 `Homestead.yaml` 檔案中的相同。一旦你將網域加進你的 `hosts` 檔案中，你就可以透過網頁瀏覽器存取到你的站台。

	http://homestead.app

繼續讀下去，你會學到如何連結到資料庫！

<a name="daily-usage"></a>
## 日常使用

### 透過 SSH 連接

因為你可能會經常需要透過 SSH 進入你的 Homestead 虛擬機器，可以考慮在你的主要機器上創建一個「別名」來快速 SSH 進入你的 Homestead:

	alias vm="ssh vagrant@127.0.0.1 -p 2222"

一旦你創建了這個別名，無論你在主要機器的哪個目錄，都可以簡單地使用 "vm" 指令來透過 SSH 進入你的 Homestead 虛擬機器。

或者，你可以在 Homestead 目錄下執行 `vagrant ssh` 指令。

### 連結資料庫

在 `Homestead` 封裝包中，MySQL 與 Postgres 兩套資料庫都已預裝其中。為了更簡便，Laravel 的 `local` 資料庫設定已經預設將其設定完成。

如果想要從本機上透過 Navicat 或者是 Sequel Pro 連接 MySQL 或者 Postgres 資料庫，你可以連接 `127.0.0.1` 的埠 33060 (MySQL) 或 54320 (Postgres)。而帳號密碼分別是 `homestead` / `secret`。

> **附註：** 從本機端你應該只能使用這些非標準的連接埠來連接資料庫。因為當 Laravel 運行在虛擬機器時，在 Laravel 的資料庫設定檔中依然是設定使用預設的 3306 及 5432 連接埠。

### 增加更多的站台

一旦 Homestead 環境上架且運行後，你可能會需要為 Laravel 應用程式增加更多的 Nginx 站台。你可以在單一個 Homestead 環境中運行非常多 Laravel 安裝程式。有兩種方式可以達成：第一種，在 `Homestead.yaml` 檔案中增加站台然後在 Homestead 目錄執行 `vagrant provision`。

> **附註：** 這個是具有破壞性的步驟。 當執行 `provision` 指令，你現有的資料庫將會被摧毀及重建。

另外，也可以使用存放在 Homestead 環境中的 `serve` 指令檔。要使用 `serve` 指令檔，請先 SSH 進入 Homestead 環境中，並執行下列命令：

	serve domain.app /home/vagrant/Code/path/to/public/directory 80

> **附註：** 在執行 `serve` 指令過後，別忘記將新的站台加進本機的 `hosts` 檔案中。

<a name="ports"></a>
## 連接埠

以下的埠將會被重導至 Homestead 環境：

- **SSH:** 2222 &rarr; Forwards To 22
- **HTTP:** 8000 &rarr; Forwards To 80
- **HTTPS:** 44300 &rarr; Forwards To 443
- **MySQL:** 33060 &rarr; Forwards To 3306
- **Postgres:** 54320 &rarr; Forwards To 5432

### 加入額外的連接埠

如果你願意，可以重導額外的連接埠到 Vagrant 封裝包，以及指定他們的協定：

	ports:
	    - send: 93000
	      to: 9300
	    - send: 7777
	      to: 777
	      protocol: udp

<a name="blackfire-profiler"></a>
## Blackfire 分析器

SensioLabs 所發佈的 [Blackfire 分析器](https://blackfire.io) 會在你的程式執行時自動搜集相關的資訊，像是 RAM，CPU 耗時，及硬碟 I/O。在 Homestead 中對你的應用程式使用此分析器是相當輕而易舉的一件事。

所有必須的安裝包已經安裝於你的 Homestead 虛擬環境，你只需要在 `Homestead.yaml` 檔案中設定 Blackfire **Server** ID 及 token ：

	blackfire:
	    - id: your-server-id
	      token: your-server-token
	      client-id: your-client-id
	      client-token: your-client-token

一旦你設定完了 Blackfire 的憑證，請從你的 Homestead 目錄透過 `vagrant provision` 重置你的虛擬機器。當然，請確定你已經看過 [Blackfire 文件](https://blackfire.io/getting-started)，以瞭解如何在你的瀏覽器安裝 Blackfire 的擴充套件。
