# AINAScan VibeGuard — KI-Code-Sicherheitsscanner

**Vibe-codierten Code vor dem Deployment prüfen.**

AINAScan erkennt die **33 Sicherheitslücken + 15 Vibe-Coding-Bugs = 48 Muster**, die KI-Coding-Assistenten (Claude, GPT, Cursor) zuverlässig erzeugen. 9 Sprachen. Deterministische AST-Analyse — kein LLM, keine Falsch-Positive.

[![Kostenlose Beta](https://img.shields.io/badge/Beta-kostenlos-brightgreen)](https://pleasing-transformation-production-90c2.up.railway.app)
[![9 Sprachen](https://img.shields.io/badge/Sprachen-9-blue)](#unterstützte-sprachen)
[![48 Muster](https://img.shields.io/badge/Muster-48-orange)](#erkennungsmuster)
[![P=R=F1=100%](https://img.shields.io/badge/F1-100%25-success)](#leistung)
[![GitHub](https://img.shields.io/badge/GitHub-aina--scan-181717?logo=github)](https://github.com/Moonsehwan/aina-scan)

---

## Das Vibe-Coding-Problem

KI-gestütztes Coding ist schnell. Das Problem: KI-Assistenten erzeugen wiederholt Code, der **ausgeführt werden kann, aber nicht funktioniert**.

```python
# KI-generierter Code — Speicherfunktion die nichts speichert (MISSING_WRITE)
def save_user(data: dict):
    validated = validate(data)
    return {"status": "saved", "id": validated["id"]}  # ← kein INSERT

# KI-generierter Code — blockiert den Event-Loop (FAKE_ASYNC)
async def fetch_data(url: str):
    response = requests.get(url)   # ← kein await
    return response.json()

# KI-generierter Code — SQL-Injection (SQL_INJECTION_RISK)
def get_user(username: str):
    return db.execute(f"SELECT * FROM users WHERE name='{username}'")
```

AINAScan erkennt diese Bugs **in unter 1 Sekunde** — vor dem Produktivbetrieb.

---

## Einstieg

### Option 1 — Web-App (kein Install)

**👉 [https://pleasing-transformation-production-90c2.up.railway.app](https://pleasing-transformation-production-90c2.up.railway.app)**

1. Link öffnen
2. Datei hochladen (Drag & Drop)
3. Ergebnisse sofort sehen — BLOCK (muss behoben werden) / WARN (Überprüfung empfohlen)

Freier API-Schlüssel `vg_free_test` ist vorausgefüllt.

---

### Option 2 — REST API (Automatisierung / CI)

```bash
curl -X POST https://pleasing-transformation-production-90c2.up.railway.app/v1/scan \
  -H "X-API-Key: vg_free_test" \
  -F "file=@your_file.py"
```

Unterstützte Erweiterungen: `.py` `.js` `.ts` `.tsx` `.go` `.java` `.kt` `.php` `.rb` `.c` `.cpp`

---

## Erkennungsmuster (48)

### Sicherheitslücken (33)

| Muster | Schweregrad | Erkennt |
|--------|-------------|---------|
| `SQL_INJECTION_RISK` | BLOCK | f-string/% SQL-Formatierung |
| `COMMAND_INJECTION` | BLOCK | Benutzereingabe → subprocess/exec |
| `HARDCODED_SECRET` | BLOCK | api_key/password als Literal |
| `XSS_RISK` | BLOCK | innerHTML + Benutzerdaten |
| `EVAL_EXEC_RISK` | BLOCK | eval()/exec() + Benutzereingabe |
| `BUFFER_OVERFLOW` | BLOCK | C/C++ gets/strcpy/sprintf |
| + 27 weitere | WARN | SSRF, CSRF, schwache Krypto usw. |

### Vibe-Coding-Bugs (15) — KI-Code-spezifisch

| Muster | Schweregrad | KI-generiertes Muster |
|--------|-------------|----------------------|
| `STUB_SKELETON` | BLOCK | `def save(): return {}` — Platzhalter übrig |
| `MISSING_WRITE` | BLOCK | Speicherfunktion ohne DB-Schreibvorgang |
| `FAKE_ASYNC` | BLOCK | `async def` ohne `await` |
| `INPUT_OUTPUT_DISCONNECTED` | BLOCK | Parameter beeinflusst Rückgabe nicht |
| `HARDCODED_TABLE` | BLOCK | Dict-Literal statt DB-Abfrage |
| + 10 weitere | BLOCK/WARN | MOCK_PATTERN, TYPE_UNSAFE_ACCESS usw. |

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

## Leistung

| Version | Precision | Recall | F1 | Testfälle |
|---------|-----------|--------|----|-----------|
| **v7.3** | **100%** | **100%** | **100%** | **90** |

- Scan-Geschwindigkeit: Dateien unter 500 KB in unter 2 Sekunden
- Null Falsch-Positive auf 10 Open-Source-Repos mit 100k+ ⭐

---

## Unterstützte Sprachen

Python · JavaScript · TypeScript · Go · Java · PHP · Ruby · Kotlin · C · C++

---

## Links

- **GitHub:** [github.com/Moonsehwan/aina-scan](https://github.com/Moonsehwan/aina-scan)
- **Issues:** [github.com/Moonsehwan/aina-scan/issues](https://github.com/Moonsehwan/aina-scan/issues)
- **E-Mail:** shanyshany3528@gmail.com
