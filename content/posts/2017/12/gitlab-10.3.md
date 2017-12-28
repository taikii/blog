---
title: "GitlabがMermaidをサポートした話"
date: 2017-12-28T23:44:02+09:00
tags: ["GitLab"]
series: []
---
先日[GitLab 10.3](https://about.gitlab.com/2017/12/22/gitlab-10-3-released/)がリリースされましたが、その中でMermaidをサポートされるようになりました。
<!--more-->

[Mermaid](https://mermaidjs.github.io/)はJavaScriptのチャートを作成できるライブラリで、私はこれを気に入ってよく使っています。以前[RedmineのMermaidプラグイン]({{< relref "redmine-mermaid-macro-plugin.md" >}})を作ったりしました。

GitLabは以前よりPlantUMLをサポートしていましたが、私はUMLに準拠したドキュメントを書ことがほとんどないので、UMLとはいえないゆるい図を書けるMermaidはすごく重宝しています。見た目もきれいですしね。

また、AtomにはMermaidグラフを書くことができるパッケージ（[Markdown Preview Enhanced](https://atom.io/packages/markdown-preview-enhanced)とか）もあるので、AtomでMermaid入りのMarkdownを書いて、それがGitLab上でも同じように見えるのは嬉しいですね。

しかし、GitLabの開発ペースは本当に速いなぁ。