---
title: Beancount 的 open 與 close — 帳戶開立與關閉
meta_title: Beancount open / close 入門
description: 介紹 beancount 中宣告帳戶的 open 與 close 指令、語法與常見錯誤。
date: 2026-04-28
image: /images/posts/2026-04-28/2026-04-28-beancount-open-close.jpg
tags: [基礎, 記帳]
categories: [notes]
authors: [chialin, AI]
draft: false
---

要在 beancount 裡記任何一筆交易之前，每個帳戶都必須先用 `open` 宣告。這篇整理 `open` / `close` 的語法與常見錯誤。

範例都用台幣 TWD，先不討論多幣別。

## 基本語法

```beancount
2024-01-01 open Assets:Bank:Cathay              TWD
2024-01-01 open Liabilities:CreditCard:Citi     TWD
2024-01-01 open Income:Salary                   TWD
2024-01-01 open Expenses:Food                   TWD
```

格式是 `{日期} open {帳戶名} {貨幣}`。

帳戶名的第一段必須是五大根帳戶之一：`Assets`、`Liabilities`、`Income`、`Expenses`、`Equity`。後面用冒號 `:` 分階層，想開幾層都可以，例如 `Expenses:Food:Eatout` 就是「外食」分類。

最後的 `TWD` 是這個帳戶允許出現的貨幣。寫上去之後，如果某天交易把別的幣別塞進來，[`bean-check`](/blog/2026-04-28-beancount-cli-tools/) 會發出 warning，防止不同幣別混用的問題。

## Close 帳戶

```beancount
2024-12-31 close Assets:Bank:OldAccount
```

關閉前帳戶餘額必須是 `0`，否則 `bean-check` 會擋。實務流程是先轉移餘額到新帳戶，下一行再 `close`：

```beancount
2024-12-30 * "Migrate balance to new account"
  Assets:Bank:OldAccount      -12345.00 TWD
  Assets:Bank:NewAccount       12345.00 TWD

2024-12-31 close Assets:Bank:OldAccount
```

關閉之後不能再有任何 transaction 動到這個帳戶。

## 規則整理

- 一個帳戶只能 `open` 一次、`close` 一次
- `open` 的日期必須早於該帳戶任何 transaction
- 沒 `open` 就用，`bean-check` 會報錯
- `close` 之前餘額必須歸零

## 常見錯誤

- **`open` 日期太晚**：你 1 月就在用的悠遊卡帳戶卻寫成 6 月 open，等於跟 beancount 說「這個帳戶 6 月才開」。`bean-check` 執行時，1 到 5 月每一筆儲值與扣款都會被當成「動到一個還沒存在的帳戶」因此報錯。
