# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 概要

FreeRADIUS v4(開発ブランチ)の Docker イメージを `linux/amd64` と `linux/arm64` でビルドし、Docker Hub の `cayenneryo/freeradius-server` へ push するためのリポジトリ。FreeRADIUS のコードは持たない。中身は、ビルド対象を固定する `build.env`、GitHub Actions のワークフロー、README のみ。

利用者は Koitohan(`jolly-jive/Koitohan`、このホストでは `/srv/docker`)。Koitohan は `freeradius/build/Dockerfile` の `FROM` でタグだけを固定し、digest は付けない。

## 仕組み

- `build.env` が唯一の設定元。タグは `<バージョン>-devel-${DIST}-<UPSTREAM_REF の先頭7桁>-r${BUILD}`(例: `4.0.40537-devel-ubuntu26-14fdfb3-r1`)。バージョンは手で書かず、`config` ジョブが上流の `./version.sh major . minor . commit_depth` で求める。`build` のテストが、`freeradius -v` の表示と一致することを確かめる。`devel` は開発ビルドの印。
- `.github/workflows/build.yml` は3つのジョブからなる。`config` がタグを決め、`build` がアーキテクチャごとにネイティブランナー(`ubuntu-24.04` / `ubuntu-24.04-arm`)でビルド・テストして digest で push し、`merge` が digest をまとめてタグを付ける。QEMU は使わない。
- Dockerfile は上流の `scripts/docker/build/<dist>/Dockerfile.service` をそのまま使う。このリポジトリには置かない。
- プルリクエストではビルドとテストのみ。push は `workflow_dispatch` で `push` を有効にしたときだけ行う。

## 守ること

- **公開済みのタグは上書きしない。** Koitohan が digest なしで固定しているため。同じコミットをビルドし直すときは `BUILD` を上げる。`config` ジョブの存在チェックを外さないこと。
- **上流の checkout は `fetch-depth: 0` と `fetch-tags: true` のままにする。** `version.sh` が `git describe`(タグ `branch_4_0_0` からのコミット数)でバージョンを決めるため、浅いクローンだとバージョン番号が変わる。Dockerfile もビルドコンテキスト内で `git clean` / `git reset` を実行するので `.git` が必要。
- `4.0.0-devel-` で始まるタグと `latest` は、今の命名にする前のもの。`4.0.0-devel-ubuntu26-14fdfb3` と `latest` は手で push した arm64 のみのイメージ。ワークフローからは触らない。
- 開発ビルド(`DEVELOPER BUILD`)のまま出している。`--disable-developer` への切り替えは未着手で、行うときは Dockerfile を上流のまま使う方針との兼ね合いを先に決める。

## 確認済みの事実(コミット `14fdfb3`、arm64)

- サーバーのバイナリは `freeradius`。`radiusd` というコマンドはなく、エントリーポイントが別名として受けるだけ。
- `docker run --rm <image> -v` は `FreeRADIUS version 4.0.40537 (git #14fdfb33)` を表示する。
- `docker run --rm <image> -XC` はイメージ標準の設定で通る(`Configuration appears to be OK`)。
- コンテナは root で起動し、サーバーが `freerad`(uid 101、gid 101)に降格する。
- ビルド中に `packages.networkradius.com` へアクセスする。amd64 以外では libkqueue をソースからビルドする。

## ローカルでの確認

このホスト(Raspberry Pi、arm64)でできるのは arm64 のビルドだけ。`~/Docker/build-and-push.sh` が `~/Docker/freeradius-server` のクローンを使ってビルドする(`PUSH=0` で push しない)。ただし既定の `DIST` は `ubuntu24`、タグは古い命名(`4.0.0-devel-<dist>-<hash>`)になる。push は必ずワークフローから行う。
