---
title: Rails API (5)
date: 2015-05-07 05:15 UTC
tags: rails
layout: post
---

#### API Versioning
隨著時間的推進，系統架構也會逐漸地成長，所以可能我們在新增一些功能的時候給新客戶的時候，會對於舊客戶有所影響，這時候比較好的做法就是把兩者切開，舊客戶使用 v1，新客戶使用 v2，這時候 `namespace` 就剛好能夠解決這個問題

````ruby
# config/routes.rb
namespace :v1 do
  resources :recipes    # http://apo.myrecipe.com/v1/recipes
end

namespace :v2 do
  resources :recipes    # http://apo.myrecipe.com/v2/recipes
end
````
````
$ rake routes

          Prefix  Verb    URI Pattern               Controller#Action
  api_v1_recipes  GET     /v1/recipes(.:format)     v1/recipes#index
                  POST    /v1/recipes(.:format)     v1/recipes#create
                  ...
  api_v2_recipes  GET     /v2/recipes(.:format)     v2/recipes#index
                  POST    /v2/recipes(.:format)     v2/recipes#create
                  ...
````
接著一樣藉由 test 來確定有被導向至正確的 controller#action

````ruby
# test/integration/routes_test.rb

class RoutesTest < ActionDispatch::IntegrationTest
   test 'routes version' do
     assert_generates '/v1/recipes', { controller: 'v1/recipes', action: 'index' }
     assert_generates '/v2/recipes', { controller: 'v2/recipes', action: 'index' }
   end
end
````
````ruby
# app/controllers/v1/recipes_controller.rb
module V1
  class RecipesController < ApplicationController
  end
end

# app/controllers/v2/recipes_controller.rb
module V2
  class RecipesController < ApplicationController
  end
end
````
如此一來我們就可以設定出不同版本的 API，但是如果兩個 API 都可能同時有相同的 method 要做，可能兩個都會先有個 before_action 是要先找出是台灣菜的食譜，那麼如果都要分別在每個版本的 controller 內加上去，這樣如果之後要找的是廚師而不是食譜這樣就要改兩個不同的地方，可能一不小心就忘記了，所以更好的方法就是在他們所繼承的 ApplicationController 內加上去

> 在這裡所提供的這個方法只限定現在的情況是單純做 Web API，而沒有 Website 的功能

````ruby
# test/integration/changing_api_versions_test.rb

class ChangingApiVersionsTest < ActionDispatch::IntegrationTest

  setup { @ip = '123.123.12.12' }

  test '/v1 returns version 1' do
    get '/v1/recipes', {}, { 'REMOTE_ADDR' => @ip }
    assert_equal 200, response.status
    assert_equal "#{@ip} Version One!", response.body
￼￼end

￼ test '/v2 returns version 2' do
    get '/v2/recipes', {}, { 'REMOTE_ADDR' => @ip }
    assert_equal 200, response.status
    assert_equal "#{@ip} Version Two!", response.body
￼￼end
end
````
````ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  before_action ->{ @remote_ip = request.headers['REMOTE_ADDR'] }
end
````
上面的情況是要將兩個相同的合併在一起，但是如果是只限定於某一個版本的呢？在 Rails 使用了一個特別的 method 叫做 `abstract!`

````ruby
# app/controllers/v2/version_controller.rb
module V2
  class VersionController < ApplicationController
    abstract!

    before_action :best_chef_for_v2

    def best_chef_for_v2
  end
end
````
我們讓他直接繼承 Version Controller，

````ruby
# app/controllers/v2/recipes_controller.rb

module V2
  class RecipesController < VersionController
  end
end
````
````ruby
# app/controllers/v2/ingredients_controller.rb

module V2
  class IngredientsController < VersionController
  end
end
````
我們就可以藉此解決版控上的問題。
