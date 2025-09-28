# 掲示板app
※English translation is available below.

このフォルダには、会員登録から投稿・編集・削除まで備えたシンプルな掲示板アプリケーションが含まれています。PHP + MySQL (MAMP想定) で動作し、ログインユーザーのみが投稿を管理できます。

## 主な機能
- 会員登録・ログイン・ログアウト
- 会員情報の閲覧・更新 (`user_edit.php`)
- 投稿の新規作成、編集、削除
- 投稿一覧のページネーション (10件/ページ)
- 投稿の論理削除 (`deleted_at` 列)
- ログアウト時に自分の投稿をまとめて非表示化

## 必要環境
- PHP 8 系 (opcache 無効でも可)
- MySQL 5.7 以降 / MariaDB 同等
- MAMP を想定した接続設定 (ホスト `localhost`, ポート `8889`, ユーザー `root`, パスワード `root`)

## セットアップ手順
1. MAMP などで Web + DB サーバーを起動します。
2. MySQL に `bbs_app` データベースを作成します。
3. 以下のテーブルを作成します。必要に応じて `created_at` / `updated_at` のデフォルト値を調整してください。

```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE posts (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    name VARCHAR(255) NOT NULL,
    title VARCHAR(255) NOT NULL,
    message TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP NULL,
    CONSTRAINT FK_posts_user FOREIGN KEY (user_id) REFERENCES users(id)
);
```

4. `掲示板app/` を MAMP のドキュメントルートに配置し、`http://localhost:8888/掲示板app/login.php` などからアクセスします。
   - 日本語パスに対応していない環境では、別名 (例: `bbs-app/`) にリネームして利用してください。

## 画面の使い方
- **register.php**: 必須項目を入力後、確認画面 (`register_confirm.php`) を経由して会員登録します。登録完了後は自動でログイン状態になります。
- **login.php**: メールアドレス/パスワードでログインします。認証成功後は掲示板トップへリダイレクトされます。
- **index.php**: 投稿一覧と投稿フォームを表示します。自分の投稿のみ「編集」「削除」ボタンが出現します。
- **user_edit.php**: 氏名・メールアドレス・パスワードを更新できます。パスワードを空欄のまま送信すると、パスワードは据え置きになります。
- **logout.php**: セッションを破棄し、同時に自分の投稿 (`posts.deleted_at`) を `NOW()` で更新して非表示にします。

## 補足事項
- ページネーション: `page` パラメータで 1 ページ 10件表示。`deleted_at IS NULL` の投稿のみ取得します。
- 投稿フォーム: 編集ボタンを押すとフォームが編集モードに切り替わり、更新後は再び一覧に戻ります。
- 入力値は `htmlspecialchars` / `nl2br` でエスケープしてから表示します。
- セッション未ログイン時は `index.php` へアクセスすると `login.php` へリダイレクトされます。

## 開発メモ
- 認証関連のセッションキー: `$_SESSION['user']` (id, name) を使用。
- 新規機能を追加するときは CSRF 対策やより詳細なバリデーションの追加を検討してください。
- ログアウト時の一括論理削除は要件依存です。投稿を残したい場合は `logout.php` の更新 SQL を削除してください。

---

## English Description
### Overview
This folder contains a simple bulletin board application that supports user registration, authentication, posting, editing, and deletion. It runs on PHP and MySQL (MAMP defaults) and only authenticated users can manage their posts.

### Features
- User sign-up, login, and logout
- Profile editing through `user_edit.php`
- Create, edit, and delete posts
- Paginated post list (10 posts per page)
- Soft deletion using the `deleted_at` column
- Optional bulk soft-delete of the current user's posts on logout

### Requirements
- PHP 8.x (opcache may be disabled)
- MySQL 5.7+ or MariaDB equivalent
- MAMP-style connection settings (`localhost:8889`, user `root`, password `root`)

### Setup
1. Start your web and database server (e.g., MAMP).
2. Create a database named `bbs_app`.
3. Create the `users` and `posts` tables with the schema shown above.
4. Place `掲示板app/` inside your document root and open `http://localhost:8888/掲示板app/login.php`.
   - Rename the directory (e.g., `bbs-app`) if your environment cannot serve paths with Japanese characters.

### Usage
- `register.php`: Enter the required fields, confirm, and complete registration. Successful registration signs the user in automatically.
- `login.php`: Authenticate with email and password and you will be redirected to the board.
- `index.php`: View the post list and create new posts. Edit/Delete buttons appear only for your own posts.
- `user_edit.php`: Update your name, email, and optionally your password.
- `logout.php`: Destroys the session and marks all of your posts as deleted by setting `deleted_at` to `NOW()`.

### Notes
- The post list only shows rows where `deleted_at IS NULL` and supports page navigation via the `page` query parameter.
- Form inputs are sanitized with `htmlspecialchars` and `nl2br` before rendering.
- Unauthenticated access to `index.php` redirects to `login.php`.
- Consider adding CSRF protection and stricter validation if you extend the app.
