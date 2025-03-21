---
title: Paraview-Superbuildで失敗しないための操作メモ
tags:
  - 備忘録
  - paraview
private: false
updated_at: '2020-02-07T19:48:12+09:00'
id: 01141befdc5f8eefa95a
organization_url_name: null
slide: false
ignorePublish: false
---
ほぼほぼ自分用です。半分は備忘録で半分は愚痴。

# これはなに
Paraview-Superbuild使って任意の環境でParaviewをmakeするだけの簡単な作業。（簡単ではない）

公式READMEを読めという話ではあるのだけど、前提知識が多いのかなんなのか、行間を読む力が求められており、しょっちゅう詰まってしまう。。。[^2]

[^2]: CMake使ってmakefile作るのは一般教養の範囲らしい。苦しい。

何度も地雷踏んでしまって随分時間が掛かってしまったけど、うまく行ったところだけ抜き出すとすごく少ない作業で済んだので、作業ログとして残しておきます。

# 環境の用意

以下、Ubuntu想定です。

- ディレクトリを用意する

`/opt`はなんでも。sudo持って無くてかつ自分しか使うつもりがないなら`~/.local`とかですかね。

``` bash
# superbuildの作業用
mkdir -p /opt/paraview/src/superbuild
mkdir -p /opt/paraview/src/superbuild/ccmake_workspace
# インストール先
mkdir -p /opt/paraview/5.7.0
```

- apt install系

いくらsuperbuildとはいえあまりに基本的なライブラリはaptで入れろと怒られます。（build_essentials+CUDAくらいなら元々入ってそうだけど他は怪しい）

OpenGLは特に入れ忘れないようにしましょう。
HPCクラスタだと入ってないことが多いし、忘れててもcmakeは通ってしまうので。
make -jで初めてエラーが出るので、ドローバックが極めて大きくなる問題があります。[^1]

``` bash
sudo apt install -y cmake cmake-curses-gui mesa-common-dev
```

[^1]: 2020/2/7追記：cmakeのオプションによってはOpenGL無くても通るらしい（pvserverのみとか）

sudo持ってない人は必要ライブラリがすべてインストールされていることを祈りましょう。


- その他環境

CUDAとかMPIとか使うなら事前にPATH通しておかないとcmakeがエラーで落ちる。
module前提の環境だと忘れがちなので気をつけたい。。。

``` bash
# 例
module load cuda/10.1 openmpi/4.0.2
```

# superbuild開始

- superbuildのソース取ってくる

``` bash
cd /opt/paraview/src/superbuild
git clone --recursive https://gitlab.kitware.com/paraview/paraview-superbuild.git
cd paraview-superbuild
git checkout v5.7.0 -b v5.7.0_build
git submodule update
```

ここに罠がいくつかあります。

1. `--recursive`付けないとsubmoduleがおかしくなります。忘れないようにしましょう。[^1]

[^1]: つけ忘れたらあとで`git submodule init && git submodule update`としても動くけど、そもそもそんなこと思いつくなら`--recursive`忘れない

2. paraview-superbuild.gitのディレクトリとccmakeの作業用ディレクトリは入れ子になるとエラー吐きます。ディレクトリは分けましょう。

3. 最後の`git submodule update`を忘れるとバージョン不整合が起こるかも

- ccmake叩く

簡単だけど難しい。エラーが出なくなるまでオプションをつけたり消したりする無限ループに陥りがち。
以下は上手く行った例。

``` bash
cd ccmake_workspace
ccmake \
  -D CMAKE_INSTALL_PREFIX:STRING=/opt/paraview/5.7.0 \
  -D CMAKE_BUILD_TYPE:STRING=RelWithDebInfo \
  -D ENABLE_cuda:BOOL=ON \
  -D CMAKE_CUDA_HOST_COMPILER:STRING=g++ \
  -D ENABLE_ffmpeg:BOOL=ON \
  -D ENABLE_vortexfinder2:BOOL=ON \
  -D ENABLE_vtkm:BOOL=ON \
  -D ENABLE_python:BOOL=ON \
  -D ENABLE_numpy:BOOL=ON \
  -D ENABLE_matplotlib:BOOL=ON \
  -D ENABLE_scipy:BOOL=ON \
  -D ENABLE_ospray:BOOL=ON \
  -D ENABLE_png:BOOL=ON \
  -D ENABLE_paraview:BOOL=OFF \
  -D ENABLE_paraviewpluginsexternal:BOOL=ON \
  -D ENABLE_nvidiaindex:BOOL=ON \
   ../paraview-superbuild.git
```

`ENABLE_*`は目的別で完全に好みだけど、私の場合pvserverでNVIDIA-IndeXプラグインを使いたかったのでこんな感じになりました。

ここでは変数は`-D`で指定したけど、ccmakeの中でやっても大丈夫なはず。

- make

``` bash
make && make install
```

1行だけですね。簡単。（簡単じゃない）

だいたいここで失敗するので、エラーメッセージ見ながらccmakeのオプション変えてmakeやり直してを繰り返します。

あとめんどくさいけどやり直すときは必ずccmake_workspaceを空にしましょう。ゴミが残ってて次のmakeが無為にエラーを吐く場合があります。

``` bash
# PWD=/opt/paraview/src/superbuild
rm -rf ccmake_workspace
mkdir ccmake_workspace && cd ccmake_workspace
ccmake -D ....
```

- PATH通す

``` bash:.bashrc
paraview_path=/opt/paraview/5.7.0
export PATH+=:$paraview_path/bin
export LD_LIBRARY_PATH+=:$paraview_path/lib
```

- 念のため確認

``` console
$ pvserver --version
paraview version 5.7.0

$ pvserver
Waiting for client...
Connection URL: cs://***:11111
Accepting connection(s): ***:11111
Client connected.
```

よかったですね。

なお肝心のNVIDIA-IndeXでボリュームレンダリングすると、エラー吐いてクラッシュします。うーん……

# 参考

- https://www.rccm.co.jp/icem/pukiwiki/index.php?ParaView-Superbuild

- https://gitlab.kitware.com/paraview/paraview-superbuild

- https://www.paraview.org/Wiki/ParaView_SuperBuild

# 変更履歴
- 2020/2/7: 微妙にディレクトリを間違えていたので修正。その他文面の修正。
