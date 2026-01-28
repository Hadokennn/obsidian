> 日期：2026-01-28
> 背景：前端工程师转型学习 Python

---

## 1. 列表推导式 + f-string

### 题目
将用户列表转换成前端需要的格式。

```python
users = [
    {"id": 1, "name": "Alice", "age": 25},
    {"id": 2, "name": "Bob", "age": 30},
]

# 目标：[{"label": "Alice (25岁)", "value": 1}, ...]
```

### 答案
```python
users = [
    {"label": f"{user['name']} ({user['age']}岁)", "value": user["id"]}
    for user in users
]
```

### 要点
- f-string 用 `{}` 嵌入表达式
- 列表推导式：`[表达式 for 变量 in 可迭代对象]`

---

## 2. 函数参数：*args, **kwargs

### 题目
实现灵活的日志函数，支持任意参数和关键字参数。

```python
log("用户登录", "user_id=123", "ip=192.168.1.1", level="DEBUG")
# 输出：[DEBUG] 用户登录 - user_id=123, ip=192.168.1.1
```

### 答案
```python
def log(message: str, *args, level: str = "INFO", timestamp=None):
    ts = f"[{timestamp}] " if timestamp else ""
    extra = ", ".join(args) if args else ""
    print(f"{ts}[{level}] {message} - {extra}")
```

### 要点

| 写法 | 接收形式 | 内部类型 | 用途 |
|------|---------|---------|------|
| `*args` | 多余位置参数 | **元组 tuple** | 收集任意数量的位置参数 |
| `**kwargs` | 多余关键字参数 | 字典 dict | 收集任意数量的关键字参数 |

**展开用法：**
```python
data = [1, 2, 3]
config = {"x": 10, "y": 20}
foo(*data, **config)  # 等价于 foo(1, 2, 3, x=10, y=20)
```

---

## 3. 强制关键字参数（单独的 *）

### 语法
```python
def connect(host, port, *, timeout=30, retry=3):
    pass
```

`*` 后面的参数**必须用关键字传参**：
```python
connect("localhost", 8080, timeout=30, retry=3)  # ✅
connect("localhost", 8080, 30, 3)                 # ❌ 报错！
```

### 目的
提高代码可读性，避免参数顺序混淆。

---

## 4. enumerate 与解包

### 题目
遍历列表同时获取索引和值。

```python
items = ["apple", "banana", "cherry"]
# 输出：
# 0: apple
# 1: banana
# 2: cherry
```

### 答案
```python
for index, item in enumerate(items):
    print(f"{index}: {item}")
```

### 要点
- `enumerate()` 返回 `(索引, 值)` 的迭代器
- 注意顺序：`for index, value in enumerate(...)`，不要写反

---

## 5. 字典深度合并

### 题目
合并两个字典，嵌套字典也要递归合并。

```python
default = {"api": {"host": "localhost", "port": 3000}, "debug": False}
user = {"api": {"port": 8080}, "features": ["auth"]}
# 结果：{"api": {"host": "localhost", "port": 8080}, "debug": False, "features": [...]}
```

### 答案
```python
def deep_merge(dic1: dict, dic2: dict) -> dict:
    copy = dic1.copy()
    for key, value in dic2.items():
        if isinstance(value, dict) and key in copy and isinstance(copy[key], dict):
            copy[key] = deep_merge(copy[key], value)
        else:
            copy[key] = value
    return copy
```

### 要点
- 递归处理嵌套字典
- 注意边界情况：检查两边是否都是字典

---

## 6. 装饰器

### 完整示例
```python
import time
import functools

def timer(func):
    """给函数添加执行计时功能"""
    @functools.wraps(func)  # 保留原函数信息
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)  # 执行原函数
        elapsed = time.time() - start
        print(f"{func.__name__} 执行耗时: {elapsed:.3f}秒")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)
    return "done"

slow_function()  # 输出：slow_function 执行耗时: 1.00x秒
```

### 核心概念

**装饰器 ≈ React 高阶组件（HOC）**

```javascript
// React HOC
function withLogging(WrappedComponent) {
  return function(props) {
    console.log('渲染前');
    return WrappedComponent(props);
  };
}
```

```python
# Python 装饰器
def timer(func):
    def wrapper(*args, **kwargs):
        print('执行前');
        return func(*args, **kwargs)
    return wrapper
```

### @functools.wraps(func) 的作用

**不用它：**
```python
@timer
def my_func():
    """文档"""
    pass

print(my_func.__name__)  # wrapper ❌
print(my_func.__doc__)   # None ❌
```

**用它：**
```python
@functools.wraps(func)
def wrapper(*args, **kwargs):
    ...

print(my_func.__name__)  # my_func ✅
print(my_func.__doc__)   # 文档 ✅
```

### 装饰器执行流程

```
1. @timer 把 slow_function 传给 timer
2. timer 返回 wrapper
3. slow_function 现在指向 wrapper
4. 调用 slow_function() 其实是调用 wrapper()
5. wrapper 内部：计时 → 调用原函数 → 返回结果
```


## 7. 切片（Slicing）

### 语法
```python
items[start:end:step]
# start: 起始索引（包含，默认0）
# end: 结束索引（不包含，默认末尾）
# step: 步长（默认1，负数为倒序）
```

### 示例
```python
s = "Hello, Python!"
arr = [0, 1, 2, 3, 4, 5]

s[0:5]       # "Hello"（前5个字符）
s[:5]        # "Hello"（start省略，从0开始）
s[7:]        # "Python!"（end省略，到末尾）
s[-6:-1]     # "ython"（负数索引，-1是最后一个）
s[::2]       # "Hlo yhn"（每2个取1个）
s[::-1]      # "!nohtyP ,olleH"（倒序！）

arr[1:4]     # [1, 2, 3]
arr[::-1]    # [5, 4, 3, 2, 1, 0]（数组倒序）
```

### 与 JS 对比
```javascript
// JS 没有原生切片，只能用 slice
arr.slice(1, 4)
arr.reverse()  // 会修改原数组或返回新数组，比较混乱
```

```python
# Python 切片更强大，不修改原数据
arr[1:4]    # 返回新列表，原列表不变
arr[::-1]   # 倒序，原列表不变
```

### LLM 场景应用
```python
# 截断过长的文本，符合模型 token 限制
def truncate(text, max_len=500):
    return text[:max_len] + "..." if len(text) > max_len else text

# 分批处理（batch）
batch_size = 10
for i in range(0, len(items), batch_size):
    batch = items[i:i + batch_size]  # 优雅地取一批
    process(batch)
```

---

## 8. 生成器（Generator）

### 概念
生成器是**惰性计算**的函数，用 `yield` 代替 `return`，每次只产生一个值，**不占大量内存**。

### 对比：列表 vs 生成器
```python
# 列表推导式 - 一次性生成所有数据，占用内存
lines = [line.strip() for line in open("large_file.txt")]
# 如果文件有 100万行，内存中就有 100万个字符串

# 生成器表达式 - 惰性生成，一次只处理一行
lines = (line.strip() for line in open("large_file.txt"))
# 几乎不占用内存，用的时候才取
```

### 函数形式：yield
```python
def read_large_file(filepath):
    """逐行读取大文件，内存友好"""
    with open(filepath, "r") as f:
        for line in f:
            yield line.strip()  # 每次只返回一行

# 使用
for line in read_large_file("big.txt"):
    process(line)  # 处理完这行才读下一行
```

### 与 JS 对比
```javascript
// JS 的生成器语法类似，但用得少
function* generator() {
    yield 1;
    yield 2;
}
```

```python
# Python 生成器更常用，语法简洁
def generator():
    yield 1
    yield 2
```

### LLM 场景：流式输出
```python
def stream_llm_response(prompt):
    """模拟 LLM 流式输出，一个字一个字返回"""
    response = call_llm_api(prompt)
    for char in response:
        yield char  # 用户看到"打字效果"

# 使用
for chunk in stream_llm_response("你好"):
    print(chunk, end="", flush=True)  # 实时显示，不用等全部生成
```

---

## 9. 上下文管理器（with 语句）

### 概念
自动管理资源的打开和关闭，**避免忘记关闭资源**（文件、数据库连接、锁等）。

### 基本用法
```python
# ❌ 容易忘记关闭，或异常时没关闭
f = open("file.txt", "r")
data = f.read()
f.close()

# ✅ 自动关闭，即使有异常也能保证关闭
with open("file.txt", "r") as f:
    data = f.read()
# 这里 f 已经自动关闭了
```

### 原理（简化版）
```python
with 获取资源() as 变量:
    使用资源()
# 自动调用 变量.close() 或清理方法
```

### 多个上下文
```python
# 同时打开多个文件
with open("input.txt") as f_in, open("output.txt", "w") as f_out:
    f_out.write(f_in.read().upper())
```

### 自定义上下文管理器
```python
from contextlib import contextmanager
import time

@contextmanager
def timer(name):
    """计时上下文，自动计算代码块执行时间"""
    start = time.time()
    yield  # 这里执行 with 块内的代码
    print(f"{name} 耗时: {time.time() - start:.3f}秒")

# 使用
with timer("数据处理"):
    process_data()  # 自动计时这段代码
```

### LLM 场景应用
```python
# API 客户端连接池
with get_http_client() as client:
    response = client.post("/chat", json={"prompt": "hello"})
    # 自动归还连接到连接池

# 数据库事务
with db.transaction() as tx:
    tx.execute("INSERT INTO ...")
    tx.execute("UPDATE ...")
    # 成功自动 commit，异常自动 rollback
```

---

## 练习：综合应用

### 练习 1：生成器逐行读取
**题目**：用 `yield` 实现逐行读取文件（去掉换行符）。

```python
def read_lines(filepath):
    with open(filepath, "r") as f:
        for line in f:
            yield line.strip()
```

**要点**：
- `yield` 让函数变成生成器，惰性读取省内存
- `with` 确保文件自动关闭

---

### 练习 2：切片文本处理
**题目**：
1. 截断文本前10个字符，加 "..."
2. 每5个字符切一刀，生成列表

```python
text = "这是一个很长的文本需要分批处理"

# 截断
truncated = text[:10] + "..." if len(text) > 10 else text

# 分批（列表推导式）
chunks = [text[i:i+5] for i in range(0, len(text), 5)]
# 结果：['这是一个很', '长的文本需', '要分批处理']
```

**要点**：
- `range(0, len(text), 5)` 生成 0, 5, 10, 15...
- Python 列表用 `append()`，不是 `push()`

---

## 补充内容要点总结

| 特性 | 核心优势 | LLM 开发场景 |
|------|---------|-------------|
| **切片** | 语法简洁，不修改原数据 | 文本截断、分批处理 |
| **生成器** | 惰性计算，省内存 | 大文件处理、流式输出 |
| **with** | 自动资源管理 | API 客户端、数据库连接 |


---

## 前端 vs Python 对照表

| Python | JavaScript/React |
|--------|------------------|
| `*args` | `...args`（但 args 是数组）|
| `**kwargs` | `...config`（展开对象）|
| 装饰器 `@timer` | 高阶组件 `withTimer()` |
| `functools.wraps` | 手动复制组件 displayName |
| 字典 `{a:1}` | 对象 `{a:1}` |
| 元组 `(1,2,3)` | 无直接对应（不可变数组）|
| 列表 `[1,2,3]` | 数组 `[1,2,3]` |


## 易错点总结

1. **enumerate 顺序**：`for index, value in enumerate(...)`，不是 `value, index`
2. **`*args` 是元组**：不是列表，不可修改
3. **装饰器忘记 `@wraps`**：导致函数名变成 wrapper，调试困难
4. **`*` 位置**：`*args` 中的 `*` 是收集，单独 `*` 是强制关键字参数
5. **字典合并**：浅拷贝 `dict.copy()` 不会递归，深度合并需要递归处理

---

*现在基础语法很扎实啦！下一个阶段：Python 工程化（包管理、虚拟环境）*
