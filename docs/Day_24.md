# Day 24 Lightning 留言功能

看完文章後，會想要留下自己的感想或發表意見。本篇就開始來實作文章留言功能。

## 新增 Comment Model 和相關檔案

依然還是新增 Model 和相關檔案：

```php
php artisan make:model Comment -mf
```

和規劃資料表結構，這邊為了更符合語意，提交留言的用戶改用 `committer_id`：

*database/migrations/2020_09_26_093544_create_comments_table.php*
```php
Schema::create('comments', function (Blueprint $table) {
    $table->id();
    $table->text('content');
    $table->foreignId('post_id')->constrained()->onDelete('cascade');
    $table->foreignId('committer_id')->constrained('users')->onDelete('cascade');
    $table->timestamps();
});
```

然後跑 Migrate：

```php
php artisan migrate
```

還有 Comment 的一些基本設定和關聯：

*app/Comment.php*
```php
protected $fillable = [
    'content',
];

protected $casts = [
    'post_id' => 'integer',
    'committer_id' => 'integer',
];

public function post()
{
    return $this->belongsTo(Post::class);
}

public function committer()
{
    return $this->belongsTo(User::class);
}
```

在 Post 裡也要新增 `comments` 關聯：

*app/Post.php*
```php
public function comments()
{
    return $this->hasMany(Comment::class);
}
```

然後配置 Comment 的假文 Factory 和 Seeder 部分：

*database/factories/CommentFactory.php*
```php
$factory->define(Comment::class, function (Faker $faker) {
    return [
        'content' => $faker->realText(20),
    ];
});
```

這次就不新增 Seeder，直接在 `PostSeeder` 裡隨機產生 Comment：

*database/seeds/PostSeeder.php*
```php
use App\Comment;

public function run()
{
    factory(Post::class, 12)->make()->each(function (Post $post) {
        ...

        factory(Comment::class, random_int(0, 3))->make()
            ->each(function (Comment $comment) use ($post) {
                $comment->committer()->associate(User::inRandomOrder()->first());
                $comment->post()->associate($post);
                $comment->save();
            });
    });
}
```

## 留言功能路由

為了讓 Controller 直接產生在 `Post` 下，執行 Artisan 新增此 Resource Controller：

```php
php artisan make:controller Post/CommentController -r --model=Comment
```

留言只會用到儲存和刪除留言兩個方法，其他可以刪掉，路由也一樣 (用 `only` 指定)：

*routes/web.php*
```php
// Comments
Route::resource('posts.comments', 'Post\CommentController')->shallow()->only('store', 'destroy');
```

那這個陌生的 `shallow()` 到底是什麼？先看這裡我們用到了嵌套路由，產生出來的路由表會長這樣：

| Method | URI                             | Name                   | Action                                              |
| ------ | ------------------------------- | ---------------------- | --------------------------------------------------- |
| POST   | posts/{post}/comments           | posts.comments.store   | App\Http\Controllers\Post\CommentController@store   |
| DELETE | posts/{post}/comments/{comment} | posts.comments.destroy | App\Http\Controllers\Post\CommentController@destroy |

但會發現，有些路由會同時有父層 ID (`post`) 和子層 ID (`comment`)，既然有了 `comment` ID 這個資源的唯一 ID，那前面的父層 `post` ID 就不需要了。套用 `shallow()` 後就會把多餘的部分去除，變成淺層嵌套路由：

| Method | URI                   | Name                 | Action                                              |
| ------ | --------------------- | -------------------- | --------------------------------------------------- |
| POST   | posts/{post}/comments | posts.comments.store | App\Http\Controllers\Post\CommentController@store   |
| DELETE | comments/{comment}    | comments.destroy     | App\Http\Controllers\Post\CommentController@destroy |

## 新增 CommentPresenter

```bash
php artisan make:presenter CommentPresenter
```

*app/Presenters/CommentPresenter.php*
```php
<?php

namespace App\Presenters;

use AdditionApps\FlexiblePresenter\FlexiblePresenter;
use App\Concens\HasAuthUser;

class CommentPresenter extends FlexiblePresenter
{
    use HasAuthUser;

    public function values(): array
    {
        return [
            'id' => $this->id,
            'content' => $this->content,
            'committer' => UserPresenter::make($this->committer)->get(),
            'created_at' => $this->created_at->diffForHumans(),
            'can_delete' => $this->userCan('delete', $this->resource),
        ];
    }
}
```

## 總結

前置作業已準備好，下一篇可以開始新增留言了！

> Lightning 範例程式碼：https://github.com/ycs77/lightning

## 參考資料

* [Nested Resources - Controllers - Laravel](https://laravel.com/docs/controllers#restful-nested-resources)
