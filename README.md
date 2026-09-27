# RuoyiScanX — 若依专项漏洞扫描工具

<div align="center">

![Python](https://img.shields.io/badge/python-3.8+-blue.svg)
![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-009688)
![Plugins](https://img.shields.io/badge/POC-51-orange)
![Tests](https://img.shields.io/badge/Tests-1000%2B%20cases-success)
![License](https://img.shields.io/badge/License-MIT-yellow)

**基于 Python 3.8+ / FastAPI 的若依（RuoYi）专项漏洞扫描器**

[快速开始](#-快速开始) • [Docker 部署](#-docker-部署) • [文档中心](#-文档中心) • [二次开发](#-二次开发)

</div>

---

## 📋 项目简介

RuoyiScanX 是一款合法授权的若依（RuoYi）专项漏洞扫描工具，采用插件化架构，内置 51 个 POC，支持批量扫描、WAF 绕过、漏洞利用链、Web API 等企业级特性。

### 🙏 项目来源与致敬

> **RuoyiScanX 基于 [Ruoyi-Scan](https://github.com/xiabai2008/Ruoyi-Scan)（作者：XIABAI）进行二次开发。**
>
> 原项目 Ruoyi-Scan 是国内首个若依专项漏洞扫描器，首创了三态判定系统、插件化架构、WAF 绕过策略矩阵等核心能力，为若依生态的安全审计树立了标杆。
>
> 本项目在原项目 MIT License 授权范围内，继承其优秀的安全检测能力，并在此基础上进行功能扩展、架构优化与工程化改进。向原作者 XIABAI 致敬，感谢你的开源贡献为社区带来的价值。

### ✨ 核心特性

- 🎯 **三态判定系统**：CONFIRMED（确认存在）/ SAFE（确认不存在）/ UNKNOWN（无法判定），网络异常时强制返回 UNKNOWN，绝不冒充 SAFE
- 🔍 **若依专项深度**：识别 5 种若依变体（Vue3 / App / Plus / Cloud / Cloud-Plus），版本感知 POC 过滤（4.2 / 4.7 / v5 / 3.9.x）
- 🧩 **插件化架构**：51 个 POC 覆盖若依 18 / Spring 14 / 通用 11 / JeecgBoot 8 个插件包
- 🛡️ **WAF 绕过**：11 种策略 + 三态判定保护矩阵
- 🔗 **漏洞利用链**：DAG 拓扑编排，内置 3 条利用链
- 📊 **7 种报告格式**：HTML / JSON / CSV / PDF / Word / Excel / SARIF
- 🌐 **Web API + 控制台**：FastAPI REST + WebSocket 实时推送
- 🖥️ **桌面端支持**：React + Tauri 打包，单 exe 双击即用
- 🐳 **Docker 一键部署**：容器化部署，开箱即用
- 🧪 **完整测试覆盖**：51 个测试文件 / 1000+ 条用例
- 🤖 **AI POC 生成**：自然语言生成插件，LLM 自验证回灌
- 🌍 **分布式扫描**：Redis Master-Worker 队列 + Standalone 降级

### 🛠️ 技术栈

| 分类 | 技术 |
|------|------|
| **核心引擎** | Python 3.8+, requests, FastAPI, Docker |
| **插件系统** | PluginBase 抽象基类, entry_points 动态注册 |
| **Web API** | FastAPI, WebSocket, SQLite 任务持久化 |
| **Web 控制台** | Alpine.js（零构建，单文件 HTML） |
| **桌面端** | React 18, TypeScript, Vite, Tauri 2 |
| **测试** | pytest, requests_mock, 1000+ 用例 |
| **部署** | Docker, Docker Compose, PyPI |
| **监控** | Prometheus, Grafana |

---

## 🚀 快速开始

### ⚡ 方式一：pip 安装（推荐）

```bash
pip install ruoyi-scan

# 快速漏洞扫描
ruoyi-scan -p http://target:8080/

# 综合扫描（目录 + 漏洞 + 爆破）
ruoyi-scan -u http://target:8080/

# 批量扫描
ruoyi-scan -f targets.txt -p --report ./reports
```

✅ 安装即用，无需额外配置  
📖 可选依赖：`pip install pyyaml redis aiohttp`

---

### 💻 方式二：桌面端（双击即用）

前往 [Releases 页面](https://github.com/xiabai2008/ruoyi-scan/releases) 下载 `ruoyi-scan-desktop.exe`，约 46 MB，无需安装无需 Python 环境。

✅ 单 exe 文件，双击运行  
🖥️ 引擎与 51 个 POC 全部内嵌

---

### 🐳 方式三：Docker 部署

```bash
docker compose up -d                        # 一键启动（扫描器 + API + 2 个靶场）
docker compose run --rm scanner -p http://lab-ruoyi:8080/   # 扫描内置靶场
```

✅ 自动启动扫描器、API 服务、2 个签名靶场  
📖 详细说明：[桌面端文档](docs/desktop.md)

---

### 🔧 方式四：源码开发

```bash
git clone https://github.com/ZapcoMan/RuoyiScanX.git
cd RuoyiScanX
pip install -r requirements.txt

python main.py -p http://target:8080/
```

---

## 📁 项目结构

```
RuoyiScanX/
├── 📄 README.md                    # 项目说明（本文件）
├── 📄 main.py                      # CLI 入口（~440 行）
├── 📄 requirements.txt             # 依赖清单
├── 📄 pyproject.toml               # 现代打包配置
├── 📄 mkdocs.yml                   # 文档站配置
├── 🐳 Dockerfile                   # Docker 镜像构建
├── 🐳 docker-compose.yml           # Docker 服务编排
├── 📂 core/                        # 核心引擎层
│   ├── runner.py                   # 扫描编排器
│   ├── engine.py                   # 并发编排 + 令牌桶限速
│   ├── models.py                   # 数据模型（三态判定）
│   ├── loader.py                   # 插件动态发现
│   ├── fingerprint.py              # 指纹识别
│   ├── router.py                   # 指纹 → 插件路由
│   ├── session.py                  # HTTP 会话封装
│   ├── chain.py                    # 漏洞利用链引擎
│   ├── report.py                   # 报告渲染
│   └── cache.py                    # SQLite 结果缓存
├── 📂 plugins/                     # 插件系统（51 个 POC）
│   ├── base.py                     # PluginBase 抽象基类
│   ├── ruoyi/                      # 若依专项 18 个 POC
│   ├── spring/                     # Spring Boot 14 个 POC
│   ├── common/                     # 通用漏洞 11 个 POC
│   └── jeecgboot/                  # JeecgBoot 8 个 POC
├── 📂 lib/                         # 工具库（33 个模块）
│   ├── waf_features.py             # WAF 绕过策略
│   ├── ai_generator.py             # AI POC 生成
│   ├── ai_validate.py              # AI 生成即验证
│   ├── auth_chain.py               # 认证链
│   ├── captcha_solver.py           # 验证码处理
│   ├── report_template.py          # docx 报告模板引擎
│   ├── remediation.py              # 整改复测工作流
│   └── ...                         # 更多工具模块
├── 📂 api/                         # FastAPI Web API + WebSocket
├── 📂 cli/                         # CLI 子命令分发
├── 📂 chains/                      # 漏洞利用链（DAG 编排）
├── 📂 config/                      # 全局配置 + 指纹库 + 模板
├── 📂 data/                        # 字典文件 + CVE 离线库
├── 📂 common/                      # 通用工具
├── 📂 lab/                         # 签名靶场（Flask）
├── 📂 tests/                       # 51 个测试文件 / 1000+ 条用例
├── 📂 docs/                        # 📁 统一文档管理
├── 📂 web/                         # Web 控制台前端（Alpine.js）
├── 📂 desktop/                     # 桌面端（React + Tauri）
├── 📂 monitoring/                  # Prometheus + Grafana
├── 📂 examples/                    # 示例配置与 nuclei 模板
└── 📂 scripts/                     # 辅助脚本
```

---

## 🧪 测试说明

项目包含完整的测试套件，覆盖所有插件、核心引擎、API 端点。

| 类别 | 测试数量 | 说明 |
|------|---------|------|
| 插件回归 | 51 个文件 | 每个 POC 对应测试 |
| 核心引擎 | 200+ 用例 | runner / engine / models / fingerprint |
| API 端点 | 100+ 用例 | REST + WebSocket |
| 误报基线 | 50+ 用例 | 安全靶场零误报门禁 |
| **总计** | **1000+ 用例** | **pytest 一键运行** |

**快速运行：**
```bash
python -m pytest tests/ -q                    # 运行所有测试
python tests/regression_ruoyi.py              # 若依插件回归
python tests/regression_spring.py             # Spring 插件回归
python -m pytest tests/test_fp_baseline.py -q # 误报基线门禁
```

📖 详细文档：[贡献指南](docs/CONTRIBUTING.md)

---

## 📖 扫描模式

核心命令只有两个：`-p`（单目标漏洞扫描）和 `-u`（综合扫描）。

| 对比项 | `-p` 单目标漏洞扫描 | `-u` 综合扫描 |
|--------|----------------------|----------------|
| 执行内容 | 仅执行 `vuln` 类插件 | 全流程：recon → vuln → brute |
| 特点 | 速度快、请求量小 | 完整风险评估，耗时较长 |
| 适合场景 | 已知目标，快速确认漏洞 | 需要对目标做全面评估 |

### 主要功能模块

| 模块 | 说明 |
|------|------|
| 🔍 指纹识别 | 5 种若依变体 + 版本感知 |
| 🧩 漏洞检测 | 51 个 POC 插件 |
| 🛡️ WAF 绕过 | 11 种策略 |
| 🔗 利用链 | DAG 拓扑编排 |
| 📊 报告输出 | 7 种格式 |
| 🌐 Web API | FastAPI + WebSocket |
| 🖥️ 桌面端 | React + Tauri |
| 🐳 Docker | 一键部署 |

---

## 🔐 安全与合规

- **三态判定纪律**：网络异常强制返回 UNKNOWN，绝不冒充 SAFE
- **存在性验证**：涉及利用的插件默认仅做存在性验证，不做实际破坏
- **授权范围**：本工具仅用于授权范围内的安全测试与学习研究
- **供应链验证**：Release 产物附带 SHA256 校验和，v1.4.3 起提供 SLSA 构建来源证明
- **合规映射**：等保 2.0 / OWASP 报告级章节输出

### 三态判定系统详解

RuoyiScanX 的核心区别点，所有插件必须严格遵守：

```
STATUS_CONFIRMED   ← 漏洞确实存在（有明确证据链）
STATUS_SAFE        ← 漏洞确实不存在（有明确排除证据）
STATUS_UNKNOWN     ← 无法判定（网络异常 / 验证码 / 目标不匹配等）
```

**红线纪律：**
- 网络异常 → **必须返回 UNKNOWN**，绝不冒充 SAFE
- 未命中版本指纹 → **必须返回 UNKNOWN**（让上层过滤器处理跳过）
- safe 靶场零 CONFIRMED → CI 误报基线门禁强制

### 插件系统详解

| 插件包 | 数量 | 说明 |
|--------|------|------|
| `plugins/ruoyi/` | 18 | 文件读取、SQL 注入、RCE、SSTI、未授权等 |
| `plugins/spring/` | 14 | Actuator、Gateway、Jolokia、Spring4Shell 等 |
| `plugins/common/` | 11 | .git/.env 泄露、备份文件、CORS、Swagger 等 |
| `plugins/jeecgboot/` | 8 | JeecgBoot 拓展框架 |
| `plugins/chain/` | 3 | 漏洞利用链（DAG 拓扑编排） |

**新增插件：**
```bash
python main.py --plugin-init ruoyi_new_vuln --category ruoyi
python main.py --plugin-check plugins/ruoyi/ruoyi_new_vuln.py
python main.py --plugin-list   # 确认加载成功
```

### Web 控制台详解

**技术栈：**
- Alpine.js（CDN 引入，零构建）
- 内联 CSS（CSS Variables + Grid 布局）
- Fetch API + WebSocket 通信

**关键特性：**
1. **任务管理**：创建、查看、选择扫描任务
2. **实时日志**：WebSocket 推送扫描进度和结果
3. **结果展示**：三态判定可视化（CONFIRMED / SAFE / UNKNOWN）
4. **报告下载**：HTML / JSON / CSV / PDF / Word 格式
5. **指纹识别**：CMS 类型和置信度显示
6. **WAF 检测**：WAF 类型和绕过策略显示

### 桌面端详解

**技术栈：**
- React 18 + TypeScript
- Vite 6 构建
- Tailwind CSS v4 样式
- ECharts 图表
- Tauri 2 桌面壳（Rust 后端）

**目录结构：**
```
desktop/
├── src/
│   ├── components/           # React 组件
│   ├── views/                # 页面视图
│   ├── services/             # 后端通信
│   ├── state/                # 状态管理
│   └── styles/               # 主题样式
├── src-tauri/                # Tauri Rust 后端
└── package.json
```

---

## 🗄️ 为什么不直接用通用扫描器

RuoyiScanX **不与 nuclei / xray 竞争，而是互补** —— 它们负责广度，我们负责若依生态的深度与判定纪律。

| 维度 | nuclei | xray | **RuoyiScanX** |
|------|--------|------|----------------|
| 定位 | 通用 POC 引擎 | 通用 Web 扫描器 | **若依 / 国产 Java 框架专项** |
| 若依变体识别 | — | — | **5 变体**（Vue3 / App / Plus / Cloud / Cloud-Plus） |
| 版本感知 POC 过滤 | — | — | **4.2 / 4.7 / v5 / 3.9.x 版本矩阵** |
| 三态判定 | 无此概念 | 无此概念 | **CONFIRMED / SAFE / UNKNOWN** |
| 网络异常情形 | 报错或跳过 | — | **强制 UNKNOWN，绝不冒充 SAFE** |
| 合规映射 | — | — | 等保 2.0 / OWASP **报告级章节** |
| 安服交付物 | — | 部分 | 7 种格式 + **docx 模板引擎 + 整改复测** |
| nuclei 模板 | 原生 | 不支持 | **兼容执行**（http 协议子集 + 安全白名单） |

**一句话**：通用扫描器告诉你「这里可能有东西」；RuoyiScanX 告诉你「这个漏洞确实存在 / 确实不存在 / 无法判定」，并直接产出可交付的报告。

若依是国内应用最广的开源 Java 后台框架之一，二次开发项目在政企与外包中大量存在。通用工具在它面前的问题是**不知道对面是什么** —— 指纹粗糙、变体不辨、版本无感，于是误报与漏报同时发生。我们把「扫得准」做在前面。

---

## ❓ 常见问题

### 1. 如何关闭扫描结尾的 Star 提示？

```bash
# 运行时加参数
ruoyi-scan -p http://target:8080/ --no-cta

# 或设置环境变量（永久）
export RUOYI_SCAN_NO_CTA=1
```

### 2. 如何启用 WAF 绕过？

```bash
# 自动检测（推荐）
ruoyi-scan -p http://target:8080/ --bypass-waf auto

# 强制启用
ruoyi-scan -p http://target:8080/ --bypass-waf on
```

### 3. 如何启动 Web API 服务？

```bash
python main.py --serve --host 0.0.0.0 --port 8000
```

✅ API 地址：http://localhost:8000  
✅ Web 控制台：http://localhost:8000/web/  
✅ OpenAPI 文档：http://localhost:8000/docs

### 4. Docker 端口被占用

修改 `docker-compose.yml` 中的端口映射：
```yaml
ports:
  - "8081:8000"    # 将 API 端口改为 8081
  - "8082:8080"    # 将靶场端口改为 8082
```

### 5. 如何新增一个漏洞检测插件？

```bash
python main.py --plugin-init my_new_vuln --category ruoyi
# 编辑生成的插件文件，实现 verify() 方法
python main.py --plugin-check plugins/ruoyi/my_new_vuln.py
python main.py --plugin-list   # 确认加载成功
```

📖 详细说明：[二次开发指南](docs/SECONDARY_DEV.md)

---

## 🛠️ 二次开发

RuoyiScanX 采用 **MIT License** 开源，允许自由使用、修改、分发和商业化。

### 快速开始二次开发

```bash
# 1. 克隆仓库
git clone https://github.com/ZapcoMan/RuoyiScanX.git
cd RuoyiScanX

# 2. 创建虚拟环境
python -m venv .venv
.\.venv\Scripts\Activate.ps1    # Windows
source .venv/bin/activate       # Linux/macOS

# 3. 安装开发依赖
pip install -r requirements.txt
pip install -r requirements-dev.txt

# 4. 运行测试确认环境就绪
python -m pytest tests/ -q

# 5. 代码风格检查
ruff check core/ lib/ api/ plugins/ chains/ main.py
ruff format --check core/ lib/ api/ plugins/ chains/ main.py
```

### 常见二次开发场景

| 场景 | 说明 |
|------|------|
| 新增漏洞检测插件 | `--plugin-init` 生成骨架 → 实现 `verify()` → `--plugin-check` 验证 |
| 新增框架变体 | 参考 JeecgBoot 实现，扩展 fingerprint / router / versions |
| 修改判定逻辑 | 调整插件 `verify()` 内的三态判定条件 |
| 集成外部工具 | nuclei / Burp / DefectDojo / 通知系统 |
| 分布式扫描扩展 | 扩展 `core/distributed.py` 支持其他调度方式 |
| Web API 定制 | FastAPI 新增端点，权限继承三级矩阵 |

📖 **完整二次开发指南**：[docs/SECONDARY_DEV.md](docs/SECONDARY_DEV.md)

---

## 📚 文档中心

> 全部文档已统一管理在 `docs/` 目录下，推荐从 **[在线文档站](https://xiabai2008.github.io/ruoyi-scan/)** 开始阅读。

### 📖 用户文档

| 文档 | 说明 |
|------|------|
| [快速上手](docs/quickstart.md) | 5 分钟从安装到出第一份报告 |
| [用户指南](docs/usage.md) | 安装配置、扫描模式、详细参数说明 |
| [CLI 参考](docs/cli-reference.md) | 全部 CLI 参数按功能分组速查 |
| [核心能力总览](docs/features.md) | 27 个功能模块系统性梳理 |
| [插件开发教程](docs/plugin_dev.md) | PluginBase、三态判定、entry_points 注册 |
| [插件模板仓库](docs/template_repo.md) | 社区模板分发 + Ed25519 验签 |
| [API 文档](docs/api.md) | REST 端点、WebSocket 事件、OpenAPI 规范 |
| [DevSecOps 集成](docs/devsecops.md) | CI/CD 流水线模板、SARIF、DefectDojo |
| [版本矩阵](docs/version-matrix.md) | 若依各版本漏洞检出矩阵 |
| [检出能力矩阵](docs/verification-matrix.md) | 51 个 POC 的验证级别与证据等级 |
| [桌面端](docs/desktop.md) | 单 exe 架构、进程回收、卸载清理等实现细节 |

### 🛠️ 开发者文档

| 文档 | 说明 |
|------|------|
| **[二次开发指南](docs/SECONDARY_DEV.md)** | 🆕 面向二次开发者：架构总览、开发环境、插件扩展点、常见场景、打包分发 |
| [贡献指南](docs/CONTRIBUTING.md) | 开发流程、代码规范、PR 约定、测试规范 |
| [社区总览](docs/community.md) | Issue / PR / 社区治理 |
| [发布流程](docs/release.md) | Release 产物构建与发布流程 |
| [安全报告说明](docs/security_report.md) | 安全评估报告的解读 |
| [发展路线图](docs/ROADMAP.md) | G1-G5 阶段规划 + 度量仪表盘 |

### 🔧 工程文档

| 文档 | 说明 |
|------|------|
| [变更日志](docs/CHANGELOG.md) | 版本历史与变更记录（Keep a Changelog 格式） |
| [安全策略](docs/SECURITY.md) | 漏洞报告流程 + 合法使用声明 |

---

## 🤝 贡献指南

欢迎贡献 POC 与改进：

1. Fork 本仓库
2. 创建特性分支：`git checkout -b feature/amazing-feature`
3. 提交 POC：`python main.py --plugin-init <name>` 生成骨架 → 实现 `verify()`（三态判定）→ `--plugin-check` 验证
4. 提交更改：`git commit -m 'feat: add amazing vuln plugin'`
5. 推送分支：`git push origin feature/amazing-feature`
6. 提交 Pull Request

📖 详细规范：[贡献指南](docs/CONTRIBUTING.md)

---

## 📄 许可证

MIT License © 2026 XIABAI

本工具仅用于**授权范围内**的安全测试与学习研究。不得用于未授权目标。

---

## 📞 联系方式

- **本仓库**：https://github.com/ZapcoMan/RuoyiScanX
- **原项目**：https://github.com/xiabai2008/Ruoyi-Scan（致敬 XIABAI）
- **PyPI**：https://pypi.org/project/ruoyi-scan/
- **Issue**：https://github.com/ZapcoMan/RuoyiScanX/issues

如有问题或建议，欢迎提 Issue

---

*最后更新时间：2026-09-27*  
*当前版本：1.4.3*