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

Las herramientas estándar (Semgrep, Bandit, Snyk) no detectan los patrones específicos de IA. AINAScan está diseñado específicamente para bugs de vibe-coding.

---

## Comenzar

### Opción 1 — Aplicación Web (sin instalación, más fácil)

**👉 [https://pleasing-transformation-production-90c2.up.railway.app](https://pleasing-transformation-production-90c2.up.railway.app)**

1. Abre el enlace
2. Arrastra y suelta tu archivo
3. Resultados inmediatos — BLOCK (debe corregirse) / WARN (revisar)

Clave gratuita `vg_free_test` pre-completada. Sin registro.

> **Privacidad:** Los archivos subidos se eliminan de la memoria inmediatamente después del análisis. Nada se guarda en disco.

---

### Opción 2 — REST API (automatización / CI)

```bash
# Clave gratuita — sin registro
curl -X POST https://pleasing-transformation-production-90c2.up.railway.app/v1/scan \
  -H "X-API-Key: vg_free_test" \
  -F "file=@your_file.py"
```

**Clave beta gratuita:** `vg_free_test` — acceso Pro completo gratis durante la beta abierta.

Extensiones soportadas: `.py` `.js` `.ts` `.tsx` `.go` `.java` `.kt` `.php` `.rb` `.c` `.cpp`

---

## Escanear un Proyecto Completo

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

## Referencia de API

### 1. Escaneo de Código — POST /v1/scan

```bash
curl -X POST .../v1/scan \
  -H "X-API-Key: YOUR_KEY" \
  -F "file=@app.py"
```

**Respuesta:**
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

**Parámetros de consulta:**

| Parámetro | Por defecto | Descripción |
|-----------|-------------|-------------|
| `include_senior` | `true` | Incluir patrones de calidad de código senior (Python) |
| `enable_aina_advisor` | `true` | Activar análisis de cadena causal AINA |

---

### 2. Generador de Documentación del Proyecto — POST /v1/docs ⭐ Nuevo

Sube un zip del proyecto y obtén un documento estructurado para contexto de IA.

```bash
zip -r myproject.zip ./src
curl -X POST .../v1/docs \
  -H "X-API-Key: vg_free_test" \
  -F "file=@myproject.zip" \
  -G -d "tier=pro&lang=es"
```

**Secciones generadas (7):**

| Sección | Nivel | Contenido |
|---------|-------|-----------|
| 📁 Estructura del Proyecto | Gratis | Árbol de archivos + conteo de funciones/clases |
| 🔧 Lista de Funciones | Gratis | Todas las funciones con parámetros, tipos de retorno, números de línea |
| 📦 Dependencias | Gratis | Paquetes externos del requirements.txt |
| 🔗 Grafo de Importaciones | Pro | Diagrama Mermaid — qué archivos importan qué |
| 🌐 Endpoints API | Pro | Todos los endpoints (método, ruta, handler, número de línea) |
| 🛡️ Estado de Seguridad | Pro | Conteo de BLOCK/WARN por archivo |
| 🪦 Código Muerto | Pro | Funciones que no se llaman desde ningún lugar |

Pega el markdown generado en tu asistente IA antes de pedir nuevas funciones — previene código duplicado, imports erróneos y conflictos de rutas.

---

### 3. Reporte de Falso Positivo — POST /v1/feedback

```bash
curl -X POST .../v1/feedback \
  -H "X-API-Key: vg_free_test" \
  -H "Content-Type: application/json" \
  -d '{
    "scan_id": "your-scan-id",
    "vuln_type": "HARDCODED_TABLE",
    "is_false_positive": true,
    "note": "Tabla de configuración intencional, no necesita consulta DB"
  }'
```

---

### 4. Otros Endpoints

```bash
# Historial de escaneos (últimos 20)
curl -H "X-API-Key: vg_free_test" .../v1/report/history

# Estadísticas de patrones
curl -H "X-API-Key: vg_free_test" .../v1/stats
```

---

### 5. Desactivación en línea — ainascan-disable

```python
# ainascan-disable HARDCODED_TABLE
COUNTRY_CODES = {"ES": "España", "US": "Estados Unidos"}  # tabla de lookup intencional
```

---

## Patrones de Detección (48)

### Vulnerabilidades de Seguridad (33)

| Patrón | Severidad | Detecta |
|--------|-----------|---------|
| `SQL_INJECTION_RISK` | BLOCK | SQL con f-string/% |
| `COMMAND_INJECTION` | BLOCK | Input usuario → subprocess/exec/system |
| `HARDCODED_SECRET` | BLOCK | api_key/password literal |
| `PATH_TRAVERSAL` | BLOCK | Manipulación de ruta de archivo no validada |
| `XSS_RISK` | BLOCK | innerHTML/document.write + datos de usuario |
| `EVAL_EXEC_RISK` | BLOCK | eval()/exec() + input usuario |
| `DESERIALIZATION_RISK` | BLOCK | pickle.loads/unserialize datos no confiables |
| `DEBUG_MODE_RISK` | BLOCK | debug=True + host="0.0.0.0" |
| `IDOR_MISSING_AUTH_CHECK` | BLOCK | Acceso directo a objeto sin verificación de autorización |
| `NOSQL_INJECTION` | BLOCK | Consultas MongoDB no validadas |
| `BUFFER_OVERFLOW` | BLOCK | C/C++ gets/strcpy/sprintf |
| `FORMAT_STRING` | BLOCK | C/C++ printf(var) sin argumento de formato |
| `USE_AFTER_FREE` | BLOCK | C/C++ uso de puntero después de free() |
| `NULL_DEREF` | BLOCK | C/C++ verificación NULL de malloc faltante |
| `INTEGER_OVERFLOW` | BLOCK | C/C++ malloc(atoi(input)) |
| `SSRF_RISK` | WARN | URL no validada en fetch del lado servidor |
| `OPEN_REDIRECT` | WARN | Destino de redirección no validado |
| `CSRF_MISSING_PROTECTION` | WARN | Endpoint que cambia estado sin token CSRF |
| `WEAK_CRYPTO` | WARN | Uso de MD5/SHA1 |
| `CORS_WILDCARD` | WARN | origins='*' |
| + 13 más | WARN | XXE, inyección LDAP, inyección de plantilla, etc. |

### Bugs de Vibe-Coding (15) — Específicos de IA

| Patrón | Severidad | Patrón generado por IA |
|--------|-----------|----------------------|
| `STUB_SKELETON` | BLOCK | `def save(): return {}` — placeholder sin completar |
| `MISSING_WRITE` | BLOCK | Función de guardado/inserción sin escritura real en DB |
| `FAKE_ASYNC` | BLOCK | `async def` sin `await` — bloquea event loop silenciosamente |
| `INPUT_OUTPUT_DISCONNECTED` | BLOCK | Parámetros no tienen efecto alguno en el valor de retorno |
| `DEAD_CALL_RESULT` | BLOCK | 3 llamadas a módulos, todos los resultados descartados |
| `HARDCODED_TABLE` | BLOCK | Dict literal de 8+ claves donde debería ir consulta DB |
| `TRIVIAL_IF_CHAIN` | BLOCK | Cadena de 7+ elif donde debería ir consulta DB |
| `MOCK_PATTERN` | BLOCK | unittest.mock dejado en código de producción |
| `ENCODING_CORRUPTION` | BLOCK | Comillas tipográficas/BOM → UnicodeDecodeError en ejecución |
| `TYPE_UNSAFE_ACCESS` | BLOCK | `float(d.get('key'))` — TypeError cuando es None |
| `DB_SCHEMA_DRIFT` | BLOCK | SQL referencia columna que no existe en DB |
| `LLM_DELEGATION` | BLOCK | Respuesta LLM devuelta textualmente sin razonamiento propio |
| `CONST_SQL_NO_PARAM` | BLOCK | Valor codificado en cláusula SQL WHERE |
| `TRIVIAL_ASSERT` | WARN | Solo `assert True` en funciones de test |
| `FAKE_ASYNC` | WARN | Solo palabra clave async, sin await (baja confianza) |

---

## Reducción Automática de Falsos Positivos

**Detección de código de biblioteca** — cuando `__all__`, `ABCMeta`, `@abstractmethod` aparecen 3+ veces, los patrones de alto ruido se degradan automáticamente a WARN.

**Detección de archivos de test** — en `test_*.py`, `*_test.py`, `conftest.py`, los patrones STUB/MOCK se degradan automáticamente a WARN.

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

## Rendimiento

| Versión | Precisión | Recall | F1 | Casos de prueba |
|---------|-----------|--------|----|----------------|
| v5.0 | 93.8% | 83.3% | 88.2% | 30 |
| v7.0 | 100% | 100% | 100% | 60 |
| **v7.3** | **100%** | **100%** | **100%** | **90** |

- Velocidad de escaneo: archivos de menos de 500 KB en menos de 2 segundos
- Caché: archivos idénticos devueltos instantáneamente en re-escaneo
- Cero falsos positivos en 10 repos open-source con 100k+ ⭐

---

## Lenguajes Soportados

| Lenguaje | Extensiones |
|----------|------------|
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

## Comparación

| | AINAScan | Bandit | Semgrep | CodeRabbit |
|--|---------|--------|---------|------------|
| Patrones vibe-coding (15) | ✅ dedicado | ❌ | ❌ | ❌ |
| Generador de documentación | ✅ /v1/docs | ❌ | ❌ | ❌ |
| AST determinista (sin LLM) | ✅ | ✅ | ✅ | ❌ |
| 9 lenguajes | ✅ | ❌ Solo Python | ✅ | ✅ |
| Sin almacenamiento de código | ✅ | ✅ | ✅ | ❌ |
| GitHub Action | ✅ | ✅ | ✅ | ✅ |
| Nivel gratuito | ✅ 50 archivos/día | ✅ | Limitado | ❌ |

---

## Privacidad y Seguridad

> Los archivos subidos nunca se almacenan. El análisis ocurre solo en memoria y se descarta inmediatamente después de la solicitud.

| Elemento | Tratamiento |
|----------|-------------|
| Archivos subidos | Eliminados de la memoria tras el análisis. Sin escritura en disco. |
| Código fuente | Nunca almacenado, registrado ni usado para entrenamiento |
| Resultados del análisis | Solo devueltos al solicitante. Sin retención en el servidor. |
| Estadísticas de patrones | Solo agregadas anónimamente |
| Direcciones IP | Logs estándar de infraestructura Railway (fuera de nuestro control) |

Toda la comunicación usa HTTPS (TLS 1.2+). Las claves API se pasan solo en cabeceras de solicitud.

Consultas de privacidad: shanyshany3528@gmail.com

---

## Precios

| | Gratis | Pro | Equipo |
|--|--------|-----|--------|
| Archivos/día | 50 | Ilimitado | Ilimitado |
| Patrones de seguridad | ✅ | ✅ | ✅ |
| Patrones vibe-coding | ✅ | ✅ | ✅ |
| Documentación de proyecto | Básica | Completa | Completa |
| Análisis entre archivos | ❌ | ❌ | ✅ |
| Precio | Gratis | $19/mes | $60/mes |

---

## Endpoints

**URL base:** `https://pleasing-transformation-production-90c2.up.railway.app`

Todos los endpoints requieren el header `X-API-Key`.

| Método | Ruta | Descripción |
|--------|------|-------------|
| `POST` | `/v1/scan` | Escanear archivo |
| `POST` | `/v1/docs` | ZIP del proyecto → documento de estructura |
| `POST` | `/v1/feedback` | Reportar falso positivo |
| `GET` | `/v1/validate` | Validez de clave API + slots restantes |
| `GET` | `/v1/slots` | Escaneos restantes hoy |
| `GET` | `/v1/report/history` | Historial de escaneos (últimos 20) |
| `GET` | `/v1/stats` | Estadísticas de patrones |
| `GET` | `/v1/status` | Estado del servidor |

---

## Enlaces

- **GitHub:** [github.com/Moonsehwan/aina-scan](https://github.com/Moonsehwan/aina-scan)
- **Issues:** [github.com/Moonsehwan/aina-scan/issues](https://github.com/Moonsehwan/aina-scan/issues)
- **Email:** shanyshany3528@gmail.com
