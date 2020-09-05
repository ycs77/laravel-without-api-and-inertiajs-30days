# Day 03 初始化 Lightning - 安裝 Laravel & Vue.js & Inertia.js

準備好 Laravel 的開發環境後，就可以開始初始化專案了。

## 安裝 Laravel

說到 Laravel 最近的大事件，就是 Laravel 8 要準備發布了！但在本系列要使用的套件有部分還不支援 Laravel 8，因此我們還是用 Laravel 7 來建構 Lightning：

```bash
composer create-project --prefer-dist "laravel/laravel:7.*" lightning
```

因為最後要把程式碼部署到 Heroku，不要讓編譯好的前端資源進 Git 版控，需要將其排除掉：

*.gitignore*

```
/node_modules
/public/css
/public/js
...
mix-manifest.json
```

設定完後，把 Git 倉庫初始化：

```bash
git init
git add --all
git commit -m "Initial"
```

接者設定 `.env` 檔：

> 注意一下這裡有把 `FILESYSTEM_DRIVER` 設成 `public`

```
APP_NAME=Lightning
...
APP_URL=http://lightning.test
...
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=lightning
DB_USERNAME=[dm-name]
DB_PASSWORD=[dm-password]
...
FILESYSTEM_DRIVER=public
```

然後新增本地 Storage 的公開連結：

```bash
php artisan storage:link
```

因為我個人習慣是在 PHP 用 4 個空格縮進，CSS、JS、Vue 等前端檔案都用 2 個空格縮進，所以需要修改一下 `.editorconfig` 設定：

```ini
...
[*.{css,js,vue,yml,yaml}]
indent_size = 2
```

再來就是本地化的設定，在這一整個系列裡我會用繁體中文，所以要做一些相關設定。如果你不想要改也可以跳過此步驟：

*config/app.php*

```php
    'timezone' => 'Asia/Taipei',
    // ...
    'locale' => 'zh_TW',
    // ...
    'faker_locale' => 'zh_TW',
```

`faker_locale` 是設定假文產生器的語言，但中文部分不完整，有些產生出來的還是英文，就將就著用吧！

雖然設定了中文語系，但現在網站還是英文的，因為少了翻譯檔案，這裡我下載 [Laravel-lang](https://github.com/Laravel-Lang/lang) 翻譯好的檔案。把 Laravel-lang 中的 `src/zh_TW` 資料夾和 `json/zh_TW.json` 檔複製到 Lightning 專案的 `resources/lang` 下。

最後我們要來安裝 Laravel 端的 Inertia 套件：

```bash
composer require inertiajs/inertia-laravel
```

然後新增 SPA 的入口 `resources/views/app.blade.php`。`@inertia` 是 Blade 指令，會編譯成 `<div id="app" data-page="{...}"></div>` 來啟動 SPA：

*resources/views/app.blade.php*

```html
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{{ config('app.name') }}</title>
    <link href="{{ mix('/css/app.css') }}" rel="stylesheet" />
    <script src="{{ mix('/js/app.js') }}" defer></script>
</head>

<body>
    @inertia
</body>
</html>
```

> 這裡的 `resources/views/app.blade.php` 是 Inertia 在 Laravel 預設的進入點，如果要自訂可以使用 `Inertia::setRootView()` 設定新的進入點。

至此，後端就告一段落了。然後來準備前端的部分。

## 安裝 Vue.js & Inertia.js

除了要裝 Vue.js & Inertia.js 之外，還有 Vue Meta 是要來處理前端頁面的 Title、Meta 等。安裝前端套件我個人習慣使用 Yarn，如果你用 NPM 只需替換掉指令就可以了：

```bash
yarn add vue@^2.6 vue-meta@^2.4 vue-template-compiler@^2.6 \
  @inertiajs/inertia@^0.2 @inertiajs/inertia-vue@^0.2
```

> 雖然 Vue.js 3 已經到 Beta 版了，但它和 2 版的差異實在不小，且還有套件相容性等問題，目前不考慮使用。在本系列還是使用 Vue.js 2。

然後是初始化前端應用，打開 `resources/js/app.js` 改成以下：

*resources/js/app.js*

```js
import Vue from 'vue'
import VueMeta from 'vue-meta'
import { InertiaApp } from '@inertiajs/inertia-vue'

Vue.use(InertiaApp)
Vue.use(VueMeta)

const app = document.getElementById('app')
const appName = document.title

new Vue({
  metaInfo: {
    titleTemplate: title => title ? `${title} - ${appName}` : appName
  },
  render: h => h(InertiaApp, {
    props: {
      initialPage: JSON.parse(app.dataset.page),
      resolveComponent: name => import(`@/Pages/${name}`).then(module => module.default)
    }
  })
}).$mount(app)
```

前端接到 **Inertia Page 物件** (`app.dataset.page`) 後，把它丟給 `InertiaApp` 的 `initialPage` 作為 SPA 啟動初始頁面的資料。另一個 `resolveComponent` 是設定要如何解析頁面組件，為了自動切分、按需加載每個頁面，這裡有使用到 [Dynamic import](https://github.com/tc39/proposal-dynamic-import)，要安裝 `@babel/plugin-syntax-dynamic-import` 才能使用。不過 Laravel Mix 已經幫我們裝好了，可以直接使用。

> 需要注意 Laravel Mix 4 在用 Dynamic import 會有個問題，就是不能在 Vue 單文件組件使用 style，如果你還是要用，只能降回 Laravel Mix 3。不過在此系列全程使用 Tailwind CSS，只會在 HTML 使用 Class 來調樣式，不會受到影響。

然後還需要增加一點 Webpack 的設定：

*webpack.mix.js*

```js
const mix = require('laravel-mix')

mix
  .js('resources/js/app.js', 'public/js')
  .sass('resources/sass/app.scss', 'public/css')
  .webpackConfig({
    output: {
      chunkFilename: 'js/[name].js?id=[chunkhash]'
    },
    resolve: {
      alias: {
        vue$: 'vue/dist/vue.runtime.esm.js',
        '@': path.resolve('resources/js')
      }
    }
  })
  .sourceMaps()
  .version()
```

這裡有設定了別名 `@` 對應到 `resources/js`，所以上面的 `@/Pages/` 的路徑實際對應到 `resources/js/Pages/`。另一個別名 `vue$` 是設定 Webpack 在打包時 Vue 時，使用運行時版本 (不包含編譯器的版本)，參考：[运行时 + 编译器 vs. 只包含运行时](https://cn.vuejs.org/v2/guide/installation.html#%E8%BF%90%E8%A1%8C%E6%97%B6-%E7%BC%96%E8%AF%91%E5%99%A8-vs-%E5%8F%AA%E5%8C%85%E5%90%AB%E8%BF%90%E8%A1%8C%E6%97%B6)。

> 如果你眼尖會發現，剛才有裝 `vue-template-compiler` 這個套件，因為 Laravel Mix 需要，就這樣。(雖然不會用到...)

先不用急著執行編譯 (`yarn dev`)，這是下篇的事情。

## 總結

本篇把建構 Lightning 的第一步完成了，安裝前端和後端套件們和基本的設定。其實這一連串動作是可以包裝成 [Preset](https://usepreset.dev/) 一行指令搞定，但為了可以了解 Inertia 我們還是一步一步自己操作。那什麼時候要 Hello world 呢？這個部分就留到下一篇吧！

> Lightning 範例程式碼：https://github.com/ycs77/lightning

## 參考資料

* [Server-side setup - Inertia.js](https://inertiajs.com/server-side-setup)
* [Client-side setup - Inertia.js](https://inertiajs.com/client-side-setup)
* [运行时 + 编译器 vs. 只包含运行时 - 安装 — Vue.js](https://cn.vuejs.org/v2/guide/installation.html#%E8%BF%90%E8%A1%8C%E6%97%B6-%E7%BC%96%E8%AF%91%E5%99%A8-vs-%E5%8F%AA%E5%8C%85%E5%90%AB%E8%BF%90%E8%A1%8C%E6%97%B6)
