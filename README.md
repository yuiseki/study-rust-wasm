# study-rust-wasm

## Official Document

- https://www.rust-lang.org/ja/what/wasm
- https://rustwasm.github.io/
- https://rustwasm.github.io/wasm-pack/installer/
- https://rustwasm.github.io/docs/book/introduction.html
- https://moshg.github.io/rustwasm-book-ja/introduction.html
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

#### `wasm-pack build` でプロジェクトをビルドする
```bash
cd hello-wasm-pack
wasm-pack build
```

#### `npm init wasm-app www` でwasm-appを生成する
```bash
npm init wasm-app www
cd www
```

#### 重要
#### `www` の `.git` ディレクトリを消す

`npm init wasm-app www` で生成されたディレクトリはgitリポジトリになっているが、
今回は一つのリポジトリで複数のrust wasmプロジェクトを置きたいので、
`.git` ディレクトリを消す必要がある。

`.git` のある子ディレクトリはGitHubなどにPushできない。

```bash
rm -rf .git
```

#### `www` の `package.json` を更新する

edit `hello-wasm-pack/www/package.json`

dependenciesに `hello-wasm-pack` を追加する。

```json
  "dependencies": {
    "hello-wasm-pack": "file:../pkg"
  },
```

`package.json` を変更したので `npm i` を実行

```bash
npm i
```

#### 動作確認

```bash
npm run start
```

Webブラウザで http://localhost:8080/ にアクセスする


#### 変更をコミットする

```
git add .
git commit -m "add hello-wasm-pack"
```

## ライフゲームを作ってみる

#### rust-wasmプロジェクトの生成

```bash
cargo generate --git https://github.com/rustwasm/wasm-pack-template --name wasm-game-of-life
```

以下の手順をもう一度やる
- `npm init wasm-app www` でwasm-appを生成する
- `www` の `.git` ディレクトリを消す
- `www` の `package.json` を更新する
  - package名は `wasm-game-of-life`

#### `www` の `index.js` で `wasm-game-of-life` を読み込む

edit `wasm-game-of-life/www/index.js`

このファイルの `hello-wasm-pack` を `wasm-game-of-life` に変更する。
また、実装も変更する。

```js
import { Universe } from "wasm-game-of-life";

const pre = document.getElementById("game-of-life-canvas");
const universe = Universe.new();

const renderLoop = () => {
  pre.textContent = universe.render();
  universe.tick();

  requestAnimationFrame(renderLoop);
};

requestAnimationFrame(renderLoop);
```

#### `www` の `index.html` を変更する

edit `wasm-game-of-life/www/index.html`

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Hello wasm-pack!</title>
    <style>
      body {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
      }
    </style>
  </head>
  <body>
    <pre id="game-of-life-canvas"></pre>
    <script src="./bootstrap.js"></script>
  </body>
</html>
```

#### wasmの実装を変更していく

edit `wasm-game-of-life/src/lib.rs`

このファイルの `alert` と `greet` は消して以下の内容に変更する。

```rust
#[wasm_bindgen]
#[repr(u8)]
#[derive(Clone, Copy, Debug, PartialEq, Eq)]
pub enum Cell {
    Dead = 0,
    Alive = 1,
}

#[wasm_bindgen]
pub struct Universe {
    width: u32,
    height: u32,
    cells: Vec<Cell>,
}

impl Universe {
    fn get_index(&self, row: u32, column: u32) -> usize {
        (row * self.width + column) as usize
    }

    fn live_neighbor_count(&self, row: u32, column: u32) -> u8 {
        let mut count = 0;
        for delta_row in [self.height - 1, 0, 1].iter().cloned() {
            for delta_col in [self.width - 1, 0, 1].iter().cloned() {
                if delta_row == 0 && delta_col == 0 {
                    continue;
                }
                let neighbor_row = (row + delta_row) % self.height;
                let neighbor_col = (column + delta_col) % self.width;
                let idx = self.get_index(neighbor_row, neighbor_col);
                count += self.cells[idx] as u8;
            }
        }
        count
    }
}

/// Public methods, exported to JavaScript.
#[wasm_bindgen]
impl Universe {
    pub fn tick(&mut self) {
        let mut next = self.cells.clone();
        for row in 0..self.height {
            for col in 0..self.width {
                let idx = self.get_index(row, col);
                let cell = self.cells[idx];
                let live_neighbors = self.live_neighbor_count(row, col);
                let next_cell = match (cell, live_neighbors) {
                    // Rule 1: Any live cell with fewer than two live neighbours
                    // dies, as if caused by underpopulation.
                    (Cell::Alive, x) if x < 2 => Cell::Dead,
                    // Rule 2: Any live cell with two or three live neighbours
                    // lives on to the next generation.
                    (Cell::Alive, 2) | (Cell::Alive, 3) => Cell::Alive,
                    // Rule 3: Any live cell with more than three live
                    // neighbours dies, as if by overpopulation.
                    (Cell::Alive, x) if x > 3 => Cell::Dead,
                    // Rule 4: Any dead cell with exactly three live neighbours
                    // becomes a live cell, as if by reproduction.
                    (Cell::Dead, 3) => Cell::Alive,
                    // All other cells remain in the same state.
                    (otherwise, _) => otherwise,
                };
                next[idx] = next_cell;
            }
        }
        self.cells = next;
    }

    pub fn new() -> Universe {
        let width = 64;
        let height = 64;
        let cells = (0..width * height)
            .map(|i| {
                if i % 2 == 0 || i % 7 == 0 {
                    Cell::Alive
                } else {
                    Cell::Dead
                }
            })
            .collect();
        Universe {
            width,
            height,
            cells,
        }
    }

    pub fn render(&self) -> String {
        self.to_string()
    }
}

use std::fmt;

impl fmt::Display for Universe {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        for line in self.cells.as_slice().chunks(self.width as usize) {
            for &cell in line {
                let symbol = if cell == Cell::Dead { '◻' } else { '◼' };
                write!(f, "{}", symbol)?;
            }
            write!(f, "\n")?;
        }
        Ok(())
    }
}
```

#### `wasm-pack build` でプロジェクトをビルドする
```bash
wasm-pack build
```

#### 動作確認

```
npm i
npm run start
```

Webブラウザで http://localhost:8080/ を開く