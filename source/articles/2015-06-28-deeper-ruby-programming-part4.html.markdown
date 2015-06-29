---
title: Deeper Ruby Programming (4)
date: 2015-06-28 06:00 UTC
tags: ruby
layout: post
---
> Choose a job you love and you will never have to work a day of your life. - Confucius

#### Module
接續 Class 之後要來談談 Module，Module 的角色很特別，跟 Class 全然不同，首先 Module 不能被繼承，像是這樣，

```ruby
module Speaker
  def quack
    ...
  end
end

class MakeSound < Speaker
  ...
end
```
這樣是會出事的，會出現 `superclass must be a Class` 這樣的錯誤訊息，除此之外也不能被 new 出物件來，因為在 Module 的原始方法就沒有定義 new 的方法，那麼通常 Module 使用的方法是什麼呢？

```ruby
class Post
  def share_on_facebook
  end
end

class Image
  def share_on_facebook
  end
end

class Tweet
  def share_on_facebook
  end
end
```
這三個 class 都具有相同的方法，這樣的寫法就有太多的重複，看起來就不夠 DRY，這時候有兩個選擇，一是將這些相同的行為都拉出來，自成一個 class，再讓這些 class 繼承，但是這樣的作法可能會有潛在的風險。

首先，在 Ruby 的世界裡，所有的 class 都只能繼承自單一 class，所以如果繼承了這個自成的 class 會限制了這個 class 之後的繼承，再來就是如果在繼承的 class 之中的方法並不適用於上述其中一個 class 那就必須要再額外去處理這個例外狀況，這樣並不是個好的做法，所以在這個時候使用 Module 就是很好的時機，

```ruby
class Post
  include Shareable
  include Favoritable
end

class Image
  include Shareable
  include Favoritable
end

class Tweet
  include Shareable
end

########################

module Shareable
  def share_on_facebook
  end
end

module Favoritable
  def add_to_delicious
  end
end
```
Module 的原理我們可以用瀏覽器來類比一下，Module 就像是我們在瀏覽器上面安裝的 plug-in，但是我和你的瀏覽器安裝的 plug-in 可能不一定會相同，因為需求不見得相同，所以這就可以寫出相當優雅的做法，來分類到底該 class 真正需要的 Module 是什麼？

那麼到底 class 是怎麼使用 Module 的呢？在我們上面的原始碼可以看見用 `include` 來外掛 Module，除此之外還可以使用 `extend` 來外掛 Module，那這兩者之間的差別在哪裡呢？我們從程式碼來看兩者之間的不同，

```ruby
class Tweet
  extend Searchable
end

Tweet.find_all_from('@wellsonjain')

######################################

class Image
  include ImageUtils
end

image = user.image
image.preview
```
`extend` 就是讓 Module 的 method 定義為 class method，而 `include` 則是 instance method。

再深入一點來談 Module 的 Hook，如果說我們在 Module 上面做一點變化，像是在 Module 裡面再掛一個 Module 這是可行的，也就是說也可以這樣寫

```ruby
module ImageUtils
  def preview
    puts "I am previewing."
  end

  def transfer(destination)
  end

  module Twitter
    def fetch_from_twitter(user)
      puts "I am #{user}."
    end
  end
end

class MyImage
  include ImageUtils
  extend ImageUtils::Twitter
end
```
我們在 class 裏頭用了 extend 來把 Twitter 這個 module 給外掛成 class method，當然我們也可以讓 class 只要 `include ImageUtils` 就可以達到一樣的效果，但是那就必須要在 module 裡面動一點手腳，

```ruby
module ImageUtils

  def self.included(base)
    base.extend(Twitter)
  end

  def preview
    puts "I am previewing."
  end

  def transfer(destination)
  end

  module Twitter
    def fetch_from_twitter(user)
      puts "I am #{user}."
    end
  end
end

class MyImage
  include ImageUtils
end
```
`self.included` 會在 module 被包進去一個 class 的時候被 Ruby 呼叫起來，base 就是指這個被包進去的 class，就可以直接的增強這個 Module 的功能。
