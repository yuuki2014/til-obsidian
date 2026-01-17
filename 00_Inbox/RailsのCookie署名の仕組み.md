---
date: 2026-01-17 17:55
tags:
  - TIL
  - Status/書きかけ
  - Rails
  - Security
---

# RailsのCookie操作とセキュリティ署名
## 💡 学んだこと・解決した課題
RailsにおけるCookieの基本的な読み書き方法と、セキュリティリスクを防ぐための「署名付き（Signed）」や「暗号化（Encrypted）」Cookieの使い分けについて。
## 📝 詳細・原因
* **基本:** RailsのCookieは`ActionController::Cookies`経由で操作する。値は基本的にString型。
* **リスク:** 通常のCookieはユーザー側で中身を見たり**改ざん**したりできるため、認証情報などをそのまま保存するのは危険。
* **対策:** 用途に応じてメソッドを使い分ける必要がある。 
	* `signed`: 中身は見えるが、改ざんされると無効になる（改ざん防止）。
	* `encrypted`: 中身も見えず、改ざんもできない（機密情報用）。
## 💻 解決策・コード
### 1. 基本的な書き込みと読み出し  
```ruby
# 書き込み (基本はString)
cookies[:user_name] = "david"
# 複雑なデータはJSONにする
cookies[:lat_lon] = JSON.generate([47.68, -122.37])
# 読み出し
puts cookies[:user_name] # => "david"
```


## 🔗 参考リンク
## 💭 感想・次への課題