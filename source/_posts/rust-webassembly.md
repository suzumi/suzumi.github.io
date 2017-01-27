---
title: RustからWebAssemblyにコンパイルしてみる
date: 2017-01-05 05:46:24
tags:
- rust
- webassembly
thumbnailImage: rust.png
thumbnailImagePosition: left
clearReading: true
---

Rust入門者です。WebAssembly(wasm)入門者です。
今までNightlyではサポートされてたようですが、stableなv1.14でも試験的にWebAssemblyがサポートされたそうなので試してみます。

<!-- more -->

## 準備

Rustを入れる必要があるので入れてない場合はrustupでインストールします。<br>
既にrustupでRustをインストールしてるけどバージョンが古い場合は`rustup update`で最新版にできます。multirust使ってる人は知りません。

```sh
# バージョン確認
$ rustc -V
rustc 1.14.0 (e8a012324 2016-12-16)
```

新しいwasmにコンパイルするためのターゲットをインストールします。
```sh
$ rustup target add wasm32-unknown-emscripten
```

後述の処理でcmakeが必要なので先に入れておく
```sh
$ brew install cmake
```

Emscripten SDKをダウンロードして解凍します。（ポチポチでDLしたい場合は[こちら](http://kripken.github.io/emscripten-site/docs/getting_started/downloads.html#download-and-install)）
```sh
# DL＆解凍
$ curl -O https://s3.amazonaws.com/mozilla-games/emscripten/releases/emsdk-portable.tar.gz
$ tar zxvf emsdk-portable.tar.gz

# インストール
$ cd emsdk_portable/
$ emsdk update
$ emsdk install sdk-incoming-64bit
$ emsdk activate sdk-incoming-64bit
```

`emsdk install sdk-incoming-64bit`を中でCのビルドする工程があるんですが、これがかなり長いです。(40分かかりました)

Emscriptenの環境変数を設定
```sh
$ source ./emsdk_env.sh
```

`emcc`のバージョンを確認します。
`1.37.0`以上が必要です。
```sh
$ emcc -v
emcc (Emscripten gcc/clang-like replacement + linker emulating GNU ld) 1.37.1
（後略）
```

## WebAssemblyの生成と実行

hello.rsを作ります。
```rust hello.rs
fn main() {
  println!("Hello world!!");
}
```

以下のコマンドで最適化オプションをつけてビルドします。
```sh
$ rustc --target=wasm32-unknown-emscripten hello.rs -o hello.html
```

コマンドを叩いた後、2〜3分プロンプトが返って来なくてヒヤヒヤしましたが成功すると以下のファイルが生成されました。
いっぱい生成されたけど使ってるのは`hello.js`と`hello.wasm`だけかな？(asm.jsはオマケで生成されてるのだろうか…)

- hello.asm.js
- hello.html
- hello.js
- hello.rs # さっき作ったファイル
- hello.wasm
- hello.wast

Chrome CanaryもしくはFirefox Nightlyで`chrome://flags/`にアクセスしてWebAssemblyを有効にします。
[f:id:suzumidokoro:20170105175622p:plain]


pythonでもphpでもnodeでもいいですが、今回はpythonでサクッとWebサーバを立てます。
```sh
$ python -m SimpleHTTPServer
```

`http://localhost:8000/hello.html`にアクセスするとコンソールに文字が出力されていると思います。

{% wide_image emscripten.png "emscriptenのコンソールに表示されている" %}

ということで無事WebAssemblyをブラウザで動かすことができました。

## まとめ

整理するとこんな感じでしょうか。

- WebAssemblyは簡単に言うとブラウザでの実行を想定したバイナリフォーマット。
- EmscriptenはLLVMベースにC/C++をJSやasm.jsに変換するコンパイラ
- Rustコード→LLVM IRを生成する(RustのEmsctipten対応)
- Emscriptenからwasmの生成
- hello.html中の`<script>`内でwasmをロード、hello.jsの中でwasmをコンパイルして実行

んーややこしい。

ここらへん疎いんでよくわかってないですが、もう少し深掘りしていきたいと思います。


## 参考
[Compiling Rust to WebAssembly Guide – Hacker Noon](https://medium.com/@chicoxyzzy/compiling-rust-to-webassembly-guide-411066a69fde#.ww7iitlup)
