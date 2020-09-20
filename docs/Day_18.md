# Day 18 Lightning 文章、草稿列表

## 文章列表

文章列表的路由已經存在 Resource 裡，只要再加一個草稿列表的路由，而且一定要放在 ShowPost 上面：

*routes/web.php*
```php
// Posts
...
Route::get('posts/drafts', 'Post\PostController@drafts');
Route::get('posts/{post}', 'Post\ShowPost');
```

然後是文章列表和草稿列表的 Controller 部分：

*app/Http/Controllers/Post/PostController.php*
```php
public function index()
{
    $posts = $this->user()
        ->posts()
        ->where('published', true)
        ->latest()
        ->get();

    return Inertia::render('Post/List', [
        'type' => 'published',
        'typeText' => '文章',
        'posts' => PostPresenter::collection($posts)
            ->preset('list')
            ->get(),
    ]);
}

public function drafts()
{
    $posts = $this->user()
        ->posts()
        ->where('published', false)
        ->latest()
        ->get();

    return Inertia::render('Post/List', [
        'type' => 'drafts',
        'typeText' => '草稿',
        'posts' => PostPresenter::collection($posts)
            ->preset('list')
            ->get(),
    ]);
}
```

還有屬於列表的 preset，裡面要放文章作者，需要的欄位不用那麼多，可以用 `only()` 指定：

*app/Presenters/PostPresenter.php*
```php
public function presetList()
{
    return $this->with(fn (Post $post) => [
        'author' => fn () => UserPresenter::make($post->author)
            ->only('id', 'name', 'avatar')
            ->get(),
    ]);
}
```

兩個列表是共用同一個列表頁面，只差會放不同的文章集合：

*resources/js/Pages/Post/List.vue*
```vue
<template>
  <div class="py-6 md:py-8">
    <alert v-if="$page.flash.success" class="shadow mb-6">{{ $page.flash.success }}</alert>

    <div class="card card-main">
      <div>
        <div class="flex justify-between items-center">
          <h2 class="text-3xl">我的{{ typeText }}</h2>
          <div>
            <inertia-link href="/posts/create" class="link">
              <icon icon="heroicons-outline:pencil" />
              撰寫文章
            </inertia-link>
          </div>
        </div>
        <hr class="mt-4">

        <tabs class="mt-4" :active="type">
          <tab name="published" url="/posts">已發布</tab>
          <tab name="drafts" url="/posts/drafts">草稿</tab>
        </tabs>
      </div>

      <div class="mt-6">
        <post-list :posts="posts" hide-author :empty="`目前沒有${typeText}`" />
      </div>
    </div>
  </div>
</template>

<script>
import AppLayout from '@/Layouts/AppLayout'
import Alert from '@/Components/Alert'
import Tabs from '@/Components/Tabs'
import Tab from '@/Components/Tab'
import PostList from '@/Lightning/PostList'

export default {
  layout: AppLayout,
  metaInfo() {
    return {
      title: `我的${this.typeText}`
    }
  },
  components: {
    Alert,
    Tabs,
    Tab,
    PostList
  },
  props: {
    type: String,
    typeText: String,
    posts: Array
  }
}
</script>
```

還有要做使用到的組件們，首先是真正的文章列表組件，只要是文章列表都可以用這個組件渲染：

*resources/js/Lightning/PostList.vue*
```vue
<template>
  <div>
    <ul v-if="posts.length" class="divide-y -my-6">
      <li v-for="post in posts" class="py-6">
        <h2>
          <inertia-link :href="`/posts/${post.id}`" class="text-xl font-medium hover:text-purple-500 transition-colors duration-100">{{ post.title }}</inertia-link>
        </h2>
        <div class="text-gray-500 font-light mt-1">{{ post.description }}</div>
        <div class="flex items-center space-x-4 text-gray-500 text-sm font-light" :class="hideAuthor ? 'mt-1' : 'mt-3'">
          <inertia-link v-if="!hideAuthor" :href="`/user/${post.author.id}`" class="inline-flex items-center hover:text-purple-500 font-normal">
            <img :src="post.author.avatar" class="w-6 h-6 rounded-full">
            <span class="ml-2">{{ post.author.name }}</span>
          </inertia-link>
          <div>
            <icon class="w-4 h-4 text-purple-500" icon="heroicons-outline:clock" />
            {{ post.created_ago }}
          </div>
          <slot name="info-after" :post="post" />
        </div>
      </li>
    </ul>

    <div v-else class="text-center text-gray-400 mt-8 mb-4">{{ empty }}</div>
  </div>
</template>

<script>
export default {
  props: {
    posts: {
      type: Array,
      required: true
    },
    hideAuthor: {
      type: Boolean,
      default: false
    },
    empty: {
      type: String,
      default: '目前沒有文章'
    }
  }
}
</script>
```

再來是一組簡易的 Tab 組件：

*resources/js/Components/Tabs.vue*
```vue
<template>
  <div class="flex space-x-2">
    <slot />
  </div>
</template>

<script>
export default {
  props: {
    active: String
  }
}
</script>
```

*resources/js/Components/Tab.vue*
```vue
<template>
  <div v-if="name === active" class="tab tab-active">
    <slot />
  </div>
  <inertia-link v-else :href="url" class="tab tab-link">
    <slot />
  </inertia-link>
</template>

<script>
export default {
  props: {
    name: {
      type: String,
      required: true
    },
    url: String
  },
  computed: {
    active() {
      return this.$parent.$options._componentTag === 'tabs'
        ? this.$parent.active
        : null
    }
  }
}
</script>
```

Tab 組件的樣式：

*resources/css/components.css*
```css
/* Tab */
.tab {
  @apply px-2 py-1 text-sm font-light rounded select-none;
}
.tab-link {
  @apply text-purple-500 transition-colors duration-100;
  &:hover {
    @apply text-purple-700;
  }
}
.tab-active {
  @apply bg-purple-100 text-purple-700;
}
```

然後就可以看到列表頁面，還可以切換草稿頁面：

![](../images/day18-01.jpg)

![](../images/day18-02.jpg)

## 總結

列表做好了，但如果文章多了起來，頁面就會很長，載入會很久，因此，下回將要來做列表分頁了。

> Lightning 範例程式碼：https://github.com/ycs77/lightning
