
## 一、 开发环境：使用 `uv` 现代工具链

对于 Node.js 开发者来说，`uv` 就是 Python 界的 `Bun`。它极大地简化了环境配置。

- **项目初始化**：
    
    Bash
    
    ```
    uv init my-ai-project  # 类似于 npm init
    cd my-ai-project
    ```
    
- **依赖管理**：
    
    Bash
    
    ```
    uv add requests        # 类似于 npm install requests
    uv run main.py         # 类似于 node main.js，但它会自动处理虚拟环境
    ```
    
- **关键差异**：Python 项目强制建议使用**虚拟环境 (venv)**。`uv` 会自动帮你创建并在 `.venv` 目录下隔离依赖，防止全局库冲突。
    

---

## 二、 语法骨架：缩进与作用域

Python 取消了 `{}` 和 `;`，改用**缩进**控制代码块。

### 1. 缩进规则

- 必须使用统一的空格（推荐 4 个）或 Tab。
    
- **TS 逻辑：** `if (true) { console.log("Hi"); }`
    
- **Python 逻辑：**
    
    Python
    
    ```
    if True:
        print("Hi")
    # 缩进结束代表代码块结束
    ```
    

### 2. 变量与解构 (Destructuring)

Python 是动态强类型语言，变量不需要 `let/const`。

- **多重赋值/解构**：
    
    Python
    
    ```
    x, y, z = 1, 2, 3
    a, *rest = [1, 2, 3, 4]  # 类似 TS: [a, ...rest] = [1, 2, 3, 4]
    ```
    
- **类型注解 (Type Hints)**：
    
    虽然 Python 不强制，但在大型项目中（尤其是大模型开发），我们会写类似 TS 的类型提示：
    
    Python
    
    ```
    def process_text(text: str, limit: int = 10) -> str:
        return text[:limit]
    ```
    

---

## 三、 数据容器：List, Tuple, Dict

这是处理模型输入输出最常用的部分。

### 1. List (列表) vs Tuple (元组)

- **`List[]`**: 对应 JS 数组，可变。
    
- **`Tuple()`**: **不可变**。
    
    - _场景_：函数返回多个值（例如 `return score, label`）时，本质是返回一个元组。
        
    - _优势_：由于不可变，它的性能比 List 略高，且可以作为字典的 Key。
        

### 2. 列表推导式 (List Comprehension) —— Python 的精髓

这是 Python 替代 `map` 和 `filter` 的最常用方式。

- **需求**：把一个数字列表中的偶数翻倍。
    
- **TS 写法**：`list.filter(x => x % 2 === 0).map(x => x * 2)`
    
- **Python 写法**：
    
    Python
    
    ```
    nums = [1, 2, 3, 4, 5, 6]
    doubled_evens = [x * 2 for x in nums if x % 2 == 0]
    ```
    

---

## 四、 面向对象：`self` 的执念

在 Python 中，`class` 的定义非常频繁，尤其是自定义模型层时。

Python

```
class ChatBot:
    def __init__(self, model_name):
        # __init__ 相当于 constructor
        self.model_name = model_name 
    
    def greet(self, user):
        # 所有方法第一个参数必须是 self
        print(f"I am {self.model_name}, hello {user}")
```

- **避坑指南**：在类方法内访问属性**必须**加 `self.`。漏掉 `self.` 会导致 Python 去找全局变量，从而报 `NameError`。
    

---

## 五、 异步编程：Asyncio 核心逻辑

这是你作为 Node 开发者最需要转换思维的地方。

| **特性**       | **Node.js (TS)** | **Python (Asyncio)**        |
| ------------ | ---------------- | --------------------------- |
| **执行机制**     | 隐式、自动运行          | 显式、需启动 Loop                 |
| **顶层 Await** | 支持 (ESM)         | 脚本中不支持，需在 `asyncio.run()` 内 |
| **并发**       | `Promise.all()`  | `asyncio.gather()`          |

**典型范式：**

Python

```python
import asyncio

async def call_llm(prompt):
    await asyncio.sleep(1) # 模拟网络 IO
    return f"Response to {prompt}"

async def main():
    # 并行执行多个任务 (类似 Promise.all)
    results = await asyncio.gather(call_llm("Hi"), call_llm("How are you"))
    print(results)

if __name__ == "__main__":
    # 这是打火启动 Event Loop 的唯一入口
    asyncio.run(main())
```

---

## 六、 今日小结：给 TS 开发者的 3 条建议

1. **忘记 `null` 和 `undefined`**：统一使用 `None`。判断时用 `if val is None:`。
    
2. **强制缩进是朋友**：虽然初学痛苦，但它消除了 TS 中常见的“大括号地狱”。
    
3. **不要怕 `self`**：把它看作是一个必须显式传递的 `this` 指针，这让 Python 的底层调用逻辑更透明。
    

---

**💡 明日任务预告**：

我们将攻克 **Dictionary (JSON 操作)** 的高级用法，并正式安装 **NumPy**。我们将尝试用代码“手动”实现大模型中最核心的计算单元：**向量的点积运算**。