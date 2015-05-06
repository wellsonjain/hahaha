---
title: Rails API (1)
date: 2015-05-05 09:03 UTC
tags: rails
layout: post
---

> You take your life in your own hands, and what happens? A terrible thing, no one to blame. - Erica Jong

對於 API 的製作一直是我的軟肋，不對啊事實上什麼都是軟肋啊!!!(抱頭哭喊) 不過這樣一直懞懞懂懂下去好像也不是辦法，最近在看 Code School 的 Rails API 課程覺得收穫良多，於是來記錄一下幾個蠻重要的部分，但是由於這個課程屬於比較進階的，所以會以一邊寫 Testing 一邊開發程式的方式，如果說對於寫 Testing 還不是這麼熟悉的話恐怕會沒辦法理解的很好 (像我本人當初沒寫過測試就來看這個，結果看了一兩個月還是看不懂.....)

故事的開始是由介紹 REST API 開始，當初剛開始接數 [REST](http://zh.wikipedia.org/zh-tw/REST) 這個名詞的時候也是一頭霧水，而裡頭給了一個相當簡潔的定義，

>A set of <font color= blue>commands</font> performed on <font color=orange>things</font> generates <font color="green">responses.</font>

並且以這個為中心思想來延續接下來的討論。

先來解釋一下為什麼會有這樣的定義，首先我們先從 Website 這種 Client Server 架構來理解，假設我在 Spotify 上面 <font color= blue>Create</font> 我的播放清單 <font color=orange>Wellson's Collection</font>，Spotify <font color="green">回應並成功建立播放清單</font>，這樣的一連串的行為即是 REST，有了這樣的理解之後我們便可以開始 API 之旅。

####Rails Routes
在 Rails project 中，routes 等於是整個網站的地圖，經由 routes 來指引我們所要 access 的 resource 在哪裡，以及我們能夠對他們做哪些 access，

````ruby
# config/routes.rb
  resouces :recipes
````
接著我們在終端機執行 `rake routes`

```shell
      recipes GET    /recipes(.:format)             recipes#index
              POST   /recipes(.:format)             recipes#create
   new_recipe GET    /recipes/new(.:format)         recipes#new
  edit_recipe GET    /recipes/:id/edit(.:format)    recipes#edit
       recipe GET    /recipes/:id(.:format)         recipes#show
              PATCH  /recipes/:id(.:format)         recipes#update
              PUT    /recipes/:id(.:format)         recipes#update
              DELETE /recipes/:id(.:format)         recipes#destroy
```
但是在設計 API 的時候不見得每一條路都給使用，有些只給新增，有些只給刪除，

````ruby
# config/routes.rb
  resources :recipes, except: [:edit, :update, :destroy]
  resources :chefs,   only:   [:index, :show]
````
上述的方式就可以限制，哪些能用哪些不能用，而且在 Rails 上這樣的寫法是相當直觀，可以馬上就知道哪些是白名單，哪些是黑名單。

#### Constraints and Subdomain
通常 API 的設計會與主網站的架構分開，所以會切出 subdomain 來達到 `load balancing` 的效果，而在 Rails 我們就可以用 constraints 來達到這樣的效果。

````ruby
# config/routes.rb
  constraints subdomain: 'api' do
    resources :recipes
    resources :chefs
  end

# 則會呈現這樣的結果
# http://api.myrecipes.com/recipes
# http://api.myrecipes.com/chefs
````

#### Namesapce
在開發上面，在程式上面因為我們會切開 API 與 Website 不同的 controller，所以這樣就有必要分開 folder 來進行管理，而在 Rails 裡面我們使用 namespace，

````ruby
# config/routes.rb
  constraints subdomain: 'api' do
    namespace :api do
      resources :recipes
    end
  end
````
````ruby
# app/controllers/api/recipes_controller.rb
module Api
  class RecipesController < ApplicationController
  end
end
````
可是這樣會造成 URL 上的重複，URL 顯示為 http://<font color=red>api</font>.myrecipe.com/<font color=red>api</font>/recipes，解決方法為在 namespace 後面加上 path

````ruby
constraints subdomain: 'api' do
  namespace :api, path: '/' do
     resources :recipes
  end
end
````
再額外提一下，我們都知道 API 就是 Application Programming Interface 的簡稱，但是因為受限於 Rails 命名規則的關係我們並不能直接在 module 裡面這樣宣告，必須在 `config/initializers/inflections.rb` 新增例外，

````ruby
# config/initializers/inflections.rb
ActiveSupport::Inflector.inflections(:en) do |inflect|
￼ inflect.acronym 'API'
end
````
於是我們便可以把 module 改為，

````ruby
# app/controllers/api/recipes_controller.rb
module API
  class RecipesController < ApplicationController
  end
end
````
這樣我們就已經規劃出很簡潔的 flow，下一步就是依據需求來開發功能。
