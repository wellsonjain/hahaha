---
title: Rails4 Outlaw (5)
date: 2015-06-04 15:28 UTC
tags: rails
layout: post
---
> Start where you are. Use what you can. Do what you can. - Arthur Ashe

#### ETag
接下來要介紹的是<del>遠通的</del> ETag，ETag 的目的是用來辨識當前的頁面是否有所改變，流程大概是這樣，

- Client 第一次拜訪 Rails App
  * render  body
  * Create ETag `headers['ETag'] = Digest::MD5.hexdigest(body)`
  * 將 Etag 和 Body 包在一起

- Client cache 住 Response

- Client 第二次拜訪 Rails App
  * render  body
  * Create ETag
  * 將 ETag 和 Client 傳來的 `headers[If-None-Match]` 進行比對，如果相同則不 response body
(真搞不懂為什麼要兩個 header 名字取不一樣)

如果說進去看 header 的話第二次的 status code 將會是 `304 Not Modified`，雖然這樣就不需要再重新 response body，但是每次 request 都必須再把整個 body 去 generate 一次 ETag 這是相當沒有效率的事，所以我們可以 set 一個 custom ETag

```ruby
class ItemsController < ApplicationController
  def show
    @item = Item.find(params[:id])
    fresh_when(@item)
  end
end

# fresh_when(@item)
# -> headers['ETag'] = Digest::MD5.hexdigest(@item.cache_key)
# -> <model name>/ <id>-<updated_at>
```
這樣一來就只要 `id` 和 `update_at` 兩個就可以知道到底整個頁面有沒有東西變動過，除此之外 fresh_when 也可接受 multiple arguments 像是 `fresh_when([@item, current_user.id])` 因為畢竟有時候我們還是需要定義得更明確，但是可能一個 controller 裡面我們有好幾個 action 都會需要放進去，就會有重複的 code 出現，所以可以這麼寫

```ruby
class ItemsController < ApplicationController
  etag { current_user.id }
  etag { current_user.age }

  def show
    @item = Item.find(params[:id])
    fresh_when(@item)
    # => fresh_when([@item, current_user.id, current_user.age])
  end
end
```
