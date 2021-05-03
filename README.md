# study-rust-wasm

## Official Document

- https://www.rust-lang.org/ja/what/wasm
- https://rustwasm.github.io/
- https://rustwasm.github.io/docs/book/introduction.html
- https://developer.mozilla.org/en-US/docs/WebAssembly

## Install on Ubuntu 20.04

```bash
curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh
cargo install cargo-generate
npm install npm@latest -g
```

## 作業用ディレクトリ＆gitリポジトリを作る

```bash
mkdir study-rust-wasm
cd study-rust-wasm
git init
touch README.md
echo "target" > .gitignore
git add .
git commit -m "init"
```

## Hello worldをやってみる

#### rust-wasmプロジェクトの生成
```bash
cargo generate --git https://github.com/rustwasm/wasm-pack-template --name hello-wasm-pack
```

#### rust-wasmプロジェクトのビルド
```bash
cd hello-wasm-pack
wasm-pack build
```

#### Webページに組み込む
```bash
npm init wasm-app www
cd www
npm i
```

edit `hello-wasm-pack/www/package.json`

```json
  "dependencies": {
    "hello-wasm-pack": "file:../pkg"
  },
```

`package.json` を変更したのでもう一度 `npm i`

```bash
npm i
```

#### 動作確認

```bash
npm run start
```

Webブラウザで http://localhost:8080/ にアクセスする

#### 重要
#### `www` の `.git` ディレクトリを消す

`npm init wasm-app www` で生成されたディレクトリはgitリポジトリになっているが、
今回は一つのリポジトリで複数のrust wasmプロジェクトを置きたいので、
`.git` ディレクトリを消す必要がある。

`.git` のある子ディレクトリはGitHubなどにPushできない。

```bash
rm -rf .git
```


#### 変更をコミットする

```
git add .
git commit -m "add hello-wasm-pack"
```