---
title: "JavaScript再入門を読んだ"
author: 'miruo'
tags: ['JavaScript']
date: 2021-04-10T23:04:31+09:00
draft: false
---

これ読んだ

[JavaScript 「再」入門](https://developer.mozilla.org/ja/docs/Web/JavaScript/A_re-introduction_to_JavaScript)

経緯: React分からん → TypeScript分からん → JavaScript分からん → ？？？？？

## JavaScriptとは

- マルチパラダイムの動的型付け言語
- 値の型（データ型），演算子，メソッド，そして **オブジェクト**

## データ型

プリミティブ型とオブジェクトの二種類からなる。

- プリミティブ型
    - `Number`
    - `String`
    - `Boolean`
    - `null`
    - `undefined`
    - `Symbol`
    - `BigInt`
- Object

プリミティブ型はオブジェクトではないが、 `null` と `undefined` 以外にはその値を内包する等価なラッパーオブジェクトが用意されている。つまりJavaScriptではほぼ全てがオブジェクトと言っても過言ではない。関数や配列もオブジェクト。

## オブジェクト

オブジェクトとは、名前と値のペアのコレクション。Pythonの辞書型に近い。以下は空のオブジェクトを生成する文。

```jsx
var obj = new Object();
```

```jsx
var obj = {};
```

二つの文は意味的に等価。2つ目の方はオブジェクトリテラルで、こっちを使うべき。 `{}` はブロックを意味するものでもあるため、文の先頭では使えない（ややこしい・・・）

```jsx
var obj = {
	name: 'Carrot',
	details: {
		color: 'orange',
		size: 12
	}
};
```

以下のどちらの方法でも属性にアクセスできる。

```jsx
obj.details.color;
obj['details']['color'];
```

## 関数

```jsx
function add(x, y) {
	var total = x + y;
	return total;
}
```

```jsx
function avg(...args) {
	var sum = 0;
	for (let value of args) {
		sum += value;
	}
	return sum / args.length;
}
```

### 無名関数

```jsx
var avg = function(...args) {
	var sum = 0;
	for (let value of args) {
		sum += value;
	}
	return sum / args.length;
}
```

ES2015では無名関数の短縮形としてアロー関数が導入された。

```jsx
var avg = (...args) => {
	var sum = 0;
	for (let value of args) {
		sum += value;
	}
	return sum / args.length;
}
```

単一式ならブラケット、 `return` を省略できる。

```jsx
var sum = (a, b) => a + b;
```

オブジェクトを返したい時は `()` でくくる。

```jsx
var sum = (a, b) => ({ sum: a + b });
var Person = () => ({});
```

### this, new

`this` は現在のオブジェクトを指し示すキーワード。グローバルで使うとグローバルオブジェクト（"The global object", 標準組み込みオブジェクトを指す"global objects"とは別物）を表す。ブラウザでのグローバルオブジェクトは `window` である。

`new` は新しく空のオブジェクトを作り、 `this` にそのオブジェクトをセットして、後に続く関数を呼ぶ。 `new` の後に呼ばれることを想定して作られた（値を返さない）関数はすなわちコンストラクタである。

```jsx
function Person(first, last) {
	this.first = first;
	this.last = last;
	this.fullName = function() {
		return this.first + ' ' + this.last;
	};
	this.fullNameReversed = function() {
		return this.last + ', ' + this.first;
	};
}
var s = new Person('LeBron', 'James');
```

このように、プロトタイプオブジェクト（ここではPersonオブジェクト）のインスタンスとして新しいオブジェクトを生成する方法を、クラスベースプログラミングに対してプロトタイプベースプログラミングと言う。

### prototype

オブジェクトのプロトタイプは、すべてのインスタンスで共有されるオブジェクト。

プロトタイプのプロパティはすべて `this` オブジェクトを通してコンストラクタの全てのインスタンスで利用できる。

```jsx
s = new Person('LeBron', 'James');

Person.prototype.firstNamecaps = function() {
	return this.first.toUpperCase();
};

s.firstNameCaps(); // "LEBRON"
```

これはすごい。
