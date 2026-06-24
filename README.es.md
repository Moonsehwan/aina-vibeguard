# AINAScan VibeGuard — Escáner de Seguridad para Código IA

**Verifica tu código generado por IA antes de desplegarlo.**

AINAScan detecta los **33 vulnerabilidades de seguridad + 15 bugs de vibe-coding = 48 patrones** que los asistentes de codificación IA (Claude, GPT, Cursor) generan repetidamente. 9 lenguajes. Análisis AST determinista — sin LLM, sin falsos positivos.

[![Beta Gratuita](https://img.shields.io/badge/beta-gratis-brightgreen)](https://pleasing-transformation-production-90c2.up.railway.app)
[![9 Lenguajes](https://img.shields.io/badge/lenguajes-9-blue)](#lenguajes-soportados)
[![48 Patrones](https://img.shields.io/badge/patrones-48-orange)](#patrones-de-detección)
[![P=R=F1=100%](https://img.shields.io/badge/F1-100%25-success)](#rendimiento)
[![GitHub](https://img.shields.io/badge/GitHub-aina--scan-181717?logo=github)](https://github.com/Moonsehwan/aina-scan)

---

## El Problema del Vibe-Coding

Programar con IA es rápido. El problema: los asistentes IA generan repetidamente código que **ejecuta pero no funciona**.

```python
# IA escribió esto — función de guardado que no guarda nada (MISSING_WRITE)
def save_user(data: dict):
    validated = validate(data)
    return {"status": "saved", "id": validated["id"]}  # ← sin INSERT

# IA escribió esto — bloquea el event loop (FAKE_ASYNC)
async def fetch_data(url: str):
    response = requests.get(url)   # ← sin await
    return response.json()

# IA escribió esto — inyección SQL (SQL_INJECTION_RISK)
def get_user(username: str):
    return db.execute(f"SELECT * FROM users WHERE name='{username}'")
```

AINAScan detecta estos bugs **en menos de 1 segundo** — antes de llegar a producción.

---

## Comenzar

### Opción 1 — Aplicación Web (sin instalación)

**👉 [https://pleasing-transformation-production-90c2.up.railway.app](https://pleasing-transformation-production-90c2.up.railway.app)**

1. Abre el enlace
2. Arrastra y suelta tu archivo
3. Resultados inmediatos — BLOCK (debe corregirse) / WARN (revisar)

Clave gratuita `vg_free_test` pre-completada. Sin registro.

---

### Opción 2 — REST API (automatización / CI)

```bash
curl -X POST https://pleasing-transformation-production-90c2.up.railway.app/v1/scan \
  -H "X-API-Key: vg_free_test" \
  -F "file=@your_file.py"
```

Extensiones soportadas: `.py` `.js` `.ts` `.tsx` `.go` `.java` `.kt` `.php` `.rb` `.c` `.cpp`

---

## Patrones de Detección (48)

### Vulnerabilidades de Seguridad (33)

| Patrón | Severidad | Detecta |
|--------|-----------|---------|
| `SQL_INJECTION_RISK` | BLOCK | SQL con f-string/% |
| `COMMAND_INJECTION` | BLOCK | Input usuario → subprocess/exec |
| `HARDCODED_SECRET` | BLOCK | api_key/password literal |
| `XSS_RISK` | BLOCK | innerHTML + datos de usuario |
| `EVAL_EXEC_RISK` | BLOCK | eval()/exec() + input usuario |
| `BUFFER_OVERFLOW` | BLOCK | C/C++ gets/strcpy/sprintf |
| + 27 más | WARN | SSRF, CSRF, criptografía débil, etc. |

### Bugs de Vibe-Coding (15) — Específicos de IA

| Patrón | Severidad | Patrón generado por IA |
|--------|-----------|----------------------|
| `STUB_SKELETON` | BLOCK | `def save(): return {}` — placeholder sin completar |
| `MISSING_WRITE` | BLOCK | Función de guardado sin escritura real en DB |
| `FAKE_ASYNC` | BLOCK | `async def` sin `await` |
| `INPUT_OUTPUT_DISCONNECTED` | BLOCK | Parámetros no afectan el valor de retorno |
| `HARDCODED_TABLE` | BLOCK | Diccionario hardcodeado en lugar de consulta DB |
| + 10 más | BLOCK/WARN | MOCK_PATTERN, TYPE_UNSAFE_ACCESS, etc. |

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

## Rendimiento

| Versión | Precisión | Recall | F1 | Casos de prueba |
|---------|-----------|--------|----|----------------|
| **v7.3** | **100%** | **100%** | **100%** | **90** |

- Velocidad de escaneo: archivos de menos de 500 KB en menos de 2 segundos
- Cero falsos positivos en 10 repos open-source con 100k+ ⭐

---

## Lenguajes Soportados

Python · JavaScript · TypeScript · Go · Java · PHP · Ruby · Kotlin · C · C++

---

## Enlaces

- **GitHub:** [github.com/Moonsehwan/aina-scan](https://github.com/Moonsehwan/aina-scan)
- **Issues:** [github.com/Moonsehwan/aina-scan/issues](https://github.com/Moonsehwan/aina-scan/issues)
- **Email:** shanyshany3528@gmail.com
