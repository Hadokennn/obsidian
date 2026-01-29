# Python 工程化基础

> 掌握 Python 项目管理的工程实践，从"能跑就行"到"专业规范"。

---

## 1. 虚拟环境（Virtual Environment）

### 1.1 为什么需要虚拟环境？

想象你同时在维护两个项目：
- **项目 A**：依赖 `requests==2.28.0`（旧版本 API）
- **项目 B**：依赖 `requests==2.30.0`（新版本 API）

如果没有隔离，全局只能装一个版本，两个项目必有一个报错。

**虚拟环境**给每个项目一个独立的 Python 环境，互不干扰。

### 1.2 uv 如何管理虚拟环境

```bash
# 这个命令背后发生了什么？
uv run main.py
```

uv 自动完成：
1. 检查当前目录是否有 `.venv/` 目录
2. 没有则自动创建（使用 `.python-version` 指定的 Python 版本）
3. 在虚拟环境中运行你的代码

### 1.3 验证隔离性

```bash
# 查看当前使用的 Python 路径
echo $VIRTUAL_ENV                  # 查看是否在虚拟环境中
uv run python -c "import sys; print(sys.executable)"
# 输出：/Users/xxx/project/.venv/bin/python

# 对比全局 Python
which python3
# 输出：/usr/local/bin/python3 或 /opt/homebrew/bin/python3
```

**关键区别**：两个路径不一样，证明是隔离的。

> **注意**：如果你之前运行过 `source .venv/bin/activate`，当前 shell 已经在虚拟环境里，这时 `which python3` 也会指向 `.venv`。退出用 `deactivate`。

---

## 2. 项目配置：pyproject.toml

### 2.1 文件作用

`pyproject.toml` 是 Python 项目的"配方表"，告诉 uv（或其他工具）如何构建和运行项目。

```toml
[project]
name = "python-learning"           # 项目名称
version = "0.1.0"                  # 项目版本
requires-python = ">=3.11"         # 最低 Python 版本要求
dependencies = [                   # 运行时依赖
    "requests>=2.32.5",
]
```

### 2.2 字段对照

| 字段 | 作用 | 前端类比 |
|------|------|----------|
| `requires-python` | 指定 Python 版本 | `.nvmrc` 指定 Node 版本 |
| `dependencies` | 运行时依赖列表 | `package.json` 的 `dependencies` |
| `version` | 项目版本号 | `package.json` 的 `version` |

---

## 3. 依赖锁定：uv.lock

### 3.1 为什么需要 lock 文件？

当你执行 `uv add requests` 时：

1. uv 解析 `pyproject.toml` 的依赖
2. 计算完整依赖树（requests 依赖 urllib3、charset-normalizer 等）
3. 把所有包的**精确版本**写入 `uv.lock`

```toml
# uv.lock 片段
[[package]]
name = "requests"
version = "2.32.5"
dependencies = [
    { name = "charset-normalizer" },
    { name = "idna" },
    { name = "urllib3" },
]
```

### 3.2 lock 文件的作用

- **版本一致**：团队协作时，所有人装的版本完全一样
- **避免"我这能跑"**：消除环境差异导致的 bug
- **可复现部署**：CI/CD 服务器能构建出相同环境

> 类比前端：`package-lock.json` 或 `yarn.lock`

---

## 4. uv 命令速查

### 4.1 日常开发

```bash
# 运行脚本（最常用）
uv run python main.py

# 进入交互式 Python
uv run python

# 直接运行（需要 shebang）
uv run main.py
```

### 4.2 依赖管理

```bash
# 添加依赖
uv add requests
uv add "requests>=2.30.0"          # 指定版本范围

# 同步依赖（按 lock 文件安装）
uv sync

# 查看依赖树
uv tree
```

### 4.3 uv vs npm 对照

| uv 命令 | npm 等价 | 作用 |
|---------|----------|------|
| `uv add <pkg>` | `npm install <pkg>` | 添加依赖 |
| `uv sync` | `npm ci` | 严格按 lock 安装 |
| `uv run <cmd>` | `npx <cmd>` | 在虚拟环境运行命令 |
| `uv lock` | `npm install` | 更新 lock 文件 |

---

## 5. Shebang 详解

### 5.1 什么是 Shebang？

文件第一行的 `#!` 标记，告诉系统"用哪个程序来执行我"。

```python
#!/usr/bin/env python3
# ↑ 这就是 shebang

def hello():
    print("Hello, World!")

hello()
```

### 5.2 词源

| 符号 | 专业叫法 | 口语叫法 |
|------|---------|---------|
| `#` | sharp | hash |
| `!` | exclamation mark | **bang** |

**hash + bang = shebang**（读快了变成 shebang）

### 5.3 使用场景

有 shebang 的文件：
```bash
chmod +x main.py    # 添加执行权限
./main.py           # 直接运行，不需要写 python3
```

没有 shebang 的文件：
```bash
python3 main.py     # 必须显式指定解释器
```

### 5.4 推荐做法

- **脚本文件**：加上 shebang，方便直接运行
- **项目代码**：不加也行，统一用 `uv run python main.py` 执行

---

## 6. 工程化最佳实践

### 6.1 项目结构

```
my_project/
├── .venv/                  # 虚拟环境（不提交 git）
├── .python-version         # Python 版本固定
├── pyproject.toml          # 项目配置
├── uv.lock                 # 锁定文件（提交 git）
├── .gitignore              # 忽略 .venv/
├── src/                    # 源代码
│   └── my_package/
│       ├── __init__.py
│       └── main.py
└── README.md
```

### 6.2 .gitignore 示例

```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so

# Virtual environments
.venv/
venv/
ENV/

# IDE
.vscode/
.idea/

# uv
*.egg-info/
dist/
build/
```

### 6.3 关键原则

| 原则 | 说明 |
|------|------|
| **虚拟环境隔离** | 每个项目独立环境，不污染全局 |
| **版本固定** | 用 `.python-version` 和 `uv.lock` 固定版本 |
| **lock 文件提交** | `uv.lock` 加入 git，确保一致性 |
| **不用全局 Python** | 统一用 `uv run` 运行，避免环境混淆 |

---

## 7. 常见问题

### Q: `uv run` 和 `source .venv/bin/activate` 有啥区别？

```bash
# ❌ 不推荐：手动激活
source .venv/bin/activate
python main.py

# ✅ 推荐：uv 自动处理
uv run python main.py
```

uv 的方式每个命令都是独立的，不会污染当前 shell 环境。

### Q: 为什么 `which python3` 和 `uv run which python` 返回一样？

说明你当前 shell 已经在虚拟环境里了（运行过 `source .venv/bin/activate`）。

```bash
deactivate  # 退出虚拟环境
which python3  # 现在应该返回全局路径了
```

### Q: `uv run main.py` 和 `uv run python main.py` 区别？

| 写法 | 作用 | 依赖条件 |
|------|------|----------|
| `uv run python main.py` | 用 python 解释器运行 | 永远可用 |
| `uv run main.py` | 当作可执行脚本运行 | 需要 shebang + 执行权限 |

推荐用 `uv run python main.py`，最稳妥。

---

## 总结

| 概念 | 一句话理解 |
|------|-----------|
| 虚拟环境 | 项目独立的 Python 环境，隔离依赖 |
| pyproject.toml | 项目的配方表，定义依赖和元信息 |
| uv.lock | 精确锁定所有依赖版本，保证一致性 |
| shebang | 文件首行的 `#!`，告诉系统用啥执行 |
| uv run | 在虚拟环境里运行命令的万能钥匙 |

掌握这些，你的 Python 项目就已经是专业级别的工程化管理了～ 🔧
