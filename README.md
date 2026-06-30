# Step 5 C++ 版

## ファイル構成

```
cpp/
├── const.h            # 共通定数（コントロール数の制約・目標値・GAパラメータ等）
├── csv_loader.h/cpp   # CSV ロード（landmarks/nodes/edges）
├── graph.h/cpp        # 道路ネットワーク + Dijkstra + 経路キャッシュ
├── evaluate.h/cpp     # 染色体デコード + 4目的関数 + 評価関数
├── ga.h/cpp           # GA 操作（選択・交叉・突然変異・メインループ）
├── main.cpp           # エントリポイント・入出力
├── Makefile           # ビルド設定（コマンドラインから make で利用）
├── input/             # 入力 CSV ファイル
│   ├── landmarks.csv
│   ├── nodes.csv
│   └── edges.csv
├── output/            # 実行結果出力先
│   ├── best_course.json
│   └── fitness_history.csv
└── README.md
```

## 入出力

| 種別 | パス |
|---|---|
| 入力（CSV） | `input/landmarks.csv`, `input/nodes.csv`, `input/edges.csv` |
| 出力（JSON） | `output/best_course.json` |
| 出力（CSV）  | `output/fitness_history.csv` |

### 入力データの形式（`input/`）

事前に Python で生成した 3 つの CSV を読み込む。すべて UTF-8（BOM 可）/ ヘッダー行あり。

**`landmarks.csv`** … コントロール候補となる地点の一覧

| 列 | 意味 |
|---|---|
| `id` | 候補の通し番号 |
| `name` | 地点名 |
| `feature` | 種別（例：`post_office`） |
| `lat`, `lon` | 緯度・経度 |
| `nearest_node` | 最寄りの道路ノード ID（`nodes.csv` の `node_id` と対応） |
| `attraction_score` | 魅力度スコア |

**`nodes.csv`** … 道路ネットワークの地点（交差点など）

| 列 | 意味 |
|---|---|
| `node_id` | ノードの一意な ID |
| `lat`, `lon` | 緯度・経度 |
| `elevation` | 標高（m） |

**`edges.csv`** … ノード間を結ぶ道（有向エッジ）

| 列 | 意味 |
|---|---|
| `from_node`, `to_node` | 始点・終点のノード ID |
| `length_m` | 道の長さ（m） |
| `elevation_change` | 標高差（+ 登り / − 下り） |
| `elevation_gain` | 累積登り（m） |

---

## ビルド方法

C++17 以降に対応したコンパイラが必要。

### リポジトリの取得（git clone）※ macOS / Windows 共通

1. **作業用フォルダをローカルに作る**
   プロジェクトを置きたい場所に、任意の名前でフォルダを作る（Finder / エクスプローラーで作ってもよい）。
   ターミナル（macOS）/ コマンドプロンプト（Windows）で作る場合：
   ```bash
   mkdir orienteering      # 好きな名前・場所でよい
   cd orienteering         # 作ったフォルダに移動
   ```

2. **リポジトリをクローンする**
   上で移動したフォルダの中で：
   ```bash
   git clone https://github.com/nhjnhjnhj/orienteering-cpp-test.git
   cd orienteering-cpp-test   # クローンしてできたフォルダに移動
   ```

> **現在地について（重要）**
> いま居る `orienteering-cpp-test/` フォルダの**直下**に、`main.cpp` や `Makefile`、`input/` などが
> 展開されている（`cpp/` という入れ子フォルダにはならない）。
> これは下の「ファイル構成」で示した **`cpp/` の中身そのもの**。
> 以降の手順で「`cpp/` フォルダ」と書かれている箇所は、この**クローンしてできたフォルダ**
> （`orienteering-cpp-test/`）に読み替えること。

### macOS（VSCode + C/C++拡張）

1. **必要なツール**
   - Visual Studio Code
   - VSCode 拡張機能：
     - `C/C++`（Microsoft 製、IntelliSense 用）
     - `C/C++ Extension Pack`（推奨）
   > 注：Gitを使えない人はコンパイラのインストールが必要

2. **プロジェクトを開く**
   VSCode で `cpp/` フォルダを作成。
   以下の方法で：
   - **方法A：ドラッグ&ドロップ**
     Finder で `cpp/` フォルダを、VSCode のウィンドウ（またはアイコン）にドラッグ&ドロップする。
   - **方法B：メニューから開く**
     VSCode のメニュー `File` →「`Open Folder...`（フォルダを開く）」→ `cpp/` フォルダを選択する。

3. **ビルド & 実行（ターミナル経由）**
   VSCode 統合ターミナル（`Ctrl + ` ` で開く）で：
   ```bash
   make
   ./orienteering
   ```

4. **デバッガを使う場合**
   - `Run and Debug` → `launch.json を作成` → `C++ (GDB/LLDB)` → `clang++ build active file` を選択
   - F5 でビルド & デバッグ実行

### Windows（Visual Studio 2022 Community）

1. **必要なもの**
   - Visual Studio 2022 Community（無料）
   - インストール時に **「C++ によるデスクトップ開発」** ワークロードを選択

2. **新規プロジェクト作成**
   - `ファイル` → `新規作成` → `プロジェクト`
   - **空のプロジェクト（C++）** を選択
   - プロジェクト名：`orienteering`（任意）
   - 場所：`cpp/` の親フォルダなど

3. **ソースファイルの追加**
   - ソリューションエクスプローラで「ソースファイル」を右クリック → `追加` → `既存の項目`
   - 以下を追加：
     - `csv_loader.cpp`
     - `graph.cpp`
     - `evaluate.cpp`
     - `ga.cpp`
     - `main.cpp`
   - 同様に「ヘッダーファイル」へ `.h` を全て追加：
     - `const.h`, `csv_loader.h`, `graph.h`, `evaluate.h`, `ga.h`

4. **C++ 言語標準の設定**
   - プロジェクト右クリック → `プロパティ`
   - `構成プロパティ` → `全般` → `C++ 言語標準` → **ISO C++17 標準 (/std:c++17)** に設定
   - 構成は `すべての構成` を選択して Debug/Release 両方に適用
   - ※ 設定しないと `inline` 変数（`const.h`）で「`/std:c++17` が必要です」エラーが出る

5. **文字コードの設定（`/utf-8`）**
   - ソースは UTF-8 で記述しているが、日本語版 Windows の MSVC は既定で Shift-JIS（CP932）として読むため、
     日本語コメント・文字列で **警告 C4819**（「現在のコードページで表示できない文字を含んでいます」）が出る。
   - プロパティ → `構成プロパティ` → `C/C++` → `コマンドライン` → 「追加のオプション」に **`/utf-8`** を追加
     （`すべての構成` に適用）。これで MSVC がソースを UTF-8 として扱い、警告が消える。
   - ※ コンソールの日本語表示が文字化けする場合は、実行前に `chcp 65001`（コンソールを UTF-8 化）。
     JSON 出力（地点名など）は `/utf-8` の有無に関わらず UTF-8 で正しく保存される。

6. **input / output フォルダの扱い（重要）**
   - `input/`・`output/` は **プロジェクトに登録しない**（「ソースファイル」「リソースファイル」等に追加する必要はない）。
     これらは実行時にプログラムが**相対パスで読み書きする外部データ**であり、ビルド対象ではない。
   - プログラムは `input/landmarks.csv` のように相対パスで開く。基準は**実行時の作業ディレクトリ**。
     そのため「実行時に `input/` が見える場所」を合わせる必要がある（`output/` は実行時に自動生成されるので用意不要）。
   - 設定方法はどちらか：
     - **方法A（推奨）**：プロパティ → `構成プロパティ` → `デバッグ` → `作業ディレクトリ` を、
       `input/` のあるフォルダに設定する（例：`$(ProjectDir)..\` など、プロジェクトの作成場所に応じて調整）。
     - **方法B**：ビルドでできた `.exe`（`x64\Debug\` 等）と同じ場所に `input/` フォルダをコピーする。

7. **ビルド & 実行**
   - `ビルド` → `ソリューションのビルド`（または `Ctrl + Shift + B`）
   - `デバッグ` → `デバッグなしで開始`（`Ctrl + F5`）

### コマンドライン（macOS / Linux / WSL）

```bash
cd orienteering-cpp-test   # クローンしてできたフォルダ（= cpp/ の中身）へ移動
make           # ビルド
./orienteering # 実行
make clean     # オブジェクトファイル削除
```

---

## 注意事項

- 文字コード：現状 **UTF-8 / LF** で記述。コンペ提出時は **Shift-JIS / CR+LF** に変換が必要
- BOM 付き UTF-8 CSV にも対応済み（CSV 先頭の BOM を自動で除去）
- Windows のコンソールでの日本語文字化けは、`main.cpp` 冒頭の `SetConsoleOutputCP(CP_UTF8)`（`_WIN32` のみ有効）で対策済み
- 乱数シードは `const.h` の `RANDOM_SEED` で固定（再現性確保のため）

---

## トラブルシューティング

### `コード1で終了` / `CSV ファイルが開けません: input/landmarks.csv`

実行時に上記エラーで終了する場合、**作業ディレクトリの指定ミス**が原因（Visual Studio で頻発）。

- プログラムは `input/landmarks.csv` のように **相対パス**で開く。基準は**実行時の作業ディレクトリ**。
- ありがちな間違い：作業ディレクトリに `input` フォルダ**そのもの**を指定してしまう。
  ```
  作業ディレクトリ = C:\orienteering\input     ← ✗
   → 探しに行くのは C:\orienteering\input\input\landmarks.csv（input が二重）→ 失敗
  ```
- 正しくは、`input` フォルダの**親（input を含むフォルダ）**を指定する。
  ```
  作業ディレクトリ = C:\orienteering           ← ◯
   → C:\orienteering\input\landmarks.csv に正しく到達
  ```
- Visual Studio：プロパティ → `構成プロパティ` → `デバッグ` → 「作業ディレクトリ」を上記に修正。
  反映されない場合は「構成（Debug/Release）」「プラットフォーム（x64/Win32）」が実行中のものと一致しているか確認する。

### 日本語が文字化けする（Windows コンソール）

`SetConsoleOutputCP(CP_UTF8)` で対策済み。それでも化ける場合は、実行前にコンソールで `chcp 65001` を実行する。
