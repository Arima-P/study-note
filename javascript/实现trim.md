# 实现 `trim()`

> 摘自[网上](https://www.cnblogs.com/rubylouvre/archive/2009/09/18/1568794.html) [原文](http://blog.stevenlevithan.com/archives/faster-trim-javascript)

### 实现 1

```js
String.prototype.trim = function () {
  return this.replace(/^\s\s*/, '').replace(/\s\s*$/, '');
};
```

- 动用了两次正则替换，速度非常惊人，主要得益于浏览器的内部优化。
- 一个著名的例子字符串拼接，直接相加比用 Array 做成的 StringBuffer 还快。base2 类库使用这种实现。

### 实现 2

```js
String.prototype.trim = function () {
  return this.replace(/^\s+/, '').replace(/\s+$/, '');
};
```

- 和实现 1 很相似，但稍慢一点，主要原因是它最先是假设至少存在一个空白符。
- Prototype.js 使用这种实现，不过其名字为 strip，因为 Prototype 的方法都是力求与 Ruby 同名。

### 实现 3

```js
String.prototype.trim = function () {
  return this.substring(
    Math.max(this.search(/\S/), 0),
    this.search(/\S\s*$/) + 1
  );
};
```

- 以截取方式取得非空白部分（当然允许中间存在空白符），总共调用了四个原生方法。
- 设计得非常巧妙，substring 以两个数字作为参数。Math.max 以两个数字作参数，search 则返回一个数字。速度比上面两个慢一点，但比下面大多数都快。
- `substring()` 方法用于提取字符串中介于两个指定下标之间的字符。
- `search()` 方法用于检索字符串中指定的子字符串，或检索与正则表达式相匹配的子字符串。

### 实现 4

```js
String.prototype.trim = function () {
  return this.replace(/^\s+|\s+$/g, '');
};
```

- 这个可以称得上实现 2 的简化版，就是利用候选操作符连接两个正则。但这样做就失去了浏览器优化的机会，比不上实现 3。
- 由于看来很优雅，许多类库都使用它，如 JQuery 与 mootools

### 实现 5

```js
String.prototype.trim = function () {
  var str = this;
  str = str.match(/\S+(?:\s+\S+)*/);
  return str ? str[0] : '';
};
```

- `match()` 是返回一个数组，因此原字符串符合要求的部分就成为它的元素。
- 为了防止字符串中间的空白符被排除，我们需要动用到非捕获性分组 `（?:exp）`。
- 由于数组可能为空，我们在后面还要做进一步的判定。
- 好像浏览器在处理分组上比较无力，一个字慢。所以不要迷信正则，虽然它基本上是万能的。

### 实现 6

```js
String.prototype.trim = function () {
  return this.replace(/^\s*(\S*(\s+\S+)*)\s*$/, '$1');
};
```

- 把非空白部分提取出来，效率差

### 实现 7

```js
String.prototype.trim = function () {
  return this.replace(/^\s*(\S*(?:\s+\S+)*)\s*$/, '$1');
};
```

- 和实现 6 很相似，但用了非捕获分组进行了优点，性能效之有一点点提升。

### 实现 8

```js
String.prototype.trim = function () {
  return this.replace(/^\s*((?:[\S\s]*\S)?)\s*$/, '$1');
};
```

- 沿着上面两个的思路进行改进，动用了非捕获分组与字符集合，用?顶替了\*，效果非常惊人。尤其在 IE6 中，可以用疯狂来形容这次性能的提升，直接秒杀火狐。

### 实现 9

```js
String.prototype.trim = function () {
  return this.replace(/^\s*([\S\s]*?)\s*$/, '$1');
};
```

- 这次是用懒惰匹配顶替非捕获分组，在火狐中得到改善，IE 没有上次那么疯狂。

### 实现 9

```js
String.prototype.trim = function () {
  var str = this,
    whitespace =
      ' \n\r\t\f\x0b\xa0\u2000\u2001\u2002\u2003\u2004\u2005\u2006\u2007\u2008\u2009\u200a\u200b\u2028\u2029\u3000';
  for (var i = 0, len = str.length; i < len; i++) {
    if (whitespace.indexOf(str.charAt(i)) === -1) {
      str = str.substring(i);
      break;
    }
  }
  for (i = str.length - 1; i >= 0; i--) {
    if (whitespace.indexOf(str.charAt(i)) === -1) {
      str = str.substring(0, i + 1);
      break;
    }
  }
  return whitespace.indexOf(str.charAt(0)) === -1 ? str : '';
};
```

- 先是把可能的空白符全部列出来，在第一次遍历中砍掉前面的空白，第二次砍掉后面的空白。全过程只用了 `indexOf()` 与 `substring()` 这个专门为处理字符串而生的原生方法，没有使用到正则。
- 速度快得惊人，估计直逼上内部的二进制实现，并且在 IE 与火狐（其他浏览器当然也毫无疑问）都有良好的表现。速度都是零毫秒级别的。

### 实现 11

```js
String.prototype.trim = function () {
  var str = this,
    str = str.replace(/^\s+/, '');
  for (var i = str.length - 1; i >= 0; i--) {
    if (/\S/.test(str.charAt(i))) {
      str = str.substring(0, i + 1);
      break;
    }
  }
  return str;
};
```

- 使用了简单的正则，替换可能的空白符（不好记）
- 10 的改进版，前面部分的空白由正则替换负责砍掉，后面用原生方法处理，效果不逊于原版，但速度都是非常逆天。

### 实现 12

```js
String.prototype.trim = function () {
  var str = this,
    str = str.replace(/^\s\s*/, ''),
    ws = /\s/,
    i = str.length;
  while (ws.test(str.charAt(--i)));
  return str.slice(0, i + 1);
};
```

- 实现 10 与实现 11 在写法上更好的改进版，注意说的不是性能速度，而是易记与使用上。和它的两个前辈都是零毫秒级别的，以后就用这个来工作与吓人。
