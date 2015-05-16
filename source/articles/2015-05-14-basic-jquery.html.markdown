---
title: Basic jQuery
date: 2015-05-14 00:31 UTC
tags: javascript
layout: post
---

> Fear does not prevent death. It prevents life. - Naguib Mahfouz

今天要記錄一些基礎的 jQuery 基本常識，基本到我本來還真的不會...首先先來看看 jQuery 到底可以做到哪些事呢？

|         |                             |
| ------- | --------------------------- |
| find    | 找到 HTML 裡的 element        |
| change  | 改變 HTML 裡的 element        |
| listen  | 聽使用者做了什麼，並做出相對的回應 |
| animate | 對 content 做特效             |
| talk    | 透過網路來得到新的 Content      |

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Basic jQuery</title>
  </head>
  <body>
    <h1>What do you want?</h1>
    <p>More basic teach</p>
  </body>
</html>
```
假如我要找 `p` 內的文字並且改變它，我可以用 jQuery 這麼做，

```javascript
// $("p") - 找到(find) p
// .text("") - 改變(change) p 裡面的文字

$(document).ready(function(){
  $("p").text("More advance teach")
});
```
可是再深入了解為什麼可以這樣用，我們必須先瞭解到底 browser 是怎麼去處理 HTML? 在這之前我們要先瞭解什麼是 `DOM`

#### Document Object Model
> Document Object Model(DOM): A <b>tree-like</b> structure created by browsers so we can quickly find HTML Elements using JavaScript.

DOM 會形成樹狀結構，所以上面的那段 HTML 就會變成

```
  html
  | head
  |   | title
  |     | Basic jQuery
  | body
      | h1
      | | What do you want?
      | p
        | More basic
```
每一個 html 的 element 都會轉變成一個個相對應的節點，但是如果你仔細看上面的 `$(document).ready(function(){`，這段又是什麼意思？
`$`代表 jQuery，`document` 則代表我們前面所提到的 DOM，`ready` 則是等待 html 都轉變為 DOM ，所以整段我們組合起來所帶代表的意思就是，我們選取了所有的 object，並且等到 DOM ready 之後我們再開始下面程式，如果我們沒有執行這段的就直接選取 p 就會出錯，這是因為整個 html 並沒有完全被轉化成 DOM，所以 jQuery 沒有辦法找到 p 節點。

在了解基本的規則以及整個 DOM 的結構之後，我想很好理解 jQuery 的行為了，首先先來解釋在前面我們提到的 `find`，jQuery 在 find 的行為跟 CSS 的 selector 相同

```
     CSS               jQuery
     -----------       --------------
     p { ... }    <=>  $("p")
     #container        $("#container")
     .articles         $(".articles")
```
jQuery 的 selector 也可以使用 `$("#container p")`，這樣的方式，選擇 container 底下的"所有" p element，也就代表如果在這個 id 下的子子孫孫 p 全部都會被選到，所以如果不要選擇到這麼多層，可以使用 ">"，`$("#container > p")`，這就表示只選擇到他的下一代。

#### Traversing
除了上面的這種用祖譜式的找到 element，我們還可以使用 traversal 直接去找就好，直觀效果強，這個的結果跟 `$("#container p")` 是相同的，

```javascript
$("#container").find("p")
```
所以說如果要使用單純找到底下一層的 element，可以使用

````javascript
$("#container").children("p")
````
Traversing 也可以走上走下這樣

````javascript
// 在同一個層級走訪
$("li").first();
$("li").first().next();
$("li").first().next().prev();

// 往上一個層級走
$("li").first().parent();
````
以上是幾種比較常用的 jQuery "find" 的行為

#### Working with the DOM
有時候我們會需要新增 class 或是 remove class 來區別我們所選擇的 element，先從 `appending` 舉例來說，我們要新增一個 node 在我們的 html 上，下面有四種方式，

````html
<!DOCTYPE html>
<html>
  ...
  <body>
    <li class="vacation">
      <h2>Taiwan</h2>
      <button>Get price</button>
    </li>
  </body>
  <script>
    $(document).ready(function() {
       var price = $('<p>From $399.99</p>');
    });
    ...
  </script>
</html>
````
| $('.vacation')       |                                  |
| -------------------- | -------------------------------- |
| .before(< element >) | 把 price 放在 .vacation 之前       |
| .after(< element >)  | 把 price 放在 .vacation 之後       |
| .prepend(< element >)| 把 price 放在 .vacation 子項最上面  |
| .append(< element >) | 把 price 放在 .vacation 子項最下面  |

當然在 jQuery 很易讀的地方在於，我們同樣可以更換主受詞得到相同的效果，

| price                      |                                  |
| -------------------------- | -------------------------------- |
| .insertBefore(< element >) | 把 price 放在 .vacation 之前       |
| .insertAfter(< element >)  | 把 price 放在 .vacation 之後       |
| .prependTo(< element >)    | 把 price 放在 .vacation 子項最上面  |
| .appendTo(< element >)     | 把 price 放在 .vacation 子項最下面  |

而 remove 就相對簡單得多 `.remove()` 就能夠做得到。

(是說我本來打算一篇就把它寫完的...看來要寫兩篇了...)
