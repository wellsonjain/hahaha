---
title: Rest API Article Review
date: 2015-05-20 16:58 UTC
tags: rails, thought
layout: post
published: false
---

由於強者我同事 [Hiroshi](https://twitter.com/hiroshiyui) 和我分享如何做出個 Rest API 的[秘訣](http://zhuanlan.zhihu.com/prattle/20034107)，所以有了以下心得分享五百字，來代表我對他的尊敬。

首先由於 API 的用途就是對外開放某些資料，以及開放某些權限，所以關鍵就在於根據不同 HTTP method/header 有適當的對應(status code)

首先要在這邊先回憶一下在 [Rails API (4)](http://wellsonjain.github.io/2015/05/06/rails-api-part4/) 所出現的精美表格，

|          | Safe                        | Idempotent                  |
| -------- | --------------------------- | --------------------------- |
| GET      | <i class="fa fa-check"></i> | <i class="fa fa-check"></i> |
| POST     | <i class="fa fa-times"></i> | <i class="fa fa-times"></i> |
| PATCH    | <i class="fa fa-times"></i> | <i class="fa fa-times"></i> |
| PUT      | <i class="fa fa-times"></i> | <i class="fa fa-check"></i> |
| DELETE   | <i class="fa fa-times"></i> | <i class="fa fa-check"></i> |

鏡頭跳回 Header 部分(跳)，在 header 內其實很多資訊都是很重要的，依據 Client 端的需求，而給予正確的 Response 是很重要的，這裏有各種的 [Status Code](http://zh.wikipedia.org/zh-tw/HTTP%E7%8A%B6%E6%80%81%E7%A0%81) ；舉例來說像是 Accept Content，如果說 Client 端要求返回 "application/xml"，但是我們 Server 端卻只提供 "application/json"，那就最好返回 `406 not acceptable` 的 status code。

文中提供了一個關於符合 RFC 規範並且相當詳細的 [Deision tree](http://clojure-liberator.github.io/liberator/assets/img/decision-graph.svg)，本來要直接在這邊把它畫出來，但是畫出來可能會<del>太花時間</del>篇幅過大，所以就放在心裡面尊重吧！

API 的是提供給一個對外的接口，所以除了我們必須謹慎地作出回應之外，也必須有安全性的考慮，一致性 (integrity)，機密性 (confidentiality) 和可用性 (availibility)。

#### 請求資料驗證
文中提到漏斗的概念來想像一個 API 的行為我覺得相當貼切，在 Request 進來的時候把不必要的過濾，

  - Request Header 是否合法： Header 內容多了或是少了，或是應該有的特殊 Header 沒出現，則一律拒絕存取，回應 4xx，降低 Server 去處理不必要的工作
  - Request URI 和 Request Body 是否合法： 如果說 Request 帶了多餘的參數，一樣一律回應 4xx，以降低可能被攻擊的風險。
