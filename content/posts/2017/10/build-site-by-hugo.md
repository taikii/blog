---
title: "Build Site by Hugo"
date: 2017-10-17T00:25:05+09:00
tags: ["Hugo", "GitHub"]
series: ["Hugo 構築"]
---
静的サイトジェネレータHugoとGitHub Pagesでブログを公開する方法についてまとめました。

<!--more-->

[Jekyllに対するHugoの優位点](jekyll-vs-hugo.md)の続きです。

私の環境は以下のようなものです。

* ジェネレータ: Hugo v0.30
* OS: Windows10
* エディタ:  Atom

HugoとGitHub Pagesでブログサイトを作成するためには、まずGitHub Pagesの概要について知っておく必要があります。

## GitHub Pagesについて
GitHub Pagesで個人として作ることのできるサイトは **ユーザページサイト**（User Pages site） と **プロジェクトページサイト**（Project Pages site owned by a user account） の2種類があります。

ユーザページサイトのURLは`https://ユーザ名.github.io`となり、プロジェクトページサイトのURLは`https://ユーザ名.github.io/リポジトリ名（プロジェクト名）`となります。

うだうだ書きますが、**公式ページを見れば一発です**。

* [User, Organization, and Project Pages - User Documentation](https://help.github.com/articles/user-organization-and-project-pages/)


### ユーザページサイトを作成する場合
ユーザページサイトを作成する際は、 **`ユーザ名.github.io`** という名前のリポジトリを作ります。それだけでサイトのデプロイは完了です。試しにリポジトリのルートディレクトリに`index.html`を作成して、`https://ユーザ名.github.io`にアクセスすると`index.html`の画面が表示されます。

ユーザページサイトは、 **`ユーザ名.github.io`という名前のリポジトリ** の **`master`ブランチ** が公開ディレクトリとなります。現時点でこれを変更することはできません。

また、`ユーザ名.github.io`という名前のプロジェクトを作成すると、自動的にGitHub Pagesとして公開されます。現時点でプロジェクトを削除する以外に公開を停止する方法はありません。

このサイトはユーザページサイトとして作成しています。GitHub上のプロジェクトは https://github.com/taikii/taikii.github.io です。


### プロジェクトページサイトを作成する場合
プロジェクトページサイトは任意の名前のリポジトリから作成できます。プロジェクトページのSetting -&gt; GitHub Pages の`Source`を`None`からそれ以外に変更することでプロジェクトページサイトが作成されます。

プロジェクトページサイトの公開ディレクトリは3種類+1から選択できます。

1. **`gh-pages branch`**: `gh-pages`ブランチを公開ディレクトリとする
2. **`master branch`**: `master`ブランチを公開ディレクトリとする（ユーザページサイトと同じ）
3. **`mastar branch /docs folder`**: `master`ブランチの`/docs`フォルダーを公開ディレクトリとする
4. **`None`**: GitHub Pagesとして公開しない ※デフォルト

#### 1. gh-pages branch
既存のリポジトリ（普通にソースを置いているようなリポジトリ）をGitHub Pagesとして公開する場合に使用することになると思います。きっと次の様に思うことでしょう。

> リポジトリと同じ名前のURLで公開したい、でもソースの中にHTMLファイルを配置したくないし、HTMLファイルを更新したコミットログを混ぜたくない。

`gh-pages`ブランチは大体において`--orphan`で作ります。
```
git checkout --orphan gh-pages
```
orphanとは孤児のことで親を持たないブランチが作成されます。これで既存の開発用コミットツリーとは無関係にサイトの管理ができるようになります。

`gh-pages`ブランチを作る上でいくつかの注意点があります。

* リポジトリ上に`gh-pages`ブランチがないと設定画面上の`gh-pages branch`選択肢は表示されません。
* `gh-pages`ブランチを作成すると、自動的に`gh-pages`が公開ディレクトリとして設定され、プロジェクトページサイトが作成されます。
* `gh-pages`ブランチを作成すると、`None`を選択できなくなります。公開を停止するには`gh-pages`ブランチを削除する他ありません。

#### 2. master branch
`master`ブランチが公開ディレクトリとして設定されます。GitHub Pagesとして公開することを目的として作ったリポジトリの場合はこれがよいでしょう。わざわざ`gh-pages`ブランチを作成する必要はありません。

#### 3. mastar branch /docs folder
既存のリポジトリで、`/docs`にマニュアルなどのHTMLドキュメント類を含んでいてそれを公開したい場合はこれになると思います。また、Hugoのような静的サイトジェネレータで生成したHTMLファイルを`/docs`に出力している場合はこれでしょう。

Hugoの公式マニュアルでは、この`/docs`を使ってプロジェクトページサイトを公開するのが[最も簡単な方法](https://gohugo.io/hosting-and-deployment/hosting-on-github/)と述べられています。

今回はこの3の`mastar branch /docs folder`を例にして説明します。

## Hugoでサイトを作って公開する
Hugoについても、**公式ページを見れば一発です**。

* [Hugo | Get Started](https://gohugo.io/getting-started/)

Hugoリポジトリの[Releases](https://github.com/gohugoio/hugo/releases)から自分の環境に合わせてた最新版をダウンロードしてきます。私の環境はWindowsなので`hugo_0.30_Windows-64bit.zip`をダウロードします。

適当なディレクトリに展開し、めんどくさいのでパスを通しておきます。

`hugo new site`でHugoのサイトを作ります。以下の例では`D:\gitrepos\gh-pages-example`というディレクトリにサイトを作ります。
```
$ mkdir D:\gitrepos
$ cd /d D:\gitrepos
$ hugo new site gh-pages-example
Congratulations! Your new Hugo site is created in D:\gitrepos\gh-pages-example.

Just a few more steps and you're ready to go:

1. Download a theme into the same-named folder.
   Choose a theme from https://themes.gohugo.io/, or
   create your own with the "hugo new theme <THEMENAME>" command.
2. Perhaps you want to add some content. You can add single files
   with "hugo new <SECTIONNAME>\<FILENAME>.<FORMAT>".
3. Start the built-in live server via "hugo server".

Visit https://gohugo.io/ for quickstart guide and full documentation.

$ cd gh-pages-example
```

以下の様なフォルダ構成となっています。
```
$ tree /f .
D:\GITREPOS\GH-PAGES-EXAMPLE
│  config.toml
│
├─archetypes
│      default.md
│
├─content
├─data
├─layouts
├─static
└─themes
```

* `config.toml`はサイトの設定ファイルです。
* `archetypes/default.md`は新しいポストを作成した際のデフォルト値を定義します。
* `content`の下にページやポストを配置します。
* `themes`の下にテーマを配置します。


ここで、GitHub上に同じ名前のリポジトリ（今回の例では`gh-pages-example`）を作っておき、Git管理対象としておきます。
```
$ cd
D:\gitrepos\gh-pages-example
$ git init
Initialized empty Git repository in D:/gitrepos/gh-pages-example/.git/
$ git remote add origin https://github.com/taikii/gh-pages-example.git
$ git fetch origin
```

また、Hugoは何かしらテーマを入れないと何も表示されません。テーマは[Hugo Themes | Complete List](https://themes.gohugo.io/)にたくさんありますが、今回はかっこいい[Liquorice](https://themes.gohugo.io/liquorice/)を入れてみます。（今回はHugoのサイトディレクトリをGitで管理しているため、`git clone`ではなく`git submodule`にします。）
```
$ git submodule add https://github.com/eliasson/liquorice.git themes/liquorice
```

Hugoはサーバ機能がついており、HTMLに変換していない状態で出来栄えを確認できます。新しいコマンドプロンプトウィンドウを開き`hugo server -t liquorice -w -D`を実行して放置します。
```
$ cd /d D:\gitrepos\gh-pages-example
$ hugo server -t liquorice -w -D
Started building sites ...

Built site for language en:
0 draft content
0 future content
0 expired content
0 regular pages created
6 other pages created
0 non-page files copied
0 paginator pages created
0 tags created
0 categories created
total in 5 ms
Watching for changes in D:\gitrepos\gh-pages-example\{data,content,layouts,static,themes}
Serving pages from memory
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```

* **`-t`**: `/themes`フォルダ配下に入れたテーマを指定します。
* **`-w`**: ファイルシステムを監視してファイルが更新された際に自動でWebページをリロードします。
* **`-D`**: `draft: true`が指定されている下書き状態のページも表示します。

ブラウザで http://localhost:1313/ にアクセスするとサイトを確認できます。まだなにも記事を作っていのでタイトルだけ表示された状態です。

新しい記事を作成してみます。
```
$ hugo new posts/hello-hugo-world.md
D:\gitrepos\gh-pages-example\content\posts\hello-hugo-world.md created
```

表示されたパスのファイルをテキストエディタで開くと以下のようになっています。
```
---
title: "Hello Hugo World"
date: 2017-10-13T20:33:56+09:00
draft: true
---

```

`---`で囲まれた内容はYAML形式で書かれたメタデータです。Front Matterといいます。`---`ではなく、`+++`で囲むとTOML形式、`{` `}`で括るとJSON形式で書くことができます。（Front Matterの詳細については[Hugo | Front Matter](https://gohugo.io/content-management/front-matter/)を参照してください。）

このFront Matterにはファイル名から自動生成したタイトルと記事の日時、ドラフトであることを示す`draft: true`があります。

Front Matterの下に記事を書いていきますが・・・ちょっとまって！！

| 文字コード | 改行コード |
| ---------- | ---------- |
| `UTF-8`    | `Lf`       |

で書きましょう。改行コードはぶっちゃけどうでもいいですが、初期値が`Lf`なので`Lf`で統一しましょう。

```
---
title: "Hello Hugo World"
date: 2017-10-13T20:33:56+09:00
draft: true
---
ほげほげ

## ふがふが
はげはげ
```

（また髪の話してる・・・。）ブラウザのページを見ると記事が表示されていることが確認できます。あとは適当にMarkdown記法で記事を書きます。記事が書けたら、Front Matterの`draft: true`を削除しておきます。

次に`config.toml`を編集します。デフォルトはTOML形式ですがYAML、JSON形式でも書けます。（Hugoのコンフィグについては[Hugo | Configure Hugo](https://gohugo.io/getting-started/configuration/)を参照してください。）

これも`UTF-8`で編集するようにしてください。`baseURL`、`languageCode`、`title`を適切なものに変更します。
```
baseURL = "https://taikii.github.io/gh-pages-example/"
languageCode = "ja"
title = "Hugo Example"
publishDir = "docs"
```

* `baseURL` を`https://ユーザ名.github.io/リポジトリ名/`に変更します。
* `languageCode` を`ja`に変更します。（日本語以外のサイトを作る場合はその言語に合わせて変更）
* `publishDir = "docs"` の行を追加します。これにより変換してできたHTMLファイルが`/docs`フォルダーに出力されるようになります。（デフォルトは`/public`フォルダーです。）

`config.toml`の変更は自動的には反映されません。一度`hugo server`を`Ctrl+C`で停止して、再度起動し直してください。また、`config.toml`の`baseURL`にサブディレクトリ`/gh-pages-example/`を含むURLを設定したので、`http://localhost:1313/`だと見れなくなっています。`http://localhost:1313/gh-pages-example/`にアクセスしてください。

ここまでできたら、`hugo`コマンドを実行してHTMLファイルを生成します。
```
$ hugo -t liquorice
Started building sites ...

Built site for language en:
0 draft content
0 future content
0 expired content
1 regular pages created
8 other pages created
0 non-page files copied
0 paginator pages created
0 tags created
0 categories created
total in 45 ms
```

`docs`フォルダーが作られて、その中にHTMLファイルが生成されているのが確認できると思います。では、コミットしてGitHub上の`master`ブランチにPushしましょう。
```
$ git add .
$ git commit -m "Initial commit"
$ git push --set-upstream origin master
```

GitHubのリポジトリ上に`/docs`フォルダーが作成されたため、設定画面を確認すると`mastar branch /docs folder`が選択できるようになっているはずです。これを選択して少し待ちます。

指定のURLをブラウザで開いてみると、見事にサイトが表示されているはずです。

## その他解説
### テーマごとのコンフィグ
上記で`config.toml`で指定した設定はHugoの基本的なものですが、テーマごとに`config.toml`に追加の設定項目が存在します。これについてはそれぞれのテーマの公式サイトを確認して下さい。

### コンフィグへのテンプレートの指定
`config.toml`に以下の行を追加することで、`hugo server`コマンドや`hugo`コマンドを実行する際に`-t`オプションでテンプレートを指定する必要がなくなります。
```
theme = "liquorice"
```

### タグ、カテゴリ
記事のヘッダに`categories`、`tags`を書くことで、カテゴリやタグを使用できます。タグもカテゴリも複数指定できるため、あまり違いは無いように思います。また、それぞれが使用できるかどうかはテンプレートによって決まります。
```
---
title: "Hello Hugo World"
date: 2017-10-13T20:33:56+09:00
categories = [ "hoge" ]
tags = [ "fuga", "foobar" ]
---
```

## 最後に
自分がつまづいた所をなるべく書いたつもりです。これからHugoを始めようと思っている方の足しになればと思います。
