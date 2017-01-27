---
title: docker composeでGoの開発環境を構築する
date: 2016-12-06 05:39:12
tags:
- docker
- golang
thumbnailImage: docker.png
thumbnailImagePosition: left
clearReading: true
---

この記事は[TokyoSWアドベントカレンダー](http://qiita.com/advent-calendar/2016/tokyosw)６日目の記事です。
業務では開発環境にDockerを使っていますが（といってもMySQLコンテナを立ててるだけ）、一から自分で設定したことはないので勉強のため導入してみようと思います。

<!-- more -->

## docker for macを入れる
開発環境はMacを想定してます。
少し前までならVirtualBox上にLinuxを乗せて更にそこにDockerを入れてコンテナを乗せて…というふうにやってましたが、その時代は終わりました。

今はMac/Win向けに公式が出してくれているのでそれを使います。
これでVirtualBoxともオサラバです。

docker for macを入れるとメニューバーにdockerちゃんのアイコンが表示され、
`docker`/`docker-machine`/`docker-compose`のコマンドが使えるようになります。

今回は主に`docker-compose`を使用して環境を構築します。

`docker-compose`とは複数のコンテナを同時に立ち上げ、さらに連携もできる便利なオーケストレーションツールです。
今回はアプリ用とデータストア用のコンテナを立ち上げるので非常に有用です。

## アプリコンテナとデータストアコンテナの設定

普段業務ではScalaを使って仕事をしていますが、今回は趣味で開発しているGolang製のナレッジ共有ツールの開発環境をDocker化させようと思います。

[snicket](https://github.com/SnippetsBucket/snicket)

ちなみにgoのhttpフレームワークである[echo](https://github.com/labstack/echo)を使っています。


アプリとデータストアを別々のコンテナとして立てます。
まずアプリ用コンテナを作るために以下の内容のDockerfileを作ります。
```Dockerfile
FROM golang:1.7
EXPOSE 5000
```

`.docker`ディレクトリを作成し、その下にDockerfileを置くようにしてみました。
これは好みがあると思いますが…
```sh
# プロジェクトルート
.docker
└── web
    └── Dockerfile
```

docker-composeは`docker-compose.yml`というファイルで設定するので、プロジェクト直下にこれを置きます。
ファイルは以下のように設定しました。

```yaml docker-compose.yml
version: '2'

services:
  app:
    build: .docker/web
    ports:
      - 5000:5000
    links:
      - mongo
    volumes:
      - .:/go/src/github.com/SnippetsBucket/snicket
    command: bash -c 'cd /go/src/github.com/SnippetsBucket/snicket/ && go run server.go'
    container_name: snicket-web

  mongo:
    image: mongo:3.4.0
    ports:
    - 27017:27017
    volumes:
    - mongo-data:/data/db
    container_name: snicket-mongo

volumes:
  mongo-data:
    driver: local

```

`app`と`mongo`の2つのコンテナを定義しています。
`app`は先程作ったDockerfileをビルドに使い、5000番ポートを開放し、mongoコンテナに繋ぐようにしています。
volumesでプロジェクトルートをコンテナにマウントし、コンテナ起動時にアプリをビルドしています。

※GOPATHについて
```sh
ホスト側と同じようにコンテナ内も同じようなディレクトリ構成にする必要があります。
Dockerfileで指定している公式イメージには既に/goが$GOPATHに設定されています。

ホスト側:$GOPATH/src/github.com/SnippetsBucket/snicket
コンテナ:$GOPATH/src/github.com/SnippetsBucket/snicket
```


`mongo`のvolumesは、通常はコンテナを再起動させた場合、コンテナ内のデータは消えてしまうためデータを永続化させるためにマウントさせています。

この時点でディレクトリ構成はこんな感じになりました。

```sh
.docker
└── web
│   └── Dockerfile
(中略)
└── docker-compose.yml
```


これで準備が整ったので`docker-compose up`をしてみます。
するとイメージのDLが行われ、ビルドされます。
おそらくmongoのコンソールがずらずらと表示されると思います。

別タブで`docker-compose ps`をしてみます。
コンテナ一覧の状態を見ることができます。

```sh
    Name                   Command               State                Ports
-----------------------------------------------------------------------------------------
snicket-mongo   /entrypoint.sh mongod            Up      0.0.0.0:27017->27017/tcp
snicket-web     bash -c cd /go/src/github. ...   Up      0.0.0.0:5000->5000/tcp, 8888/tcp
```


そして最後に`localhost:5000`にブラウザからアクセスすると画面が表示されました。

もちろん5000番ポートを開けているのでアプリ側でも5000番をリッスンするようにしておく必要があります。

```go server.go
func main() {
	router := router.Init()
	router.Run(standard.New(":5000"))
}
```

なんらかの理由でコンテナ内に入りたいときは`docker-compose exec <サービス名> bash`で入ることができます。
`<サービス名>`とはdocker-compose.ymlに書いたサービス名たちのことです。

最初`container_name`の名前を指定してハマってました…。

- コードの変更は次のようにコンテナを再起動すればOKです。
`docker-compose restart app`

- mongoにつなぎたいときは普通に`mongo`と打てば繋げます。

## まとめ
これでgolangの快適な開発環境ができました。

virtualboxを起動していない分、起動も速いですし余計なメモリも食いません。

docker-composeのお陰でdockerが簡単に扱えていいですね。

ぜひこれを機にdocker入門してみませんか？
