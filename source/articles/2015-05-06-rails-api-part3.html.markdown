---
title: Rails API (3)
date: 2015-05-06 11:59 UTC
tags: rails
layout: post
---

> There is nothing impossible to him who will try. - Alexander the Great

#### Content Negotiation
API 基本上會介接許多種裝置，可能是電腦也可能是手機，因為無法預期是由誰來呼叫，所以必須先設計好要回丟什麼樣格式的資料，在 Rails 便提供了這樣的功能，

````ruby
# config/routes.rb
  resouces :recipes

# http://api.myrecipe.com/recipes.json
# http://api.myrecipe.com/recipes.xml
````
````
      recipes GET    /recipes(.:format)             recipes#index
              POST   /recipes(.:format)             recipes#create
   new_recipe GET    /recipes/new(.:format)         recipes#new
  edit_recipe GET    /recipes/:id/edit(.:format)    recipes#edit
       recipe GET    /recipes/:id(.:format)         recipes#show
              PATCH  /recipes/:id(.:format)         recipes#update
              PUT    /recipes/:id(.:format)         recipes#update
              DELETE /recipes/:id(.:format)         recipes#destroy
````
接著一樣繼續寫測試程式，

````ruby
# test/integration/listing_recipes_test.rb
class ListingZombiesTest < ActionDispatch::IntegrationTest
  test 'returns recipes in JSON' do
    get '/recipes', {}, { 'Accept' => Mime::JSON }
    assert_equal 200, response.status
    assert_equal Mime::JSON, response.content_type
  end

  test 'returns recipes in XML' do
    get '/recipes', {}, { 'Accept' => Mime::XML }
    assert_equal 200, response.status
    assert_equal Mime::XML, response.content_type
  end
end
````
````ruby
# app/controllers/api/recipes_controller.rb
module API￼
  class RecipesController < ApplicationController
    def index
      recipes = Recipe.all

      respond_to do |format|
        format.html
        format.json { render json: recipes, status: 200 }
        format.xml  { render xml: recipes, status: 200 }
      end
    end
  end
end
````
藉由 Rails 的 helper method `respond_to`，可以產生出很多種不同的格式，Rails 會根據 request 的不同而選擇不同的格式，我們可以藉由上一篇提到的 `curl` 指令來測試

````
$ curl -IH "Accept: application/json"  http://localhost:3000/recipes

HTTP/1.1 200 OK
X-Frame-Options: SAMEORIGIN
X-Xss-Protection: 1; mode=block
X-Content-Type-Options: nosniff
Content-Type: application/json; charset=utf-8

$ curl -IH "Accept: application/xml"  http://localhost:3000/recipes

HTTP/1.1 200 OK
X-Frame-Options: SAMEORIGIN
X-Xss-Protection: 1; mode=block
X-Content-Type-Options: nosniff
Content-Type: application/xml; charset=utf-8
````
我們可以輕易地看出其中的不同，表示 it works!! (已哭)
