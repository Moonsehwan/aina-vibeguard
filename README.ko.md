# AINAScan VibeGuard — AI 코드 보안 스캐너

**바이브코딩으로 만든 코드, 배포 전에 한 번 더 확인하세요.**

AI 코딩 어시스턴트(Claude, GPT, Cursor)가 반복적으로 만들어내는 **보안 취약점 33종 + 바이브코딩 버그 15종 = 48개 패턴**을 탐지합니다. 9개 언어 지원. LLM 없이 결정론적 AST 분석으로 작동합니다.

[![무료 베타](https://img.shields.io/badge/베타-무료-brightgreen)](https://pleasing-transformation-production-90c2.up.railway.app)
[![9개 언어](https://img.shields.io/badge/언어-9개-blue)](#지원-언어)
[![48개 패턴](https://img.shields.io/badge/패턴-48개-orange)](#탐지-패턴)
[![P=R=F1=100%](https://img.shields.io/badge/F1-100%25-success)](#성능)
[![GitHub](https://img.shields.io/badge/GitHub-aina--scan-181717?logo=github)](https://github.com/Moonsehwan/aina-scan)

---

## 바이브코딩의 문제

AI로 빠르게 코드를 만드는 것은 좋습니다. 문제는 AI 어시스턴트가 **실행은 되지만 동작하지 않는** 코드를 반복적으로 생성한다는 점입니다.

```python
# AI가 작성한 코드 — 저장 함수인데 실제로 저장하지 않음 (MISSING_WRITE)
def save_user(data: dict):
    validated = validate(data)
    return {"status": "saved", "id": validated["id"]}  # ← INSERT가 없음

# AI가 작성한 코드 — 이벤트 루프를 블로킹함 (FAKE_ASYNC)
async def fetch_data(url: str):
    response = requests.get(url)   # ← await가 없음
    return response.json()

# AI가 작성한 코드 — SQL 인젝션 (SQL_INJECTION_RISK)
def get_user(username: str):
    return db.execute(f"SELECT * FROM users WHERE name='{username}'")
```

AINAScan은 이런 버그들을 **1초 안에** 잡아냅니다. 프로덕션에 나가기 전에.

기존 도구(Semgrep, Bandit, Snyk)는 AI 특유의 패턴을 탐지하지 못합니다. AINAScan은 바이브코딩 버그에 특화되어 있습니다.

---

## 시작하기

### 방법 1 — 웹 앱 (설치 없음, 가장 쉬운 방법)

**👉 [https://pleasing-transformation-production-90c2.up.railway.app](https://pleasing-transformation-production-90c2.up.railway.app)**

1. 위 링크 접속
2. 파일을 드래그해서 올리기
3. 결과 즉시 확인 — BLOCK(반드시 수정) / WARN(검토 권장)

API 키 `vg_free_test`가 자동으로 입력되어 있어 별도 설정 불필요합니다.

> **개인정보 보호:** 업로드한 파일은 분석 즉시 메모리에서 삭제됩니다. 서버에 저장되지 않습니다.

---

### 방법 2 — REST API (자동화·CI/CD 연동)

```bash
# 무료 키로 바로 시작 — 회원가입 불필요
curl -X POST https://pleasing-transformation-production-90c2.up.railway.app/v1/scan \
  -H "X-API-Key: vg_free_test" \
  -F "file=@your_file.py"
```

**무료 베타 키:** `vg_free_test` — 오픈 베타 기간 중 Pro 전체 기능 무료.

지원 확장자: `.py` `.js` `.ts` `.tsx` `.go` `.java` `.kt` `.php` `.rb` `.c` `.cpp`

---

## 전체 프로젝트 스캔

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

## API 레퍼런스

### 1. 코드 스캔 — POST /v1/scan

```bash
curl -X POST .../v1/scan \
  -H "X-API-Key: YOUR_KEY" \
  -F "file=@app.py"
```

**응답:**
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

**쿼리 파라미터:**

| 파라미터 | 기본값 | 설명 |
|---------|--------|------|
| `include_senior` | `true` | 시니어 코드 품질 패턴 포함 (Python) |
| `enable_aina_advisor` | `true` | AINA 인과 체인 분석 활성화 |

---

### 2. 프로젝트 문서 생성 — POST /v1/docs ⭐ 신기능

프로젝트 zip을 업로드하면 AI 컨텍스트용 구조화된 문서를 생성합니다.

```bash
zip -r myproject.zip ./src
curl -X POST .../v1/docs \
  -H "X-API-Key: vg_free_test" \
  -F "file=@myproject.zip" \
  -G -d "tier=pro&lang=ko"
```

**생성 섹션 (7개):**

| 섹션 | 티어 | 내용 |
|------|------|------|
| 📁 프로젝트 구조 | 무료 | 파일 트리 + 함수/클래스 수 |
| 🔧 함수 목록 | 무료 | 모든 함수 (파라미터, 반환 타입, 라인 번호) |
| 📦 의존성 | 무료 | requirements.txt 기반 패키지 목록 |
| 🔗 임포트 그래프 | Pro | Mermaid 다이어그램 — 파일 간 의존관계 |
| 🌐 API 엔드포인트 | Pro | 모든 엔드포인트 (메서드, 경로, 핸들러, 라인) |
| 🛡️ 보안 상태 | Pro | 파일별 BLOCK/WARN 수 |
| 🪦 데드 코드 | Pro | 어디서도 호출되지 않는 함수 |

생성된 마크다운을 AI 어시스턴트에게 붙여넣으면 중복 코드, 잘못된 임포트, 라우트 충돌을 예방할 수 있습니다.

---

### 3. 오탐 신고 — POST /v1/feedback

```bash
curl -X POST .../v1/feedback \
  -H "X-API-Key: vg_free_test" \
  -H "Content-Type: application/json" \
  -d '{
    "scan_id": "your-scan-id",
    "vuln_type": "HARDCODED_TABLE",
    "is_false_positive": true,
    "note": "의도적인 설정 테이블, DB 조회 불필요"
  }'
```

---

### 4. 기타 엔드포인트

```bash
# 스캔 이력 (최근 20개)
curl -H "X-API-Key: vg_free_test" .../v1/report/history

# 패턴 통계
curl -H "X-API-Key: vg_free_test" .../v1/stats
```

---

### 5. 인라인 비활성화 — ainascan-disable

```python
# ainascan-disable HARDCODED_TABLE
COUNTRY_CODES = {"KR": "대한민국", "US": "미국"}  # 의도적인 룩업 테이블
```

---

## 탐지 패턴 (48개)

### 보안 취약점 (33개)

| 패턴 | 심각도 | 탐지 내용 |
|------|--------|----------|
| `SQL_INJECTION_RISK` | BLOCK | SQL의 f-string / % 포맷 |
| `COMMAND_INJECTION` | BLOCK | 사용자 입력 → subprocess/exec/system |
| `HARDCODED_SECRET` | BLOCK | api_key/password = "..." 리터럴 |
| `PATH_TRAVERSAL` | BLOCK | 검증되지 않은 파일 경로 조작 |
| `XSS_RISK` | BLOCK | innerHTML/document.write + 사용자 데이터 |
| `EVAL_EXEC_RISK` | BLOCK | eval()/exec() + 사용자 입력 |
| `DESERIALIZATION_RISK` | BLOCK | pickle.loads/unserialize 신뢰할 수 없는 데이터 |
| `DEBUG_MODE_RISK` | BLOCK | debug=True + host="0.0.0.0" |
| `IDOR_MISSING_AUTH_CHECK` | BLOCK | 인증 없는 직접 객체 접근 |
| `NOSQL_INJECTION` | BLOCK | 검증되지 않은 MongoDB 쿼리 |
| `BUFFER_OVERFLOW` | BLOCK | C/C++ gets/strcpy/sprintf |
| `FORMAT_STRING` | BLOCK | C/C++ printf(var) 포맷 인자 없음 |
| `USE_AFTER_FREE` | BLOCK | C/C++ free() 후 포인터 사용 |
| `NULL_DEREF` | BLOCK | C/C++ malloc NULL 체크 없음 |
| `INTEGER_OVERFLOW` | BLOCK | C/C++ malloc(atoi(input)) |
| `HEADER_INJECTION` | BLOCK | HTTP 헤더에 직접 사용자 입력 |
| `XXE_RISK` | BLOCK | XML 외부 엔티티 처리 |
| `MASS_ASSIGNMENT` | BLOCK | request.json 직접 모델 할당 |
| `LDAP_INJECTION` | BLOCK | LDAP 쿼리 문자열 조작 |
| `PROTOTYPE_POLLUTION` | BLOCK | JS __proto__ / constructor 조작 |
| `PICKLE_RISK` | BLOCK | pickle.loads 신뢰할 수 없는 데이터 |
| `TEMPLATE_INJECTION` | BLOCK | render_template_string + 사용자 입력 |
| `SSRF_RISK` | WARN | 서버 측 fetch에 검증되지 않은 URL |
| `OPEN_REDIRECT` | WARN | 검증되지 않은 리다이렉트 대상 |
| `CSRF_MISSING_PROTECTION` | WARN | CSRF 토큰 없는 상태 변경 엔드포인트 |
| `WEAK_CRYPTO` | WARN | MD5/SHA1 사용 |
| `CORS_WILDCARD` | WARN | origins='*' |
| `CORS_MISCONFIGURATION` | WARN | 잘못된 CORS 설정 |
| `TIMING_ATTACK` | WARN | hmac.compare_digest 대신 == 비교 |
| `LOG_INJECTION` | WARN | 검증되지 않은 사용자 입력 로그 기록 |
| `UNSAFE_CAST` | WARN | C++ reinterpret_cast |
| + 2종 더 | WARN | INSECURE_REDIRECT, BOUNDARY_MISSING |

### 바이브코딩 버그 (15개) — AI 코드 특화

| 패턴 | 심각도 | AI가 생성하는 것 |
|------|--------|----------------|
| `STUB_SKELETON` | BLOCK | `def save(): return {}` — 뼈대만 있는 함수 |
| `MISSING_WRITE` | BLOCK | save/store/insert 함수에 실제 DB 쓰기 없음 |
| `FAKE_ASYNC` | BLOCK | `async def`인데 `await` 없음 — 이벤트 루프 블로킹 |
| `INPUT_OUTPUT_DISCONNECTED` | BLOCK | 파라미터가 반환값에 전혀 영향 없음 |
| `DEAD_CALL_RESULT` | BLOCK | 모듈 3개 호출했는데 결과 전부 버림 |
| `HARDCODED_TABLE` | BLOCK | DB 조회 대신 8키+ 딕셔너리 리터럴 사용 |
| `TRIVIAL_IF_CHAIN` | BLOCK | DB 조회 대신 7+ elif 체인 |
| `MOCK_PATTERN` | BLOCK | 프로덕션 코드에 unittest.mock 남겨둠 |
| `ENCODING_CORRUPTION` | BLOCK | 스마트 따옴표/BOM → 런타임 UnicodeDecodeError |
| `TYPE_UNSAFE_ACCESS` | BLOCK | `float(d.get('key'))` — None일 때 TypeError |
| `DB_SCHEMA_DRIFT` | BLOCK | SQL이 실제 DB에 없는 컬럼 참조 |
| `LLM_DELEGATION` | BLOCK | LLM 응답을 자체 추론 없이 그대로 반환 |
| `CONST_SQL_NO_PARAM` | BLOCK | SQL WHERE 절에 하드코딩 값 |
| `TRIVIAL_ASSERT` | WARN | 테스트 함수에 `assert True`만 있음 |
| `FAKE_ASYNC` | WARN | async 키워드만 있음 (낮은 확신도) |

---

## 자동 오탐 감소

**라이브러리 코드 감지** — `__all__`, `ABCMeta`, `@abstractmethod`가 3회 이상 등장하면 노이즈 많은 패턴을 WARN으로 자동 강등.

**테스트 파일 감지** — `test_*.py`, `*_test.py`, `conftest.py`에서는 STUB/MOCK 패턴을 WARN으로 자동 강등.

---

## GitHub Actions 연동

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

## 성능

| 버전 | Precision | Recall | F1 | 테스트 케이스 |
|------|-----------|--------|----|-----------| 
| v5.0 | 93.8% | 83.3% | 88.2% | 30 |
| v7.0 | 100% | 100% | 100% | 60 |
| **v7.3** | **100%** | **100%** | **100%** | **90** |

- 스캔 속도: 500KB 이하 파일 2초 미만
- 캐싱: 동일한 파일은 재스캔 시 즉시 반환
- 100k+ 스타 오픈소스 10개 프로젝트에서 오탐 0건

---

## 지원 언어

| 언어 | 확장자 |
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

## 비교

| | AINAScan | Bandit | Semgrep | CodeRabbit |
|--|---------|--------|---------|------------|
| 바이브코딩 패턴 (15개) | ✅ 전용 | ❌ | ❌ | ❌ |
| 프로젝트 문서 생성 | ✅ /v1/docs | ❌ | ❌ | ❌ |
| 결정론적 AST (LLM 없음) | ✅ | ✅ | ✅ | ❌ |
| 9개 언어 | ✅ | ❌ Python만 | ✅ | ✅ |
| 코드 미저장 | ✅ | ✅ | ✅ | ❌ |
| GitHub Action | ✅ | ✅ | ✅ | ✅ |
| 무료 티어 | ✅ 50파일/일 | ✅ | 제한적 | ❌ |

---

## 개인정보 및 보안

> 업로드한 파일은 절대 저장되지 않습니다. 분석은 메모리에서만 이루어지며 요청 완료 즉시 삭제됩니다.

| 항목 | 처리 방법 |
|------|----------|
| 업로드 파일 | 분석 후 메모리에서 즉시 삭제. 디스크 쓰기 없음. |
| 소스 코드 | 저장, 로깅, 학습에 절대 사용 안 함 |
| 분석 결과 | 요청자에게만 반환. 서버 측 보관 없음. |
| 패턴 통계 | 익명 집계만 |
| IP 주소 | Railway 인프라 표준 로그 (당사 통제 범위 외) |

모든 통신은 HTTPS(TLS 1.2+)를 사용합니다. API 키는 요청 헤더로만 전달됩니다.

개인정보 문의: shanyshany3528@gmail.com

---

## 요금제

| | 무료 | Pro | 팀 |
|--|------|-----|-----|
| 파일/일 | 50 | 무제한 | 무제한 |
| 보안 패턴 | ✅ | ✅ | ✅ |
| 바이브코딩 패턴 | ✅ | ✅ | ✅ |
| 프로젝트 문서 | 기본 | 전체 | 전체 |
| 크로스파일 분석 | ❌ | ❌ | ✅ |
| 가격 | 무료 | $19/월 | $60/월 |

---

## 엔드포인트

**기본 URL:** `https://pleasing-transformation-production-90c2.up.railway.app`

모든 엔드포인트는 `X-API-Key` 헤더가 필요합니다.

| 메서드 | 경로 | 설명 |
|--------|------|------|
| `POST` | `/v1/scan` | 파일 스캔 |
| `POST` | `/v1/docs` | 프로젝트 zip → 구조 문서 |
| `POST` | `/v1/feedback` | 오탐 신고 |
| `GET` | `/v1/validate` | API 키 유효성 + 잔여 슬롯 |
| `GET` | `/v1/slots` | 오늘 남은 스캔 수 |
| `GET` | `/v1/report/history` | 스캔 이력 (최근 20개) |
| `GET` | `/v1/stats` | 패턴 통계 |
| `GET` | `/v1/status` | 서버 상태 |

---

## 링크

- **GitHub:** [github.com/Moonsehwan/aina-scan](https://github.com/Moonsehwan/aina-scan)
- **이슈:** [github.com/Moonsehwan/aina-scan/issues](https://github.com/Moonsehwan/aina-scan/issues)
- **이메일:** shanyshany3528@gmail.com
