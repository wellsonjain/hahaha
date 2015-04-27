---
title: Middleman and Github Pages (1)
date: 2015-04-26 06:06 UTC
tags: middleman, github
layout: post
---

事實上要說要架自己的 blog 也說了好一段時間了，先前一度嘗試用 [Hexo](https://hexo.io/zh-tw/) 來當 bloger，但是自己畢竟對 node.js 還不太熟，之後選擇使用 [Middleman](https://middlemanapp.com)，因為 Ruby 對我來說還是比較熟悉的語言，而且搭配許多 Gem，終於架出了一個自己比較喜歡的 blog 模組，以後就可以用這個來記錄自己在 Programming 上或是在生活上的一些學習歷程以及心得。

事實上在個人 blog platform 市面上有許多選擇，像是 [Pixnet](https://www.pixnet.net/), [Google Blogger](https://www.blogger.com/) 或是蠻潮的微網誌 [Tumblr](https://www.tumblr.com/)，還有為技術文章而生的 [Logdown](logdown.com/)，雖然或多或少其中都有符合我需求的元素，但是自己組出一個，就像是自己用樂高做出一個鋼鐵人，那種成就感是難以言喻的，而且在使用上有我個人非常喜歡用 markdown 來撰寫文章，能讓文章看起來更加 organized，所以最後我選擇自己找工具組裝 my personal blog。

第一篇就先來記錄一下架 Blog 的過程，一共有三個部分

1. Install Middleman and Relavant
2. Deploy to Github Pages
3. Disqus plug-in

####1. Install Middleman and Relavant
> [Middleman](https://middlemanapp.com) 在官方的文件上也詳列了步驟細節，非常建議藉由文件來瞭解安裝的細節。

首要之急當然就是必須先安裝 Gem，但是事實上 middleman 的目的是建造一個靜態網頁所開發出來的 gem，所以要使用 blog 的 template 我們還必須安裝 `middleman-blog` 這個 gem

```
$ gem install middleman
$ gem install middleman-blog
```

安裝完成之後在想要的資料夾目錄下，輸入

```
$ middleman init MY_BLOG_PROJECT --template=blog
```

即可以立馬擁有一個超級陽春的<del>陽春麵</del> blogger，但是身為一個前端也不太熟的小菜鳥，要從頭到尾刻出一個很 Fancy 的 blog 簡直難如上青天啊!!! 只能說是天無絕人之路，後來在 [Ruby on Rails 新手村](https://www.facebook.com/groups/RailsRookie/) 認識的朋友幫我找到了一個在介面看起來相當漂亮的 template，[middleman-casper](https://github.com/danielbayerlein/middleman-casper)，於是乎讓我更快的慢慢完成(大概隔了三個月才開始用...)這個 blog

使用的方法其實很簡單，首先先建立 `~/.middleman` 這個檔案目錄，接著在該檔案目錄下，輸入指令

```
$ git clone https://github.com/danielbayerlein/middleman-casper.git ~/.middleman/casper
```

接著移動到在本機上想要建立這個 blog 的目錄下，<del>至於剛剛那碗陽春麵就把它給刪掉吧</del>，執行

```
$ middleman init MY_BLOG_PROJECT --template=casper
```

進到 blog 目錄後，執行 `bundle install`，接下來只是在 `config.rb` 內做一些微調或者是設定一些基本資訊，一個在本機上的 blog 就基本上完成了。

