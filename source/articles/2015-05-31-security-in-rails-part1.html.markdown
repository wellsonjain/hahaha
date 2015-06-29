---
title: Security in Rails (1)
date: 2015-05-31 07:34 UTC
tags: rails, rails guides
layout: post
---
> Believe in yourself. Under-confidence leads to a self-fulfilling prophecy that you are not good enough for your work. - Dr Roopleen

這是第一篇開始對於 [Rails Guides](http://guides.rubyonrails.org/index.html) 所做的筆記記錄，也不一定是按照著順序寫，最近對於 Security 有點興趣而且也在這方面的機制不是很了解，所以就先從[這篇](http://guides.rubyonrails.org/security.html)開始看起。

#### Session
HTTP 是 stateless 的 protocl，如果說在 content 裡有需要驗證使用者身份或是像是購物車狀態，seesion 的功用就是要讓為了記錄狀態，如果沒有 session 的機制，每一次的 request 都必須做一次驗證，光用想的就覺得惱人啊，一個 session 通常是由 session id 和 hash values 所組成，session id 是由隨機產生的，所以有產生 collision 的可能，不過截至目前為止都是相當安全。

有關於 session 的安全性議題最常見的稱之為 Session Hijacking，是指竊取使用者的 session id 得到使用 Web app 的使用權限，有以下幾種情境

- 藉由在不安全的網路環境進行竊聽 cookie，就可以竊取 session

  ```ruby
  # 方法：以 SSL 建立安全的網路連線，在 rails 裡可以在 config 裡面進行設定

    config.force_ssl = true
  ```
- 使用者在 public 的環境下使用卻沒有登出的習慣
- Cross-Site Scripting(XSS) 讓使用者自動送出他們的 cookie
- 將 Session 的識別碼修正成可以 attacker 的(Session Fixation)

Session Fixation 是常見的手法，就由先跟 Server 要到 session 之後，再把使用者的 session 偷偷改成自己的，當使用者登入之後，Hacker 也可以擁有對方的資料了，解決的方式其實很簡單，`reset_session`，或者是記錄 user 更細節的資訊，像是 IP address，並且在每個 request 都進行驗證，如果有不同馬上就把 session 清除。

#### Cross-Site Request Frogery (CSRF)
借刀殺人一直以來都是相當陰險的招數，在惡意攻擊也有類似的手法，讓你在 Hacker 網站植入一個對 A 網站進行惡意存取的動作，如果使用者剛好有 A 網站的使用權限並且 session 還是合法的狀態，這個攻擊就被觸發了，而從行為上來看，就像是你自己觸發這件事一樣，實際上則是這個請求是自動觸發，你甚至不知道有這件事發生，他只是把動作再加上你自己的 session，就代替你做了這個請求。

而 rails 則提供了一個方法，就是在做非 GET 的 request 的時候，都必須帶一個安全性的 token，而這個 token 只有該網站才會知道甚至只有在該網站內進行 POST request 的時候才知道，其他網站不知道，所以上述的情況則是 Hacker 網站無法得到該網站的 token，所以就算竊取了你的 session 但是因為沒有辦法得到 token 或是沒辦法 match，該請求無效。

Rails 會在 application controller 裡面 default 加入下面這段，

```ruby
  protect_from_forgery with: :exception
```
