---
title: 把記為花費的二手物賣掉 — 以電鋼琴為例
meta_title: 賣二手物的 beancount 記法（Expense 情境）
description: 當初買電鋼琴直接記成 Expense，現在賣掉收到 10,000 元，要記在哪裡？比較「認列為收入」與「沖銷原花費」兩種做法。
date: 2026-05-08
image: /images/posts/2026-05-08/2026-05-08-sell-piano.jpg
tags: [記帳, 實例]
categories: [notes]
authors: [chialin, AI]
draft: false
---

最近把家裡的電鋼琴賣掉了。當初買進時直接記成花費，沒有特別開資產帳戶。現在收到買家的 10,000 元，這筆錢要怎麼記？

## 帳上沒有這台琴

先看當初的紀錄，大致長這樣：

```beancount
2024-03-15 * "購入電鋼琴"
  Expenses:Hobbies:Music         20,000.00 TWD
  Assets:Bank:Main              -20,000.00 TWD
```

關鍵是：**這台琴從來沒有出現在資產表上**。買的當下就全數記為花費，所以帳簿裡找不到 `Assets:Equipment:Piano` 這種項目。

這代表現在賣出時，沒有「資產要移除」這件事——只剩「銀行多了 10,000 元」要處理。

## 兩種做法

### 做法 1：認列為收入

```beancount
2026-05-08 * "賣電鋼琴"
  Assets:Bank:Main                 10,000.00 TWD
  Income:Other:UsedGoods          -10,000.00 TWD
```

開一個 `Income:Other:UsedGoods` 專門收這類零星收入。賣二手書、舊衣、家電、樂器都可以丟這裡。

### 做法 2：沖銷原本的花費

```beancount
2026-05-08 * "賣電鋼琴"
  Assets:Bank:Main                  10,000.00 TWD
  Expenses:Hobbies:Music           -10,000.00 TWD
```

把 10,000 當成「負花費」，等於把當初的支出抵掉一部分。

兩種做法帳都會平，差別在報表呈現。

## 我選做法 1，理由

- **不扭曲歷史**。當初真的花了 20,000，這個事實不會因為兩年後賣掉而變成「其實只花了 10,000」。如果用做法 2，回頭看 2026 年的 `Expenses:Hobbies:Music` 會出現一筆負數，第一眼看不出來是怎麼回事。
- **「賣二手物」是獨立事件**。以後不管賣什麼舊東西，都能在 `Income:Other:UsedGoods` 一覽，不會跟原本的興趣花費混在一起。
- **報表更直覺**。賣出的錢是收入，不是「花費的反義」。

唯一適合做法 2 的情境，大概是**買進當年就賣掉**——例如三月買、八月轉售，當年度的花費就直接抵掉，比較不會誤讀過去的紀錄。

## 開帳戶

如果還沒開過 `Income:Other:UsedGoods`，記得先 [open](/blog/2026-04-28-beancount-open-close/)：

```beancount
2026-05-08 open Income:Other:UsedGoods    TWD
```

之後賣什麼舊東西都丟進去，年底看一眼就知道二手收入有多少。

## 延伸：為什麼當初是 Expense 而不是 Asset？

這篇講的是「當初記為 Expense」的情境。但同樣一台電鋼琴，也可以記成 `Assets:Equipment:Piano`，賣出時的記法就完全不同——要把資產移除、認列買賣損益。

什麼東西該記成 Asset、什麼直接當 Expense？這是另一個值得單獨講的決策，下一篇整理。
