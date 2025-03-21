---
title: docker-compose/singularity でサクッと PostgreSQLサーバ を立てる
tags:
  - postgres
  - docker-compose
  - Singularity
private: false
updated_at: '2022-03-04T23:39:57+09:00'
id: 64a1b1ebee8ac0b8ea4f
organization_url_name: null
slide: false
ignorePublish: false
---
標記の件です。ここは手を抜くべきところですのでコピペして終わりにしましょう。~~こんなところでつまづいてないでさっさと研究した方がいい~~

# docker-compose版

Dockerが使える環境にある人は運がいい。脳死で動かせます。（ネットの記事も充実しているのでそっちを読んだ方がいいかもしれません。例えば[^1]）

下記のdocker-compose をコピペして `docker-compose up`すれば完了です。（まあセキュリティの都合でポートが空いてない場合はめんどくさいが。。）

```docker-compose.yml
version: "3"
services:
  postgres:
    image: postgres:14-alpine3.15
    container_name: postgres
    ports:
      - 5432:5432
    volumes:
      - ${PWD}/postgres/init:/docker-entrypoint-initdb.d
      - ${PWD}/postgres/data:/var/lib/postgresql/data
    environment:
      TZ: Asia/Tokyo
      POSTGRES_USER: optuna
      POSTGRES_PASSWORD: optunaxxxx
      POSTGRES_DB: optuna_db
      POSTGRES_INITDB_ARGS: --encoding=UTF-8
    restart: always
```

\* ユーザ名/パスワード/DB名がoptunaなのは[Optuna](https://www.preferred.jp/ja/projects/optuna/)を使ってみたかったからですね[^2]。実用上は別に何でもいいです。

# Singularity版

「Dockerありますか？」「そこになければないですね」
令和にもなって信じられない話ですがスパコン業界では常識です。まあCPU効率最適化とかあるので、仮想化してユーザにroot与えるとかは厳しいんでしょうね。。。

というわけで代わりにSingularity[^3]を使います。こっちは少しめんどくさい[^4]。

(1)  Postgresのイメージを手元に置く
```bash
singularity pull docker://postgres:14-alpine3.15
```
(2) 環境変数用のスクリプトを書く（singularityのbashrc相当）
```env.sh
export TZ=Asia/Tokyo
export POSTGRES_USER=optuna
export POSTGRES_PASSWORD=optunaxxxx
export POSTGRES_DB=optuna_db
export POSTGRES_INITDB_ARGS="--encoding=UTF-8"
``` 
(3) singularity runを実行する
```bash
mkdir -p postgres/{data,run}
. ${PWD}/env.sh

singularity run \
    -B ${PWD}/postgres/data:/var/lib/postgresql/data \
    -B ${PWD}/postgres/run:/var/run/postgresql \
    -e \
    -C --env-file ${PWD}/env.sh \
    ${PWD}/postgres_14-alpine3.15.sif \
```
以上です。[^5]

[^1]: https://qiita.com/mabubu0203/items/5cdff1caf2b024df1d95
[^2]: 話がそれますが、Optunaで使うときは ```study = optuna.create_study(study_name=<any_study_name>, storage='postgresql://optuna:optunaxxxx@<hostname>/optuna_db', load_if_exists=True)```
などとすればいいようです。
[^3]: Singularity: https://sylabs.io
[^4]: [singularity-compose](https://singularityhub.github.io/singularity-compose/#/) なんかもあるんですが、当方の環境では`fakeroot`が使えず断念しました。。。いい環境があれば追記するかも。
[^5]: なんで `env.sh` を2回呼んでるんや？と思った方は勘がいいです。本来は`singularity -C --env-file </path/to/env_file>`だけでいいはずです。ただ、これは私も良く分からないのですが、動いたり動かなかったりするので、念の為ホスト環境で事前にsourceした方が安全です。
