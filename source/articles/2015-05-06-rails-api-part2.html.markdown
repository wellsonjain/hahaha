---
title: Rails API (2)
date: 2015-05-06 06:47 UTC
tags: rails
layout: post
---

> So many of our dreams at first seem impossible. Then they seem improbable. And then, when we summon the will, they soon become inevitable. - Nelson Mandela

#### Resource
在上一篇裡面使用到 Resouces，在 Rails 裡面 Resource 我覺得是蠻神奇的，這樣就可以自動產生出預設的 HTTP verb，可是到底 Resource 是什麼？其實 Resource 是一種概念性的東西

- 播放清單
- 一道菜的食譜
- 一首歌

這些都可以在 Website 裡被視作 Resource，也就是我們在上一篇所提到的 <font color=orange>things</font> 然後我們再來想要對這些 Resource 做什麼動作。

#### Get method
首先先來談談該"怎麼進行一個讀取的動作"，在 HTTP verb 裡面就是 GET，

````ruby
# config/routes.rb
namespace :api, path: '/', constraint: { subdomain: 'api' } do
  resources :recipes
end
````
````ruby
# test/integration/listing_recipes_test.rb
require 'test_helper'

class ListingRecipesTest < ActionDispatch::IntegrationTest
  setup { host! 'api.example.com' }
  test 'return list of all recipes' do
    get '/recipes'
    assert_equal 200, response.status  # 200 - Success status
    refute_empty response.body
  end
end
````
> 講個秘訣：在進行開發之前都先從寫一個測試開始，以確定程式執行的結果是正確的。

````ruby
# app/controllers/api/recipes_controller.rb
module API￼
  class RecipesController < ApplicationController
    def index
      recipes = Recipe.all
      render json: recipes, status: 200
    end
  end
end
````
接下來再繼續做點變化，有時候我們不會只是要找出所有的資料，可能會給一些關鍵字然後再從中去過濾出比較少量的資料，在 API 的使用上面我們可能會在網址後面增加參數，像是 `/recipes?category=taiwanese`，在`?`後面的都是一個參數，並且帶過去給該 action 再做後續的處理，接著我們再繼續新增測試範圍。

````ruby
# test/integration/listing_recipes_test.rb
require 'test_helper'

class ListingRecipesTest < ActionDispatch::IntegrationTest
  setup { host! 'api.example.com' }

  test 'return list of all recipes' do
    get '/recipes'
    assert_equal 200, response.status  # 200 - Success status
    refute_empty response.body
  end

  test 'return recipes filtered by category'
    taiwan_food = Recipe.create!(title: "Stinky tofu", category: "taiwanese")
    japan_food = Recipe.create!(title: "Sushi", category: "japanese")

    get 'recipese?category=taiwanese'
    assert_equal 200, response.status

    #parse response 回來的 content, symbolize_name 則會將字串 symbolize
    recipes = JSON.parse(response.body, symbolize_name: true)
    title = recipes.collect { |recipe| recipe[:title] }
    assert_includes title, 'Stinky tofu'
    refute_includes title, 'Sushi'
  end
end
````
````ruby
# app/controllers/api/recipes_controller.rb
module API￼
  class RecipesController < ApplicationController
    def index
      recipes = Recipe.all
      if params[:category]
        recipes = recipes.where(category: params[:category])
      end
      render json: recipes, status: 200
    end
  end
end
````
如果想再實際測試一下的話我們可以使用 `curl` 這個指令來做，

```
$ curl localhost:3000/recipes
```
就可以看到 request 回來的 json。
