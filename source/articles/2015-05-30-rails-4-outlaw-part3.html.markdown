---
title: Rails4 Outlaw (3)
date: 2015-05-30 03:45 UTC
tags: rails
layout: post
---
> One man or woman with courage is a majority. - Andrew Jackson

#### Strong Parameters
Stong parameters 是在 rails4 關注比較大的更新，在一般 form 的 submit 後，可能會有不懷好意的使用者夾帶送出不應該有的多餘參數，雖然說在 rails3 有 `attr_accessible` 這個白名單的設計，但是如果發生了夾帶多餘參數的事情時，會造成系統出現 `Can't mass-assign protected attributes:` 這樣的錯誤，所以在 rails4 就做出了 strong parameter 的更新。

先前白名單會是在 Model 裡面被定義，而現在則是拉到 Controller 裡面去做，

```ruby
  user_params = params.require(:user).permit(:name)
```
require 的意思就是，如果我們仔細去看 params 裡面所帶的參數，必須一定要有 require 這個 key 值，如果沒有則一樣會出現錯誤 `ActionController::ParameterMissing: param not found: user`。

而 permit 的功能就是白名單的角色，如果說在白名單裡面沒有的參數或是參數的型態不正確就會被濾掉，而在 default 的設定中這些被濾掉的參數還是會被記錄在 log 裡面，如果覺得還是覺得需要產生例外錯誤訊息，也可以在 config 裡面做設定，

```ruby
# config/application.rb
config.action_controller.action_on_unpermitted_parameters = :raise
```
至於如果還是想在 model 裡面設定白名單的話，則是要可以加一個 gem `protected_attributes`

#### Authenticity Token
在 Rails 中，非 GET 的 HTTP request 都會要求帶一個 Token 才能夠存取，像是 form 表單就會自動生成一個 token，但是在 rails4 如果是用 ajax 的形式，則預設會把這個地方給取消，因為我們可以在 layout 的地方就定義 <%= csrf_meta_tag %>，就是讓 javascript 來讀取驗證碼用的，所以當我們在做 ajax 的時候，其實 js 已經在 layout 找到 token 所以在 rails4 就把它給簡化了。

但也是有可能發生 client 端並不支援 Javascript，這時候就無法驗證 token 的情況發生，這時候我們只要做，

```erb
<%= form_for @some_model, :remote => true, :authenticity_token => true do |f| %>
<% end %>
```
雖然我們是有需要驗證 token，不過如果在開放 API 的時候我們可能會需要取消它，所以可以做像是，

```ruby
class ApplicationController < ActionController::Base
  protect_from_forgery
  skip_before_action :verify_authenticity_token, if: :json_request?

  protected

  def json_request?
    request.format.json?
  end
end
```
關於在 Application controller 我們在驗證 token 的時候，能夠有幾種工具來做

```ruby
# 會出現 exception: ActionController::InvalidAuthenticityToken
protect_from_forgery with: :exception

# Empty session
protect_from_forgery with: :null_session

# New session (並且把舊的清掉)
protect_from_forgery with: :reset_session

```
