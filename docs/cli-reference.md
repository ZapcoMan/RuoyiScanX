# CLI 参数速查

> 完整参数说明可运行 `python main.py -h`。以下按功能分组列出全部参数。

---

## 核心扫描模式

| 参数 | 说明 |
|------|------|
| `-h` | 帮助信息 |
| `-u <url>` | 综合扫描（目录+漏洞+爆破） |
| `-m <url>` | 目录扫描 |
| `-p <url>` | 漏洞检测 |
| `-l <url>` | 登录爆破 |
| `-f <file>` | 批量扫描（从文件读取目标列表） |
| `--cms <ruoyi\|spring>` | 手动指定 CMS（跳过指纹识别） |
| `--pass-level <lvl>` | 口令字典级别 top100/top1000/full |
| `--template <name>` | 扫描模板（quick/deep/compliance/dengbao） |
| `--template-list` | 列出所有可用模板 |
| `--config <path>` | YAML 配置文件（CLI 参数优先级高于配置） |

---

## 网络与并发

| 参数 | 说明 |
|------|------|
| `--proxy <url>` | 代理地址（如 http://127.0.0.1:8080） |
| `--proxy-file <f>` | 代理池文件（每行一个代理 URL） |
| `--proxy-rotate <s>` | 代理轮换策略 round-robin/random/least-fail |
| `--threads <n>` | 并发线程数 |
| `--rate <n>` | 每秒请求数（0=不限速） |
| `--timeout <n>` | 请求超时秒数 |
| `--debug` | 调试模式（请求日志输出到 stderr） |

---

## 信息收集

| 参数 | 说明 |
|------|------|
| `--crawl` | 启用主动爬虫 |
| `--crawl-depth <n>` | 爬虫最大深度（默认 2） |
| `--crawl-max-pages <n>` | 爬虫最大页面数（默认 50） |
| `--subdomain` | 启用子域名枚举 |
| `--js-extract` | 启用 JS 端点提取 |
| `--portscan` | 端口扫描 + 服务识别 |
| `--ports <p1,p2>` | 自定义端口列表（逗号分隔） |
| `--passive` | 启动被动代理模式 |
| `--passive-host <addr>` | 代理监听地址（默认 127.0.0.1） |
| `--passive-port <n>` | 代理监听端口（默认 8080） |

---

## 报告与输出

| 参数 | 说明 |
|------|------|
| `--report <dir>` | 报告输出目录 |
| `--report-format <f>` | 报告格式 html/json/csv/pdf/docx/xlsx/sarif |
| `--no-dedup` | 关闭结果去重聚合 |
| `--lang <zh\|en>` | 报告语言（默认 zh） |
| `--diff <old.json>` | 与历史扫描报告对比 |
| `--diff-only <old> <new>` | 仅对比两个 JSON 报告 |
| `--save-baseline` | 保存本次扫描结果为基线 |

---

## WAF 绕过与利用链

| 参数 | 说明 |
|------|------|
| `--bypass-waf <auto\|on\|off>` | WAF 绕过策略（默认 auto） |
| `--chain <name>` | 执行漏洞利用链 |
| `--chain-list` | 列出所有可用的漏洞利用链 |

---

## 认证扫描

| 参数 | 说明 |
|------|------|
| `--auth <type=value>` | 认证注入（可多次指定） |
| `--auth-file <path>` | 从文件加载认证信息 |
| `--auth-login <user:pass>` | 自动登录获取认证 |

---

## Web API 服务

| 参数 | 说明 |
|------|------|
| `--serve` | 启动 Web API 服务（FastAPI + WebSocket + Web 控制台） |
| `--host <addr>` | API 服务监听地址（默认 0.0.0.0） |
| `--port <n>` | API 服务监听端口（默认 8000） |
| `--api-key <key>` | API Key 鉴权 |
| `--cors-origins <o>` | 允许的 CORS 源（逗号分隔） |
| `--db-path <path>` | SQLite 任务持久化数据库路径 |

详细的 API 端点说明、请求/响应示例、WebSocket 事件格式请参考 [API 文档](api.md)。

---

## OAST 带外检测

| 参数 | 说明 |
|------|------|
| `--oast` | 启用 OAST 带外检测 |
| `--oast-server` | 启动 OAST 回调服务器 |
| `--oast-host <addr>` | OAST 服务器监听地址 |
| `--oast-port <n>` | OAST 服务器监听端口 |

---

## 业务逻辑检测

| 参数 | 说明 |
|------|------|
| `--logic-scan` | 业务逻辑漏洞检测（IDOR/越权/参数篡改/竞争条件） |
| `--logic-endpoints <file>` | 业务扫描端点列表文件 |
| `--logic-concurrency <n>` | 竞争条件检测并发数 |

---

## 认证后深度扫描

| 参数 | 说明 |
|------|------|
| `--auth-surface` | 登录态接口资产盘点 + 越权矩阵（需 `--auth-login user:pass` 高权凭证） |
| `--surface-account <user:pass>` | 低权账号（垂直越权对比，可多次指定，取第一个登录成功者） |
| `--surface-output <path>` | 资产清单 JSON 输出路径 |

---

## CVE 同步

| 参数 | 说明 |
|------|------|
| `--cve-sync` | 同步 NVD CVE 信息 |
| `--cve-id <CVE-ID>` | 查询单个 CVE 信息 |
| `--nvd-api-key <key>` | NVD API Key（提高速率限制） |

---

## SIEM 集成

| 参数 | 说明 |
|------|------|
| `--siem-export <fmt>` | 导出 SIEM 格式（ecs/cef/leef/json） |
| `--siem-output <path>` | SIEM 导出路径 |
| `--siem-syslog <host:port>` | 发送到 Syslog 服务器 |
| `--siem-protocol <p>` | Syslog 协议 udp/tcp |

---

## 异步引擎

| 参数 | 说明 |
|------|------|
| `--async` | 启用异步扫描引擎（ThreadPoolExecutor） |
| `--async-workers <n>` | 异步并发线程数（默认 10） |

---

## Web UI 控制台

| 参数 | 说明 |
|------|------|
| `--web-ui` | 生成 Web UI 控制台（单页 HTML） |
| `--web-ui-output <path>` | Web UI 输出路径 |
| `--web-ui-api <url>` | Web UI 连接的 API 地址 |

---

## 分布式扫描

| 参数 | 说明 |
|------|------|
| `--distributed <mode>` | 分布式模式（master/worker/standalone） |
| `--redis-url <url>` | Redis 连接 URL |
| `--distributed-rate <n>` | 分布式全局限速（每秒请求数，0 不限速） |
| `--worker-max-tasks <n>` | Worker 最大任务数（0 不限） |
| `--distributed-timeout <n>` | 分布式超时秒数（默认 600） |

---

## 结果缓存

| 参数 | 说明 |
|------|------|
| `--cache` | 启用扫描结果缓存（SQLite） |
| `--cache-ttl <n>` | 缓存有效期秒数（默认 3600） |
| `--cache-db <path>` | 缓存数据库路径 |
| `--cache-stats` | 查看缓存统计 |
| `--cache-clear` | 清除过期缓存 |
| `--cache-clear-all` | 清除全部缓存 |

---

## 通知

| 参数 | 说明 |
|------|------|
| `--notify <type=target>` | 扫描完成通知（可多次指定） |

---

## 插件 SDK

| 参数 | 说明 |
|------|------|
| `--plugin-init <name>` | 生成插件模板 |
| `--plugin-check <path>` | 验证插件文件完整性 |
| `--plugin-list` | 列出所有已加载插件 |
| `--category <cat>` | 插件类别 ruoyi/spring/common |

---

## CI/CD 集成

| 参数 | 说明 |
|------|------|
| `--ci` | CI 模式（严重度超阈值时退出码非 0） |
| `--severity-threshold <lvl>` | CI 失败阈值 low/medium/high（默认 high） |
| `--ci-init <platform>` | 生成 CI 配置（github/gitlab/jenkins） |

---

## 漏洞知识库

| 参数 | 说明 |
|------|------|
| `--wiki` | 生成漏洞知识库（HTML Wiki + JSON API） |
| `--wiki-output <path>` | 知识库输出路径 |