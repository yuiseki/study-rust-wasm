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