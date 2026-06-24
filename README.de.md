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

Standardtools (Semgrep, Bandit, Snyk) erkennen KI-spezifische Muster nicht. AINAScan ist speziell für Vibe-Coding-Bugs entwickelt.

---

## Einstieg

### Option 1 — Web-App (kein Install, am einfachsten)

**👉 [https://pleasing-transformation-production-90c2.up.railway.app](https://pleasing-transformation-production-90c2.up.railway.app)**

1. Link öffnen
2. Datei hochladen (Drag & Drop)
3. Ergebnisse sofort sehen — BLOCK (muss behoben werden) / WARN (Überprüfung empfohlen)

Freier API-Schlüssel `vg_free_test` ist vorausgefüllt. Keine Registrierung nötig.

> **Datenschutz:** Hochgeladene Dateien werden nach der Analyse sofort aus dem Speicher gelöscht. Nichts wird auf der Festplatte gespeichert.

---

### Option 2 — REST API (Automatisierung / CI)

```bash
# Kostenloser Schlüssel — keine Registrierung nötig
curl -X POST https://pleasing-transformation-production-90c2.up.railway.app/v1/scan \
  -H "X-API-Key: vg_free_test" \
  -F "file=@your_file.py"
```

**Kostenloser Beta-Schlüssel:** `vg_free_test` — voller Pro-Zugang kostenlos während der offenen Beta.

Unterstützte Erweiterungen: `.py` `.js` `.ts` `.tsx` `.go` `.java` `.kt` `.php` `.rb` `.c` `.cpp`

---

## Gesamtes Projekt scannen

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

---

## API-Referenz

### 1. Code-Scan — POST /v1/scan

```bash
curl -X POST .../v1/scan \
  -H "X-API-Key: YOUR_KEY" \
  -F "file=@app.py"
```

**Antwort:**
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

**Query-Parameter:**

| Parameter | Standard | Beschreibung |
|-----------|----------|-------------|
| `include_senior` | `true` | Senior-Code-Qualitätsmuster einschließen (Python) |
| `enable_aina_advisor` | `true` | AINA-Kausalkettenanalyse aktivieren |

---

### 2. Projekt-Dokumentationsgenerator — POST /v1/docs ⭐ Neu

Laden Sie ein Projekt-ZIP hoch und erhalten Sie ein strukturiertes Dokument für KI-Kontext.

```bash
zip -r myproject.zip ./src
curl -X POST .../v1/docs \
  -H "X-API-Key: vg_free_test" \
  -F "file=@myproject.zip" \
  -G -d "tier=pro&lang=de"
```

**Generierte Abschnitte (7):**

| Abschnitt | Tier | Inhalt |
|-----------|------|--------|
| 📁 Projektstruktur | Kostenlos | Dateibaum + Anzahl Funktionen/Klassen |
| 🔧 Funktionsliste | Kostenlos | Alle Funktionen mit Parametern, Rückgabetypen, Zeilennummern |
| 📦 Abhängigkeiten | Kostenlos | Externe Pakete aus requirements.txt |
| 🔗 Import-Graph | Pro | Mermaid-Diagramm — welche Dateien importieren was |
| 🌐 API-Endpunkte | Pro | Alle Endpunkte (Methode, Pfad, Handler, Zeilennummer) |
| 🛡️ Sicherheitsstatus | Pro | BLOCK/WARN-Anzahl pro Datei |
| 🪦 Toter Code | Pro | Funktionen, die nirgendwo aufgerufen werden |

Fügen Sie das generierte Markdown in Ihren KI-Assistenten ein, bevor Sie neue Funktionen anfordern — verhindert doppelten Code, falsche Imports und Routen-Konflikte.

---

### 3. Falsch-Positiv-Meldung — POST /v1/feedback

```bash
curl -X POST .../v1/feedback \
  -H "X-API-Key: vg_free_test" \
  -H "Content-Type: application/json" \
  -d '{
    "scan_id": "your-scan-id",
    "vuln_type": "HARDCODED_TABLE",
    "is_false_positive": true,
    "note": "Absichtliche Konfigurationstabelle, kein DB-Lookup nötig"
  }'
```

---

### 4. Weitere Endpunkte

```bash
# Scan-Verlauf (letzte 20)
curl -H "X-API-Key: vg_free_test" .../v1/report/history

# Muster-Statistiken
curl -H "X-API-Key: vg_free_test" .../v1/stats
```

---

### 5. Inline-Deaktivierung — ainascan-disable

```python
# ainascan-disable HARDCODED_TABLE
COUNTRY_CODES = {"DE": "Deutschland", "US": "USA"}  # absichtliche Lookup-Tabelle
```

---

## Erkennungsmuster (48)

### Sicherheitslücken (33)

| Muster | Schweregrad | Erkennt |
|--------|-------------|---------|
| `SQL_INJECTION_RISK` | BLOCK | f-string/% SQL-Formatierung |
| `COMMAND_INJECTION` | BLOCK | Benutzereingabe → subprocess/exec/system |
| `HARDCODED_SECRET` | BLOCK | api_key/password als Literal |
| `PATH_TRAVERSAL` | BLOCK | Unvalidierte Dateipfad-Manipulation |
| `XSS_RISK` | BLOCK | innerHTML + Benutzerdaten |
| `EVAL_EXEC_RISK` | BLOCK | eval()/exec() + Benutzereingabe |
| `DESERIALIZATION_RISK` | BLOCK | pickle.loads/unserialize nicht vertrauenswürdige Daten |
| `DEBUG_MODE_RISK` | BLOCK | debug=True + host="0.0.0.0" |
| `IDOR_MISSING_AUTH_CHECK` | BLOCK | Direkter Objektzugriff ohne Berechtigungsprüfung |
| `NOSQL_INJECTION` | BLOCK | Unvalidierte MongoDB-Abfragen |
| `BUFFER_OVERFLOW` | BLOCK | C/C++ gets/strcpy/sprintf |
| `FORMAT_STRING` | BLOCK | C/C++ printf(var) ohne Format-Argument |
| `USE_AFTER_FREE` | BLOCK | C/C++ Pointer-Verwendung nach free() |
| `NULL_DEREF` | BLOCK | C/C++ malloc NULL-Prüfung fehlt |
| `INTEGER_OVERFLOW` | BLOCK | C/C++ malloc(atoi(input)) |
| `SSRF_RISK` | WARN | Unvalidierte URL in serverseitigem Fetch |
| `OPEN_REDIRECT` | WARN | Unvalidiertes Redirect-Ziel |
| `CSRF_MISSING_PROTECTION` | WARN | Zustandsändernder Endpunkt ohne CSRF-Token |
| `WEAK_CRYPTO` | WARN | MD5/SHA1-Verwendung |
| `CORS_WILDCARD` | WARN | origins='*' |
| + 13 weitere | WARN | XXE, LDAP-Injection, Template-Injection usw. |

### Vibe-Coding-Bugs (15) — KI-Code-spezifisch

| Muster | Schweregrad | KI-generiertes Muster |
|--------|-------------|----------------------|
| `STUB_SKELETON` | BLOCK | `def save(): return {}` — Platzhalter hinterlassen |
| `MISSING_WRITE` | BLOCK | Speicher-/Einfügefunktion ohne echten DB-Schreibvorgang |
| `FAKE_ASYNC` | BLOCK | `async def` ohne `await` — blockiert Event-Loop lautlos |
| `INPUT_OUTPUT_DISCONNECTED` | BLOCK | Parameter hat keinerlei Einfluss auf Rückgabewert |
| `DEAD_CALL_RESULT` | BLOCK | 3 Modul-Aufrufe, alle Ergebnisse verworfen |
| `HARDCODED_TABLE` | BLOCK | 8-Schlüssel+-Dict-Literal statt DB-Abfrage |
| `TRIVIAL_IF_CHAIN` | BLOCK | 7+-elif-Kette statt DB-Abfrage |
| `MOCK_PATTERN` | BLOCK | unittest.mock im Produktionscode hinterlassen |
| `ENCODING_CORRUPTION` | BLOCK | Typografische Anführungszeichen/BOM → UnicodeDecodeError |
| `TYPE_UNSAFE_ACCESS` | BLOCK | `float(d.get('key'))` — TypeError bei None |
| `DB_SCHEMA_DRIFT` | BLOCK | SQL referenziert nicht existierenden DB-Spalte |
| `LLM_DELEGATION` | BLOCK | LLM-Antwort wortgetreu zurückgegeben ohne eigene Logik |
| `CONST_SQL_NO_PARAM` | BLOCK | Hartcodierter Wert in SQL WHERE-Klausel |
| `TRIVIAL_ASSERT` | WARN | Nur `assert True` in Testfunktionen |
| `FAKE_ASYNC` | WARN | Nur async-Schlüsselwort, kein await (niedrige Konfidenz) |

---

## Automatische Falsch-Positiv-Reduzierung

**Bibliotheks-Code-Erkennung** — wenn `__all__`, `ABCMeta`, `@abstractmethod` 3+ mal vorkommen, werden hochrauschige Muster automatisch auf WARN herabgestuft.

**Testdatei-Erkennung** — in `test_*.py`, `*_test.py`, `conftest.py` werden STUB/MOCK-Muster automatisch auf WARN herabgestuft.

---

## GitHub Actions

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

## Leistung

| Version | Precision | Recall | F1 | Testfälle |
|---------|-----------|--------|----|-----------|
| v5.0 | 93.8% | 83.3% | 88.2% | 30 |
| v7.0 | 100% | 100% | 100% | 60 |
| **v7.3** | **100%** | **100%** | **100%** | **90** |

- Scan-Geschwindigkeit: Dateien unter 500 KB in unter 2 Sekunden
- Caching: Identische Dateien werden beim erneuten Scan sofort zurückgegeben
- Null Falsch-Positive auf 10 Open-Source-Repos mit 100k+ ⭐

---

## Unterstützte Sprachen

| Sprache | Erweiterungen |
|---------|--------------|
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

## Vergleich

| | AINAScan | Bandit | Semgrep | CodeRabbit |
|--|---------|--------|---------|------------|
| Vibe-Coding-Muster (15) | ✅ dediziert | ❌ | ❌ | ❌ |
| Projekt-Dokumentationsgenerator | ✅ /v1/docs | ❌ | ❌ | ❌ |
| Deterministisches AST (kein LLM) | ✅ | ✅ | ✅ | ❌ |
| 9 Sprachen | ✅ | ❌ Nur Python | ✅ | ✅ |
| Kein Code-Speichern | ✅ | ✅ | ✅ | ❌ |
| GitHub Action | ✅ | ✅ | ✅ | ✅ |
| Kostenloser Tier | ✅ 50 Dateien/Tag | ✅ | Begrenzt | ❌ |

---

## Datenschutz & Sicherheit

> Hochgeladene Dateien werden niemals gespeichert. Die Analyse findet ausschließlich im Arbeitsspeicher statt und wird sofort nach der Anfrage verworfen.

| Punkt | Behandlung |
|-------|-----------|
| Hochgeladene Dateien | Nach der Analyse sofort aus dem Speicher gelöscht. Kein Festplattenschreiben. |
| Quellcode | Wird niemals gespeichert, protokolliert oder für Training verwendet |
| Analyseergebnisse | Nur an den Anfrager zurückgegeben. Keine serverseitige Aufbewahrung. |
| Muster-Statistiken | Nur anonymisiert aggregiert |
| IP-Adressen | Standard-Railway-Infrastruktur-Logs (außerhalb unserer Kontrolle) |

Alle Kommunikation verwendet HTTPS (TLS 1.2+). API-Schlüssel werden nur in Request-Headern übergeben.

Datenschutz-Anfragen: shanyshany3528@gmail.com

---

## Preise

| | Kostenlos | Pro | Team |
|--|-----------|-----|------|
| Dateien/Tag | 50 | Unbegrenzt | Unbegrenzt |
| Sicherheitsmuster | ✅ | ✅ | ✅ |
| Vibe-Coding-Muster | ✅ | ✅ | ✅ |
| Projekt-Doku | Basis | Vollständig | Vollständig |
| Cross-Datei-Analyse | ❌ | ❌ | ✅ |
| Preis | Kostenlos | $19/Monat | $60/Monat |

---

## Endpunkte

**Basis-URL:** `https://pleasing-transformation-production-90c2.up.railway.app`

Alle Endpunkte erfordern den `X-API-Key`-Header.

| Methode | Pfad | Beschreibung |
|---------|------|-------------|
| `POST` | `/v1/scan` | Datei scannen |
| `POST` | `/v1/docs` | Projekt-ZIP → Struktur-Dokument |
| `POST` | `/v1/feedback` | Falsch-Positiv melden |
| `GET` | `/v1/validate` | API-Schlüssel-Gültigkeit + verbleibende Slots |
| `GET` | `/v1/slots` | Verbleibende Scans heute |
| `GET` | `/v1/report/history` | Scan-Verlauf (letzte 20) |
| `GET` | `/v1/stats` | Muster-Statistiken |
| `GET` | `/v1/status` | Server-Status |

---

## Links

- **GitHub:** [github.com/Moonsehwan/aina-scan](https://github.com/Moonsehwan/aina-scan)
- **Issues:** [github.com/Moonsehwan/aina-scan/issues](https://github.com/Moonsehwan/aina-scan/issues)
- **E-Mail:** shanyshany3528@gmail.com
