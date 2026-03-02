# Deep Past Challenge — Colab-like Local Dev Environment (Docker)

このリポジトリは、Kaggle「Deep Past Challenge」向けの実装を **ローカルで Google Colab 風の環境**（Jupyter / Colab-like）で開発し、最終的に **Google Colab 上で実行**することを目的としています。

* **ローカル**: Docker で Colab 風環境を立ち上げて実装・検証
* **本番**: Notebook / スクリプトを Colab に持っていって実行（GPU など）

---

## 1. 前提条件

### 必須

* Docker Desktop / Docker Engine
* Docker Compose v2（`docker compose` が使える）
* NVIDIA GPU を使う場合:

  * NVIDIA Driver
  * NVIDIA Container Toolkit（Linux / WSL2 環境などで必要）

### 環境別の注意

* **Windows**: Docker Desktop + WSL2 推奨
* **Mac**: NVIDIA GPU は基本使用不可（GPU 実行は Colab 側で行う想定）

---

## 2. 使い方（Quick Start）

### 2.1 コンテナ起動

```bash
docker compose up
```

初回はイメージの pull が入るため時間がかかることがあります。

---

### 2.2 ブラウザでアクセス

```
http://localhost:8081
```

---

### 2.3 停止

```bash
docker compose down
```

---

## 3. docker-compose.yml

ローカルのカレントディレクトリ（このリポジトリ）をコンテナの `/work` にマウントします。
そのため、コンテナ内で作成/編集したファイルは **ホスト側にも反映**されます。

```yaml
services:
  colab-local:
    image: sorokine/docker-colab-local:latest
    ports:
      - "8081:8081"
    stdin_open: true
    tty: true
    volumes:
      - ./:/work
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
```

---

## 4. ディレクトリ構成（例）

推奨の置き方（必要に応じて変更してください）:

```
.
├── docker-compose.yml
├── README.md
├── notebooks/          # 実験用 Notebook
├── src/                # 学習 / 推論コード
├── data/               # Kaggleから取得したデータ
├── input/              # （任意）加工前データ
├── output/             # 特徴量 / モデル / 予測など
└── requirements.txt    # Colab側で入れるpip依存
```

### 推奨事項

* `data/` などの大容量データは `.gitignore` 対象にする
* Notebook は `notebooks/` にまとめる
* 実行コードは `src/` に整理する

---

## 5. ローカル開発 → Colab 実行のワークフロー

### Step A: ローカルで実装

1. コンテナ起動

   ```bash
   docker compose up
   ```

2. ブラウザでアクセス

   ```
   http://localhost:8081
   ```

3. `/work` 配下（= このリポジトリ）で Notebook / Python を編集・実行

4. 依存パッケージは `requirements.txt` に追記して管理

---

### Step B: Colab に持っていく

1. `notebooks/*.ipynb` または `src/` のコードを Colab にアップロード
   （または GitHub 連携でこのリポジトリを開く）

2. Colab で依存パッケージをインストール

   ```python
   !pip install -r requirements.txt
   ```

3. GPU を使う場合は Colab のランタイム設定で GPU を有効化

---

### Step C: Kaggle 提出

* 予測ファイル（例: `submission.csv`）を `output/` などに出力
* Kaggle にアップロードして submit

---

## 6. データの扱い

### Kaggle データ配置（例）

* Kaggle からダウンロードしたデータを `./data` に配置
* Notebook / コード内では `./data/...` を参照

### Colab 側での再現方法

以下のいずれかで同じディレクトリ構造を再現してください:

* Google Drive に配置
* Colab に直接アップロード
* Kaggle API (`kaggle` CLI) を使用

---

## 7. 永続化とファイルの扱い

### マウントの挙動

* ホストのカレントディレクトリ → コンテナ `/work`
* `/work` で編集した内容はホスト側に即時反映されます
* `docker compose down` してもローカルのファイルは消えません

---

## 8. よくあるトラブルシュート

### GPU が認識されない

確認項目:

* ホストで `nvidia-smi` が動くか
* NVIDIA Driver が正しくインストールされているか
* NVIDIA Container Toolkit が導入済みか

> 注意: `deploy:` は環境によって無視される場合があります。
> その場合は `gpus: all` を使う構成に変更すると解決することがあります。

---

### ファイルが見えない

* コンテナ内 `/work` = ホストのリポジトリ直下
* Notebook では以下を使用:

  * `/work/...`
  * `./...`

---

### ポートに接続できない

```bash
docker compose logs
```

でログを確認してください。

---

## 9. ライセンス / 注意

* Kaggle の利用規約に従ってデータ・成果物を扱ってください
* 本リポジトリは研究 / 学習目的です
