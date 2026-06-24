# AINAScan VibeGuard — AIコードセキュリティスキャナー

**バイブコーディングで作ったコード、デプロイ前に一度確認してください。**

AIコーディングアシスタント（Claude、GPT、Cursor）が繰り返し生成する**セキュリティ脆弱性33種 + バイブコーディングバグ15種 = 48パターン**を検出します。9言語対応。LLMなしの決定論的AST分析。

[![無料ベータ](https://img.shields.io/badge/ベータ-無料-brightgreen)](https://pleasing-transformation-production-90c2.up.railway.app)
[![9言語](https://img.shields.io/badge/言語-9種-blue)](#対応言語)
[![48パターン](https://img.shields.io/badge/パターン-48種-orange)](#検出パターン)
[![P=R=F1=100%](https://img.shields.io/badge/F1-100%25-success)](#パフォーマンス)
[![GitHub](https://img.shields.io/badge/GitHub-aina--scan-181717?logo=github)](https://github.com/Moonsehwan/aina-scan)

---

## バイブコーディングの問題

AIで素早くコードを作るのは良いことです。問題は、AIアシスタントが**実行はできても動作しない**コードを繰り返し生成することです。

```python
# AIが書いたコード — 保存関数なのに実際には保存しない (MISSING_WRITE)
def save_user(data: dict):
    validated = validate(data)
    return {"status": "saved", "id": validated["id"]}  # ← INSERTがない

# AIが書いたコード — イベントループをブロック (FAKE_ASYNC)
async def fetch_data(url: str):
    response = requests.get(url)   # ← awaitがない
    return response.json()

# AIが書いたコード — SQLインジェクション (SQL_INJECTION_RISK)
def get_user(username: str):
    return db.execute(f"SELECT * FROM users WHERE name='{username}'")
```

AINAScanはこれらのバグを**1秒以内**に検出します。本番環境に出る前に。

Semgrep、Bandit、Snykなどの既存ツールはAI特有のパターンを検出できません。AINAScanはバイブコーディングバグに特化しています。

---

## はじめに

### 方法1 — Webアプリ（インストール不要、最も簡単）

**👉 [https://pleasing-transformation-production-90c2.up.railway.app](https://pleasing-transformation-production-90c2.up.railway.app)**

1. 上記リンクにアクセス
2. ファイルをドラッグ＆ドロップ
3. 結果をすぐ確認 — BLOCK（必ず修正）/ WARN（要確認）

APIキー `vg_free_test` があらかじめ入力されています。登録不要。

> **プライバシー：** アップロードされたファイルは分析後すぐにメモリから削除されます。ディスクには保存されません。

---

### 方法2 — REST API（自動化・CI/CD連携）

```bash
# 無料キー — 登録不要
curl -X POST https://pleasing-transformation-production-90c2.up.railway.app/v1/scan \
  -H "X-API-Key: vg_free_test" \
  -F "file=@your_file.py"
```

**無料ベータキー：** `vg_free_test` — オープンベータ期間中、Pro全機能を無料で利用できます。

対応拡張子：`.py` `.js` `.ts` `.tsx` `.go` `.java` `.kt` `.php` `.rb` `.c` `.cpp`

---

## プロジェクト全体のスキャン

### Python

```python
import requests, pathlib, json

API = "https://pleasing-transformation-production-90c2.up.railway.app"
KEY = "vg_free_test"

results = []
for path in pathlib.Path(".").rglob("*"):
    if path.suffix not in {".py",".js",".ts",".go",".java",".php",".rb",".kt",".c",".cpp"} or not path.is_file():
        continue
    with open(path, "rb") as f:
        r = requests.post(f"{API}/v1/scan", headers={"X-API-Key": KEY}, files={"file": (path.name, f)})
    data = r.json()
    if data.get("block_count", 0) > 0:
        results.append({"file": str(path), "blocks": data["block_count"], "issues": data["issues"]})

print(json.dumps(results, indent=2, ensure_ascii=False))
```

### Bash

```bash
API="https://pleasing-transformation-production-90c2.up.railway.app"
KEY="vg_free_test"
find . -name "*.py" -o -name "*.js" -o -name "*.ts" | while read file; do
  result=$(curl -s -X POST "$API/v1/scan" -H "X-API-Key: $KEY" -F "file=@$file")
  blocks=$(echo "$result" | python3 -c "import sys,json; print(json.load(sys.stdin).get('block_count',0))")
  [ "$blocks" -gt 0 ] && echo "BLOCK [$blocks] $file"
done
```

---

## APIリファレンス

### 1. コードスキャン — POST /v1/scan

```bash
curl -X POST .../v1/scan \
  -H "X-API-Key: YOUR_KEY" \
  -F "file=@app.py"
```

**レスポンス：**
```json
{
  "scan_id": "a1b2c3d4-...",
  "filename": "app.py",
  "language": "python",
  "passed": false,
  "block_count": 2,
  "warn_count": 1,
  "issues": [
    {
      "kind": "SQL_INJECTION_RISK",
      "severity": "BLOCK",
      "line": 47,
      "detail": "f-string used in SQL — use parameterized queries",
      "function_name": "get_user",
      "disable_hint": "# ainascan-disable SQL_INJECTION_RISK"
    }
  ]
}
```

**クエリパラメータ：**

| パラメータ | デフォルト | 説明 |
|-----------|-----------|------|
| `include_senior` | `true` | シニアコード品質パターンを含む (Python) |
| `enable_aina_advisor` | `true` | AINA因果連鎖分析を有効化 |

---

### 2. プロジェクトドキュメント生成 — POST /v1/docs ⭐ 新機能

プロジェクトのzipをアップロードすると、AIコンテキスト用の構造化ドキュメントが生成されます。

```bash
zip -r myproject.zip ./src
curl -X POST .../v1/docs \
  -H "X-API-Key: vg_free_test" \
  -F "file=@myproject.zip" \
  -G -d "tier=pro&lang=ja"
```

**生成セクション（7つ）：**

| セクション | ティア | 内容 |
|-----------|--------|------|
| 📁 プロジェクト構造 | 無料 | ファイルツリー + 関数/クラス数 |
| 🔧 関数リスト | 無料 | 全関数（パラメータ、戻り型、行番号） |
| 📦 依存関係 | 無料 | requirements.txtからの外部パッケージ |
| 🔗 インポートグラフ | Pro | Mermaidダイアグラム — ファイル間依存関係 |
| 🌐 APIエンドポイント | Pro | 全エンドポイント（メソッド、パス、ハンドラー、行番号） |
| 🛡️ セキュリティ状態 | Pro | ファイルごとのBLOCK/WARN数 |
| 🪦 デッドコード | Pro | どこからも呼び出されていない関数 |

生成されたMarkdownをAIアシスタントに貼り付けることで、重複コード、誤ったインポート、ルートの衝突を防ぐことができます。

---

### 3. 誤検知レポート — POST /v1/feedback

```bash
curl -X POST .../v1/feedback \
  -H "X-API-Key: vg_free_test" \
  -H "Content-Type: application/json" \
  -d '{
    "scan_id": "your-scan-id",
    "vuln_type": "HARDCODED_TABLE",
    "is_false_positive": true,
    "note": "意図的な設定テーブル、DBルックアップ不要"
  }'
```

---

### 4. その他のエンドポイント

```bash
# スキャン履歴（最近20件）
curl -H "X-API-Key: vg_free_test" .../v1/report/history

# パターン統計
curl -H "X-API-Key: vg_free_test" .../v1/stats
```

---

### 5. インライン無効化 — ainascan-disable

```python
# ainascan-disable HARDCODED_TABLE
COUNTRY_CODES = {"JP": "日本", "US": "アメリカ"}  # 意図的なルックアップテーブル
```

---

## 検出パターン（48種）

### セキュリティ脆弱性（33種）

| パターン | 重大度 | 検出対象 |
|---------|--------|---------|
| `SQL_INJECTION_RISK` | BLOCK | f-string/% でSQL構築 |
| `COMMAND_INJECTION` | BLOCK | ユーザー入力 → subprocess/exec/system |
| `HARDCODED_SECRET` | BLOCK | api_key/password に文字列リテラル |
| `PATH_TRAVERSAL` | BLOCK | 未検証のファイルパス操作 |
| `XSS_RISK` | BLOCK | innerHTML/document.write + ユーザーデータ |
| `EVAL_EXEC_RISK` | BLOCK | eval()/exec() + ユーザー入力 |
| `DESERIALIZATION_RISK` | BLOCK | pickle.loads/unserialize 信頼できないデータ |
| `DEBUG_MODE_RISK` | BLOCK | debug=True + host="0.0.0.0" |
| `IDOR_MISSING_AUTH_CHECK` | BLOCK | 認可チェックなしの直接オブジェクトアクセス |
| `NOSQL_INJECTION` | BLOCK | 未検証のMongoDBクエリ |
| `BUFFER_OVERFLOW` | BLOCK | C/C++ gets/strcpy/sprintf |
| `FORMAT_STRING` | BLOCK | C/C++ printf(var) フォーマット引数なし |
| `USE_AFTER_FREE` | BLOCK | C/C++ free()後のポインタ使用 |
| `NULL_DEREF` | BLOCK | C/C++ malloc NULLチェック欠落 |
| `INTEGER_OVERFLOW` | BLOCK | C/C++ malloc(atoi(input)) |
| `SSRF_RISK` | WARN | 未検証URLのサーバーサイドfetch |
| `OPEN_REDIRECT` | WARN | 未検証のリダイレクト先 |
| `CSRF_MISSING_PROTECTION` | WARN | CSRFトークンなしの状態変更エンドポイント |
| `WEAK_CRYPTO` | WARN | MD5/SHA1の使用 |
| `CORS_WILDCARD` | WARN | origins='*' |
| + 13種 | WARN | XXE、LDAPインジェクション、テンプレートインジェクションなど |

### バイブコーディングバグ（15種）— AIコード専用

| パターン | 重大度 | AIが生成するパターン |
|---------|--------|-------------------|
| `STUB_SKELETON` | BLOCK | `def save(): return {}` — プレースホルダーが残ったまま |
| `MISSING_WRITE` | BLOCK | 保存/挿入関数に実際のDB書き込みなし |
| `FAKE_ASYNC` | BLOCK | `async def` に `await` なし — イベントループをサイレントブロック |
| `INPUT_OUTPUT_DISCONNECTED` | BLOCK | パラメータが戻り値にまったく影響しない |
| `DEAD_CALL_RESULT` | BLOCK | 3つのモジュール呼び出し結果を全て破棄 |
| `HARDCODED_TABLE` | BLOCK | 8キー+辞書リテラルをDBクエリの代わりに使用 |
| `TRIVIAL_IF_CHAIN` | BLOCK | 7+のelifチェーンをDBクエリの代わりに使用 |
| `MOCK_PATTERN` | BLOCK | プロダクションコードにunittest.mockが残存 |
| `ENCODING_CORRUPTION` | BLOCK | スマート引用符/BOM → ランタイムUnicodeDecodeError |
| `TYPE_UNSAFE_ACCESS` | BLOCK | `float(d.get('key'))` — NoneでTypeError |
| `DB_SCHEMA_DRIFT` | BLOCK | SQLがDBに存在しないカラムを参照 |
| `LLM_DELEGATION` | BLOCK | LLMの応答を自身の推論なしにそのまま返す |
| `CONST_SQL_NO_PARAM` | BLOCK | SQL WHERE句にハードコードされた値 |
| `TRIVIAL_ASSERT` | WARN | テスト関数に `assert True` のみ |
| `FAKE_ASYNC` | WARN | asyncキーワードのみ、awaitなし（低確信度） |

---

## 自動誤検知削減

**ライブラリコード検出** — `__all__`、`ABCMeta`、`@abstractmethod` が3回以上現れる場合、高ノイズパターンを自動的にWARNに降格。

**テストファイル検出** — `test_*.py`、`*_test.py`、`conftest.py` では、STUB/MOCKパターンを自動的にWARNに降格。

---

## GitHub Actions連携

`.github/workflows/ainascan.yml`：

```yaml
name: AINAScan
on: [pull_request]
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: Moonsehwan/aina-vibeguard-action@v1
        with:
          api-key: ${{ secrets.AINASCAN_API_KEY }}
          fail-on-block: 'true'
```

→ [aina-vibeguard-action](https://github.com/Moonsehwan/aina-vibeguard-action)

---

## パフォーマンス

| バージョン | Precision | Recall | F1 | テストケース |
|-----------|-----------|--------|----|------------|
| v5.0 | 93.8% | 83.3% | 88.2% | 30 |
| v7.0 | 100% | 100% | 100% | 60 |
| **v7.3** | **100%** | **100%** | **100%** | **90** |

- スキャン速度：500KB以下のファイルは2秒以内
- キャッシング：同一ファイルは再スキャン時に即座に返却
- 100k+⭐のオープンソースリポジトリ10個で誤検知ゼロを確認

---

## 対応言語

| 言語 | 拡張子 |
|------|--------|
| Python | `.py` |
| JavaScript | `.js` `.mjs` `.cjs` |
| TypeScript | `.ts` `.tsx` |
| Go | `.go` |
| Java | `.java` |
| PHP | `.php` |
| Ruby | `.rb` |
| Kotlin | `.kt` `.kts` |
| C | `.c` `.h` |
| C++ | `.cpp` `.cc` `.cxx` `.hpp` `.hxx` |

---

## 比較

| | AINAScan | Bandit | Semgrep | CodeRabbit |
|--|---------|--------|---------|------------|
| バイブコーディングパターン（15種） | ✅ 専用 | ❌ | ❌ | ❌ |
| プロジェクトドキュメント生成 | ✅ /v1/docs | ❌ | ❌ | ❌ |
| 決定論的AST（LLMなし） | ✅ | ✅ | ✅ | ❌ |
| 9言語 | ✅ | ❌ Pythonのみ | ✅ | ✅ |
| コード非保存 | ✅ | ✅ | ✅ | ❌ |
| GitHub Action | ✅ | ✅ | ✅ | ✅ |
| 無料ティア | ✅ 50ファイル/日 | ✅ | 制限あり | ❌ |

---

## プライバシーとセキュリティ

> アップロードされたファイルは絶対に保存されません。分析はメモリのみで行われ、リクエスト完了後すぐに破棄されます。

| 項目 | 処理方法 |
|------|---------|
| アップロードファイル | 分析後にメモリから即削除。ディスク書き込みなし。 |
| ソースコード | 保存、ログ記録、トレーニングに絶対使用しない |
| 分析結果 | リクエスト者にのみ返却。サーバー側での保持なし。 |
| パターン統計 | 匿名での集計のみ |
| IPアドレス | Railwayインフラの標準ログ（当社の管理外） |

すべての通信はHTTPS（TLS 1.2+）を使用します。APIキーはリクエストヘッダーのみで渡されます。

プライバシーに関するお問い合わせ：shanyshany3528@gmail.com

---

## 料金プラン

| | 無料 | Pro | チーム |
|--|------|-----|-------|
| ファイル/日 | 50 | 無制限 | 無制限 |
| セキュリティパターン | ✅ | ✅ | ✅ |
| バイブコーディングパターン | ✅ | ✅ | ✅ |
| プロジェクトドキュメント | 基本 | フル | フル |
| クロスファイル分析 | ❌ | ❌ | ✅ |
| 価格 | 無料 | $19/月 | $60/月 |

---

## エンドポイント

**ベースURL：** `https://pleasing-transformation-production-90c2.up.railway.app`

すべてのエンドポイントは `X-API-Key` ヘッダーが必要です。

| メソッド | パス | 説明 |
|---------|------|------|
| `POST` | `/v1/scan` | ファイルスキャン |
| `POST` | `/v1/docs` | プロジェクトzip → 構造ドキュメント |
| `POST` | `/v1/feedback` | 誤検知レポート |
| `GET` | `/v1/validate` | APIキー有効性 + 残余スロット |
| `GET` | `/v1/slots` | 今日の残りスキャン数 |
| `GET` | `/v1/report/history` | スキャン履歴（最近20件） |
| `GET` | `/v1/stats` | パターン統計 |
| `GET` | `/v1/status` | サーバー状態 |

---

## リンク

- **GitHub：** [github.com/Moonsehwan/aina-scan](https://github.com/Moonsehwan/aina-scan)
- **Issues：** [github.com/Moonsehwan/aina-scan/issues](https://github.com/Moonsehwan/aina-scan/issues)
- **メール：** shanyshany3528@gmail.com
