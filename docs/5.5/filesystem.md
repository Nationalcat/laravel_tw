---
layout: post
title: filesystem
tag: 5.5
---
# 檔案儲存系統

- [介紹](#introduction)
- [設定](#configuration)
    - [公用硬碟](#the-public-disk)
    - [本機驅動](#the-local-driver)
    - [驅動需求](#driver-prerequisites)
- [取得硬碟實例](#obtaining-disk-instances)
- [接收檔案](#retrieving-files)
    - [檔案 URL](#file-urls)
    - [檔案資料](#file-metadata)
- [儲存檔案](#storing-files)
    - [檔案上傳](#file-uploads)
    - [檔案可見性](#file-visibility)
- [刪除檔案](#deleting-files)
- [目錄](#directories)
- [自訂檔案系統](#custom-filesystems)

<a name="introduction"></a>
## 介紹

Laravel 提供了一個抽象的檔案系統，且這一切都要歸功於 Frank de Jonge 製作的 [Flysystem](https://github.com/thephpleague/flysystem) PHP 套件。Laravel Flysystem 整合了本機檔案系統、Amazon S3 和 Rackspace 雲端儲存。更棒的是，能像使用 API 那樣輕易的切換這些儲存方式來面對各種系統。

<a name="configuration"></a>
## 設定

檔案系統的設定檔就放在 `config/filesystems.php`。在這個檔案中，你可以設定所有的「硬碟」。每個硬碟都代表著獨特的儲存驅動與儲存位置。已經將每個有支援的驅動設定範例都寫入這個設定檔中。所以你只需要修改其中的設定來選擇你的儲存偏好與憑證。

當然，你可以根據實際需求而設定多組硬碟，甚至使用相同的驅動。

<a name="the-public-disk"></a>
### 公用硬碟

`public` 硬碟用於存放會被公開存取的檔案。預設的 `public` 硬碟是使用 `local` 驅動並將檔案儲存到 `storage/app/public`。為了要讓它們能夠從網頁上存取，你應該建立一個從 `public/storage` 到 `storage/app/public` 的符號連結。這個方式會讓你的公用可存取的檔案維持在一個目錄中，以便日後在使用像是 [Envoyer](https://envoyer.io) 這種零停機部署系統時，可以輕易的共享整個部署。

你可以使用 `storage:link` 指令來建立連結符號：

    php artisan storage:link

當然，一旦檔案被儲存並也建立了連結符號，就可以使用 `asset` 輔助函式來建立一個該檔案的 URL：

    echo asset('storage/file.txt');

<a name="the-local-driver"></a>
### 本機驅動

所有的操作是相對於設定檔中的「根目錄」設定進行。該目錄預設是 `storage/app`。因此下列方法將把檔案儲存在 s`torage/app/file.txt`：

    Storage::disk('local')->put('file.txt', 'Contents');

<a name="driver-prerequisites"></a>
### 驅動需求

#### Composer 套件

在使用 S3 或 Rackspace 驅動之前，你會需要透過 Composer 來安裝必要的套件：

- Amazon S3: `league/flysystem-aws-s3-v3 ~1.0`
- Rackspace: `league/flysystem-rackspace ~1.0`

#### S3 驅動設定

S3 驅動設定資訊就放置於 `config/filesystems.php` 設定檔中。這個檔案有一個 S3 的驅動設定陣列的範例。你可以隨意的修改這個陣列來設定自己的 S3 設定與憑證。為了方便設定，這些環境變數會採用 AWS CLI 使用的命名慣例。

#### FTP 驅動設定

Laravel 的 Flysystem 也整合了 FTP 驅動。然而，框架預設的 `filesystems.php` 設定檔並不含 FTP 的簡單範例。如果你需要設定一個 FTP 檔案系統，你可以使用以下的範例設定：

    'ftp' => [
        'driver'   => 'ftp',
        'host'     => 'ftp.example.com',
        'username' => 'your-username',
        'password' => 'your-password',

        // 可選的 FTP 設定...
        // 'port'     => 21,
        // 'root'     => '',
        // 'passive'  => true,
        // 'ssl'      => true,
        // 'timeout'  => 30,
    ],

#### Rackspace 驅動設定

Laravel 的 Flysystem 也整合了 Rackspace 驅動。框架預設的 `filesystems.php` 設定檔並不含 Rackspace 的簡單範例。如果你需要設定一個 Rackspace 檔案系統，你可以使用以下的範例設定：

    'rackspace' => [
        'driver'    => 'rackspace',
        'username'  => 'your-username',
        'key'       => 'your-key',
        'container' => 'your-container',
        'endpoint'  => 'https://identity.api.rackspacecloud.com/v2.0/',
        'region'    => 'IAD',
        'url_type'  => 'publicURL',
    ],

<a name="obtaining-disk-instances"></a>
## 取得硬碟實例

`Storage` facade 可被用於與任何設定的硬碟進行資料上的交換。例如，你可以在 Facade 上使用 `put` 方法將頭像儲存到預設的硬碟上。如果你不先使用 `dist` 方法而是直接呼叫 `Storage` Facade，那麼該方法會自動傳入預設的硬碟。

    use Illuminate\Support\Facades\Storage;

    Storage::put('avatars/1', $fileContents);

如果你的應用程式要與多個硬碟交換資料，請使用在 `Storage` Facade 上的  `disk` 方法來處理在特定硬碟上的檔案：

    Storage::disk('s3')->put('avatars/1', $fileContents);

<a name="retrieving-files"></a>
## 接收檔案

可以使用 `get` 方法來取得一個檔案的內容。檔案的原始字串內容會被這個方法所回傳。請記得，所有檔案路徑應該使用硬碟「根目錄」的相對位置：

    $contents = Storage::get('file.jpg');

`exists` 方法可被用於檢查給定的檔案是否存在於硬碟上：

    $exists = Storage::disk('s3')->exists('file.jpg');

<a name="file-urls"></a>
### 檔案 URL

你可以使用 `url` 方法來取得給定檔案的 URL。如果你正在使用 `local` 驅動，這通常只會在 `/storage` 後面加上給定的路徑，並回傳檔案的 URL。如果你正在使用 `s3` 或 `rackspace` 驅動，則會回傳完整的 URL：

    use Illuminate\Support\Facades\Storage;

    $url = Storage::url('file1.jpg');

> {note} 請記得，如果你正使用 `local` 驅動，所有可被公開存取的檔案應該都放在 `storage/app/public` 目錄中。此外，你也應該在 `public/storage` [建立一個連結符號](#the-public-disk)來指向 `storage/app/pbulic` 目錄。

#### 臨時的 URL

如果是使用 `s3` 或 `rackspace` 驅動來儲存檔案的話，你可以使用 `temporaryUrl` 方法來為給定的檔案建立一個臨時的 URL。這個方法接受一個路徑和一個用來指定 URL 有效期限的 `DateTime` 實例：

    $url = Storage::temporaryUrl(
        'file1.jpg', now()->addMinutes(5)
    );

#### 自訂本機 URL 主機

如果你想要使用 `local` 驅動來預先定義儲存在硬碟上的檔案的主機，你可以新增一個 `url` 選項到硬碟的設定陣列中：

    'public' => [
        'driver' => 'local',
        'root' => storage_path('app/public'),
        'url' => env('APP_URL').'/storage',
        'visibility' => 'public',
    ],

<a name="file-metadata"></a>
### 檔案資料

除了讀寫檔案，Laravel 也提供了關於檔案本身的資訊。例如，`size` 方法可被用來取得檔案的位元大小：

    use Illuminate\Support\Facades\Storage;

    $size = Storage::size('file1.jpg');

`lastModified` 方法會回傳最後一次修改檔案的 UNIX 時間戳記：

    $time = Storage::lastModified('file1.jpg');

<a name="storing-files"></a>
## 儲存檔案

`put` 方法可被用於儲存硬碟上的原始檔案內容。你也可以將 PHP `resource` 傳入 `put` 方法，這會使用到 Flysystem 底層 stream 支援。強烈建議使用 stream 處理大型檔案。

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents);

    Storage::put('file.jpg', $resource);

#### 自動傳輸

如果你想要 Laravel 來自動管理傳輸一個給定檔案到儲存的位置，你可以使用 `putFile` 或 `putFileAs` 方法。這個方法接受 `Illuminate\Http\File` 或 `Illuminate\Http\UploadedFile` 其中一個實例，並自動將檔案傳輸到你預期的位置：

    use Illuminate\Http\File;
    use Illuminate\Support\Facades\Storage;

    // 自動檔案名產生一個唯一的 ID...
    Storage::putFile('photos', new File('/path/to/photo'));

    // 手動指定檔案名稱...
    Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');

關於 `putFile` 方法有一些重要的事情需要注意。請注意，我們只有指定目錄名稱，而不是檔案名稱。預設的 `putFile` 方法會產生一個唯一的 ID 作為檔案名稱。該檔案的路徑會被 `putFile` 方法所回傳，以便將路徑和剛產生的檔案名稱存入資料庫中。

`putFile` 和 `putFileAs` 方法也接受一個用來指定儲存檔案的可見性的參數。如果你是將檔案儲存到像是 S3 的雲端硬碟，並希望檔案能夠被公開存取的話，這個用法會特別好用：

    Storage::putFile('photos', new File('/path/to/photo'), 'public');

#### 寫入檔案的開頭或結尾

`prepend` 和 `append` 方法可以讓你寫入資料到檔案的開頭或結尾：

    Storage::prepend('file.log', 'Prepended Text');

    Storage::append('file.log', 'Appended Text');

#### 複製與移動檔案

`copy` 方法可被用來複製一個現有的檔案到硬碟上的新位置，至於 `move` 方法可被用於重新命名或移動一個現有的檔案到新位置：

    Storage::copy('old/file1.jpg', 'new/file1.jpg');

    Storage::move('old/file1.jpg', 'new/file1.jpg');

<a name="file-uploads"></a>
### 檔案上傳

在網頁應用程式中，儲存檔案最常見的使用案例是儲存使用者上傳的檔案，像是個人照片、圖片和文件。Laravel 在上傳檔案的實例上使用 `store` 方法將儲存上傳的檔案這件事弄的很容易。只要在你想要儲存上傳的檔案的路徑呼叫 `store` 方法：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserAvatarController extends Controller
    {
        /**
         * 上傳使用者的頭像。
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            $path = $request->file('avatar')->store('avatars');

            return $path;
        }
    }

關於這個範例有一些重要的事情需要注意一下。請注意我們只有指定目錄名稱，而不是檔案名稱。預設的 `store` 方法會去產生一個唯一的 ID 作為檔案名稱。該檔案的路徑會被 `store` 方法所回傳，以便將路徑和剛產生的檔案名稱存入資料庫中。

你還能在 `Storeage` facade 上呼叫 `putFile` 方法來執行與上面範例相同的檔案處理：

    $path = Storage::putFile('avatars', $request->file('avatar'));

#### 指定檔案名稱

如果你不想要檔案名稱被自動分配到你儲存的檔案，請使用 `storeAs` 方法，它會去接收路徑、檔案名稱和（可選）硬碟作為參數：

    $path = $request->file('avatar')->storeAs(
        'avatars', $request->user()->id
    );

當然，你還可以在 `Storage` Facade 上使用 `putFileAs` 方法，它會去執行與上面範例相同的檔案處理：

    $path = Storage::putFileAs(
        'avatars', $request->file('avatar'), $request->user()->id
    );

#### 指定一個硬碟

預設這個方法會使用你預設的硬碟。如果你想要指定另外一個硬碟，請將硬碟名稱傳入 `store` 方法的第二個參數：

    $path = $request->file('avatar')->store(
        'avatars/'.$request->user()->id, 's3'
    );

<a name="file-visibility"></a>
### 檔案可見性

在 Laravel 的 Flysystem 整合中，「可見性」是跨多個平台的檔案權限的抽象概念。檔案可被宣告為 `public` 或 `private`。當一個檔案被宣告為 `public`，就表示該檔案應該要能夠被其他人存取。例如，在使用 S3 驅動的時候，你可以接收 `public` 檔案的 URL。

你能透過設定 `put` 方法時，順便設定可見性：

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents, 'public');

如果該檔案已經被儲存，還是能透過 `getVisibility` 和 `setVisibility` 方法來接收和設定檔案的可見性：

    $visibility = Storage::getVisibility('file.jpg');

    Storage::setVisibility('file.jpg', 'public')

<a name="deleting-files"></a>
## 刪除檔案

`delete` 方法接受一個檔案名稱或檔案名稱陣列，用來刪除硬碟上的檔案：

    use Illuminate\Support\Facades\Storage;

    Storage::delete('file.jpg');

    Storage::delete(['file1.jpg', 'file2.jpg']);

如果有需要，你可以指定將要被刪除的檔案所使用的硬碟：

    use Illuminate\Support\Facades\Storage;

    Storage::disk('s3')->delete('folder_path/file_name.jpg');

<a name="directories"></a>
## 目錄

#### 取得一個目錄中所有的檔案

`files` 方法回傳給定目錄下的檔案陣列。如果你希望回傳包含給定目錄下所有子目錄的檔案，你可以使用 `allFiles` 方法。

    use Illuminate\Support\Facades\Storage;

    $files = Storage::files($directory);

    $files = Storage::allFiles($directory);

#### 取得一個目錄中所有的子目錄

`directories` 方法回傳給定目錄下的目錄陣列。另外，你也可以使用 `allDirectories` 方法取得給定目錄下子目錄以及子目錄所包含的目錄。

    $directories = Storage::directories($directory);

    // 遞迴...
    $directories = Storage::allDirectories($directory);

#### 建立一個目錄

`makeDirectory` 方法可以建立給定的目錄，以及任何所需的子目錄：

    Storage::makeDirectory($directory);

#### 刪除一個目錄

最後，`deleteDirectory` 方法可被用於刪除一個目錄以及目錄中的所有檔案：

    Storage::deleteDirectory($directory);

<a name="custom-filesystems"></a>
## 自訂檔案系統

Laravel 整合的 Flysystem 提供了幾個可馬上使用的「驅動」。然而，Flysystem 不受限於這些，還具有適用於其他儲存系統的連接器。你能在 Laravel 應用程式中使用其中一個額外的連接器來建立一個自訂的驅動。

為了設定自訂的檔案系統，你會需要一個 Flysystem 連接器。讓我們新增一個由社群維護的 Dropbox 連接器到我們的專案中：

    composer require spatie/flysystem-dropbox

接著，你應該建立一個像是 `DropboxServiceProvider` 的[服務容器](/laravel_tw/docs/5.5/providers)。在該提供者的 `boot` 方法中，你可以使用 `Storage` facade 的 `extend` 方法來定義該自訂的驅動：

    <?php

    namespace App\Providers;

    use Storage;
    use League\Flysystem\Filesystem;
    use Illuminate\Support\ServiceProvider;
    use Spatie\Dropbox\Client as DropboxClient;
    use Spatie\FlysystemDropbox\DropboxAdapter;

    class DropboxServiceProvider extends ServiceProvider
    {
        /**
         * 執行註冊服務後啟動。
         *
         * @return void
         */
        public function boot()
        {
            Storage::extend('dropbox', function ($app, $config) {
                $client = new DropboxClient(
                    $config['authorizationToken']
                );

                return new Filesystem(new DropboxAdapter($client));
            });
        }

        /**
         * 在容器中註冊綁定。
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

`extend` 方法的第一個參數是驅動的名稱，第二個參數是用來接收 `$app` 和 `$config` 變數的必包。解析器閉包會回傳 `League\Flysystem\Filesystem` 實例。`$config` 變數包含了定義在 `config/filesystems.php` 對指定硬碟的設定。

一旦你建立了該服務容器到註冊擴充，你就能在 `config/filesystems.php` 設定檔中使用 `dropbox` 驅動。
