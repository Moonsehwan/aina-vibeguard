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

---

## はじめに

### 方法1 — Webアプリ（インストール不要）

**👉 [https://pleasing-transformation-production-90c2.up.railway.app](https://pleasing-transformation-production-90c2.up.railway.app)**

1. 上記リンクにアクセス
2. スキャンしたいファイルをドラッグ＆ドロップ
3. 結果をすぐ確認 — BLOCK（必ず修正）/ WARN（要確認）

APIキー `vg_free_test` があらかじめ入力されています。

---

### 方法2 — REST API（自動化・CI/CD連携）

```bash
curl -X POST https://pleasing-transformation-production-90c2.up.railway.app/v1/scan \
  -H "X-API-Key: vg_free_test" \
  -F "file=@your_file.py"
```

対応拡張子：`.py` `.js` `.ts` `.tsx` `.go` `.java` `.kt` `.php` `.rb` `.c` `.cpp`

---

## 検出パターン（48種）

### セキュリティ脆弱性（33種）

| パターン | 重大度 | 検出対象 |
|---------|--------|---------|
| `SQL_INJECTION_RISK` | BLOCK | f-string/% でSQL構築 |
| `COMMAND_INJECTION` | BLOCK | ユーザー入力 → subprocess/exec |
| `HARDCODED_SECRET` | BLOCK | api_key/password に直接文字列 |
| `XSS_RISK` | BLOCK | innerHTML + ユーザーデータ |
| `EVAL_EXEC_RISK` | BLOCK | eval()/exec() + ユーザー入力 |
| `BUFFER_OVERFLOW` | BLOCK | C/C++ gets/strcpy/sprintf |
| + 27種 | WARN | SSRF、CSRF、弱い暗号化など |

### バイブコーディングバグ（15種）— AIコード専用

| パターン | 重大度 | AIが生成するパターン |
|---------|--------|-------------------|
| `STUB_SKELETON` | BLOCK | `def save(): return {}` — プレースホルダーのまま |
| `MISSING_WRITE` | BLOCK | 保存関数に実際のDB書き込みなし |
| `FAKE_ASYNC` | BLOCK | `async def` に `await` なし |
| `INPUT_OUTPUT_DISCONNECTED` | BLOCK | パラメータが戻り値に影響しない |
| `HARDCODED_TABLE` | BLOCK | DBクエリの代わりに辞書ハードコード |
| + 10種 | BLOCK/WARN | MOCK_PATTERN、TYPE_UNSAFE_ACCESSなど |

---

## GitHub Action

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

---

## パフォーマンス

| バージョン | Precision | Recall | F1 | テストケース |
|-----------|-----------|--------|----|------------|
| **v7.3** | **100%** | **100%** | **100%** | **90** |

- スキャン速度：500KB以下のファイルは2秒以内
- 10万+⭐のオープンソースリポジトリ10個で誤検知ゼロを確認

---

## 対応言語

Python · JavaScript · TypeScript · Go · Java · PHP · Ruby · Kotlin · C · C++

---

## リンク

- **GitHub:** [github.com/Moonsehwan/aina-scan](https://github.com/Moonsehwan/aina-scan)
- **Issues:** [github.com/Moonsehwan/aina-scan/issues](https://github.com/Moonsehwan/aina-scan/issues)
- **メール:** shanyshany3528@gmail.com
