---
title: Rails4 Outlaw (2)
date: 2015-05-27 09:56 UTC
tags: rails
layout: post
---
> If you don't like where you are, change it. You're not a tree. - Jim Rohn

這幾次在寫 Code School 學習筆記的時候，通常都會自己開個專案進去根據他所說的去做點實驗，其實一方面也是幫助自己記憶，只是有時候會卡關，想不通為什麼可以這樣用，所以後來覺得去看官方的 [Rails Guide](http://guides.rubyonrails.org/) 其實是蠻好的幫助，而且裡面的解釋也是相當的淺顯易懂，是非常適合練習閱讀以及了解核心概念的方法，所以之後每個禮拜都會寫一篇 Guide 的讀書心得，前言結束，開始往下紀錄。

#### Scope
我覺得 Scope 是相當神奇的東西，宣告後就好像有了這樣一個屬性，在 rails4 裡面定義為必須使用 `proc`，

```ruby
scope :sold, ->{ where(state: 'sold') }

# default scope 則可以使用 block
default_scope { where(state: 'available') }
default_scope ->{ where(state: 'available') }
```

#### Relation
Rails4 新增了很多當我們在對於 Model 進行存取的時候可能會使用的工具，像是 `none`，因為可能有時候會有要回應空陣列的情形，但是如果在 controller 就必須去驗證是否為空，這樣多一次的 if else 還是讓整個 code 寫得很髒，none 就能夠達到這樣的需求

```ruby
# app/models/users.rb
class User < ActiveRecord::Base
  def visible_posts
    case role
    when 'Country Manager'
      Post.where(country: country)
    when 'Reviewer'
      Post.published
    when 'Bad User'
      Post.none
    end
  end
end
```
這樣當我們在 controller 需要進行一個呼叫的動作

```ruby
@posts = current_user.visible_posts
@posts.recent
```
在這樣的情形下，Post.none 並不會去對資料庫做存取，而且即使返回的是空陣列，呼叫 scope 也不會爆炸，因為空陣列是來自 ActiveRecord::Relation。

接著要談的是另一個有趣的工具 `not`，在對 Active Record 的 object 使用 where method 的時候，可能會遇到我們希望找到除了本身以外的值，傳統的作法可能是類似寫 `where('author != ?', author)` 如果 author 一直都有值那就暫時不會出現問題，但是如果 author 為空，就會出現錯誤 query

```sql
SELECT "posts".* FROM "posts" WHERE (author != NULL)
```
所以在 rails4 我們可以這麼寫

```ruby
Post.where.not(author: author)
```
如此一來就會產生出正確的 query

```sql
SELECT "posts".* FROM "posts" WHERE (author IS NOT NULL)
```
最後要談的部分是 `References`，我們可能常常會需要使用 `include`，將關聯的資料表先讀進來，以解決 N+1 query 的問題，在此之前先來談 include 其實是兩個 method 的綜合體，`preload` 和 `eager_load`，

>先講結論：從 sql 的角度來看兩者最後的結果是相同的，只是 query 不同

但是 preload 並不支援後面再繼續使用 where 等 query

```ruby
# It doesn't work
post = Post.published.preload(:tags).where(tags: {name: "Rails"})

# Correct
post = Post.published.eager_load(:tags).where(tags: {name: "Rails"})
```
然而 include 則會自動去定義應該選擇使用哪一個，然而當我們在使用 include 的時候，可以更明確的定義我們是關聯到哪個資料表，所以會出現這樣的寫法，

```ruby
Post.includes(:comments).
  where("comments.name = 'foo'").references(:comments)
```
