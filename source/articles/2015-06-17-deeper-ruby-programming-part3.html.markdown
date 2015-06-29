---
title: Deeper Ruby Programming (3)
date: 2015-06-17 00:28 UTC
tags: ruby
layout: post
---
> People will forget what you said, people will forget what you did, but people will never forget how you made them feel. - Maya Angelou

####Class
在 Ruby 這個萬物皆物件的程式語言中，我們實作上當然也最好 follow 物件導向的概念，首先先談談封裝 (Encapsulation)，實務上封裝想要達到的結果就是一個 class 不應該知道太多其他 class 的資訊，或是不要在該物件去做不是屬於他的任務，下面就有幾個必須要考慮到的狀況

- Pass string 或是 number 的 data 會打破封裝
- 在使用 data 的地方必須知道如何去 handle
- 單一地方的修改也會必須在其他很多地方修改

舉例來說，

```ruby
send_tweet("Practicing Ruby", 14)

#################################

def send_tweet(status, owner_id)
  retrieve_user(owner_id)
  ...
end
```
這樣可能就不是一個很好的寫法，因為要傳進去 Magic Number 並不直觀，而且還要再 method 裏頭去存取別的 Class 的內容，就沒有很好的實踐封裝的精神，所以可以改成這樣，

```ruby
class Tweet
  attr_accessor ...

  def owner
    retieve_user(owner_id)
  end
end

#################################

tweet          = Tweet.new
tweet.status   = "Practice Ruby"
tweet.owner_id = current_user.id

send_tweet(tweet)

#################################

def sendtweet(message)
  message.owner
end

#################################
```
然而，除了物件不應該知道太多其他物件的資訊以外，那在物件裡面呢？當然也是必須要有使用的限制，在 ruby 裡面提供了三種方法，`public`，`protected，`private`，通常我們一開始定義方法時，如果沒有特別指定，就是預設是 public，也就是任何物件都可以呼叫包含繼承自該 class 的物件，而 private 和 protected 則否，至於這兩者之間的差異則是在於 private 的方法，前面不能有 receiver，關於這三者之間的關係，在龍哥寫的這篇文章 [Public, Protected and Private Method in Ruby](http://blog.eddie.com.tw/2011/07/26/public-protected-and-private-method-in-ruby/) 裏頭有很清楚地解釋，所以這樣就除了對外清楚劃分權責，對內也一樣分割清楚權限，像是下面這個例子，

```ruby
class User
  def up_vote(friend)
    bump_karma
    friend.bump_karma
  end

  protected

  def bump_karma
    puts "karma up for #{name}"
  end
end
```
最後談一下 `super` 這個方法，通常我們在繼承父類別的時候，直接更改父類別的方法就能直接的改變子類別繼承下來的方法，但是 `super` 這個用法呢，則是再不改動父類別的狀況之下，在子類別去增強方法的內容，

```ruby
class Animal
  def move
    "I can move"
  end
end

class Bird < Animal
  def move
    super + " by flying"
  end
end
```
在 new 出 `Bird` 這個物件並且呼叫了 move 這個 method 之後， `super` 會向上父類別去尋找是否有 `move` 這個方法，如果有叫呼叫，所以結果會是 `I can move by flying`。
