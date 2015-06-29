---
title: Deeper Ruby Programming (2)
date: 2015-06-12 13:07 UTC
tags: ruby
layout: post
---
> Never give up on a dream just because of the time it will take to accomplish it. The time will pass anyway. - Earl Nightingale

#### Optional Arguments
當我們要傳入 method 的 argument 可能不是每個都是必要的，可能有些事是情況傳入的，如果說定義死的話就是必定需要傳入的，那在呼叫 method 的狀況可能是這樣，`method(arg1, nil, nil)`，這樣會對於使用上不是很大的彈性，所以我們可以先行預設給值，像是這樣，

```ruby
def method(arg1, arg2 = nil, arg3 = nil)
  ...
end
```
這樣在呼叫這個方法的時候，其實就可以不必放進去多餘的 argument，`method(arg1)`，這樣就可以了，但是這樣也並不是完全解決問題，如果一個 method 同時需要傳入多個 arguments，像是 `method(arg1, 234, 0.345, 333344)`，不過當然實務上可能不會有這麼極端的狀況 (畢竟這樣寫很不專業啊)，但是這樣畢竟可讀性和彈性都很低，所以再更好一點的做法就是傳入 Hash，

```ruby
def master_recipe(chef, options = {})
  recipe             = Recipe.new
  recipe.chef        = chef
  recipe.ingredients = options[:ingredients]
  recipe.season      = options[:seation]
  recipe.comment_id  = options[:comment_id]
end
```
這樣在傳入 arguments 時，就可以藉由 hash key 來提高可讀性，甚至可以不用擔心參數傳入的先後，而且後續再擴充也很方便，這應該是 method 設計上是比較優雅的做法。

然而在 ruby 2.0 版本以上提供了更加牛逼的方法，你可以使用 `**options` 定義 argument，就會傳入 Hash
，如果是 `*options` 就會傳入一個 Array。

#### Classes
講完 method 的定義我們再往上講一個層級，也就是 Class，讓我們更方便的管理我們物件的行為，有時候有些屬性我們設定為可以被使用者讀寫 (像是使用者姓名) 但是有時候我們又希望可以讓使用者唯讀就可以，在這裡就要介紹一下 `attr_accessor`，這個 method 可以直接為屬性產生 setter 和 getter，

```ruby
attr_accessor :status

# ->
# setter
def status=(value)
  @status = value
end
# getter
def status
  @status
end
```
所以如果在某些的 attribute 我們要做到限制的話，就可以使用 `attr_reader`，就會限制到只有 getter，如果說要使用到 setter 的話就會發生錯誤。

最後來提一下 Self，如果我們在 method 裡面做了 `self.status` 代表的意思就是 current 由該 class new 出來的 instance 的 status，但是如果是像是定義 `def self.method`，那就是 class method 必須由該 class 才能呼叫。
