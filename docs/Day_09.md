# Day 09 Lightning 用戶登入

> 小提醒：本篇長度為往常的2倍，請酌情閱讀。

調好了用戶 Model，現在可以來做登入/登出了。後端直接拿 Laravel UI 現成的登入邏輯，但前端就需要做比較多事情...，像用 Tailwind CSS 調表單樣式、增加組件等。不管，先上再說！

## Laravel UI 套件

畢竟一開始決定了用 Laravel 7...，還是乖乖先用 Laravel UI 吧：

```bash
composer require laravel/ui
```

原本 Laravel UI 附的視圖不需要，我們只要後端的 Controllers：

```bash
php artisan ui:controllers
```

雖然提供了滿多功能的，但我這次只打算做登入和註冊，留下 `LoginController` 和 `RegisterController`，其他扔掉。既然要用我們的視圖，那就要註冊進去。覆蓋原本的 `showLoginForm()` 回傳我們的 Inertia 視圖：

*app/Http/Controllers/Auth/LoginController.php*
```php
use Inertia\Inertia;

public function showLoginForm()
{
    return Inertia::render('Auth/Login');
}
```

註冊登入的路由，把不用的忘記密碼功能先關掉 (如果你要做也是可以啦...)：

*routes/web.php*
```php
Auth::routes(['reset' => false]);
```

然後改一下登入成功跳轉的頁面：

*app/Providers/RouteServiceProvider.php*
```php
public const HOME = '/';
```

暫時先把狀態先調成未登入的狀態，有登入跟註冊兩個按鈕：

*resources/js/Layouts/AppLayout.vue*
```vue
<template>
  ...
            <template v-if="true">
              ...
            </template>
  ...
</template>
```

## 登入頁面

用戶登入在前端的部分要處裡的比較多，登入頁面表單、文字輸入框組件、表單樣式... 等，一個一個來。先是登入頁面 `Auth/Login`：

*resources/js/Pages/Auth/Login.vue*
```vue
<template>
  <div class="py-6 md:py-8">
    <form @submit.prevent="submit" class="bg-white rounded-lg shadow max-w-sm p-6 md:p-8 mx-auto">
      <h1 class="text-3xl text-center">登入</h1>
      <div class="w-12 mt-1 mx-auto border-b-4 border-purple-400"></div>

      <div class="grid gap-6 mt-6">
        <text-input v-model="form.email" label="E-mail" autocomplete="email" ref="emailInput" />
        <text-input v-model="form.password" type="password" label="密碼" />
        <div>
          <label>
            <input type="checkbox" class="form-checkbox" v-model="form.remember"> 記住我
          </label>
        </div>
        <div>
          <loading-button :loading="loading" class="inline-flex items-center px-5 py-2 rounded-md transition-colors duration-150 bg-purple-500 text-white hover:bg-purple-700 disabled:bg-purple-300">登入</loading-button>
        </div>
      </div>
    </form>
  </div>
</template>

<script>
import AppLayout from '@/Layouts/AppLayout'
import TextInput from '@/Components/TextInput'
import LoadingButton from '@/Components/LoadingButton'

export default {
  layout: AppLayout,
  metaInfo: {
    title: '登入'
  },
  components: {
    TextInput,
    LoadingButton
  },
  data() {
    return {
      form: {
        email: '',
        password: '',
        remember: true
      },
      loading: false
    }
  },
  methods: {
    submit() {
      //
    }
  },
  mounted() {
    this.$refs.emailInput.focus()
  }
}
</script>
```

上面 `<loading-button>` 的 class 裡有 `disabled:bg-purple-300`，但預設是關閉，要用它需要開 `disabled` 的 variants：

*tailwind.config.js*
```js
module.exports = {
  variants: {
    ...
    backgroundColor: ['responsive', 'hover', 'focus', 'disabled'],
  }
}
```

`Login` 裡面已經先引入了 `TextInput` 和 `LoadingButton` 兩個組件，卻都不存在，所以會報錯。這兩個組件都是我從 Inertia.js 官方 Demo [PingCRM](https://github.com/inertiajs/pingcrm) 拿的，稍微調整後變成下方這樣。文字輸入框：

*resources/js/Components/TextInput.vue*
```vue
<template>
  <div>
    <label v-if="label" class="form-label" :for="id">{{ label }}：</label>
    <input :id="id" ref="input" v-bind="$attrs" class="form-input" :type="type" :value="value" @input="$emit('input', $event.target.value)">
  </div>
</template>

<script>
export default {
  inheritAttrs: false,
  props: {
    id: {
      type: String,
      default() {
        return `text-input-${this._uid}`
      }
    },
    type: {
      type: String,
      default: 'text'
    },
    value: String,
    label: String
  },
  methods: {
    focus() {
      this.$refs.input.focus()
    },
    select() {
      this.$refs.input.select()
    },
    setSelectionRange(start, end) {
      this.$refs.input.setSelectionRange(start, end)
    }
  }
}
</script>
```

附有載入圖示的按鈕：

*resources/js/Components/LoadingButton.vue*
```vue
<template>
  <button :disabled="loading" class="inline-flex items-center">
    <svg v-if="loading" class="animate-spin -ml-1 mr-3 h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
      <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
      <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
    </svg>
    <slot />
  </button>
</template>

<script>
export default {
  props: {
    loading: Boolean
  }
}
</script>
```

![](../images/day09-01.jpg)

嗯...再調一下 CSS。先改 Tailwind CSS Custom Forms 的設定，把輸入框變寬、顏色改成紫色、把半透明陰影改掉：

*tailwind.config.js*
```js
module.exports = {
  theme: {
    ...
    customForms: theme => ({
      default: {
        'input, textarea, select': {
          width: theme('width.full'),
          borderColor: theme('colors.gray.300'),
          '&:focus': {
            borderColor: theme('colors.purple.500'),
            boxShadow: `0 0 0 1px ${theme('colors.purple.500')}`,
          },
        },
        'checkbox, radio': {
          color: theme('colors.purple.500'),
          borderColor: theme('colors.gray.300'),
          '&:focus': {
            borderColor: theme('colors.purple.500'),
            boxShadow: `0 0 0 1px ${theme('colors.purple.500')}`,
          },
        },
      },
    }),
    ...
  },
}
```

還有輸入框的 Label，要在 CSS 檔裡修改。表單的部分要獨立出來，新增一個 `form.css`：

*resources/css/form.css*
```css
.form-label {
  @apply mb-2 block text-gray-600 select-none;
}
```

記得註冊對位置：

*resources/css/app.css*
```css
...

@import 'tailwindcss/components';
@import 'form';
...
```

看看改得如何：

![](../images/day09-02.jpg)

剛才的按鈕 class 滿長的，按鈕也是個很常用的元素，把它抽出來變成 class：

*resources/js/Pages/Auth/Login.vue*
```html
<loading-button :loading="loading" class="btn btn-purple">登入</loading-button>
```

新增 `button.css`：

*resources/css/button.css*
```css
.btn {
  @apply inline-flex items-center px-5 py-2 rounded-md transition-colors duration-150;
}

.btn-purple {
  @apply bg-purple-500 text-white;
  &:hover {
    @apply bg-purple-700;
  }
  &:disabled {
    @apply bg-purple-300 !important;
  }
}
```

*resources/css/app.css*
```css
...

@import 'tailwindcss/components';
@import 'button';
...
```

`.card` 也抽出來：

*resources/js/Pages/Auth/Login.vue*
```html
<form @submit.prevent="submit" class="card max-w-sm p-6 md:p-8 mx-auto">
  ...
</form>
```

*resources/css/components.css*
```css
/* Card */
.card {
  @apply bg-white rounded-lg shadow;
}
```

CSS 搞定後要傳帳號資料給後端驗證和登入。Inertia.js 傳資料很簡單，用法跟 Axios 差不多 (可以用 `put`、`patch`、`delete` 等方法)：

*resources/js/Pages/Auth/Login.vue*
```vue
<script>
export default {
  ...
  methods: {
    submit() {
      this.loading = true
      this.$inertia.post('/login', this.form).then(() => this.loading = false)
    }
  },
  ...
}
</script>
```

基本這樣就可以登入了，把前面新增的帳密輸入進去並送出。

![](../images/day09-03.jpg)

![](../images/day09-04.jpg)

## 顯示登入用戶資訊

當然登入後看起來沒啥變化，因為前端還不知道目前登入用戶的資料，回到 `AppServiceProvider` 補上：

*app/Providers/AppServiceProvider.php*
```php
use Inertia\Inertia;

public function registerInertia()
{
    Inertia::share([
        ...
        'auth' => fn() => [
            'user' => Auth::user() ? [
                'id' => Auth::user()->id,
                'name' => Auth::user()->name,
                'email' => Auth::user()->email,
                'description' => Auth::user()->description,
                'avatar' => Auth::user()->avatar,
            ] : null,
        ],
    ]);
}
```

然後前端就可以用 `this.$page.auth.user` 取得用戶資料。

> `auth` 裡面有包一層 Function，這個是 [延遲載入 (Lazy evaluation)](https://inertiajs.com/responses#lazy-evaluation) 功能，之後的篇章會解釋。

這裡說一下，用戶的頭像會有可能是完整的 URL，或只有後面的 Path，需要有個 Function 可以統一輸出完整的 URL。先新增 `app/Utils/helpers.php` 和 `storage_url()`：

*app/Utils/helpers.php*
```php
use Illuminate\Support\Facades\Storage;

function storage_url(string $path = null): ?string
{
    if (! $path) {
        return null;
    }

    return filter_var($path, FILTER_VALIDATE_URL)
        ? $path
        : Storage::url($path);
}
```

還要在 `composer.json` 裡註冊，讓 Composer 自動引入：

*composer.json*
```json
{
    "autoload": {
        ...
        "files": [
            "app/Utils/helpers.php"
        ]
    }
}
```

和跑 `composer dump-autoload`。

把用戶頭像套用 `storage_url()`：

*app/Providers/AppServiceProvider.php*
```php
public function registerInertia()
{
    Inertia::share([
        ...
        'auth' => fn() => [
            'user' => Auth::user() ? [
                ...
                'avatar' => storage_url(Auth::user()->avatar),
            ] : null,
        ],
    ]);
}
```

回到前端串資料，把 `user` 替換成後端傳的資料，打掉 `data`，用 `computed` 確保可以取得最新的用戶資料。這裡看不懂的應該有 `this.$page.auth?.user` 裡的 `?.` 這個用法，這是 JavaScript 的一個新特性 [Optional chaining](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining)，等同 `this.$page.auth && this.$page.auth.user`：

*resources/js/Layouts/AppLayout.vue*
```vue
<template>
  ...
            <template v-if="!user">
              ...
            </template>
  ...
</template>

<script>
export default {
  ...
  computed: {
    user() {
      return this.$page.auth?.user
    }
  }
}
</script>
```

現在前端就可以顯示當前用戶的資料了：

![](../images/day09-05.jpg)

## 登出

登出也很簡單，後端登出的路由是 post 到 `/logout`，把 `<inertia-link>` 的 method 改成 `post` 就可以了，但要先增加 `DropdownItem` 的 Props：

*resources/js/Components/DropdownItem.vue*
```vue
<template>
  <inertia-link :href="href" :method="method" class="flex items-center px-4 py-2 text-gray-700 hover:bg-gray-100 focus:bg-gray-100" v-on="$listeners">
    ...
  </inertia-link>
</template>

<script>
export default {
  props: {
    ...
    method: String,
    ...
  }
}
</script>
```

然後 Layout 裡就可以傳 `method="post"` 了 (還有路徑是 `/logout`)：

*resources/js/Layouts/AppLayout.vue*
```html
<dropdown-item href="/logout" method="post" icon="heroicons-outline:logout" @click="close">
  登出
</dropdown-item>
```

登出功能完成~~ 開瀏覽器試試。

## 登入失敗提示

現在如果打錯帳密它不會跟你講，但其實 Laravel 背後都已經產好訊息了，只是前端還沒看到。

在 `inertia-laravel` 的 `0.2.9` 版增加自動處裡表單驗證錯誤訊息的功能，如果 `inertia-laravel` 低於 `0.2.9` 要先升級：

```bash
composer require inertiajs/inertia-laravel:^0.2.9
```

然後可以使用 `$page.errors.email` 取得 E-mail 的驗證錯誤訊息：

*resources/js/Pages/Auth/Login.vue*
```html
<text-input v-model="form.email" :error="$page.errors.email" label="E-mail" autocomplete="email" ref="emailInput" />
```

調整一下文字輸入框：

*resources/js/Components/TextInput.vue*
```vue
<template>
  <div>
    ...
    <input :id="id" ref="input" v-bind="$attrs" class="form-input" :class="{ error }" :type="type" :value="value" @input="$emit('input', $event.target.value)">
    <div v-if="error" class="form-error">{{ error }}</div>
  </div>
</template>

<script>
export default {
  ...
  props: {
    ...
    error: String
  },
  ...
}
</script>
```

還有錯誤時的紅字紅框：

*resources/css/form.css*
```css
.form-error {
  @apply text-red-700 mt-2 text-sm;
}

.form-input.error,
.form-textarea.error,
.form-select.error {
  @apply .border-red-600;

  &:focus {
    box-shadow: 0 0 0 1px theme('colors.red.600');
  }
}
```

![](../images/day09-06.jpg)

## 解決記住我始終為 True 問題

原本後端在讀「記住我」的值是用 `$request->filled('remember')`，判斷存在且不為空，可是用 XHR 傳過去是 Boolean 值，不管 `true` 或 `false` 都會回傳 `true` (存在)，需要修正一下此s問題：

*app/Http/Controllers/Auth/LoginController.php*
```php
use Illuminate\Http\Request;

protected function attemptLogin(Request $request)
{
    return $this->guard()->attempt(
        $this->credentials($request), $request->input('remember')
    );
}
```

## 載入進度條

Inertia.js 整合了 [NProgress](https://ricostacruz.com/nprogress/)，頁面載入時頂部有那條細長的進度條。Inertia.js 還設定讓它在頁面載入超過 250 毫秒時才顯示，且預設不載入 CSS，所以才看不到。那就讓我們自己來調吧：

*resources/css/components.css*
```css
/* NProgress */
#nprogress {
  @apply pointer-events-none;
  .bar {
    @apply fixed top-0 inset-x-0 w-screen bg-purple-400;
    height: 3px;
    z-index: 9999;
  }
}
```

![](../images/day09-07.jpg)

進度條出現~~

## 總結

跟之前比，本篇真的......很長，不過也表示 Lightning 多了一些功能，離部署之日也進了一步。下一篇會來處裡 Presenter，自己的資料自己掌控！

> Lightning 範例程式碼：https://github.com/ycs77/lightning

## 參考資料

* [PingCRM](https://github.com/inertiajs/pingcrm)
