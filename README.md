# AINAScan VibeGuard — AI Code Security Scanner

**Ship vibe-coded projects without the security holes.**

AINAScan catches the **33 security vulnerabilities + 15 vibe-coding bugs = 48 patterns** that AI coding assistants (Claude, GPT, Cursor) reliably produce. 9 languages. Deterministic AST analysis — no LLM, no false positives.

[![Free Beta](https://img.shields.io/badge/beta-free-brightgreen)](https://pleasing-transformation-production-90c2.up.railway.app)
[![9 Languages](https://img.shields.io/badge/languages-9-blue)](#supported-languages)
[![48 Patterns](https://img.shields.io/badge/patterns-48-orange)](#detection-patterns)
[![P=R=F1=100%](https://img.shields.io/badge/F1-100%25-success)](#performance)
[![GitHub](https://img.shields.io/badge/GitHub-aina--scan-181717?logo=github)](https://github.com/Moonsehwan/aina-scan)

---

## The Vibe-Coding Problem

AI-assisted coding is fast. The problem is that AI assistants repeatedly generate code that **runs but doesn't work**.

```python
# AI wrote this — save function that never saves (MISSING_WRITE)
def save_user(data: dict):
    validated = validate(data)
    return {"status": "saved", "id": validated["id"]}  # ← no INSERT

# AI wrote this — blocks the event loop (FAKE_ASYNC)
async def fetch_data(url: str):
    response = requests.get(url)   # ← no await
    return response.json()

# AI wrote this — SQL injection (SQL_INJECTION_RISK)
def get_user(username: str):
    return db.execute(f"SELECT * FROM users WHERE name='{username}'")
```

AINAScan catches these bugs **in under 1 second** — before they reach production.

Standard tools (Semgrep, Bandit, Snyk) miss AI-specific patterns. AINAScan is purpose-built for vibe-coding bugs.

---

## Get Started

### Option 1 — Web App (no install, easiest)

**👉 [https://pleasing-transformation-production-90c2.up.railway.app](https://pleasing-transformation-production-90c2.up.railway.app)**

1. Open the link above
2. Drag and drop your file
3. Get results instantly — BLOCK (must fix) / WARN (review recommended)

Free key `vg_free_test` is pre-filled. No signup required.

> **Privacy:** Uploaded files are deleted from memory immediately after analysis. Nothing is stored on disk.

---

### Option 2 — REST API (automation / CI)

```bash
# Free key — no signup needed
curl -X POST https://pleasing-transformation-production-90c2.up.railway.app/v1/scan \
  -H "X-API-Key: vg_free_test" \
  -F "file=@your_file.py"
```

**Free beta key:** `vg_free_test` — full Pro access free during open beta.

Supported extensions: `.py` `.js` `.ts` `.tsx` `.go` `.java` `.kt` `.php` `.rb` `.c` `.cpp`

---

## Scan an Entire Project

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

print(json.dumps(results, indent=2))
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

### PowerShell
```powershell
$API = "https://pleasing-transformation-production-90c2.up.railway.app"
$KEY = "vg_free_test"
Get-ChildItem -Recurse -Include "*.py","*.js","*.ts","*.go" | ForEach-Object {
    $client = [System.Net.Http.HttpClient]::new()
    $client.DefaultRequestHeaders.Add("X-API-Key", $KEY)
    $form = [System.Net.Http.MultipartFormDataContent]::new()
    $bytes = [System.Net.Http.ByteArrayContent]::new([IO.File]::ReadAllBytes($_.FullName))
    $form.Add($bytes, "file", $_.Name)
    $resp = $client.PostAsync("$API/v1/scan", $form).Result
    $json = $resp.Content.ReadAsStringAsync().Result | ConvertFrom-Json
    if ($json.block_count -gt 0) { Write-Host "BLOCK [$($json.block_count)] $($_.FullName)" }
}
```

---

## API Reference

### 1. Code Scan — POST /v1/scan

```bash
curl -X POST .../v1/scan \
  -H "X-API-Key: YOUR_KEY" \
  -F "file=@app.py"
```

**Response:**
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
    },
    {
      "kind": "MISSING_WRITE",
      "severity": "BLOCK",
      "line": 23,
      "detail": "save_record() has no DB write statement",
      "function_name": "save_record"
    }
  ]
}
```

**Query params:**

| Param | Default | Description |
|-------|---------|-------------|
| `include_senior` | `true` | Include senior code quality patterns (Python) |
| `enable_aina_advisor` | `true` | Enable AINA causal chain analysis |

---

### 2. Project Docs Generator — POST /v1/docs ⭐ New

Upload a project zip and get a structured doc for AI context.

```bash
zip -r myproject.zip ./src
curl -X POST .../v1/docs \
  -H "X-API-Key: vg_free_test" \
  -F "file=@myproject.zip" \
  -G -d "tier=pro&lang=en"
```

**Generated sections (7):**

| Section | Tier | Content |
|---------|------|---------|
| 📁 Project Structure | Free | File tree + function/class counts |
| 🔧 Function List | Free | All functions with params, return types, line numbers |
| 📦 Dependencies | Free | External packages from requirements.txt |
| 🔗 Import Graph | Pro | Mermaid diagram — which files import which |
| 🌐 API Endpoints | Pro | All endpoints (method, path, handler, line number) |
| 🛡️ Security Status | Pro | BLOCK/WARN count per file |
| 🪦 Dead Code | Pro | Functions never called from anywhere |

Paste the generated markdown into your AI assistant before asking for new features — prevents duplicate code, wrong imports, and route conflicts.

---

### 3. False Positive Report — POST /v1/feedback

```bash
curl -X POST .../v1/feedback \
  -H "X-API-Key: vg_free_test" \
  -H "Content-Type: application/json" \
  -d '{
    "scan_id": "your-scan-id",
    "vuln_type": "HARDCODED_TABLE",
    "is_false_positive": true,
    "note": "Intentional config table, no DB lookup needed"
  }'
```

---

### 4. Other Endpoints

```bash
# Scan history (last 20)
curl -H "X-API-Key: vg_free_test" .../v1/report/history

# Pattern stats
curl -H "X-API-Key: vg_free_test" .../v1/stats
```

---

### 5. Inline Disable — ainascan-disable

```python
# ainascan-disable HARDCODED_TABLE
COUNTRY_CODES = {"US": "United States", "KR": "Korea"}  # intentional lookup
```

---

## Detection Patterns (48)

### Security Vulnerabilities (33)

| Pattern | Severity | Detects |
|---------|----------|---------|
| `SQL_INJECTION_RISK` | BLOCK | f-string / % format in SQL |
| `COMMAND_INJECTION` | BLOCK | User input → subprocess/exec/system |
| `HARDCODED_SECRET` | BLOCK | api_key/password = "..." literal |
| `PATH_TRAVERSAL` | BLOCK | Unvalidated file path manipulation |
| `XSS_RISK` | BLOCK | innerHTML/document.write + user data |
| `EVAL_EXEC_RISK` | BLOCK | eval()/exec() + user input |
| `DESERIALIZATION_RISK` | BLOCK | pickle.loads/unserialize untrusted data |
| `DEBUG_MODE_RISK` | BLOCK | debug=True + host="0.0.0.0" |
| `IDOR_MISSING_AUTH_CHECK` | BLOCK | Direct object access without authz check |
| `NOSQL_INJECTION` | BLOCK | Unvalidated MongoDB queries |
| `BUFFER_OVERFLOW` | BLOCK | C/C++ gets/strcpy/sprintf |
| `FORMAT_STRING` | BLOCK | C/C++ printf(var) without format arg |
| `USE_AFTER_FREE` | BLOCK | C/C++ pointer use after free() |
| `NULL_DEREF` | BLOCK | C/C++ malloc NULL check missing |
| `INTEGER_OVERFLOW` | BLOCK | C/C++ malloc(atoi(input)) |
| `SSRF_RISK` | WARN | Unvalidated URL in server-side fetch |
| `OPEN_REDIRECT` | WARN | Unvalidated redirect target |
| `CSRF_MISSING_PROTECTION` | WARN | State-changing endpoint without CSRF token |
| `WEAK_CRYPTO` | WARN | MD5/SHA1 usage |
| `CORS_WILDCARD` | WARN | origins='*' |
| + 13 more | WARN | XXE, LDAP injection, template injection, etc. |

### Vibe-Coding Bugs (15) — AI Code Specific

| Pattern | Severity | What AI produces |
|---------|----------|-----------------|
| `STUB_SKELETON` | BLOCK | `def save(): return {}` — placeholder left in |
| `MISSING_WRITE` | BLOCK | save/store/insert function with no actual DB write |
| `FAKE_ASYNC` | BLOCK | `async def` with no `await` — silently blocks event loop |
| `INPUT_OUTPUT_DISCONNECTED` | BLOCK | Parameters have zero effect on return value |
| `DEAD_CALL_RESULT` | BLOCK | 3 module calls, all results discarded |
| `HARDCODED_TABLE` | BLOCK | 8-key+ dict literal where DB lookup belongs |
| `TRIVIAL_IF_CHAIN` | BLOCK | 7+ elif chain where DB lookup belongs |
| `MOCK_PATTERN` | BLOCK | unittest.mock left in production code |
| `ENCODING_CORRUPTION` | BLOCK | Smart quotes/BOM → runtime UnicodeDecodeError |
| `TYPE_UNSAFE_ACCESS` | BLOCK | `float(d.get('key'))` — TypeError when None |
| `DB_SCHEMA_DRIFT` | BLOCK | SQL references a column that doesn't exist in DB |
| `LLM_DELEGATION` | BLOCK | LLM response returned verbatim without own reasoning |
| `CONST_SQL_NO_PARAM` | BLOCK | Hardcoded value in SQL WHERE clause |
| `TRIVIAL_ASSERT` | WARN | `assert True` only in test functions |
| `FAKE_ASYNC` | WARN | async keyword only, no await (lower confidence) |

---

## Auto FP Reduction

**Library code detection** — when `__all__`, `ABCMeta`, `@abstractmethod` appear 3+ times, high-noise patterns auto-downgrade to WARN.

**Test file detection** — in `test_*.py`, `*_test.py`, `conftest.py`, STUB/MOCK patterns auto-downgrade to WARN.

---

## GitHub Action

`.github/workflows/ainascan.yml`:

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

## Performance

| Version | Precision | Recall | F1 | Test Cases |
|---------|-----------|--------|----|------------|
| v5.0 | 93.8% | 83.3% | 88.2% | 30 |
| v7.0 | 100% | 100% | 100% | 60 |
| **v7.3** | **100%** | **100%** | **100%** | **90** |

- Scan speed: files under 500 KB in under 2 seconds
- Caching: identical files return instantly on rescan
- Zero false positives on 10 open-source repos with 100k+ stars

---

## Supported Languages

| Language | Extensions |
|----------|-----------|
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

## Comparison

| | AINAScan | Bandit | Semgrep | CodeRabbit |
|--|---------|--------|---------|------------|
| Vibe-coding patterns (15) | ✅ dedicated | ❌ | ❌ | ❌ |
| Project docs generator | ✅ /v1/docs | ❌ | ❌ | ❌ |
| Deterministic AST (no LLM) | ✅ | ✅ | ✅ | ❌ |
| 9 languages | ✅ | ❌ Python only | ✅ | ✅ |
| No code storage | ✅ | ✅ | ✅ | ❌ |
| GitHub Action | ✅ | ✅ | ✅ | ✅ |
| Free tier | ✅ 50 files/day | ✅ | Limited | ❌ |

---

## Privacy & Security

> Uploaded files are never stored. Analysis happens in memory only and is discarded immediately after the request.

| Item | How it's handled |
|------|-----------------|
| Uploaded files | Deleted from memory after analysis. No disk write. |
| Source code | Never stored, logged, or used for training |
| Analysis results | Returned to requester only. Not retained server-side. |
| Pattern stats | Aggregated anonymously only |
| IP addresses | Standard Railway infrastructure logs (outside our control) |

All communication uses HTTPS (TLS 1.2+). API keys are passed in request headers only.

Privacy inquiries: shanyshany3528@gmail.com

---

## Pricing

| | Free | Pro | Team |
|--|------|-----|------|
| Files/day | 50 | Unlimited | Unlimited |
| Security patterns | ✅ | ✅ | ✅ |
| Vibe-coding patterns | ✅ | ✅ | ✅ |
| Project docs | Basic | Full | Full |
| Cross-file analysis | ❌ | ❌ | ✅ |
| Price | Free | $19/mo | $60/mo |

---

## Endpoints

**Base URL:** `https://pleasing-transformation-production-90c2.up.railway.app`

All endpoints require `X-API-Key` header.

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/v1/scan` | Scan a file |
| `POST` | `/v1/docs` | Project zip → structure doc |
| `POST` | `/v1/feedback` | Report false positive |
| `GET` | `/v1/validate` | API key validity + remaining slots |
| `GET` | `/v1/slots` | Remaining scans today |
| `GET` | `/v1/report/history` | Scan history (last 20) |
| `GET` | `/v1/stats` | Pattern statistics |
| `GET` | `/v1/status` | Server status |

---

## Links

- **GitHub:** [github.com/Moonsehwan/aina-scan](https://github.com/Moonsehwan/aina-scan)
- **Issues:** [github.com/Moonsehwan/aina-scan/issues](https://github.com/Moonsehwan/aina-scan/issues)
- **Email:** shanyshany3528@gmail.com
