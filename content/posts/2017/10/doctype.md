---
title: "HTML5のDOCTYPE宣言は<!DOCTYPE html>か<!doctype html>か"
date: 2017-10-02T00:00:00+09:00
tags: ["TIL", "HTML5"]
---
正しいHTML5を書こうとしていろんなサイトのソースを見ていたところ、Googleの多くは小文字の`<!doctype html>`だったので違いを調べてみました。
<!--more-->

## 結論
HTML5として記述する場合は、大文字・小文字の区別はないため、どのように記述してもよい。

* `<!DOCTYPE html>`
* `<!doctype html>`
* `<!Doctype html>`

XHTML5として記述する場合は、大文字・小文字が区別されるため `<!DOCTYPE html>` として記述する必要がある。

## HTMLとXHTMLの歴史
そもそも、`HTML4.01`までの`HTML`は`SGML`の拡張であり、DTDによって規定されていた。その後、より拡張性があるものが求められたため、`HTML4.01`をXMLで定義し直した`XHTML1.0`が作成され、この`XHTML1.0`はW3Cによって協力に推し進められた。

`HTML5`は従来のHTMLとは異なり`SGML`の拡張ではなく、`DOM5 HTML`（現：`DOM HTML`、または`the DOM`、`DOM HTML`）として規定されている。`DOM5 HTML`はその名の通り、メモリ上のDOMオブジェクトとして抽象化された規定である。`HTML5`および`XHTML5`は`DOM5 HTML`のシリアライズ仕様である。

## 参考文献
* [DOM5 HTML](https://wiki.suikawiki.org/n/DOM5%20HTML)
* [HTML5“とか”アプリ開発入門（3）：HTML5の登場で、XHTMLは結局どうなったの？ (1/2) - ＠IT](http://www.atmarkit.co.jp/ait/articles/1011/12/news121.html)
* [HTML - XHTMLとHTML5の違いについて(16300)｜teratail](https://teratail.com/questions/16300)
