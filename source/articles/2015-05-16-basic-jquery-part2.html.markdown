---
title: Basic jQuery (2)
date: 2015-05-16 08:24 UTC
tags: javascript
layout: post
---
> The world is moving so fast that the man who says it can't be done is generally interrupted by someone doing it. - Elbert Hubbard

#### Acting on Interaction
在上一篇我們有提到 jQuery 有一個行為叫做 `listen`，這就是聽有沒有什麼事件發生了呢? `on`就是其中一種，

```javascript
// jQuery Object Methods
$('button').on('click', function(){})
```
上面這段的意思是，當我們<del>臨幸</del>選擇的 `button` 這個物件，只要發生了 `click` 這個事件，就會執行後面定義的 function

```javascript
$(document).ready(function() {
  $('button').on('click', function() {
    var price = $('<p>From $399.99</p>');
    $('.vacation').append(price);
    $('button').remove();
  });
});
```
在上面我們看到，當 `button` 發生了 `click` 這個事件，就會在 `vacation` 的子層加上 `price` 這個 node，並且把 `button` 移除，但是這會發生一個問題，就是如果在同一個 DOM 裡面有多個 `button` 或是多個 `vacation` 那就會發生一次都改變的問題，所以在 jQuery 就提供了一個相當方便也相當重要的方法，叫做 `this`，`this` 所代表的意思為當前的物件，所以上面的程式，我們就可以將他換成下面這樣，

```javascript
$(document).ready(function() {
  $('button').on('click', function() {
    var price = $('<p>From $399.99</p>');
    $(this).closest('.vacation').append(price);
    $(this).remove();
  });
});

```
我們在程式中定義了 `button` 去 listen 事件，所以在這邊的 `this` 代表的物件就是 `button`，在邊使用 `closest` 顧名思義就是最近的 `vacation` class ，之所以不用先前提到的 `.parents()` 是因為我們可能無法確定會不會有很多相同的 class name，所以在這樣的情況之下使用 `closest` 會是更好的選擇。

在經過一連串的 refactor 之後，我們目前的程式碼看起來可讀性很高，但是唯一比較寫死的就是 `price` 這個物件，那我們有更好的方法可以去 improve 嗎? 當然，

```html
<li class="vacation onsale" data-price='399.99'>
  <h3>Hawaiian Vacation</h3>
  <button>Get Price</button>
  <ul class='comments'>
    <li>Amazing deal!</li>
    <li>Want to go!</li>
  </ul>
</li>
```
首先我們先在 html 裡面加上一個 data attribute `data-price`，這樣我們就可以在我們的 jQuery 裡面去呼叫方法 `.data(<name>)`，

```javascript
$(document).ready(function() {
  $('button').on('click', function() {
    var amount = $(this).closest('.vacation').data('price');
    var price = $('<p>From $'+amount+'</p>');
    $(this).closest('.vacation').append(price);
    $(this).remove();
  });￼￼
});
```
這是個很方便的做法，於是我們便可以在 html 上面去設定多個自定義的 attribute，然後再使用上述的方法個取值。

接著我們繼續延伸說明 `on` 的用法，先看看上述的程式當中，我們設定為當 `button` 這個物件發生了 `click` 事件，但是可能整個 DOM 裡面會有多個 button 物件，所以我們可以藉由在傳入的 argument 去指定在這個物件底下的哪個物件要被執行 function。

```javascript
$(document).ready(function() {
  $('.vacation').on('click', 'button', function() {
    ...
  });￼￼
});
```
>  on() 的各種 Event: <http://api.jquery.com/category/events/>

>  Mouse Events: <http://api.jquery.com/category/events/mouse-events/>

>  Keyboard Events: <http://api.jquery.com/category/events/keyboard-events/>

>  Form Events: <http://api.jquery.com/category/events/form-events/>

#### Link Layover
接下來要講一個我覺得蠻有趣的東西，叫做 bubble up，也就是當事件發生之後該 node 會把這個事件發生的訊息告訴所有父層級，

```html
<!DOCTYPE html>
...
<ul>
  <li class="vacation">
    <a href="#" class="expand">Show comment</a>
    <ul class="comment">
      <li>...</li>
    </ul>
  </li>
</ul>
```
在上面這段 html 裡面當我們按下了 a link，因為 # 的原因，不管我們在哪個地方按下，都會回到最頂端，所以我們必須在我們的 callback function 再帶入一個 argument `event`，來為我們所觸發的 event 再做一些處理，在這邊我要介紹的是兩種處理的方式，一種叫做 `.stopPropagation()` 這個方法的意思就是當這個事件觸發的時候，只有這個物件會知道，父層級的 node 都會被擋下來，雖然如此因為 hashtag 我們還是會有置頂的狀況發生，所以有另外一種方法，叫做 `.preventDefault()`，這與我們第一個提到的方法有點不同，他會讓事件發生的訊息告訴所有父層級，但是卻不會進行處理。

#### Animation
最後要說的是上一篇提到的 animate 行為，也就是做出一點動態的效果來

```javascript
$(document).ready(function() {
  $('#vacations').on('click', '.vacation', function() {
    $(this).toggleClass('highlighted');

    if($(this).hasClass('highlighted')) {
      $(this).animate({'top': '-10px'});
    } else {
      $(this).animate({'top': '0px'});
    }￼
  });
});
```
上面這邊所做的事情就是當我們按一下 click，就會給他加上 `highlighted` 的 class，那如果這個物件有這個 class 那就給他做出一點動態的效果，往上滑動 10px，但是畢竟這是 CSS 語法，我們希望能夠切割乾淨，所以最好的方法還是使用 CSS3 的函式，`transition: top 0.2s;` 這樣加上這個 class 也會有相同的效果。

(太棒了寫完基礎篇了!~~)
