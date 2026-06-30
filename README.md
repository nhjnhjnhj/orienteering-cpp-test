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

> 実行は `cpp/` ディレクトリから行う前提。相対パスはこの場所を基準とする。
> `cpp/` フォルダ単体で自己完結しているため、別の環境へコピーする場合は `cpp/` まるごと持っていけば動く。

---

## ビルド方法

C++17 以降に対応したコンパイラが必要。

### macOS（VSCode + C/C++拡張）

> 注：Visual Studio for Mac は 2024年8月にサポート終了。代替として VSCode を使う。

1. **必要なツール**
   - Xcode Command Line Tools（`clang++` が含まれる）
     ```bash
     xcode-select --install
     ```
   - Visual Studio Code
   - VSCode 拡張機能：
     - `C/C++`（Microsoft 製、IntelliSense 用）
     - `C/C++ Extension Pack`（推奨）

2. **プロジェクトを開く**
   ```bash
   code /path/to/orienteering-course/cpp
   ```

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

5. **作業ディレクトリの設定（重要）**
   - プロパティ → `構成プロパティ` → `デバッグ` → `作業ディレクトリ`
   - `$(ProjectDir)..\` などに変更し、実行時に `cpp/input/` を相対パスで参照できるようにする
   - もしくは、ビルドした `.exe` を `cpp/` 直下にコピーして実行

6. **ビルド & 実行**
   - `ビルド` → `ソリューションのビルド`（または `Ctrl + Shift + B`）
   - `デバッグ` → `デバッグなしで開始`（`Ctrl + F5`）

### コマンドライン（macOS / Linux / WSL）

```bash
cd cpp
make           # ビルド
./orienteering # 実行
make clean     # オブジェクトファイル削除
```

---

## 注意事項

- 文字コード：現状 **UTF-8 / LF** で記述。コンペ提出時は **Shift-JIS / CR+LF** に変換が必要
- BOM 付き UTF-8 CSV にも対応済み（CSV 先頭の BOM を自動で除去）
- 乱数シードは `const.h` の `RANDOM_SEED` で固定（再現性確保のため）
