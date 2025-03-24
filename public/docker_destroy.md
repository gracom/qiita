---
title: Dockerグループ（非sudo）のユーザが `sudo rm -rf` する
tags:
    - "Docker"
    - "ネタ"
    - "備忘録"
private: false
updated_at: ""
organization_url_name: ""
slide: false
id: null
---

# これはなに

戒めです。環境構築芸人を自称しながらこんなことに5年気づかないとは。。。

# 想定される状況

あなたはラボのサーバ管理者です。一般権限のユーザから、Dockerが使いたいとの依頼がありました。

# 管理者
わかる、環境構築めんどくさいよね、Dockerじゃないと動かせない環境とかあるし。（[NGC](https://www.nvidia.com/ja-jp/gpu-cloud/)とか、[Optuna](https://github.com/optuna/optuna)使うときのPostgreSQLささっと立ち上げたいときとか[^1]）
Dockerグループに入れるだけなら、管理者権限渡すわけではないし、まあ、ええやろ。。。
というわけで

```bash
sudo usermod -aG docker sudou
```

# 須藤さん！？

何を思ったか一般ユーザさん、以下のコマンドを実行してしまいました。

```bash
docker run -v /:/root_of_host alpine rm -rf /root_of_host
```

サーバは無事死にました。。。[^2]

# じゃあどうすれば

真に管理者権限（ぽいもの）を渡さないdocker（のようなもの）があれば良いんですけどね。
たとえば以下が候補ですが、私の中ではまだ答えがないです。（だれかいい案あったらコメントください）（あと3年位前の知識なので、間違ってたら修正コメントください）

- [Singularity](https://sylabs.io/): スパコンユーザにはなじみ深いソフトですね。docker pull, run, execあたりは代替できそう。でもこいつbuild出来ないんだよなあ。。。
- [podman](https://podman.io/): そもそも非root用なのか？Docker Desktopが有料化された時の退避先という印象。~~筆者はインストールでこけた~~
- [Docker Rootless mode](https://docs.docker.jp/engine/security/rootless.html): Docker公式の非root版だ！と意気揚々と試したんですが、NFSマウントされたディレクトリをマウントできないという致命的な仕様があり、断念した記憶。
- [enroot](https://github.com/NVIDIA/enroot): 名前だけ知ってて試したことがない。


[^1]: 選定に悪意はないです
[^2]: 一応動作確認のために野良のWSLで叩いてみたんですが、`/mnt/c`の下が全削除されて危うく死ぬところでした。`$Recycle.Bin` の時点で気づいて何とか助かった。。。