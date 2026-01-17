---
date: 2026-01-17 21:45
tags:
  - TIL
  - Status/書きかけ
---
# DockerのPostgreSQLバージョン不整合と起動エラーの解決

## 💡 学んだこと・解決した課題
- 昨日まで起動していたウェブアプリをローカルで起動したらエラーで起動せず
- `docker compose up` をすると、DB コンテナだけエラーで落ちる状態
## 📝 詳細・原因
```bash title:エラーログ
% docker compose logs db
db-1  | Error: in 18+, these Docker images are configured to store database data in a
db-1  |        format which is compatible with "pg_ctlcluster" (specifically, using
db-1  |        major-version-specific directory names).  This better reflects how
db-1  |        PostgreSQL itself works, and how upgrades are to be performed.
db-1  | 
db-1  |        See also https://github.com/docker-library/postgres/pull/1259
db-1  | 
db-1  |        Counter to that, there appears to be PostgreSQL data in:
db-1  |          /var/lib/postgresql/data (unused mount/volume)
db-1  | 
db-1  |        This is usually the result of upgrading the Docker image without
db-1  |        upgrading the underlying database using "pg_upgrade" (which requires both
db-1  |        versions).
db-1  | 
db-1  |        The suggested container configuration for 18+ is to place a single mount
db-1  |        at /var/lib/postgresql which will then place PostgreSQL data in a
db-1  |        subdirectory, allowing usage of "pg_upgrade --link" without mount point
db-1  |        boundary issues.
db-1  | 
db-1  |        See https://github.com/docker-library/postgres/issues/37 for a (long)
db-1  |        discussion around this process, and suggestions for how to do so.
```
- ログを見てわかるように、postgresqlが18以上になってDockerイメージの指定場所が変わっているので変更してくれと出てくる
## 💻 解決策・コード
```diff title:compose.yaml
services:
	db:
-		image: postgres
+		image: postgres:18
		restart: always
		environment:
			TZ: Asia/Tokyo
			POSTGRES_PASSWORD: password
+			POSTGRES_DB: myapp_development
		volumes:
-			- postgresql_data:/var/lib/postgresql/data
+			- postgresql_data:/var/lib/postgresql
		ports:
			- 5432:5432
		healthcheck:
			test: ["CMD-SHELL", "pg_isready -d myapp_development -U postgres"]
```
- `image: postgres:18` バージョンを指定しておくことで、次回バージョンアップ時でも起動できるようにしておく
- `POSTGRES_DB: myapp_development` 起動時に自動でこの名前のDBを作成
- `postgresql_data:/var/lib/postgresql` バージョン18に合わせた場所指定

```bash title:コンテナ停止とボリューム削除
docker compose down -v
```
- これでDockerの中にあった「ライブラリの保存場所（node_modules）」を完全に消去するため、色々入れ直さないといけない。
- データベースの中身も削除されるのでマイグレーションをし直す

```bash title:コンテナビルドし直し
docker compose build
```

```bash title:マイグレーションし直し
docker compose exec web rails db:migrate
```

```bash title:gemのインストールし直し
docker compose exec web bundle install
```

```bash title:Javascriptパッケージインストールし直し
docker compose exec web yarn install
```
## 🔗 参考リンク
https://qiita.com/jojo__xxxxx/items/325a9ae66290066ee8cc
## 💭 感想・次への課題
- ログを見ればすぐにわかることだったので、ログをしっかり読むこと
- 本番環境のpostgresqlのバージョンと開発環境を合わせるようにしておく