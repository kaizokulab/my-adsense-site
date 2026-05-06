# AdSense サイト構築手順書

## 概要
GitHub Pages を使った静的サイトの構築・公開・独自ドメイン設定までの手順。

---

## 1. サイト作成

### ファイル構成
```
my-adsense-site/
├── index.html          # メインページ（記事10本）
├── about.html          # Aboutページ
├── privacy-policy.html # プライバシーポリシー
└── SETUP.md            # 本ドキュメント
```

### 各ページの役割
| ファイル | 内容 |
|---|---|
| `index.html` | 東京旅行Tips 記事10本・AdSlot配置済み |
| `about.html` | サイト紹介・連絡先 |
| `privacy-policy.html` | AdSense審査必須ページ |

---

## 2. GitHub リポジトリ設定

### 初期設定
```bash
# ブランチ名を main に変更
git branch -m master main

# リモートURLにトークンを含める形式で設定
git remote set-url origin https://<USERNAME>:<TOKEN>@github.com/kaizokulab/my-adsense-site.git

# push
git push -u origin main
```

### Personal Access Token (PAT) の取得
1. GitHub → Settings → Developer settings
2. Personal access tokens → Tokens (classic)
3. Generate new token → `repo` にチェック → Generate
4. 表示されたトークンをコピー（再表示不可）

### 認証情報の保存
```bash
git config --global credential.helper store
git push  # Username・Passwordを入力すると以後省略される
```

---

## 3. GitHub Pages の公開設定

1. リポジトリ → **Settings** → **Pages**
2. Source: `Deploy from a branch`
3. Branch: `main` / `/ (root)` → **Save**
4. 公開URL: `https://kaizokulab.github.io/my-adsense-site/`

> ⚠️ Private リポジトリは GitHub Pages 使用不可。Public に変更が必要。

---

## 4. 独自ドメイン設定

### 4-1. ドメイン取得
- Xserver Domain（https://www.xdomain.ne.jp）で取得
- 汎用的なドメイン名を推奨（内容変更に対応できるため）

### 4-2. Xserver DNS レコード追加

| タイプ | ホスト名 | 内容 |
|---|---|---|
| A | @ または空欄 | 185.199.108.153 |
| A | @ または空欄 | 185.199.109.153 |
| A | @ または空欄 | 185.199.110.153 |
| A | @ または空欄 | 185.199.111.153 |
| CNAME | www | kaizokulab.github.io |
| TXT | `_github-pages-challenge-kaizokulab` | `6f6c799498fb4035c4ea77464e1f96` |

> NSレコード（ns1.xdomain.ne.jp 等）は変更不要

### 4-3. GitHub Pages にドメインを設定
1. リポジトリ → Settings → Pages
2. Custom domain に `kaizokulab.com` を入力 → Save
3. DNS反映後（数時間〜24時間）に **Enforce HTTPS** をオン

### 4-4. ネームサーバー設定（重要）

DNSレコードを設定しただけでは不十分。ドメインのネームサーバーがXdomainを向いていないと世界中に反映されない。

**確認・変更手順:**
1. [XServer Domain](https://www.xdomain.ne.jp) にログイン
2. 対象ドメイン → **ネームサーバー設定**
3. 以下が設定されていることを確認（されていなければ変更して「設定を変更する」を押す）

```
ネームサーバー1: ns1.xdomain.ne.jp
ネームサーバー2: ns2.xdomain.ne.jp
ネームサーバー3: ns3.xdomain.ne.jp
```

> ネームサーバー変更後の反映には最大48時間かかる。

**トラブル症状:** `dig kaizokulab.com` で `SERVFAIL` または応答なし → ネームサーバーの設定ミスを疑う（"Lame Delegation" エラー）

### 4-5. DNS反映確認
```bash
# ネームサーバーに直接問い合わせ（設定直後でも確認可能）
dig @ns1.xdomain.ne.jp kaizokulab.com A +short

# Google DNS経由で確認（反映後）
curl -s "https://dns.google/resolve?name=kaizokulab.com&type=A" | python3 -c \
  "import sys,json; d=json.load(sys.stdin); [print(a['data']) for a in d.get('Answer',[])]"

# グローバル反映確認
dig kaizokulab.com A +short
```

---

## 5. 今後の作業（残タスク）

- [ ] DNS反映確認・HTTPS有効化
- [ ] Google AdSense 申請（URL: `https://kaizokulab.com`）
- [ ] AdSense審査通過後、広告コードを `.ad-slot` に貼り付け

---

## トラブルシューティング

| エラー | 原因 | 解決方法 |
|---|---|---|
| `src refspec main does not match` | ローカルブランチが `master` | `git branch -m master main` |
| `Authentication failed` | パスワード認証廃止 | PAT（Personal Access Token）を使用 |
| `Repository not found` | リポジトリ未作成 or URL誤り | GitHub でリポジトリを新規作成 |
| `Upgrade or make public to enable Pages` | リポジトリが Private | Public に変更 |
| `DNS check unsuccessful` | DNS未反映 | 24時間待って再確認 |
| `SERVFAIL` / 応答なし | Lame Delegation（NSとゾーンの不一致） | Xdomain「ネームサーバー設定」でns1-3.xdomain.ne.jpを設定 |
