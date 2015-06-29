---
title: Rails4 Outlaw (4)
date: 2015-06-02 17:19 UTC
tags: rails
layout: post
---
> You may not control all the events that happen to you, but you can decide not to be reduced by them. - Maya Angelou

#### Filter
在 rails3 有 `before_filter` 幫我們處理一些我們可能每次在執行某些 function 的時候會有些先行條件要完成，但是語意上可能會顯得不是那麼直觀，所以在 rails4 就改成為 `before_action`，但是值得一提的是 rails4 並沒有將 filter 給封掉，所以一樣還是可以使用。

#### Session
這個部分跟我上一篇所提的剛好可以做個相呼應，session 有點像 global variable，我們可以在 rails project 的任何地方都可以呼叫出來，因為他就是儲存在 server 上，Rails 採用的是 `Cookies session storage` 來儲存 session，概念上是這樣

```
            |     initial request     |
Browser     |------------------------>|session[:user_id] = user.id
            |   Send session data     |
{Cookeis}   |<------------------------| APP
            |                         |
            |   Sends Cookie data     |
            |------------------------>|@current_user ||=
                                          User.find(session[:user_id])
```
Session 透過編碼之後再存在 Cookies 裡面，在 rails3 裡雖然 session 裡面的值不能改變，但是卻可以讀得到，只要用 `Rack::Session::Cookie::Base64::Marshal.new.decode(cookie)` 就可以把 cookies 裡面的內容給讀取出來，但是在 rails4 加強了安全性，即使使用上述的方法依然沒辦法。

#### Flash Type
flash 是 rails 所提供的一個 hash，可以在前端顯示一些訊息，像是當我們在新增成功一筆資料可能導回前端之後，`Your item has created successfully.` 這樣一個訊息一起帶過去，在 rails4 有一些 mehthod 可以很方便的使用，

```erb
<p id="notice"><%= flash[:notice] %></p>
<p id="alert"><%= flash[:alert] %></p>


<p id="notice"><%= notice %></p>
<p id="alert"><%= alert %></p>
```
而且事實上不只這些預設的可以使用，我們也可以自定義我們想要的 flash，只要在 Application controller 加上，

```ruby
class ApplicationController < ActionController::Base
  add_flash_type :warning
end
```
就一樣可以在前端直接使用這個 flash 的 method。
