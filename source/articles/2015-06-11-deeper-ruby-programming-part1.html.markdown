---
title: Deeper Ruby Programming (1)
date: 2015-06-11 13:17 UTC
tags: ruby
layout: post
---
> The secret of change is to focus all of your energy, not on fighting the old but on building the new. - Unknown

Rails 要學得好，Ruby 才是 Rails 開發者的根基，而且其實有很多用 Ruby 寫成的應用也常常讓我覺得很有趣，只是有很多比較深入的用法如果平常沒有接觸，有些技巧性的寫法就會覺得實在跟魔法沒兩樣，但是又很難參透其中的道理，所以今天要來談的就是比較深入一點的 Ruby 奇淫技巧。

#### Control flow
先來點簡單的暖身一下，大家知道在 ruby 的 Control flow (if-else) 裏頭 `0`, `""`, `[]` 這三個我們以為被當作 false 的 condition 事實上都是 true，`nil` 才是 false，於是乎像是，

```ruby
if not name.length
  warn "Name is required"
end
```
裏頭的警告訊息是不會發生的，而且事實上在判斷是裡面也只有一個操作，其實可以寫得更精簡，

```ruby
warn "Name is too short" if not name > 8
```
同樣在判斷式的精簡上面，如果條件允許，也可以把兩個 if 用 && 合併起來

```ruby
if user && user.sign_in?
  ...
end
```
這樣當第一個為 false 就不會再去做第二個判斷處理。

#### Short Circuit
接下來我們再更深入的討論 controll flow，其中的 `||` 代表邏輯閘中的 OR，在 ruby 的邏輯判斷中，只要有一者為真就為真，如果賦值的話，前者有值則 assign 前者的值，如無則 assign 後者的值，用解釋好像有點饒口，不過這樣就可以做一些很棒的設計，

```ruby
# 如果這個 chef.recipe 為空，則回傳給他一個空陣列
recipe = chef.recipe || []
```
或者可以給予初始值，

```ruby
# 這樣寫雖然很易讀，但是這就是要給予一個預設值，我們可以寫得更優雅
options[:country] = 'us' if options[:country].nil?
options[:privacy] = true if options[:privacy].nil?
options[:geotag]  = true if options[:geotag].nil?

########################################################

options[:country] ||= 'us'
options[:privacy] ||= true
options[:geotag]  ||= true
```
`||=` 就像是 `+=` 一樣，`a = a || b` 的意思，剛開始看的時候還想說這到底為什麼可以這樣寫，聯想力不夠有點哀傷

#### Condition Return Value
這也是有點神奇的語法，就是 assign if statement 的值，

```ruby
option[:path] = if last_name
  "/#{user_name}/#{list_name}"
else
  "/#{user_name}"
end
```
其實這是因為，ruby 會 return 最後一行的值，所以在最後一行 assign 值再回傳其實沒有必要，所以也可以在 method 裡面寫這樣，其實不只是 if statement，像是 case when 這種流程控制都是相同的道理，

```ruby
def list_url(user_name, list_name)
  if list_name
    "https://twitter.com/#{user_name}/#{list_name}"
  else
    "https://twitter.com/#{user_name}"
  end
end
```
