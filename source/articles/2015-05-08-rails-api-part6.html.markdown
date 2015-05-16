---
title: Rails API (6)
date: 2015-05-08 01:31 UTC
tags: rails
layout: post
---

> Everything around you that you call life was made up by people, and you can change it - Steve Jobs

這是這個 API 系列文的最後一篇，最後要談的是 Authentication，前一陣子有機會接觸 [Doorkeeper](https://github.com/doorkeeper-gem/doorkeeper) 覺得 Oauth 這根本是外星知識阿!! 而且就算看了網路上以鴨七介紹的最為仔細的[系列文](http://blog.yorkxin.org/posts/2013/09/30/oauth2-1-introduction/)，依然看不懂，不過真的是自己的基礎不夠，所以之後會再一次的深入研究，會再記錄下來。

回到主題，當我們設計 API 的時候當然不一定會希望所有人都可以進來存取資料，這樣可能會導致重要的資料被更改，所以我們會 `進行一個加密的動作`，也就是我把它鎖起來，沒有鑰匙就當然打不開，在這邊會介紹兩種 Authentication 的方式

- Basic Auth
- Token Base Auth

首先是 Basic Auth，在介紹 Auth 之前我們先要了解 Credentials(憑證) 也就是所謂的鑰匙，必須在使用 Authorization Header 的 Http request 裡被提供，像是這樣，

```
  GET /Recipes HTTP/1.1
  Host: localhost:3000
  Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```
在 Basic Auth 裡所使用的加密技術是 `Base64 encoded`，我們可以直接在 `irb` 裡面使用 library，

```
  $ irb
  $ irb > require 'base64'
    => true
  $ irb > Base64.encode64('foo:secret')
    => "Zm9vOnNlY3JldA==\n"
```
接下來我們來寫一段測試

````ruby
# test/integration/listing_recipes.rb
class ListingRecipesTest < ActionDispatch::IntegrationTest !
  setup { @user = User.create!(username: 'foo', password: 'secret') }

  test 'valid username and password' do
    get '/recipes', {},'Authorization' => 'Basic Zm9vOnNlY3JldA=='}
    assert_equal 200, response.status
  end

  test 'missing credentials' do
    get '/recipes', {}, {}
    assert_equal 401, response.status
  end
end
````
````ruby
class RecipesController < ApplicationController
  before_action :authenticate
  def index
    recipes = Recipe.all
    render json: recipes, status: 200
  end

  private
    def authenticate
      authenticate_or_request_with_http_basic do |username, password|
        User.authenticate(username, password)
      end
    end
end
````
上面的測試這樣寫其實有點醜，所以我們其實可以把加密的部分放到 test_helper 裡面，

````ruby
# test/test_helper.rb
ENV["RAILS_ENV"] ||= "test"
require File.expand_path('../../config/environment', __FILE__)
require 'rails/test_help'

class ActiveSupport::TestCase
  ActiveRecord::Migration.check_pending!
  fixtures :all

  def encode_credentials(username, password)
    ActionController::HttpAuthentication::Basic.encode_credentials(username, password)
  end
end
````
剛剛的測試我們就可以寫成這樣，

````ruby
# test/integration/listing_recipes.rb
class ListingRecipesTest < ActionDispatch::IntegrationTest !
  setup { @user = User.create!(username: 'foo', password: 'secret') }

  test 'valid username and password' do
    get '/recipes', {},'Authorization' => encode_credentials(@user.username, @user.password)}
    assert_equal 200, response.status
  end
  ...
end
````
如果沒有 Authorized，HTTP response head 將會有 `WWW-Authenticate`

````
HTTP/1.1 401 Unauthorized 
WWW-Authenticate: Basic realm="Application"
Content-Type: text/html; charset=utf-8
````
> Basic 表示為使用 HTTP Basic Authentication

> realm = "Application" 表示我們的 resource 是在 Application 這個範圍內的

`Realm` 很有意思，他可以讓我們去切割出不同的被保護區域，所以就可以設計出一套個人化的授權，在我們前面所使用的 helper `authenticate_or_request_with_http_basic` 事實上就有可以帶入 Realm 的參數，

```ruby
authenticate_or_request_with_http_basic('Recipes') do |username, password|
  User.authenticate(username, password)
end
```
但是使用 `authenticate_or_request_with_http_basic` 的話，Response 都會是 `text/html`，可是我們設計的 API，應該預期所有的回應都必須要是 json，所以我們可以使用另外一個 helper

```ruby
class RecipesController < ApplicationController
  before_action :authenticate
  ...
  protected
    def authenticate
      authenticate_basic_auth || render_unauthorized
    end

    def authenticate_basic_auth
      authenticate_with_http_basic do |username, password|
        User.authenticate(username, password)
      end
    end

    def render_unauthorized
      self.headers['WWW-Authenticate'] = 'Basic realm="Recipes"'

      respond_to do |format|
        format.json { render json: 'Bad Credential', status: 401 }
        format.xml { render xml: 'Bad Credential', status: 401 }
      end
    end
end
```
但是，事實上 Basic Auth 並不安全，因為我們只要用 Sniffer 就可以竊取到憑證，只要在用 Ruby 內建的 Decode 就可以得到權限進行存取，所以接下來，要分享的是另外一種 `Token Base Auth`

基本上 Token Base Auth 在實作上與 Basic Auth 其實差不多，但是 Token Base 卻有了更多好處

- 更加方便，我們可以在不影響 user account 的狀況下，expire 或是重新 generate 新的 token。
- 更加安全，因為並不影響 user account，就算受到影響也僅限於 API 的存取。
- 一個 user 可以有多個 token，可以在多個不同的 API 取得權限存取。
- 每個 token 就可以有更好的控制，可以有不同的規則可以設計。

```
GET /recipes HTTP/1.1
Host: localhost:3000
Authorization: Token token=16d7d6089b8fe0c5e19bfe10bb156832
```
上面就是使用 Token base 的 Header

接下來我們一樣繼續寫測試，

```ruby
# test/integration/listing_recipes_test.rb

class ListingRecipesTest < ActionDispatch::IntegrationTest
  setup do
    @user = User.create!
  end

  def token_header(token)
    ActionController::HttpAuthentication::Token.encode_credentials(token)
  end

  test 'valid authentication with token' do
    get '/recipes', {}, { 'Authorization' => token_header(@user.auth_token)}
    assert_equal 200, response.status
    assert_equal Mime::JSON, response.content_type
  end

  test 'invalid authentication' do
    get '/episodes', {}, { 'Authorization' => @auth_header + 'fake' }
    assert_equal 401, response.status
  end
end
```
```ruby
# app/models/user.rb
class User < ActiveRecord::Base
  before_create :set_auth_token

  private

    def set_auth_token
      return if auth_token.present?
      self.auth_token = generate_auth_token
    end

    def generate_auth_token
      loop do
        token = SecureRandom.hex
        break if not self.class.exists?(auth_token: token)
      end
    end
end
```
在 Rails 裡面當然也有提供 token 的 method `authenticate_or_request_with_http_token`，但是與之前一樣的問題，這個 method 最後只會 response `text/html`，所以我們一樣將它換成

```ruby
class RecipesController < ApplicationController
   before_action :authenticate
   ...
   protected
      def authenticate
        authenticate_token || render_unauthorized
      end

      def authenticate_token
        authenticate_with_http_token do |token, options|
          User.find_by(auth_token: token)
        end
      end

      def render_unauthorized
        self.headers['WWW-Authenticate'] = 'Token realm="Recipes"'

        respond_to |format|
          format.json { render json: "Bad credencial", status: 401}
          format.xml { render xml: "Bad credencial", status: 401}
        end
      end
```
這樣一個具有基本架構的 API 就算是完成了!! 這個系列文也總算是告個段落了...已累...
