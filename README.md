# LINE Web Client (Unofficial)

公式の **LINE Chrome拡張機能** をパッチして、普通のWebサイトとして動くようにした非公式クライアントです。

> **注意**  
> これは非公式の実装です。LINEの利用規約に違反する可能性があります。  
> 自己責任で利用してください。

対応デプロイ先: **Render** / **Railway**（どちらも同じコード）

リポジトリ: https://github.com/shunichi19990314-oss/line

---

## 必要なもの

- Node.js 18以上
- Python 3 + `requests`（拡張機能の取得用）
  ```bash
  pip install requests
  ```

---

## セットアップ手順（ローカル）

### 1. リポジトリをクローン

```bash
git clone https://github.com/shunichi19990314-oss/line.git
cd line
```

### 2. 公式拡張機能を取得してパッチ

```bash
python3 scripts/fetch_and_patch.py
```

これで `www/` フォルダにパッチ済みのファイルが展開されます。

### 3. 依存関係をインストール

```bash
npm install
```

### 4. ローカルで起動

```bash
npm start
```

ブラウザで http://localhost:3000 を開いてください。

---

## Railway へのデプロイ手順

1. [Railway](https://railway.app) にログインし、**New Project → Deploy from GitHub repo** でこのリポジトリを選ぶ

2. 必要に応じて次を確認する

   | 項目 | 推奨 |
   |------|------|
   | **Build Command** | `pip install requests && python3 scripts/fetch_and_patch.py && npm install` |
   | **Start Command** | `npm start` |
   | **Healthcheck Path** | `/healthz` |

   `railway.toml` / `nixpacks.toml` がある場合、上記は自動で使われます。

3. **Settings → Networking → Generate Domain** で公開 URL を発行する

### 再デプロイ

GitHub の `main` が更新されたら Railway が自動で再ビルドします。  
手動の場合は Dashboard で **Redeploy** を実行してください。

---

## Render へのデプロイ手順

1. [Render Dashboard](https://dashboard.render.com) → **New → Web Service**
2. この GitHub リポジトリを接続
3. 設定:

   | 項目 | 値 |
   |------|-----|
   | **Runtime** | `Node` |
   | **Build Command** | `pip install requests && python3 scripts/fetch_and_patch.py && npm install` |
   | **Start Command** | `npm start` |

---

## 仕組みの概要

1. 公式 LINE Chrome 拡張機能をダウンロード
2. `main.js` / `ltsmSandbox.js` をパッチ
   - R4 を同一オリジン経由に変更
   - API ホストを same-origin に（path は `/api/...` 維持）
   - ログイン成功後に accessToken を先にセットしてから getProfile
3. Node.js (Hono) で静的配信 + プロキシ

- `/api/*` → `https://line-chrome-gw.line-apps.com`
- `/R4` → `https://ci.line-apps.com`

---

## 環境変数（任意）

| 変数 | 説明 |
|------|------|
| `PORT` | 待受ポート |
| `HOST` | 待受アドレス（既定 `0.0.0.0`） |
| `LINE_CHROME_VERSION` | 拡張バージョン上書き |
| `DEBUG_PROXY` | `1` でパスのみデバッグログ |

---

## 利用上の注意

1. **メインの大切なアカウントでは使わない**
2. 自動化・BOT 用途はしない
3. 公式アプリ／公式拡張がある場合はそちらを優先

非公式クライアントであるため、制限や BAN の可能性は残ります。
