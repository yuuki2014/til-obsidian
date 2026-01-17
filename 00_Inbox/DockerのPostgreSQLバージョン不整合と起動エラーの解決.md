---
date: 2026-01-17 21:45
tags:
  - TIL
  - Status/書きかけ
---
# DockerのPostgreSQLバージョン不整合と起動エラーの解決

## 💡 学んだこと・解決した課題
- 昨日まで起動していたウェブアプリをローカルで起動したらエラーで起動せず
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
- ログを見てわかるように、postgresqlが18以上になってDockerイメージの保存場所の指定方法が変わりました。
## 💻 解決策・コード
```ruby

```


## 🔗 参考リンク
## 💭 感想・次への課題