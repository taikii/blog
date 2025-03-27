---
title: "Cosense（Scrapbox） TIL用カレンダー2025年版"
date: 2025-03-27T22:42:49+09:00
tags: ["Scrapbox", "TIL"]
---

以前 [ScrapboxでTIL]({{< relref "/posts/2022/10/scrapbox-til.md" >}}) という記事を書いたんですが、今更その2025年版カレンダーを作ったのでメモ。

<!--more-->

実際には月毎のページを作っています。

{{< gist taikii d1413115fa8302a6104c6370c40023f6 >}}

カレンダー作成用

```text
(
cal 01 2025 | tail -n +3 | sed -r 's/_\x08//g' | awk -v FIELDWIDTHS='2 3 3 3 3 3 3' -v OFS='\t' '$1=$1{for(i=1;i<=NF;i++){gsub(" ","",$i);if($i!=""){$i=sprintf("[2025-01-%02d]",$i);}else{$i="-"}}print}'
cal 02 2025 | tail -n +3 | sed -r 's/_\x08//g' | awk -v FIELDWIDTHS='2 3 3 3 3 3 3' -v OFS='\t' '$1=$1{for(i=1;i<=NF;i++){gsub(" ","",$i);if($i!=""){$i=sprintf("[2025-02-%02d]",$i);}else{$i="-"}}print}'
cal 03 2025 | tail -n +3 | sed -r 's/_\x08//g' | awk -v FIELDWIDTHS='2 3 3 3 3 3 3' -v OFS='\t' '$1=$1{for(i=1;i<=NF;i++){gsub(" ","",$i);if($i!=""){$i=sprintf("[2025-03-%02d]",$i);}else{$i="-"}}print}'
cal 04 2025 | tail -n +3 | sed -r 's/_\x08//g' | awk -v FIELDWIDTHS='2 3 3 3 3 3 3' -v OFS='\t' '$1=$1{for(i=1;i<=NF;i++){gsub(" ","",$i);if($i!=""){$i=sprintf("[2025-04-%02d]",$i);}else{$i="-"}}print}'
cal 05 2025 | tail -n +3 | sed -r 's/_\x08//g' | awk -v FIELDWIDTHS='2 3 3 3 3 3 3' -v OFS='\t' '$1=$1{for(i=1;i<=NF;i++){gsub(" ","",$i);if($i!=""){$i=sprintf("[2025-05-%02d]",$i);}else{$i="-"}}print}'
cal 06 2025 | tail -n +3 | sed -r 's/_\x08//g' | awk -v FIELDWIDTHS='2 3 3 3 3 3 3' -v OFS='\t' '$1=$1{for(i=1;i<=NF;i++){gsub(" ","",$i);if($i!=""){$i=sprintf("[2025-06-%02d]",$i);}else{$i="-"}}print}'
cal 07 2025 | tail -n +3 | sed -r 's/_\x08//g' | awk -v FIELDWIDTHS='2 3 3 3 3 3 3' -v OFS='\t' '$1=$1{for(i=1;i<=NF;i++){gsub(" ","",$i);if($i!=""){$i=sprintf("[2025-07-%02d]",$i);}else{$i="-"}}print}'
cal 08 2025 | tail -n +3 | sed -r 's/_\x08//g' | awk -v FIELDWIDTHS='2 3 3 3 3 3 3' -v OFS='\t' '$1=$1{for(i=1;i<=NF;i++){gsub(" ","",$i);if($i!=""){$i=sprintf("[2025-08-%02d]",$i);}else{$i="-"}}print}'
cal 09 2025 | tail -n +3 | sed -r 's/_\x08//g' | awk -v FIELDWIDTHS='2 3 3 3 3 3 3' -v OFS='\t' '$1=$1{for(i=1;i<=NF;i++){gsub(" ","",$i);if($i!=""){$i=sprintf("[2025-09-%02d]",$i);}else{$i="-"}}print}'
cal 10 2025 | tail -n +3 | sed -r 's/_\x08//g' | awk -v FIELDWIDTHS='2 3 3 3 3 3 3' -v OFS='\t' '$1=$1{for(i=1;i<=NF;i++){gsub(" ","",$i);if($i!=""){$i=sprintf("[2025-10-%02d]",$i);}else{$i="-"}}print}'
cal 11 2025 | tail -n +3 | sed -r 's/_\x08//g' | awk -v FIELDWIDTHS='2 3 3 3 3 3 3' -v OFS='\t' '$1=$1{for(i=1;i<=NF;i++){gsub(" ","",$i);if($i!=""){$i=sprintf("[2025-11-%02d]",$i);}else{$i="-"}}print}'
cal 12 2025 | tail -n +3 | sed -r 's/_\x08//g' | awk -v FIELDWIDTHS='2 3 3 3 3 3 3' -v OFS='\t' '$1=$1{for(i=1;i<=NF;i++){gsub(" ","",$i);if($i!=""){$i=sprintf("[2025-12-%02d]",$i);}else{$i="-"}}print}'
) | clip
```