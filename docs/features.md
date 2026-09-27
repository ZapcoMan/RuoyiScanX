# 核心能力总览

> 本文档系统性地梳理 Ruoyi-Scan 的全部功能模块。CLI 参数速查请见 [CLI 参考](cli-reference.md)，
> 二次开发指南请见 [SECONDARY_DEV.md](SECONDARY_DEV.md)。

---

## 一、插件系统

| 模块 | 说明 |
|------|------|
| `plugins/ruoyi/` | 若依 18 个插件（文件读取、SQL 注入、RCE、SSTI、未授权等） |
| `plugins/spring/` | Spring Boot 14 个 POC（Actuator、Gateway、Jolokia、Spring4Shell 等） |
| `plugins/common/` | 通用漏洞 11 个插件（.git/.env 泄露、备份文件、CORS、Swagger、中间件未授权等） |
| `plugins/jeecgboot/` | JeecgBoot 拓展框架 8 个插件（首个非若依框架拓展实证） |
| 插件模板仓库 | 导出/manifest/Ed25519 强制验签/`--plugin-update` 社区分发闭环 |

## 二、指纹识别

| 模块 | 说明 |
|------|------|
| 变体识别 | 5 变体（Vue3 / App / Plus / Cloud / Cloud-Plus）细分 |
| 版本感知 | 4.2 / 4.7 / v5 / v3.9.x 版本矩阵 + POC 自动过滤 |
| 特征源 | favicon hash + 特征路径 + 关键字 |

## 三、三态判定引擎

| 模块 | 说明 |
|------|------|
| 核心纪律 | CONFIRMED / SAFE / UNKNOWN 三态 |
| 网络异常 | 强制返回 UNKNOWN，绝不冒充 SAFE |
| WAF 绕过保护矩阵 | 11 种绕过策略 + 三态判定联动保护 |

## 四、WAF 绕过

| 模块 | 说明 |
|------|------|
| 绕过策略 | 11 种（大小写变换 / URL 编码 / 注释注入 / 路径折叠 / Content-Type 变形等） |
| 判定保护 | WAF 存在时自动启用，失败强制 UNKNOWN 而非误报 SAFE |
| 成功率追踪 | 记录每种绕过策略的命中情况 |

## 五、漏洞利用链

| 模块 | 说明 |
|------|------|
| 引擎 | DAG 拓扑编排 + 条件分支 |
| 内置链 | 3 条（如 ruoyi_sql_to_rce） |
| 自定义 | `chains/` 目录下添加 YAML 定义 |

## 六、nuclei 模板兼容

| 模块 | 说明 |
|------|------|
| 引擎 | http 协议子集 + 安全白名单 |
| 执行 | `--nuclei <dir>` 直接执行 YAML |
| 校验 | `--nuclei-validate` 不扫描仅校验模板结构 |
| 社区模板包 | `contrib/nuclei-templates/` 自建专项模板 |

## 七、AI 能力

| 模块 | 说明 |
|------|------|
| POC 生成 | `--ai "描述"` 自然语言生成插件，LLM 自验证回灌 |
| 安全降级 | 无 API Key 时降级规则模板 |
| AI 报告解读 | `--ai-report zh\|en` 自动生成漏洞分析与修复优先级 |
| UNKNOWN 降噪 | `--ai-triage` 无法判定结果按插件聚类分流（固定标签，AI 不得输出三态） |

## 八、批量扫描

| 模块 | 说明 |
|------|------|
| 目标源 | `-f targets.txt` 多目标 |
| 汇总 | 批量汇总报告 + 版本对照表 |
| 并发 | ThreadPoolExecutor + 令牌桶限速（锁外 sleep，无并发退化） |

## 九、报告输出

| 格式 | 说明 |
|------|------|
| HTML | SVG 图表 + 合规映射章节 |
| JSON | 结构化原始数据 |
| CSV | 表格格式，Excel 可读 |
| PDF | 可直接打印 |
| Word (docx) | 安服交付标准格式 |
| Excel (xlsx) | 按严重度分 sheet |
| SARIF | GitHub Code Scanning / GitLab 直接导入 |
| docx 模板引擎 | `--report-template` 套用安服公司自己的模板一键出交付物 |

## 十、Web API 与控制台

| 模块 | 说明 |
|------|------|
| 框架 | FastAPI REST + WebSocket 实时推送 |
| 控制台 | Web 控制台（任务管理 + 实时日志流 + 结果可视化） |
| 权限 | 三级（read / scan / admin），X-API-Key 头鉴权 |
| OpenAPI | `python scripts/export_openapi.py` 导出规范 |

## 十一、端口扫描

| 模块 | 说明 |
|------|------|
| TCP 扫描 | 多线程端口探测 |
| 服务识别 | Banner 抓取 |
| 联动 | 扫描结果自动关联漏洞插件 |

## 十二、被动代理

| 模块 | 说明 |
|------|------|
| 模式 | HTTP/HTTPS 被动代理 |
| 功能 | 捕获流量自动扫描 |
| 启动 | `python main.py --passive --passive-port 8080` |

## 十三、OAST 带外检测

| 模块 | 说明 |
|------|------|
| 回调服务器 | 自建 HTTP OAST 端点 |
| Payload 模板 | 6 种（SSRF / XXE / SQL 盲注 / RCE 盲注 / LDAP / 命令注入） |

## 十四、业务逻辑检测

| 模块 | 说明 |
|------|------|
| IDOR | 越权访问检测 |
| 参数篡改 | 请求参数修改探测 |
| 竞争条件 | 并发请求资源状态不一致检测 |

## 十五、认证后深度扫描

| 模块 | 说明 |
|------|------|
| 资产盘点 | 登录态接口资产提取 |
| 越权矩阵 | 匿名重放判未授权 / 低权重放判垂直越权 |
| JS 端点提取 | SPA 应用 JS 包 API 路径提取 |

## 十六、CVE 同步

| 模块 | 说明 |
|------|------|
| 数据源 | NVD + GHSA + CNVD（best-effort）+ 离线库四源 |
| 缓存 | 24h TTL |
| 合规映射 | CWE → OWASP / 等保 2.0 自动映射 |
| 组件检测 | 20 个 Java 组件（fastjson / SpringBoot / Shiro / Nacos / Log4j / Tomcat / Jenkins / Grafana 等）版本比对 CVE |

## 十七、SIEM 集成

| 模块 | 说明 |
|------|------|
| 格式 | ECS / CEF / LEEF / JSON |
| 传输 | Syslog UDP / TCP |

## 十八、异步引擎

| 模块 | 说明 |
|------|------|
| 并发模型 | ThreadPoolExecutor |
| HTTP | 可选 aiohttp 异步客户端 |

## 十九、分布式扫描

| 模块 | 说明 |
|------|------|
| 架构 | Redis Master-Worker 队列 |
| 降级 | Standalone 模式（无 Redis 时自动降级为单机） |
| 全局限速 | `--distributed-rate` 所有 worker 共享速率 |

## 二十、结果缓存

| 模块 | 说明 |
|------|------|
| 存储 | SQLite 持久化 |
| 键 | SHA256（目标 URL + 插件名） |
| TTL | 默认 3600 秒，可配置 |
| 统计 | 命中率 + 过期时间 |

## 二十一、扫描模板

| 模板 | 说明 |
|------|------|
| quick | 快速扫描，仅高置信度插件 |
| deep | 深度扫描，启用全部插件 + 爬虫 + OAST |
| compliance | 合规导向，关联等保条款插件 |
| dengbao | 等保 2.0 专项模板 |

## 二十二、认证扫描

| 模式 | 说明 |
|------|------|
| Cookie | 手动注入 Cookie |
| Token | 手动注入 Bearer Token |
| 自动登录 | `--auth-login user:pass` 自动获取 Token |
| 登录爆破 | `-l` 弱口令探测 |

## 二十三、验证码处理

| 模式 | 说明 |
|------|------|
| 自动探测 | 自动识别验证码接口 |
| OCR 识别 | 识别图像验证码 |
| 跳过 | 遇到验证码直接 UNKNOWN |

## 二十四、国际化

| 模块 | 说明 |
|------|------|
| 报告 | `--lang zh\|en` 中英文切换 |
| 核心文案 | 控制台输出统一中文，可在 `common/console.py` 扩展 |

## 二十五、插件 SDK

| 命令 | 说明 |
|------|------|
| `--plugin-init <name>` | 生成插件模板骨架 |
| `--plugin-check <path>` | 验证插件文件完整性 |
| `--plugin-list` | 列出已加载插件 |
| entry_points | 第三方插件通过 pip install 自动注册 |

## 二十六、CI/CD 集成

| 特性 | 说明 |
|------|------|
| 严重度阈值退出 | `--ci --severity-threshold` CI 模式下严重度超阈值时退出码非 0 |
| GitHub Code Scanning | SARIF 格式直接导入 |
| 模板生成 | `--ci-init github\|gitlab\|jenkins` 生成流水线配置 |

## 二十七、漏洞知识库

| 模块 | 说明 |
|------|------|
| 离线 HTML Wiki | `--wiki` 生成 |
| JSON API | 程序化获取 |
| 数据源 | 全部插件的 verify 特征 + CVE 详情聚合 |