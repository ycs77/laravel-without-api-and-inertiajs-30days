# Day 01 為什麼我用 Laravel 寫 SPA 卻可以不寫 API?

在回答這個問題之前，我們先回來看看網頁前後端的區別吧！

以後端為基礎的網頁應用，優點是天生SEO良好，缺點是載入速度較慢。後來以AJAX為基礎，誕生了前端的網頁應用，優點是有良好的使用者互動，缺點是SEO先天不佳（依賴後天補足）而且要寫API（重點在這）。以上大家都耳熟能詳，這裡只大概帶過，想要詳細了解前後端的差異請參考 [Rendering on the Web](https://developers.google.com/web/updates/2019/02/rendering-on-the-web) 這篇文章。

> 如果以上的部分你壓根兒都沒聽過，可能比較不適合閱讀本文。建議可以右上角點X，然後去 [Google 搜尋: SPA MPA](https://www.google.com/search?q=spa+mpa) 了解相關的知識。

接下來我們來了解一下前後端「架構」的差異，什麼？MVC？MVVM？那些都不是本文的重點。在這裡我用 Jonathan Reinink 的[文章](https://reinink.ca/articles/server-side-apps-with-client-side-rendering)裡談到的架構，以**路由**、**資料**、**視圖**區分，理解前後端架構差異：

![前後端架構差異](../images/day01-01.jpg)

為了要發揮前端和後端的優勢，有必要將以上兩種架構混和一下，結果就是接下來要介紹的這個有點古怪的架構，在**經典後端應用**中使用**前端渲染**，又稱 **「前端渲染的後端應用」**：

> 其實這個名稱是根據我的獨斷與偏見翻譯出來的。嘻~
>
> 提出這個架構的 Reinink 在[原文](https://reinink.ca/articles/server-side-apps-with-client-side-rendering)裡是寫「Server-side apps with client-side rendering」。其中的 Server-side 和 Client-side 中文本來應該是「服務端」和「用戶端」，但我比較想用「前端」和「後端」，因為比較簡短、更直覺而且都通用，在本系列中都會使用此名稱。

![後端路由+ORM資料+前端組件渲染=前端渲染的後端應用架構](../images/day01-02.jpg)

「蛤？這看起來不就是 SPA 嘛！我也會！」

別急，這可不完全是 SPA。剛才有說道這是「後端應用」，不只後端管路由 (Router)，後端還管除了視圖 (View) 以外的所有東西。然後視圖 (View) 才交給前端，使用「前端組件」來舒服地刻前端頁面。

意思就是，使用這種架構我可以前端用 Vue.js 刻版面，和後端用 Laravel 的 Controller、Middleware、Auth、Eloquent ORM...既有功能——來做 SPA 應用。(爽不爽？)

要使這種架構得以運行，就必須要有一個東西來做前後端的溝通橋樑。這就是這個系列的主題——[Inertia.js](https://inertiajs.com/)，同時也是**我用 Laravel 寫 SPA 卻可以不寫 API** 的關鍵。那 Inertia.js 是什麼？他只是個 Javascript 套件嗎？這些我會在下篇來詳細介紹。

## 關於此系列

這個系列會觸及到前後端，會使用到的框架有以下：

* [Laravel](https://laravel.com/) v7
* [Vue.js](https://cn.vuejs.org/) v2.6
* [Tailwind CSS](https://tailwindcss.com/) v1.8

Laravel 和 Vue.js 都是非常熱門的框架，放眼網路世界都可以找到許多學習資源，在鐵人賽中也有不少以此為主題的文章，因此我不會從入門開始講起。如果你想要繼續往下看，會基本的 Laravel 和 Vue.js 會比較適合。而要會到哪種程度？Laravel 至少會 Controller、Model，Vue.js 至少會基本的組件，有以上知識應該可以跟得上吧！(再不行就單純複製貼上...)

以前刻板大多都使用 Bootstrap，但樣式看起來都長得很「Bootstrap」，且少了高度客製化的靈活度。這裡我用 [Tailwind CSS](https://tailwindcss.com/) 來代替它，Tailwind CSS 是一個 Utility-First 的 CSS 框架，沒聽過？沒關係，後面我會介紹，然後跟者入坑就對了。

在這個系列文章我將使用以上框架，建構一個簡易的部落格平台 **Lightning**，來帶大家了解 Inertia.js。用實際的專案來了解在開發時會遇到的各種問題。主要會實作以下功能：

* 用戶登入/註冊 *(放心可以登出)*
* 撰寫文章/編輯/刪除 *(只能用 Markdown 寫文章)*
* 文章留言/刪除 *(不做回覆留言)*
* 部屬到 Heroku *(zzz...)*
* Server Side Rendering 功能 *(只對機器人做渲染，解決 SEO 問題)*

**準 備 好 了 嗎？**

要開始這趟 Inertia.js 之旅囉~~

## 參考資料

* [Server-side apps with client-side rendering](https://reinink.ca/articles/server-side-apps-with-client-side-rendering)
* [Who is Inertia.js for? - Inertia.js](https://inertiajs.com/who-is-it-for)
