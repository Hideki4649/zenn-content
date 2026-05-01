---
title: "【PortSwigger】SQL injection vulnerability in WHERE clause"
emoji: "🛡️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["security", "ctf", "web"]
published: true
---

# ■ 概要

このLabでは、WHERE句に対するSQL Injectionを利用して非公開の製品情報を取得します。

---

# ■ 結論

- 脆弱性 : SQL Injection(WHERE句)
- 攻撃手法 : `' OR 1=1--`
- 結果 : 非公開データを含むすべての製品をWeb画面上に表示

---

# ■ 解法

SQL Injectionで非公開の製品情報の取得手順を示します。

## ■ 観察
まず、Labの説明で、「When the user selects a category, the application carries out a SQL query like the following:」とSQLクエリの記載がありました。
通常のCTFでは、このような説明はないと思いますが、この内容はSQL Injectionを発生させる大きなヒントとなりました。

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

Labを起動した後、ページに「Refine your search:」に、SQLクエリの例にもあった「Gifts」があったため、選択して動きを観察しました。
すると、URLの末尾に以下の内容が追加されました。

```
filter?category=Gifts
```

## ■ 仮説

この挙動から以下のように推察しました。

- 選択したカテゴリがGETパラメータとして送信される
- URL中の「category=Gifts」がSQLクエリ中の「category = 'Gifts'」として入力される。

SQLクエリでは、選択されたカテゴリがシングルクォート(')で囲まれていましたので、URLの末尾に「'--」を追加することで、SQLクエリが以下のように変化して、「released = 1」を無効にできるのでは、と仮説を立てました。

## ■ 検証


```検証に使用するGETリクエスト
filter?category=Gifts'--
```

URL末尾への「'--」の追加によって、新たに製品が表示されたことで、SQL Queryは以下のように変化したと推察しました。

```sql
SELECT * FROM products WHERE category = 'Gifts'-- AND released = 1
```

これは、コメントアウトを示す「--」を追加したことで、以降の「released = 1」が無効になったことを示します。

## ■ 攻撃
このLabでは、「To solve the lab, perform a SQL injection attack that causes the application to display one or more unreleased products.」とあり、未発売の製品を1つ以上表示させる、というのがゴールになるので、今度は全製品を表示させるように変更しました。
以下のクエリで、すべてのcategoryの全製品を出す、という内容になります。

```
filter?category=Gifts' OR 1=1--
```

これにより、アプリケーションでは以下のようなSQLクエリが処理されます。

```sql
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--AND released = 1
```

上記のクエリでは、「OR 1=1」が常に真であるため、すべてのカテゴリが表示できます。
また、最後の「--」によって、その後の「released = 1」が無効になるため、非公開の製品も表示できます。
これによってLabのステータスが「Solved」になったため、クリアとなります。

# ■ なぜ攻撃は成立するのか

このLabにおけるSQLインジェクションが成立する理由は以下となります。

- 入力値がそのままSQLに埋め込まれている
- 「OR 1=1」により条件が常に真となる
- 「--」により後続の条件(released = 1)が無効化される

# ■ このLabにおける学び

- WHERE句では条件を崩すことが重要
- コメントアウトによる条件無効化はSQL Injectionの基本テクニック

# ■ 攻撃に対する対策

本LabにあるWHERE句に対するSQL Injectionに対して以下の対策が考えられます。

- Prepared Statementの使用
- 入力のサニタイズ

