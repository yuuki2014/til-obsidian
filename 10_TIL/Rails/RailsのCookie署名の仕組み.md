---
date: 2026-01-17 17:55
tags:
  - TIL
  - Status/完了
  - Rails
  - Security
---
# RailsのCookie操作とセキュリティ署名
## 💡 学んだこと・解決した課題
- RailsにおけるCookieの基本的な読み書き方法と、セキュリティリスクを防ぐための「署名付き（Signed）」や「暗号化（Encrypted）」Cookieの使い分けについて。
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

### 2.セキュリティ（署名と暗号化）
```ruby
# 【署名付き】 改ざん防止 (ユーザーIDなど)
# ユーザーは値を見れるが、変更すると無効になる
cookies.signed[:user_id] = current_user.id
# 読み出し
cookies.signed[:user_id] 

# 【暗号化】 完全隠蔽 (割引コード、一時的な認証トークンなど)
# ユーザーは値を見ることも変更することもできない
cookies.encrypted[:discount] = 45
# 読み出し
cookies.encrypted[:discount]

# 【期限付き・永続化】
cookies.permanent[:login] = "XJ-122" # 20年有効
cookies.signed.permanent[:login] = "XJ-122" # チェーンも可能
```
### 3. オプション設定と削除
削除時の注意点：書き込み時に `:domain` を指定した場合、**削除時にも同じドメインを指定しないと消えない**。
```ruby
# オプション付きで書き込み
cookies[:name] = {
  value: 'cookie value',
  expires: 1.hour,       # 期限
  domain: 'domain.com',  # ドメイン指定
  secure: true,          # HTTPSのみ
  httponly: true         # JavaScriptからアクセス不可 (XSS対策)
}

# 削除 (ドメイン指定がある場合は必須！)
cookies.delete(:name, domain: 'domain.com')
```
## 🔗 参考リンク
- https://api.rubyonrails.org/v7.2/classes/ActionDispatch/Cookies.html
## 💭 感想・次への課題
- cookieを使う時は基本的に「cookies.signed」で良さそう
- permanentで期限を20年にしても、ブラウザのせいか、1年とちょっとの期限になるっぽい