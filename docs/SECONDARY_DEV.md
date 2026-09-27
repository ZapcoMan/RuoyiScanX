# 二次开发指南

> 本文档面向在 Ruoyi-Scan 基础上进行二次开发的开发者。在动手之前，请确保你已阅读
> [贡献指南](CONTRIBUTING.md)，熟悉项目代码规范与开发流程。

---

## 一、授权声明

Ruoyi-Scan 采用 **MIT License** 开源，允许自由使用、修改、分发和商业化，但请遵守以下原则：

| 允许 | 不允许 |
|------|--------|
| 私人学习研究 | 移除原始版权声明 |
| 内部安全评估工具 | 用工具进行未授权攻击 |
| 二次开发并发布衍生版本 | 冒充作者或声称官方背书 |
| 商业化集成 | 利用漏洞检测能力进行破坏性利用 |

如果你的二次开发是面向**商业产品**，建议同时关注项目 [Roadmap](ROADMAP.md) 中的 Open Core 边界设计，避免与未来闭源功能冲突。

---

## 二、项目架构总览

```
Ruoyi-Scan/
├── main.py                  # CLI 入口（~440 行，纯参数解析+分发）
├── config/settings.py       # 全局配置（版本号、路径常量等）
├── core/                    # 核心引擎层
│   ├── runner.py            # 扫描编排器
│   ├── engine.py            # 并发编排 + 令牌桶限速
│   ├── models.py            # 数据模型（三态判定）
│   ├── loader.py            # 插件动态发现
│   ├── fingerprint.py       # 指纹识别
│   ├── router.py            # 指纹 → 插件路由
│   ├── session.py           # HTTP 会话封装
│   ├── chain.py             # 漏洞利用链引擎
│   ├── report.py            # 报告渲染
│   └── cache.py             # SQLite 结果缓存
├── plugins/                 # 插件系统
│   ├── base.py              # PluginBase 抽象基类（必须继承）
│   ├── ruoyi/               # 若依专项 18 个 POC
│   ├── spring/              # Spring Boot 14 个 POC
│   ├── jeecgboot/           # JeecgBoot 8 个 POC
│   └── common/              # 通用漏洞 11 个 POC
├── lib/                     # 工具库（33 个模块）
│   ├── waf_features.py      # WAF 绕过策略
│   ├── auth_chain.py        # 认证链
│   ├── captcha_solver.py    # 验证码处理
│   ├── ai_generator.py      # AI POC 生成
│   ├── ai_validate.py       # AI 生成即验证
│   ├── auth_surface.py      # 认证后深扫
│   ├── component_detect.py  # 组件版本检测
│   ├── cve_sync.py          # CVE 多源同步
│   ├── report_template.py   # docx 报告模板引擎
│   ├── remediation.py       # 整改复测工作流
│   └── ...                  # 更多工具模块
├── chains/                  # 漏洞利用链（DAG 编排）
├── api/                     # FastAPI Web API + WebSocket
├── cli/                     # CLI 子命令分发
├── common/                  # 通用工具（console / console 编码强制）
├── data/                    # 字典文件 + CVE 离线库
├── lab/                     # 签名靶场（Flask 模拟靶标）
├── tests/                   # 测试套件（1000+ 用例）
└── web/                     # Web 控制台前端
```

---

## 三、开发环境搭建

```bash
# 1. 克隆仓库
git clone https://github.com/<your-username>/Ruoyi-Scan.git
cd Ruoyi-Scan

# 2. 创建虚拟环境
python -m venv .venv
.\.venv\Scripts\Activate.ps1    # Windows PowerShell
source .venv/bin/activate       # Linux/macOS

# 3. 安装开发依赖
pip install -r requirements.txt
pip install -r requirements-dev.txt

# 4. 运行测试确认环境就绪
python -m pytest tests/ -q

# 5. 代码风格检查（提交前必过）
ruff check core/ lib/ api/ plugins/ chains/ main.py
ruff format --check core/ lib/ api/ plugins/ chains/ main.py
```

---

## 四、核心概念：三态判定系统

Ruoyi-Scan 的核心区别点，所有插件必须严格遵守：

```
STATUS_CONFIRMED   ← 漏洞确实存在（有明确证据链）
STATUS_SAFE        ← 漏洞确实不存在（有明确排除证据）
STATUS_UNKNOWN     ← 无法判定（网络异常 / 验证码 / 目标不匹配等）
```

**红线纪律：**
- 网络异常 → **必须返回 UNKNOWN**，绝不冒充 SAFE
- 未命中版本指纹 → **必须返回 UNKNOWN**（让上层过滤器处理跳过）
- safe 靶场零 CONFIRMED → CI 误报基线门禁强制

```python
from plugins.base import PluginBase
from core.models import STATUS_CONFIRMED, STATUS_SAFE, STATUS_UNKNOWN


class YourPlugin(PluginBase):
    def verify(self, target, session):
        resp = session.get(f"{target}/vulnerable-path")
        if resp.status_code == 200 and "漏洞特征关键词" in resp.text:
            return self._build_result(STATUS_CONFIRMED, url=resp.url, evidence="特征1 + 特征2")
        elif resp.status_code == 404:
            return self._build_result(STATUS_SAFE, url=resp.url)
        else:
            return self._build_result(STATUS_UNKNOWN, url=resp.url, evidence=f"状态码 {resp.status_code}")
```

---

## 五、常见二次开发场景

### 场景 A：新增一个漏洞检测插件

```bash
python main.py --plugin-init ruoyi_new_vuln --category ruoyi
```

生成骨架在 `plugins/ruoyi/ruoyi_new_vuln.py`，填写元数据（`name` / `cve` / `severity` / `cvss_vector` / `compliance` 等）和 `verify()` 方法。完成后：

```bash
python main.py --plugin-check plugins/ruoyi/ruoyi_new_vuln.py
python main.py --plugin-list   # 确认加载成功
```

**必须：** 新增对应的签名靶场响应（`lab/`）和单元测试（`tests/test_*.py`，使用 `requests_mock` mock 网络请求）。

### 场景 B：新增一个框架变体

如果需要支持新的框架（如芋道 yudao），参考 JeecgBoot 的实现：

1. 在 `plugins/` 下新建目录 `plugins/yudao/`
2. 在 `core/fingerprint.py` 添加变体指纹特征
3. 在 `core/router.py` 添加指纹 → 插件目录路由映射
4. 在 `core/ruoyi_versions.py` 添加变体元数据表

### 场景 C：修改判定逻辑

所有插件的 `verify()` 返回值直接影响报告结果。如果要调整某个插件的判定策略：

1. 找到 `plugins/<category>/<plugin_name>.py`
2. 修改 `verify()` 内的三态判定条件
3. 更新该插件的单元测试（`tests/regression_ruoyi.py` 等）
4. **在 lab 靶场验证**：vuln 模式 CONFIRMED / safe 模式零误报

### 场景 D：集成外部工具

项目已预留多个集成点：

| 集成点 | 位置 | 说明 |
|--------|------|------|
| nuclei 模板 | `lib/nuclei_loader.py` | http 协议子集 + 安全白名单，直接执行 nuclei YAML |
| Burp 联动 | `core/proxy_server.py` | 被动代理模式 `--passive`，流量自动捕获扫描 |
| DefectDojo | `lib/siem_connector.py` | SIEM ECS/CEF/LEEF/JSON 格式导出 |
| 钉钉/企微/飞书 | `lib/notifier.py` | 通知骨架，扩展 `notify()` 实现推送 |

### 场景 E：分布式扫描扩展

当前实现基于 **Redis Master-Worker** 队列（`--distributed master|worker`），降级模式为 standalone。如需支持其他调度方式（如 RabbitMQ / Celery），扩展 `core/distributed.py` 即可。

### 场景 F：Web API 定制

`api/` 基于 FastAPI，新增端点只需：

```python
from fastapi import APIRouter, HTTPException

router = APIRouter()

@router.post("/custom/scan")
async def custom_scan(request: CustomRequest):
    # 调用 core.runner 执行扫描
    ...
```

权限继承系统的三级矩阵（read / scan / admin），通过 `X-API-Key` 头鉴权。

---

## 六、测试与调试

```bash
# 运行全部测试
python -m pytest tests/ -q

# 仅运行某个插件的回归测试
python tests/regression_ruoyi.py
python tests/regression_spring.py

# 手动验证插件（需要靶场运行中）
# 1. 启动靶场
docker compose up -d lab-ruoyi
# 2. 跑单插件扫描
python main.py -p http://localhost:8080/ --plugin-list

# 误报基线门禁（对完全正常的响应断言零 CONFIRMED）
python -m pytest tests/test_fp_baseline.py -q
```

**调试技巧：**
- `--debug` 输出详细请求日志到 stderr
- 设置环境变量 `RUOYI_SCAN_DEBUG=1`
- 插件级调试：在 `verify()` 内临时 `print()` 证据链内容（开发期）

---

## 七、打包与分发

### CLI 版本（PyPI）

```bash
# 构建 wheel
python -m build
# 本地安装验证
pip install dist/ruoyi_scan-*.whl --force-reinstall
```

### 桌面端 exe

桌面端通过 **PyInstaller + Tauri** 打包，CI 流程在 `.github/workflows/release.yml`。

关键实现点（`desktop/` 目录）：
- 引擎编译期嵌入壳二进制
- 运行时自解压到 `%LOCALAPPDATA%\Ruoyi-Scan\engine\`
- 子进程挂 Windows JobObject，壳崩溃时引擎树自动回收
- NSIS 安装包含卸载清理钩子

### Docker 镜像

```bash
docker build -t ruoyi-scan .
docker compose up -d   # 一键启动（扫描器 + API + 2 个靶场）
```

---

## 八、提交清单 Checklist

在提交 PR 之前，请确保：

- [ ] 功能代码 + 单元测试全部通过（`pytest` 零失败）
- [ ] `ruff check` 零错误、`ruff format --check` 通过
- [ ] 新增漏洞插件已在 lab 靶场验证：vuln 模式 CONFIRMED / safe 模式零误报
- [ ] 误报基线门禁 `tests/test_fp_baseline.py` 对新插件零 CONFIRMED
- [ ] `mypy core/ --strict` 通过（如修改了 core 模块）
- [ ] CHANGELOG.md 已新增条目
- [ ] 文档已同步更新（如涉及新参数或新插件）

---

## 九、获取帮助

如果在二次开发中遇到问题：

1. 先查 [插件开发教程](plugin_dev.md) —— 覆盖 PluginBase、三态判定、entry_points 注册
2. 再查 [API 文档](api.md) —— REST 端点、WebSocket 事件、OpenAPI 规范
3. 最后开 Issue —— 附带复现步骤和你期望的行为