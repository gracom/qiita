---
title: sudoなしの環境にTimes New Romanを入れてmapltotlibをいい感じにする
tags:
  - Linux
  - font
  - matplotlib
  - 備忘録
private: false
updated_at: '2023-01-19T18:55:12+09:00'
id: 2562dce68fdf79cea9cb
organization_url_name: null
slide: false
ignorePublish: false
---
sudoがないのでいつも憤死しそうになる。あればaptなりyumなりで`msttcore-fonts`を入れるだけなんですけどね。。
まあ頑張りましょう。


# フォントのインストール

フォントですがあまり知られていませんが`$HOME`配下にインストールすることが可能です。
あとTimes New Romanは`msttcore-fonts`なるパッケージに入っておりRPMが入手可能ですので、ここからフォントファイルを取り出してインストールします。具体的には以下：

```bash
# RPM取り出したら`etc`と`usr`なるディレクトリが作られて後で鬱陶しいのでtmpの下で作業する
mkdir fonts_downloads
cd fonts_downloads
# `msttcore fonts RPM`とかでググるとRPM配布ページが見つかる。以下は例
wget https://rpmfind.net/linux/sourceforge/m/ms/mscorefonts2/rpms/msttcore-fonts-installer-2.6-1.noarch.rpm
# `rpm2cpio`と`cpio`が無かったら泣きましょう。こいつらのインストール方法は知らん
rpm2cpio msttcore-fonts-installer-2.6-1.noarch.rpm | cpio -id
# うまくいけば`usr/share/fonts`の下にフォントが展開される。これを`$HOME`配下の適切なディレクトリにコピーする
# フォントインストール用のディレクトリは環境によって違うかもしれない（以下はRHEL7.8の場合）
font_install_dir=$HOME/.fonts
rsync -auv usr/share/fonts/ $HOME/.fonts/
```

# matplotlibの設定更新

キャッシュを消さないと新しいフォントを見つけてくれないらしいので消しましょう。@`matplotlib/3.6.2`
```
rm -rv .cache/matplotlib
```

フォントの設定は、`plt.rcParams`でやるのが常套だけどめんどくさい、と思っていたら`matplotlibrc`なるものがあるそうですね[^matplotlibrc]。
これ使えばスクリプトごとにrcParamsを書き直す必要がないので良いですね。

まずは自分の環境におけるmatplotlibrcの置き場所を調べましょう。
```
python3 -c 'import matplotlib as mpl; print(mpl.get_configdir())'
# 手元の（というかssh先だが）環境では`$HOME/.config/matplotlib`と出てきた
```

で、出てきたディレクトリの下にファイルを作成します。なおフォント以外の項目は趣味です。

```$HOME/.confg/matplotlibrc
font.family       : serif
font.serif        : Times New Roman
mathtext.fontset  : stix

axes.linewidth    : 0.5
xtick.major.width : 0.5
ytick.major.width : 0.5
figure.figsize    : 5.8, 4.1
font.size         : 12
```

以上です。あとは適当にプロットしたらデフォルトでTimes New Romanが有効になっています。

# テスト

```python
from matplotlib import pyplot as plt
import numpy as np

x = np.arange(0, 1, 0.01)
plt.plot(x, np.sin(2*np.pi*x), label=r'sin $2\pi x$')
plt.xlabel(r'$x$')
plt.ylabel(r'$y$')
plt.legend()
```

![hogehoge.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/450648/6db60b4f-4149-9579-08e9-4be5ee18d052.png)

よかったですね。


[^matplotlibrc]: https://hogelog.com/python/about-matplotlibrc.html

