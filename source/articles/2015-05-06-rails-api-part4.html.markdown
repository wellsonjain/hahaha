---
title: Rails API (4)
date: 2015-05-06 13:07 UTC
tags: rails
layout: post
---

> Look up at the stars and not down at your feet. Try to make sense of what you see, and wonder about what makes the universe exist. Be curious. - Stephen Hawking

#### POST
談完了如何讀取資料，接下來談的是如何新增資料，在 HTTP verb 當中，新增資料所使用的是 POST，而新增成功的 status code 201 - Created (GET 是 200)，接下來我們一樣來寫個測試，

````ruby
# test/integration/creating_recipes_test.rb

class CreatingRecipesTest < ActionDispatch::IntegrationTest
  test 'create recipe' do
    post '/recipes',
      { recipe:
        { title: 'Stinky tofu', category: 'taiwanese', description: 'really stinky' }
      }.to_json,
      { 'Accept' => Mime::JSON, 'Content_Type' => Mime::JSON.to_s}

    assert_equal 201, response.status
    assert_equal Mime::JSON, response.content_type

    recipe = json(response.body)
    # 可以藉由 location 來對照是否新增了該 resource
    assert_equal recipe_url(recipe[:id]), response.location
  end
end
````
````ruby
# controller/recipes_controller.rb

class RecipesController < ApplicationController

  def create
    @recipe = Recipe.new(recipe_params)
    if @recipe.save
      render json: recipe, status: 201, location: recipe
    end
  end
  private

    def recipe_params
      params.require(:recipe).permit(:title, :description, :category)
    end
end
````
這樣測試應該就能通過了，接著我們一樣使用 `curl` 來驗證(根本神兵利器)，

````
$ curl -i -X POST -d 'recipe[title]=StinkyTofu' http://localhost:3000/recipes
````
但是卻會出現 `422 Unprocessable Entity` 這樣的結果，而不是我們預期的結果，這是因為 Rails 在 POST, PUT, PATCH, DELETE 這幾個 verb 會需要 check 是否有 `authenticity token`，來防禦 [CSRF](http://zh.wikipedia.org/zh-tw/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0) 但是因為透過 api 我們並不會有表單的傳送，所以我們可以在 `/controller/application.rb` 進行下面的修改，

````ruby
# app/controllers/application_controller.rb

class ApplicationController < ActionController::Base
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :null_session
end
````
這樣就可以成功地出現 201，但是有時候我們新增 Resource 的時候不見得需要 response body，這樣可以增加速度，這時候就可以這樣設計，

````ruby
# controller/recipes_controller.rb

class RecipesController < ApplicationController

  def create
    @recipe = Recipe.new(recipe_params)
    if @recipe.save
      render nothing: true , status: 204, location: recipe
    end
  end
  ...
end
````
#### PUT and PATCH
新增結束之後就要提到更新，在 HTTP 這兩個 Verb，所使用的 method 都是 update method，但是它們究竟有甚麼分別呢？ iHower 在 [這篇文章](https://ihower.tw/blog/archives/6483) 有相當清楚的解釋；我們可能一筆資料有十個欄位，甚至更多，但是如果只是要更新其中一個欄位，使用 PUT 卻要全部更新一次，那就可能會多浪費一點時間，所以便改成 PATCH 只是做部分更新，而且在文中提到 idempotent 我覺得蠻重要的，順便記錄一下，

|          | Safe                        | Idempotent                  |
| -------- | --------------------------- | --------------------------- |
| GET      | <i class="fa fa-check"></i> | <i class="fa fa-check"></i> |
| POST     | <i class="fa fa-times"></i> | <i class="fa fa-times"></i> |
| PATCH    | <i class="fa fa-times"></i> | <i class="fa fa-times"></i> |
| PUT      | <i class="fa fa-times"></i> | <i class="fa fa-check"></i> |
| DELETE   | <i class="fa fa-times"></i> | <i class="fa fa-check"></i> |

> Safe 特性會影響是否可以快取 (POST/PUT/PATCH/DELETE 一定都不可以快取)
 Idempotent 特性則是會影響可否 Retry，而且結果相同

接下來就來新增 Update 的測試

````ruby
# test/integration/updating_recipes_test.rb

class UpdatingRecipesTest < ActionDispatch::IntegrationTest
  setup { @recipe = Recipe.create!(title: 'Stinky Tofu')}

  test 'successful update' do
    patch '/recipes/#{@recipe.id}',
      { recipe: {title: 'Really Stinky Tofu' } }.to_json,
      { 'Accept' => Mime::JSON, 'Content-Type' => Mime::JSON.to_s }

    assert_equal 200, response.status
    assert_equal 'Really Stinky Tofu', @recipe.reload.title
  end
end
````
````ruby
# app/controllers/recipes_controller.rb

class RecipesController < ApplicationController

  def update
    recipe = Recipe.find(params[:id])
    if recipe.update(recipe_params)
      render json: recipe, status: 200
    else
      render json: recipe.errors, status: 422
    end
  end

  private
    def recipe_params
      params.require(:recipe).premit(:title, :category, :description)
    end
end
````

#### Delete
最後是如何刪除資料，說到刪除資料有兩種方式可以做

- 人間蒸發
- 藏起來讓你看不到

第一種方式有點殘忍，我們先寫測試程式來測試看看怎麼蒸發(嗯?)

````ruby
# test/integration/deleting_recipes_test.rb

class DeletingRecipesTest < ActionDispatch::IntegrationTest
  setup { @recipe = Recipe.create(title: 'Disgusting food') }

  test 'deletes existing recipe' do
    delete "/recipes/#{@recipe.id}"
    assert_equal 204, response.status
  end
end
````
````ruby
# app/controllers/recipes_controller.rb

class RecipesController < ApplicationController

  def destroy
    recipe = Recipe.find(params[:id])
    recipe.destory
    head 204
  end
end
````
上面的這種方式是讓該筆資料直接從資料庫消失，而接下來是第二種的方式，設定個 Flag 讓他並不是直接從資料庫被刪除，而只是狀態是看不到，

````ruby
# app/models/recipe.rb

class Recipe < ActiveRecord::Base
  def self.find_unarchived(id)
    find_by!(id: id, archived: false)
  end

  def archive
    self.archived = true
    self.save
  end
````
````ruby
# app/controllers/recipes_controller.rb

class RecipesController < ApplicationController

  def destroy
    recipe = Recipe.find(params[:id])
    recipe.archive
    head 204
  end
end
````
這樣 API 功能的新增，修改，刪除就完成了。
