# AINAScan VibeGuard — AI代码安全扫描器

**在部署之前检查您的AI生成代码。**

AINAScan检测AI编程助手（Claude、GPT、Cursor）反复生成的**33种安全漏洞 + 15种Vibe编码错误 = 48种模式**。支持9种语言。基于确定性AST分析——无LLM，零误报。

[![免费测试版](https://img.shields.io/badge/测试版-免费-brightgreen)](https://pleasing-transformation-production-90c2.up.railway.app)
[![9种语言](https://img.shields.io/badge/语言-9种-blue)](#支持语言)
[![48种模式](https://img.shields.io/badge/模式-48种-orange)](#检测模式)
[![P=R=F1=100%](https://img.shields.io/badge/F1-100%25-success)](#性能)
[![GitHub](https://img.shields.io/badge/GitHub-aina--scan-181717?logo=github)](https://github.com/Moonsehwan/aina-scan)

---

## Vibe编码的问题

AI辅助编码速度很快。问题在于AI助手反复生成**能运行但无法正常工作**的代码。

```python
# AI编写的代码 — 保存函数但实际上没有保存 (MISSING_WRITE)
def save_user(data: dict):
    validated = validate(data)
    return {"status": "saved", "id": validated["id"]}  # ← 没有INSERT语句

# AI编写的代码 — 阻塞事件循环 (FAKE_ASYNC)
async def fetch_data(url: str):
    response = requests.get(url)   # ← 没有await
    return response.json()

# AI编写的代码 — SQL注入 (SQL_INJECTION_RISK)
def get_user(username: str):
    return db.execute(f"SELECT * FROM users WHERE name='{username}'")
```

AINAScan在**1秒内**捕获这些错误——在部署到生产环境之前。

Semgrep、Bandit、Snyk等工具无法检测AI特有的模式。AINAScan专为Vibe编码错误而设计。

---

## 快速开始

### 方式1 — Web应用（无需安装，最简单）

**👉 [https://pleasing-transformation-production-90c2.up.railway.app](https://pleasing-transformation-production-90c2.up.railway.app)**

1. 打开上述链接
2. 拖拽您的文件上传
3. 即时获取结果 — BLOCK（必须修复）/ WARN（建议审查）

免费密钥 `vg_free_test` 已预填，无需注册。

> **隐私保护：** 上传的文件在分析后立即从内存中删除，不会存储在磁盘上。

---

### 方式2 — REST API（自动化/CI/CD集成）

```bash
# 免费密钥 — 无需注册
curl -X POST https://pleasing-transformation-production-90c2.up.railway.app/v1/scan \
  -H "X-API-Key: vg_free_test" \
  -F "file=@your_file.py"
```

**免费测试版密钥：** `vg_free_test` — 公测期间免费提供完整Pro功能。

支持扩展名：`.py` `.js` `.ts` `.tsx` `.go` `.java` `.kt` `.php` `.rb` `.c` `.cpp`

---

## 扫描整个项目

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

## API参考

### 1. 代码扫描 — POST /v1/scan

```bash
curl -X POST .../v1/scan \
  -H "X-API-Key: YOUR_KEY" \
  -F "file=@app.py"
```

**响应示例：**
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

**查询参数：**

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `include_senior` | `true` | 包含高级代码质量模式 (Python) |
| `enable_aina_advisor` | `true` | 启用AINA因果链分析 |

---

### 2. 项目文档生成 — POST /v1/docs ⭐ 新功能

上传项目zip，获取AI上下文的结构化文档。

```bash
zip -r myproject.zip ./src
curl -X POST .../v1/docs \
  -H "X-API-Key: vg_free_test" \
  -F "file=@myproject.zip" \
  -G -d "tier=pro&lang=zh"
```

**生成章节（7个）：**

| 章节 | 层级 | 内容 |
|------|------|------|
| 📁 项目结构 | 免费 | 文件树 + 函数/类数量 |
| 🔧 函数列表 | 免费 | 所有函数（参数、返回类型、行号） |
| 📦 依赖关系 | 免费 | requirements.txt中的外部包 |
| 🔗 导入图谱 | Pro | Mermaid图 — 文件间依赖关系 |
| 🌐 API端点 | Pro | 所有端点（方法、路径、处理器、行号） |
| 🛡️ 安全状态 | Pro | 每个文件的BLOCK/WARN数量 |
| 🪦 死代码 | Pro | 未被任何地方调用的函数 |

将生成的Markdown粘贴到AI助手中，可防止重复代码、错误导入和路由冲突。

---

### 3. 误报反馈 — POST /v1/feedback

```bash
curl -X POST .../v1/feedback \
  -H "X-API-Key: vg_free_test" \
  -H "Content-Type: application/json" \
  -d '{
    "scan_id": "your-scan-id",
    "vuln_type": "HARDCODED_TABLE",
    "is_false_positive": true,
    "note": "这是故意设置的配置表，不需要DB查询"
  }'
```

---

### 4. 其他端点

```bash
# 扫描历史（最近20条）
curl -H "X-API-Key: vg_free_test" .../v1/report/history

# 模式统计
curl -H "X-API-Key: vg_free_test" .../v1/stats
```

---

### 5. 内联禁用 — ainascan-disable

```python
# ainascan-disable HARDCODED_TABLE
COUNTRY_CODES = {"CN": "中国", "US": "美国"}  # 故意设置的查找表
```

---

## 检测模式（48种）

### 安全漏洞（33种）

| 模式 | 严重程度 | 检测内容 |
|------|----------|----------|
| `SQL_INJECTION_RISK` | BLOCK | SQL中使用f-string/% 格式化 |
| `COMMAND_INJECTION` | BLOCK | 用户输入 → subprocess/exec/system |
| `HARDCODED_SECRET` | BLOCK | api_key/password = "..." 字面量 |
| `PATH_TRAVERSAL` | BLOCK | 未验证的文件路径操作 |
| `XSS_RISK` | BLOCK | innerHTML/document.write + 用户数据 |
| `EVAL_EXEC_RISK` | BLOCK | eval()/exec() + 用户输入 |
| `DESERIALIZATION_RISK` | BLOCK | pickle.loads/unserialize不可信数据 |
| `DEBUG_MODE_RISK` | BLOCK | debug=True + host="0.0.0.0" |
| `IDOR_MISSING_AUTH_CHECK` | BLOCK | 无授权检查的直接对象访问 |
| `NOSQL_INJECTION` | BLOCK | 未验证的MongoDB查询 |
| `BUFFER_OVERFLOW` | BLOCK | C/C++ gets/strcpy/sprintf |
| `FORMAT_STRING` | BLOCK | C/C++ printf(var) 缺少格式参数 |
| `USE_AFTER_FREE` | BLOCK | C/C++ free()后使用指针 |
| `NULL_DEREF` | BLOCK | C/C++ malloc NULL检查缺失 |
| `INTEGER_OVERFLOW` | BLOCK | C/C++ malloc(atoi(input)) |
| `SSRF_RISK` | WARN | 服务端fetch使用未验证URL |
| `OPEN_REDIRECT` | WARN | 未验证的重定向目标 |
| `CSRF_MISSING_PROTECTION` | WARN | 无CSRF令牌的状态变更端点 |
| `WEAK_CRYPTO` | WARN | 使用MD5/SHA1 |
| `CORS_WILDCARD` | WARN | origins='*' |
| + 13种 | WARN | XXE、LDAP注入、模板注入等 |

### Vibe编码错误（15种）— AI代码专项

| 模式 | 严重程度 | AI生成的模式 |
|------|----------|-------------|
| `STUB_SKELETON` | BLOCK | `def save(): return {}` — 占位符未替换 |
| `MISSING_WRITE` | BLOCK | 保存/插入函数无实际DB写入 |
| `FAKE_ASYNC` | BLOCK | `async def` 无 `await` — 静默阻塞事件循环 |
| `INPUT_OUTPUT_DISCONNECTED` | BLOCK | 参数对返回值无任何影响 |
| `DEAD_CALL_RESULT` | BLOCK | 调用3个模块但全部结果被丢弃 |
| `HARDCODED_TABLE` | BLOCK | 8键+字典字面量代替DB查询 |
| `TRIVIAL_IF_CHAIN` | BLOCK | 7+个elif链代替DB查询 |
| `MOCK_PATTERN` | BLOCK | 生产代码中残留unittest.mock |
| `ENCODING_CORRUPTION` | BLOCK | 智能引号/BOM → 运行时UnicodeDecodeError |
| `TYPE_UNSAFE_ACCESS` | BLOCK | `float(d.get('key'))` — None时TypeError |
| `DB_SCHEMA_DRIFT` | BLOCK | SQL引用不存在于DB的列 |
| `LLM_DELEGATION` | BLOCK | 直接返回LLM响应，无自有推理 |
| `CONST_SQL_NO_PARAM` | BLOCK | SQL WHERE子句硬编码值 |
| `TRIVIAL_ASSERT` | WARN | 测试函数中只有 `assert True` |
| `FAKE_ASYNC` | WARN | 只有async关键字，无await（低置信度） |

---

## 自动减少误报

**库代码检测** — 当 `__all__`、`ABCMeta`、`@abstractmethod` 出现3次以上时，高噪声模式自动降级为WARN。

**测试文件检测** — 在 `test_*.py`、`*_test.py`、`conftest.py` 中，STUB/MOCK模式自动降级为WARN。

---

## GitHub Actions集成

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

## 性能

| 版本 | 精确率 | 召回率 | F1 | 测试用例 |
|------|--------|--------|----|---------|
| v5.0 | 93.8% | 83.3% | 88.2% | 30 |
| v7.0 | 100% | 100% | 100% | 60 |
| **v7.3** | **100%** | **100%** | **100%** | **90** |

- 扫描速度：500KB以下文件2秒内完成
- 缓存：相同文件重复扫描即时返回
- 在10个拥有100k+⭐的开源项目上零误报

---

## 支持语言

| 语言 | 扩展名 |
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

## 对比

| | AINAScan | Bandit | Semgrep | CodeRabbit |
|--|---------|--------|---------|------------|
| Vibe编码模式（15种） | ✅ 专用 | ❌ | ❌ | ❌ |
| 项目文档生成 | ✅ /v1/docs | ❌ | ❌ | ❌ |
| 确定性AST（无LLM） | ✅ | ✅ | ✅ | ❌ |
| 9种语言 | ✅ | ❌ 仅Python | ✅ | ✅ |
| 不存储代码 | ✅ | ✅ | ✅ | ❌ |
| GitHub Action | ✅ | ✅ | ✅ | ✅ |
| 免费层级 | ✅ 50文件/天 | ✅ | 有限 | ❌ |

---

## 隐私与安全

> 上传的文件永远不会被存储。分析仅在内存中进行，请求完成后立即销毁。

| 项目 | 处理方式 |
|------|----------|
| 上传文件 | 分析后立即从内存删除，不写入磁盘 |
| 源代码 | 永远不会存储、记录或用于训练 |
| 分析结果 | 仅返回给请求者，不在服务器端保留 |
| 模式统计 | 仅匿名聚合 |
| IP地址 | Railway基础设施标准日志（非我方控制） |

所有通信使用HTTPS（TLS 1.2+）。API密钥仅通过请求头传递。

隐私咨询：shanyshany3528@gmail.com

---

## 定价

| | 免费 | Pro | 团队 |
|--|------|-----|------|
| 文件/天 | 50 | 无限 | 无限 |
| 安全模式 | ✅ | ✅ | ✅ |
| Vibe编码模式 | ✅ | ✅ | ✅ |
| 项目文档 | 基础 | 完整 | 完整 |
| 跨文件分析 | ❌ | ❌ | ✅ |
| 价格 | 免费 | $19/月 | $60/月 |

---

## 端点列表

**基础URL：** `https://pleasing-transformation-production-90c2.up.railway.app`

所有端点需要 `X-API-Key` 请求头。

| 方法 | 路径 | 说明 |
|------|------|------|
| `POST` | `/v1/scan` | 扫描文件 |
| `POST` | `/v1/docs` | 项目zip → 结构文档 |
| `POST` | `/v1/feedback` | 报告误报 |
| `GET` | `/v1/validate` | API密钥有效性 + 剩余配额 |
| `GET` | `/v1/slots` | 今日剩余扫描次数 |
| `GET` | `/v1/report/history` | 扫描历史（最近20条） |
| `GET` | `/v1/stats` | 模式统计 |
| `GET` | `/v1/status` | 服务器状态 |

---

## 链接

- **GitHub：** [github.com/Moonsehwan/aina-scan](https://github.com/Moonsehwan/aina-scan)
- **问题反馈：** [github.com/Moonsehwan/aina-scan/issues](https://github.com/Moonsehwan/aina-scan/issues)
- **邮箱：** shanyshany3528@gmail.com
