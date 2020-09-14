# Day 14 Lightning 文章功能

Lightning 作為一個部落格平台，最重要的自然是發文功能。本篇要先準備好文章需要的 Migration、Model、Presenter 等等。

## 新增 Post Model 和相關檔案

為了快速產生需要的檔案，新增 Model 後面還帶上 `-mfr`，`m` 是順便新增 Migration，`f` 是新增 Factory，`r` 是新增 Resource Controller：

```bash
php artisan make:model Post -mfr
```

然後規劃我們的資料表，這裡是用 `author_id` 指向 `users` 資料表：

*database/migrations/2020_09_16_090156_create_posts_table.php*
```php
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->string('description');
    $table->text('content');
    $table->string('thumbnail')->nullable();
    $table->unsignedInteger('visits')->default(0);
    $table->boolean('published')->default(false);
    $table->foreignId('author_id')->constrained('users')->onDelete('cascade');
    $table->timestamps();
});
```

然後跑 Migrate：

```bash
php artisan migrate
```

增加一些 Post 裡的設定，`$fillable` 是批量填充資料時允許的欄位；`$casts` 是型別轉換；`updateDescription()` 是更新 `description`，擷取 `content` 的前80個字，並在新增和更新 Model 時擷取 `description`；還有關聯作者：

*app/Post.php*
```php
use Illuminate\Support\Str;

protected $fillable = [
    'title', 'description', 'content', 'thumbnail', 'visits', 'published',
];

protected $casts = [
    'visits' => 'integer',
    'published' => 'boolean',
    'author_id' => 'integer',
];

protected static function booted()
{
    static::creating(function (self $post) {
        $post->updateDescription();
    });

    static::updating(function (self $post) {
        $post->updateDescription();
    });
}

public function updateDescription()
{
    $this->description = Str::limit($this->content, 80);

    return $this;
}

public function author()
{
    return $this->belongsTo(User::class);
}
```

User 也要設定和 Post 的關聯：

*app/User.php*
```php
public function posts()
{
    return $this->hasMany(Post::class, 'author_id');
}
```

Post 的假資料 Factory：

*database/factories/PostFactory.php*
```php
$factory->define(Post::class, function (Faker $faker) {
    return [
        'title' => $faker->realText(10),
        'content' => $faker->realText(1000),
        'visits' => $faker->numberBetween(10, 1000),
        'published' => $faker->boolean(80),
    ];
});
```

和假資料 Seeder：

```bash
php artisan make:seeder PostSeeder
```

這裡隨機指派文章作者：

*database/seeds/PostSeeder.php*
```php
use App\Post;
use App\User;

public function run()
{
    factory(Post::class, 12)->make()->each(function (Post $post) {
        User::inRandomOrder()->first()->posts()->save($post);
    });
}
```

*database/seeds/DatabaseSeeder.php*
```php
public function run()
{
    ...
    $this->call(PostSeeder::class);
}
```

## 新增 PostPresenter

Presenter 也先準備好：

```bash
php artisan make:presenter PostPresenter
```

*app/Presenters/PostPresenter.php*
```php
public function values(): array
{
    return [
        'id' => $this->id,
        'title' => $this->title,
        'description' => $this->description,
        'thumbnail' => $this->thumbnail,
        'visits' => $this->visits,
        'created_at' => $this->created_at->format('Y-m-d H:i:s'),
        'created_ago' => $this->created_at->diffForHumans(),
        'published' => $this->published,
    ];
}
```

## 總結

前置作業準備好，下一篇可以開始新增文章了！

> Lightning 範例程式碼：https://github.com/ycs77/lightning
