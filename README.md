# ds-template-project

データサイエンス用テンプレート。Python のバージョン管理や依存関係は Astral の uv を使用する前提で構成されています。

## 重要なポイント
- Python の実行要件は [pyproject.toml](pyproject.toml) の `requires-python`（>=3.13）を参照してください。
- 依存関係のロックは [uv.lock](uv.lock) を使用します（pyenv は使用しません）。
- エントリポイント: [`main.main`](main.py)

## フォルダ構成（簡易ツリー）
```
.
├── .devcontainer/            # VS Code DevContainer 設定
│   └── devcontainer.json
├── .vscode/                  # ローカル VS Code 設定（推奨拡張・formatter等）
│   ├── settings.json
│   └── extensions.json
├── app/                      # アプリケーション（Streamlit / FastAPI 等）を置く想定
├── model/                    # 学習済みモデルやモデル定義を置く想定（大きなファイルは除外）
├── src/                      # ライブラリ／パッケージ化するソースコード
├── tests/                    # 単体テスト（pytest）を置く（存在しない場合は作成）
├── Dockerfile                # uv を使った Docker ビルド定義
├── compose.yaml              # Docker Compose / DevContainer 連携設定
├── pyproject.toml            # プロジェクト定義（依存／メタ情報）
├── uv.lock                   # uv によるロック済み依存（再現性確保）
├── main.py                   # 簡易エントリポイント（例）
├── .env / .env.example       # コンテナ起動時の環境変数サンプル
├── .python-version           # 情報表示用（uv を使うため pyenv ではない）
└── .gitignore
```

各フォルダの目的（短め）
- .devcontainer/: VS Code のコンテナ開発設定。DevContainer で compose.yaml の app サービスを拡張して起動します。参照: [.devcontainer/devcontainer.json](.devcontainer/devcontainer.json)
- .vscode/: 推奨拡張や保存時フォーマット設定が入っています。参照: [.vscode/settings.json](.vscode/settings.json)
- app/: Streamlit や簡易 Web UI を置く場所。Docker の CMD で起動することを想定（例: streamlit run app/app.py）。
- model/: 学習済みモデルや出力物を置くディレクトリ（通常は .gitignore で無視）。
- src/: パッケージ本体。パッケージ化してプロジェクトルートからインストールできる構成にする想定。
- tests/: pytest 用テストケースを置きます（DevContainer では pytest が有効）。
- uv.lock / pyproject.toml: 依存管理は uv を使い、ロックファイル（uv.lock）を CI・本番で使用して再現性を担保します。

## クイックスタート（ローカル）
1. uv をインストール（公式手順）:
   - curl -sS https://astral.sh/uv/install.sh | sh
2. 依存を同期:
   - 開発: uv sync
   - 再現性重視: uv sync --frozen（[uv.lock](uv.lock) を厳密に使用）
3. 実行:
   - python main.py  または uv 環境内の python を使用

## VS Code DevContainer / Docker
- DevContainer は [`.devcontainer/devcontainer.json`](.devcontainer/devcontainer.json) から compose.yaml を参照して起動します。
- Dockerfile は uv を使う構成になっています（[Dockerfile](Dockerfile) を参照）。

## 参照ファイル
- プロジェクト設定: [pyproject.toml](pyproject.toml)  
- 依存ロック: [uv.lock](uv.lock)  
- エントリポイント: [`main.main`](main.py)  
- コンテナ設定: [Dockerfile](Dockerfile) / [compose.yaml](compose.yaml)

## DevContainer 起動 / 開発フロー（実践手順）

以下はこのリポジトリを使って実際に開発を進める際の具体的な手順です。ファイル参照: [`.devcontainer/devcontainer.json`](.devcontainer/devcontainer.json), [compose.yaml](compose.yaml), [Dockerfile](Dockerfile), [uv.lock](uv.lock), [pyproject.toml](pyproject.toml), [main.py](main.py)。

前提
- Docker と docker-compose（または Docker Compose V2）がインストール済み
- VS Code と Remote - Containers（または Dev Containers）拡張がインストール済み

1) 環境変数の準備
   - リポジトリルートでサンプルからコピー:
     - cp .env.example .env
   - 必要であれば .env の値（CONTAINER_NAME / PORT）を調整

2) DevContainer（VS Code）で開発する（推奨）
   - VS Code を開き、コマンドパレットで「Dev Containers: Reopen in Container」または「Remote-Containers: Reopen Folder in Container」を選択
   - これにより [.devcontainer/devcontainer.json](.devcontainer/devcontainer.json) が参照する [compose.yaml](compose.yaml) の `app` サービスを使ってコンテナが起動します
   - コンテナ内ターミナルで依存を同期（起動時に自動で行われない場合）:
     - uv sync --frozen
   - 開発中の実行例:
     - ユニットテスト: pytest
     - アプリ起動（Streamlit 例）: streamlit run app/app.py --server.port ${PORT} --server.address 127.0.0.1
   - 終了: VS Code の Remote メニューで「Close Remote Connection」またはホストで:
     - docker compose -f compose.yaml down

3) CLI でコンテナを使う（VS Code を使わない場合）
   - ビルド & 起動:
     - docker compose -f compose.yaml up --build
   - コンテナに入る:
     - docker exec -it ${CONTAINER_NAME} /bin/bash
   - コンテナ内で依存同期:
     - uv sync --frozen
   - テスト／実行:
     - pytest
     - python main.py
     - streamlit run app/app.py --server.port ${PORT} --server.address 127.0.0.1
   - 停止:
     - docker compose -f compose.yaml down

4) ローカル（非コンテナ）での開発（uv を使う場合）
   - uv をインストール（ホスト側）:
     - curl -sS https://astral.sh/uv/install.sh | sh
   - プロジェクトで依存を同期:
     - uv sync
     - 再現性重視: uv sync --frozen（[uv.lock](uv.lock) を使用）
   - 実行 / テスト:
     - python main.py
     - pytest
     - streamlit run app/app.py --server.port ${PORT}

5) 開発のヒント
   - 依存を変更したら必ず uv lock を更新してコミット（uv の手順に従う）
   - 大きなデータ / 学習済みモデルは [model/](model/) に置くが .gitignore で除外済み
   - VS Code の設定や拡張は [.vscode/](.vscode/) と [.devcontainer/](.devcontainer/) を参照

