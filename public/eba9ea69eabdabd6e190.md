---
title: '[Vim] カーソル上の単語でvimgrep'
tags:
  - Vim
private: false
updated_at: '2019-06-30T21:51:06+09:00'
id: eba9ea69eabdabd6e190
organization_url_name: null
slide: false
ignorePublish: false
---
皆さんvimgrep使ってますか？便利ですよねvimgrep。ただ、「*」みたいな単語検索が無くて不便だったので作りました。
# 概要
``` ~/.vimrc
nnoremap " *:vimgrep /<C-r>// **/*.{cpp,cu,h,hpp}
```
これで「"」を押せば一発でvimgrepが走ります。以下詳細。

#やりたいこと
たとえば、関数の引数を変えたので関数を呼んでいるところをすべて修正したいという状況が考えられます（void foo(int a)をvoid foo(int a, float b)に書き直したなど）。
同一ファイルであれば単語検索（*ないし#）を使えばいいんですが、大抵の場合コードが複数ファイルに跨っているので、vimgrepなりtabnewなりで虱潰しに直していく必要があります。とはいえいずれも手数が掛かってめんどくさいので、mapに登録して楽をしましょうというやつです。
というわけで、冒頭に書いた一行の各要素を解説していきます。

## vimgrep
``` vim
:vimgrep [word] {src1 src2 ...]
```
検索文字列（word）を列挙したファイル（sec1 src2 ...）から探して一覧を出力してくれます。便利。
vimgrepでググるといろいろ出てくるけど、下記がわりと簡潔でわかりやすい（個人の感想です）
https://qiita.com/tsumac/items/3972d5347a5f6e37ca09

ただデフォルトだと検索文字列もファイル名も全部手打ちで書かないといけなくてめんどくさいので、そこも自動化していきます。

## *
カーソル上の単語を検索するVim標準コマンドです。すごく便利。hjklの次に重要といっても過言ではない（言い過ぎ）

## /\<C-r>//
「*」はファイル内検索でしか無いので、vimgrepに持っていくには一工夫が必要です。
そこでレジスタというやつを使ってやります（CPUの話ではないです）

https://qiita.com/0829/items/0b3f63798b6910623efc
↑の9. にある通り、vimでの検索文字列は「"/」なる名前のレジスタに登録されています。
このレジスタを貼り付けるにはCtrl+rのあとに/（スラッシュ）を打てばよいです。

また、左右をスラッシュで括っているのは正規表現に対応するためです。
（*での検索は単語の開始と終了を表す正規表現（\<と\>）がついているので）

## \*\*/\*.{cpp,cu,h,hpp}
「**/」はカレントディレクトリから階層的にファイル探索する、という意味らしいです。たとえば、

```
src/hoge/*
src/hoge/fuga/*
src/include/foobar/*
```

この辺りがすべてヒットするということですね。これも便利。

また、「\*.{cpp,cu,h,hpp}」は拡張子が.cpp, .cu, .hまたは.hppのファイルを指します。bashと同じです。
Pythonの人なら\*.py、FORTRANの人なら\*.{f,f90}などど変えればよいですね。

# おまけ
併せて使うと便利そうな設定をいくつか書いておきます。

## Gitで管理されているソースコードを対象としてvimgrep
Vimのコマンドモードでは任意のシェルコマンドが走るそうなので、それを使えば小題のようなこともできます。

``` vim
:vimgrep [word] `git ls-list`
```

## :cn（次の検索結果への移動）の簡略化
vimgrepでの検索結果の移動には:cnを打つ必要がありますが、数が多くなってくるとこれもわりとめんどくさいです。
これの簡略化方法としては、
① :cnをmapに登録する

```~/.vimrc
nnoremap > :cn<CR>
```
② Quickfixを使う

``` ~/.vimrc
autocmd QuickFixCmdPost vigrep cwindow$
```
の②通りが考えられますが、どちらでもお好みで良いかと思います。

# 参考文献（再掲含む）
1. vimgrep & 検索と置換
https://qiita.com/tsumac/items/3972d5347a5f6e37ca09

2. vimのレジスタ
https://qiita.com/0829/items/0b3f63798b6910623efc

3. Vimのレジスタを使う
https://qiita.com/a4_nghm/items/d29b35f5d056ef5da0d9

4. 【vimめも】 10. マッピング
https://qiita.com/r12tkmt/items/b89df403f587216802f1

5. vimgrepとQuickfix知らないVimmerはちょっとこっち来い
https://qiita.com/yuku_t/items/0c1aff03949cb1b8fe6b
