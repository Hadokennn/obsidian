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

---

## 易错点总结

1. **enumerate 顺序**：`for index, value in enumerate(...)`，不是 `value, index`
2. **`*args` 是元组**：不是列表，不可修改
3. **装饰器忘记 `@wraps`**：导致函数名变成 wrapper，调试困难
4. **`*` 位置**：`*args` 中的 `*` 是收集，单独 `*` 是强制关键字参数
5. **字典合并**：浅拷贝 `dict.copy()` 不会递归，深度合并需要递归处理

---

*继续加油！下一个阶段：Python 工程化（包管理、虚拟环境）*
