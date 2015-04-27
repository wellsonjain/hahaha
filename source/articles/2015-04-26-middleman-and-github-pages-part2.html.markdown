---
title: Middleman And Github Pages (2)
date: 2015-04-26 07:40 UTC
tags: middleman, github
layout: post
---

接下來就要介紹到如何將 blog 部署到 Github Page 上，Github Pages 也是為了滿足 deploy 至靜態網站的需求。

####2. Deploy to Github Page
Middleman 提供了許多 deploy 方式的 gem，不過因為一開始我就是以 Middleman + Github 的 Solution，所以就沒有特別去研究其他的方式，不過之後會對於 Rails 專案的紀錄比較多種 deploy 組合的方式。

首先先在 `Gemfile` 內加入

```
gem 'middleman-deploy'
```
並執行 `bundle install`，現在我們已經具備了可以 deploy 到 Github 上的 method 了

接著要做的就是到 Github 新增一個給 blog 的 repository，在這邊要注意一下因為我們是要單純做一個個人的 blog，所以我們在設定上就相對簡單許多，在 Domain 上相當單純只有使用者帳號，但是要注意的是一個帳號也只能有這麼一個名稱為 <em>username.github.io </em> 的 Domain。
> 在 [Github Pages](https://pages.github.com/) 上也是有很簡單易懂的教學。

當我們完成了上面這兩件事情之後，我們就可以開始極度<del>無腦</del>簡單的 deploy 步驟

```
$ middleman build
$ middleman deploy
```
這樣就把我們的 blog 赤裸裸的一絲不掛的一滴不漏的推到全世界的眼前了!! (水拉!!!) 但是既然我們把 blog 公開出來，當然也希望能得到一些回應，所以我就很跟風的看著許多技術社群的大大都會在文章的底下外掛上套件 [Disqus](https://disqus.com/) 基於如果剛剛就結束的話篇幅有點短，所以再繼續為大家介紹如何外掛上 Disqus。

####3. Disqus plug-in
在技術社群上總是有很多高手幫助我們造好輪子，這樣我們就只要專心在如何加強引擎就好，一樣有前輩為了這個需求而開發好了 [middleman-disqus](https://github.com/simonrice/middleman-disqus) 這個 gem，接下來一樣是在 `Gemfile` 內加入

```
gem 'middleman-disqus'
```
然後執行 `bundle install`，接著在 `config.rb` 內將這個功能給開啟

```
# config.rb
activate :disqus do |d|
  d.shortname = 'your-shortname' # Replace with your Disqus shortname.
end
```
>`your_shortname` 這是要去 [Disqus](https://disqus.com/) 上面申請的

接著在你的 layout 上新增

Haml:

```haml
/ link with `#disqus_thread` is optional if not using `disqus_count` -->
%a{:href => "http://example.com/foo.html#disqus_thread"} Comments

= disqus
= disqus_count
```
ERB:

```erb
<!-- link with `#disqus_thread` is optional if not using `disqus_count` -->
<a href="http://example.com/foo.html#disqus_thread">Comments</a>

<%= disqus %>
<%= disqus_count %>
```

這樣一個具備完整功能的 blog 就完成了，接下來就開始一點一點的累積生活的日常吧!!
