---
title: 什麼該記成資產？什麼當花費就好？
meta_title: Beancount 資產 vs 花費 — 大型購買的記帳判斷
description: 什麼東西該記成 Asset，什麼直接當 Expense 就好？整理三個判斷面向，加上房子、車子的實際記帳範例。
date: 2026-05-09
image: /images/posts/2026-05-09/2026-05-09-asset-or-expense.jpg
tags: [基礎, 記帳]
categories: [notes]
authors: [chialin, AI]
draft: false
---

[上一篇](/blog/2026-05-08-sell-piano) 結尾留了一個問題：什麼東西該記成資產（Asset）、什麼直接當花費（Expense）？

買進當下就要決定，事後再改很麻煩。所有後續的轉帳、賣出、報表都會被影響。這篇整理三個判斷面向，加上幾個實際範例。

## 初學者：先用最直覺的分類就好

如果你還沒遇到「不知道該記哪邊」的情況，記帳一開始可以很簡單：

- 銀行、現金、買的股票 → **Asset**
- 其他消費 → **Expense**

遇到不確定的個別情境，再回頭查資料解決就好。一開始就想把每一條規則都搞清楚，學習曲線會陡到還沒享受到記帳的好處就先放棄。

下面整理的判斷面向，是**遇到不確定怎麼分類的大型購買時再回來看**的進階指引。

## 三個判斷面向

### 1. 有沒有殘值？

簡單問：賣掉的時候，這東西能不能換回現金？

- **房子（自住）**：通常有，而且常增值 → Asset
- **車子（新車）**：有，但會折舊 → 都行；折舊麻煩，多數人選 Expense
- **一杯咖啡**：完全沒有 → Expense

### 2. 金額與使用年限

- 高價、長期使用（>1 年）→ 偏 Asset
- 低價、即時消耗 → Expense

### 3. 想不想追蹤損益

- 想知道「賣掉時賺/賠多少」→ Asset（記下 cost basis，賣出時與賣價比對）
- 不在乎，買了就當花掉 → Expense

三個面向不必全部「是」才記為 Asset，但**至少要有一個強烈的理由**。

## 常見項目的建議

| 項目 | 建議分類 | 理由 |
| --- | --- | --- |
| 房子（自住） | `Assets:RealEstate:[地點]` | 金額大、有殘值、會賣 |
| 車子（多數情況） | `Expenses:Transport:Vehicle` | 折舊攤提太麻煩，賣車收入另計 |
| 車子（想追蹤殘值，進階） | `Assets:Vehicle:[車名]` | 需自定折舊方式 |
| 機車、腳踏車 | `Expenses:Transport` | 金額小，不值得追蹤 |
| 電鋼琴、家電 | 多數情況 → Expense | 見 [上一篇](/blog/2026-05-08-sell-piano) |

## 房子的記帳範例

**買進**（頭期款 + 房貸）：

```beancount
2026-05-09 * "購入自住房 - 信義區"
  Assets:RealEstate:Xinyi         15,000,000.00 TWD
  Assets:Bank:Main                -3,000,000.00 TWD  ; 頭期款
  Liabilities:Mortgage:Main      -12,000,000.00 TWD  ; 房貸
```

**每月房貸**（本金 + 利息）：

```beancount
2026-06-01 * "房貸月繳"
  Liabilities:Mortgage:Main          45,000.00 TWD  ; 還本金
  Expenses:Interest:Mortgage          8,000.00 TWD  ; 利息
  Assets:Bank:Main                  -53,000.00 TWD
```

關鍵差別：本金部分是「負債減少」（從 `Liabilities:Mortgage` 拿走），利息才是真正的「花費」。

**賣出（假設貸款已還清）**：

```beancount
2030-XX-XX * "賣房"
  Assets:RealEstate:Xinyi        -15,000,000.00 TWD
  Assets:Bank:Main                18,000,000.00 TWD
  Income:CapitalGains:RealEstate  -3,000,000.00 TWD  ; 增值
```

賣價高於買進成本的部分認列為 `Income:CapitalGains`，賣低就反過來認列為 `Expenses:Loss`。

## 車子的記帳範例

折舊每年攤提太麻煩。對個人記帳來說，這個複雜度通常不值得換來的資訊量。我自己直接把車子當成花費記掉，賣車時用 [賣二手物](/blog/2026-05-08-sell-piano) 的方式處理收入。

**買進**：

```beancount
2026-05-09 * "購入車輛"
  Expenses:Transport:Vehicle      800,000.00 TWD
  Assets:Bank:Main               -800,000.00 TWD
```

**幾年後賣車**：

```beancount
2030-XX-XX * "賣車"
  Assets:Bank:Main                400,000.00 TWD
  Income:Other:UsedGoods         -400,000.00 TWD
```

帳上沒有資產要移除，剛好對應上一篇的情境。

如果真的想追蹤殘值與買賣損益，可以記成 `Assets:Vehicle:[車名]`，但要自定折舊方式。對多數人來說太重。

## 判斷流程

買進當下，依序問自己：

1. 金額 > 50,000 TWD？
2. 使用年限 > 3 年？
3. 想追蹤賣出時的損益？

**三個都「是」→ Asset**；**三個都「否」→ Expense**；中間就看你願意花多少力氣記帳。

電鋼琴常常落在「金額中等、用得久、但不在意賣出損益」這類情況，多數人會選 Expense，記帳簡單，剛好對應 [上一篇](/blog/2026-05-08-sell-piano) 的情境。
