---
title: "Hugo のテーマ Whiteplain に Bluesky シェアボタン作った"
date: 2024-02-29T22:38:24+09:00
tags: ["Hugo", "Whiteplain", "Bluesky"]
---

Bluesky がバージョンアップして念願のハッシュタグが実装されました！おめでとう！それと同時にインテントにも対応したため、せっかくなのでWhiteplainにシェアボタンを作りました。

> Bluesky: "📢 1.70 is rolling out now (5/6) (This one is a bit techy) Compose intent URLs! https:// bsky . app /intent/compose?text=”…” to open Bluesky with the composer open and prefilled with the text parameter." — Bluesky https://bsky.app/profile/bsky.app/post/3kmjboaopvn2f

<!--more-->

デフォルトでは非表示にしているので、Hugoのサイトルートの下に `layouts/partials/` というディレクトリを作成し、Whiteplainの `share.html` を持ってきてコメントアウトを解除します。

```sh
$ mkdir -p layouts/partials/
$ themes/whiteplain/layouts/partials/share.html layouts/partials/
```

こういう感じ。

```diff
--- themes/whiteplain/layouts/partials/share.html       2024-02-29 22:37:01.991898402 +0900
+++ layouts/partials/share.html 2024-02-29 22:37:07.331897687 +0900
@@ -15,14 +15,12 @@
     </a>
   </li>
   <!-- Sharingbutton Bluesky -->
-  <!--
   <li>
     <a class="resp-sharing-button__link" href="{{ printf "https://bsky.app/intent/compose?text=%s - %s %s" (.Title | safeURL) (.Site.Title | safeURL) (replace (.Permalink | absURL) "%" "%25") | safeURL }}" target="_blank" rel="noopener" aria-label="Bluesky">
       <div class="resp-sharing-button resp-sharing-button--bluesky resp-sharing-button--medium"><div aria-hidden="true" class="resp-sharing-button__icon resp-sharing-button__icon--solid">
         <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 568 501"><path d="M123.121 33.664C188.241 82.553 258.281 181.68 284 234.873c25.719-53.192 95.759-152.32 160.879-201.21C491.866-1.611 568-28.906 568 57.947c0 17.346-9.945 145.713-15.778 166.555-20.275 72.453-94.155 90.933-159.875 79.748C507.222 323.8 536.444 388.56 473.333 453.32c-119.86 122.992-172.272-30.859-185.702-70.281-2.462-7.227-3.614-10.608-3.631-7.733-.017-2.875-1.169.506-3.631 7.733-13.43 39.422-65.842 193.273-185.702 70.281-63.111-64.76-33.89-129.52 80.986-149.071-65.72 11.185-139.6-7.295-159.875-79.748C9.945 203.659 0 75.291 0 57.946 0-28.906 76.135-1.612 123.121 33.664Z"/></svg></div>Bluesky</div>
     </a>
   </li>
-  -->
 </ul>
 
 {{- end }}
 ```

よいBlueskyライフを。
