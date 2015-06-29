---
title: Rails4 Outlaw
date: 2015-05-24 03:56 UTC
tags: rails
layout: post
---
> As you think, so shall you become. - unknown

Hello Hello，今天依舊不原創的我，要紀錄一下的一樣在 [Code School](https://www.codeschool.com/) 的課後筆記，首先先用 rails new 一個新 project，接著使用 `scaffold` 這個 rails 的懶人包工具，他會將基本的 CRUD 都生出來，有些人會新手不應該習慣使用 scaffold 這個工具應該自己從頭到尾都好好的把怎麼寫 CRUD 給記熟，我也是這麼認為，但是學了一段時間之後可以回來再看看到底 scaffold 這個工具是生出什麼樣的程式碼，就可以知道官方可能是比較推薦大家該怎麼寫 rails (上面這段話是強者我同事 [Andy](https://www.facebook.com/chouandy625?fref=ts) 哥告訴我的金玉良言)

在 Rails4 將 Update 的 http verb 改成為使用 `PATCH`，這是因為 put 是將整個 resource 更新，但是我們可能只是要更新部分的 resource，而 patch 就是滿足這個需求。

#### Routes

接者在 routes 的部分可能一個 resource 底下會再有幾個 resource，但是如果是多個 resource 底下有多個"相同的"resource，我們可以這麼寫，

```ruby
# config/routes.rb
  consern :sociable do |options|
    resource :comments,   options
    resource :categories, options
    resource :tags,       options
  end

  resource :messages, concerns: :sociable
  resource :posts,    concerns: :sociable
  resource :items do
    concerns: :sociable, only: :create
  end
```
使用 `concerns` 就有點像是 ruby 使用 module 的概念將他外掛加強模組，同時還可以在後綴參數，可以再限縮使用的範圍

#### Finder
在 rails3 裡面有很多種 `find_by_xxx`，`find_last_by_xxx`，其實這些都是透過 ruby 的 method missing 去實做出來的，但是這樣卻會造成一些效能上的問題，所以在 rails4 比較建議的寫法是

```ruby
# 找出所有 title 為 'Rails 4' 的 post
Post.find_by(title: 'Rails 4')

# 找出所有 title 為 'Rails 4' 並且 author 為 'admin' 的 post
Post.find_by(title: 'Rails 4', author: 'admin')
```
除此之外還有可以更 dynamic 的方式來做 finder，因為有可能資料的來源是 json，而 `find_by` 也能夠接受資料型態為 hash

```ruby
post_params = { title: 'Rails 4', author: 'admin' }
Post.find_by(post_params)
```
除了 `find_by` 之外，還有一些變化的型態，像是 `find_or_create_by` 意思是如果找不到就新增，在 rails3 有提供一個類似的方法 `first_or_create`，其實這兩個方法做的事情是相同的，但是唯一會出現問題的地方就在於是否有使用 active record callback，尤其是當在 callback 裡面又有一段 query

```ruby
# app/controllers/posts_controller.rb
# type 1
Post.where(title: 'Rails 4').first_or_create

# type 2
Post.find_or_create_by(title: 'Rails 4')

#############################################

# app/models/post.rb
class Post < ActiveRecord::Base
  after_create :foo

  def foo
    post = Post.where(author; 'admin')
    ...
  end
end
```
可是這樣做到底會造成什麼問題呢？ 如果說我們可能有個需求就是當新增了之後要更新所有作者為 admin 的某個欄位值，這樣會造成 query 上的不一致，在 `first_or_create` 所做的 query 為

```sql
-- type 1
SELECT "posts".* FROM "posts"
  WHERE "posts"."title" = 'Rails 4' AND "posts"."author" = 'admin'
```
可是我們在 callback 裡面只有呼叫 author 為 admin 的啊，這樣反而限制的更嚴格，和我們原本所設想的有落差，而 `find_or_create` 所做的 query 則是

```sql
-- type 2
SELECT "posts".* FROM "posts" WHERE "posts"."author" = 'admin'
```
接著 find 之後來說 update，在 rails4 裡提供了兩種 update 方式，分別是，

```ruby
@post.update(post_params)

# skip validations
@post.update_columns(post_params)
```
因為第二種會直接對資料庫做存取，假設我們有設定 post title 不得為空，但是如果用第二種，即使 title 設定為空，依然會直接更新到資料庫，並且不會有 updated_at 這個日期的更動，所以還是使用第一種比較安全。


