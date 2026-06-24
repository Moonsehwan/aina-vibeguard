# AINAScan VibeGuard — AI代码安全扫描器

**在部署之前检查您的AI生成代码。**

AINAScan检测AI编程助手（Claude、GPT、Cursor）反复生成的**33种安全漏洞 + 15种Vibe编码错误 = 48种模式**。支持9种语言。基于确定性AST分析——无LLM，无误报。

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

### 方式1 — Web应用（无需安装）

**👉 [https://pleasing-transformation-production-90c2.up.railway.app](https://pleasing-transformation-production-90c2.up.railway.app)**

1. 打开上述链接
2. 拖拽您的文件上传
3. 即时获取结果 — BLOCK（必须修复）/ WARN（建议审查）

免费密钥 `vg_free_test` 已预填，无需额外设置。

---

### 方式2 — REST API（自动化/CI/CD）

```bash
curl -X POST https://pleasing-transformation-production-90c2.up.railway.app/v1/scan \
  -H "X-API-Key: vg_free_test" \
  -F "file=@your_file.py"
```

支持扩展名：`.py` `.js` `.ts` `.tsx` `.go` `.java` `.kt` `.php` `.rb` `.c` `.cpp`

---

## 检测模式（48种）

### 安全漏洞（33种）

| 模式 | 严重程度 | 检测内容 |
|------|----------|----------|
| `SQL_INJECTION_RISK` | BLOCK | f-string/% 格式化SQL |
| `COMMAND_INJECTION` | BLOCK | 用户输入 → subprocess/exec |
| `HARDCODED_SECRET` | BLOCK | api_key/password 硬编码 |
| `XSS_RISK` | BLOCK | innerHTML + 用户数据 |
| `EVAL_EXEC_RISK` | BLOCK | eval()/exec() + 用户输入 |
| `BUFFER_OVERFLOW` | BLOCK | C/C++ gets/strcpy/sprintf |
| + 27种 | WARN | SSRF、CSRF、弱加密等 |

### Vibe编码错误（15种）— AI代码专项

| 模式 | 严重程度 | AI生成的模式 |
|------|----------|-------------|
| `STUB_SKELETON` | BLOCK | `def save(): return {}` — 占位符未替换 |
| `MISSING_WRITE` | BLOCK | 保存函数无实际DB写入 |
| `FAKE_ASYNC` | BLOCK | `async def` 无 `await` |
| `INPUT_OUTPUT_DISCONNECTED` | BLOCK | 参数对返回值无影响 |
| `HARDCODED_TABLE` | BLOCK | 硬编码字典代替DB查询 |
| + 10种 | BLOCK/WARN | MOCK_PATTERN、TYPE_UNSAFE_ACCESS等 |

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

## 性能

| 版本 | 精确率 | 召回率 | F1 | 测试用例 |
|------|--------|--------|----|---------|
| v7.3 | **100%** | **100%** | **100%** | 90 |

- 扫描速度：500KB以下文件2秒内完成
- 零误报：在10个拥有10万+⭐的开源项目上验证

---

## 支持语言

Python · JavaScript · TypeScript · Go · Java · PHP · Ruby · Kotlin · C · C++

---

## 链接

- **GitHub:** [github.com/Moonsehwan/aina-scan](https://github.com/Moonsehwan/aina-scan)
- **问题反馈:** [github.com/Moonsehwan/aina-scan/issues](https://github.com/Moonsehwan/aina-scan/issues)
- **邮箱:** shanyshany3528@gmail.com
