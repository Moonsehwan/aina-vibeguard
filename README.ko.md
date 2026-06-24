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
2. 스캔하고 싶은 파일을 드래그해서 올리기
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

**무료 베타 키:** `vg_free_test` — Pro 전체 기능 무료.

지원 확장자: `.py` `.js` `.ts` `.tsx` `.go` `.java` `.kt` `.php` `.rb` `.c` `.cpp`

---

## 전체 프로젝트 스캔

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

---

## 탐지 패턴 (48종)

### 보안 취약점 (33종)

| 패턴 | Severity | 탐지 대상 |
|------|----------|----------|
| `SQL_INJECTION_RISK` | BLOCK | f-string/% 포맷으로 SQL 구성 |
| `COMMAND_INJECTION` | BLOCK | 사용자 입력 → subprocess/exec/system |
| `HARDCODED_SECRET` | BLOCK | api_key/password = "..." 직접 할당 |
| `PATH_TRAVERSAL` | BLOCK | 비검증 파일 경로 조작 |
| `XSS_RISK` | BLOCK | innerHTML/document.write + 사용자 데이터 |
| `EVAL_EXEC_RISK` | BLOCK | eval()/exec() + 사용자 입력 |
| `DESERIALIZATION_RISK` | BLOCK | pickle.loads/unserialize 신뢰 불가 |
| `DEBUG_MODE_RISK` | BLOCK | debug=True + host="0.0.0.0" |
| `BUFFER_OVERFLOW` | BLOCK | C/C++ gets/strcpy/sprintf |
| `SSRF_RISK` | WARN | 비검증 URL 서버측 페치 |
| + 23종 더 | WARN | CSRF, 약한 암호화, CORS 와일드카드 등 |

### 바이브코딩 버그 (15종) — AI 코드 특화

| 패턴 | Severity | AI가 만드는 패턴 |
|------|----------|-----------------|
| `STUB_SKELETON` | BLOCK | `def save(): return {}` — 플레이스홀더 그대로 |
| `MISSING_WRITE` | BLOCK | save/insert 함수에 실제 DB write 없음 |
| `FAKE_ASYNC` | BLOCK | `async def` 인데 `await` 없음 |
| `INPUT_OUTPUT_DISCONNECTED` | BLOCK | 파라미터가 반환값에 전혀 영향 없음 |
| `DEAD_CALL_RESULT` | BLOCK | 모듈 호출 결과 전부 버림 |
| `HARDCODED_TABLE` | BLOCK | DB 조회 대신 8키+ 딕셔너리 하드코딩 |
| `DB_SCHEMA_DRIFT` | BLOCK | 코드가 참조한 SQL 컬럼이 DB에 없음 |
| + 8종 더 | BLOCK/WARN | MOCK_PATTERN, TYPE_UNSAFE_ACCESS 등 |

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

---

## 성능

| 버전 | Precision | Recall | F1 | 테스트 케이스 |
|------|-----------|--------|----|-------------|
| v5.0 | 93.8% | 83.3% | 88.2% | 30 |
| v7.0 | 100% | 100% | 100% | 60 |
| **v7.3** | **100%** | **100%** | **100%** | **90** |

- 스캔 속도: 500KB 이하 파일 2초 이내
- 10만+ ⭐ 오픈소스 레포 10개 대상 오탐 0건

---

## 지원 언어

Python · JavaScript · TypeScript · Go · Java · PHP · Ruby · Kotlin · C · C++

---

## 비교

| | AINAScan | Bandit | Semgrep | CodeRabbit |
|--|---------|--------|---------|------------|
| 바이브코딩 패턴 15종 | ✅ 전용 | ❌ | ❌ | ❌ |
| 프로젝트 문서 자동 생성 | ✅ /v1/docs | ❌ | ❌ | ❌ |
| LLM 없는 결정론적 분석 | ✅ | ✅ | ✅ | ❌ |
| 9개 언어 | ✅ | ❌ Python만 | ✅ | ✅ |
| 무료 티어 | ✅ 50파일/일 | ✅ | 제한적 | ❌ |

---

## 링크

- **GitHub:** [github.com/Moonsehwan/aina-scan](https://github.com/Moonsehwan/aina-scan)
- **이슈:** [github.com/Moonsehwan/aina-scan/issues](https://github.com/Moonsehwan/aina-scan/issues)
- **이메일:** shanyshany3528@gmail.com
