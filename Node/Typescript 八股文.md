# JavaScript 事件循环（Event Loop）详细讲解

TypeScript 本身**没有独立的事件循环**，因为TS是JavaScript的超集，最终会编译为纯JS代码运行，其事件循环完全遵循**JavaScript的事件循环规则**，且运行时（浏览器/Node.js）的事件循环机制是TS事件循环的核心载体。学习TS事件循环，本质是掌握JS事件循环在不同运行时的工作原理，以及TS类型系统对事件循环相关API的类型约束。

本文从**核心基础**、**浏览器/Node.js 差异**、**TS类型加持**、**实战案例**、**常见误区**五个维度详细讲解，兼顾原理和实际开发应用，确保上下文逻辑连贯、内容完整。

## 一、事件循环核心基础：为什么需要事件循环？

JavaScript/TypeScript 的核心执行特性是**单线程**（主线程只有一个，同一时间只能执行一段代码），但实际开发中需要处理**异步任务**（如网络请求、定时器、DOM事件），如果异步任务阻塞主线程，会导致页面卡死、程序无响应。

**事件循环（Event Loop）**是JS/TS实现**非阻塞异步编程**的核心机制：它通过协调**同步任务**和**异步任务**的执行顺序，让单线程的JS/TS能高效处理多类任务，既保证主线程不阻塞，又能按规则执行异步回调。

### 核心前置概念（必须掌握）

在事件循环中，所有任务被分为**同步任务**和**异步任务**，异步任务又细分为**宏任务（Macrotask）**和**微任务（Microtask）**，这三类任务的执行优先级是核心规则。

#### 1. 同步任务（Synchronous Task）

- **定义**：立即执行、阻塞主线程的任务，执行完才会继续处理后续任务；
    
- **例子**：变量声明、函数同步调用、for循环、DOM同步操作（如`document.getElementById`）；
    
- **TS特性**：无特殊差异，仅需按TS类型规范编写（如指定变量类型、函数返回值）。
    

#### 2. 宏任务（Macrotask）

- **定义**：**异步任务的基础类型**，执行粒度较粗，每次事件循环仅执行一个宏任务，执行后会先清空微任务队列，再进入下一次循环；
    
- **核心特点**：**队列式执行**，先进先出（FIFO），且**每次事件循环只处理一个宏任务**；
    
- **常见宏任务**（浏览器+Node.js）：
    
    - 定时器：`setTimeout`、`setInterval`、`setImmediate`（Node.js专属）；
        
    - I/O操作：网络请求（fetch/axios）、文件读写（Node.js）、DOM事件（click/input）；
        
    - 入口任务：全局执行上下文（script整体代码）；
        
    - 其他：`requestAnimationFrame`（浏览器，视觉帧相关）、`process.nextTick`（Node.js，特殊宏任务，优先级高于普通宏任务）。
        

#### 3. 微任务（Microtask）

- **定义**：**异步任务的精细类型**，执行粒度更细，**宏任务执行完成后，会立即清空所有微任务队列**（直到微任务队列为空），再处理UI渲染/下一个宏任务；
    
- **核心特点**：**高优先级**，宏任务后的“即时执行”，同样遵循FIFO原则；
    
- **常见微任务**（浏览器+Node.js）：
    
    - Promise 相关：`Promise.then()`、`Promise.catch()`、`Promise.finally()`（注意：`new Promise((resolve) => {})` 内部是同步代码）；
        
    - `async/await`：`await` 后面的代码（本质是Promise.then的语法糖，属于微任务）；
        
    - 专属API：`queueMicrotask()`（通用）、`MutationObserver`（浏览器，DOM变化监听）。
        

### 核心执行规则（通用，浏览器/Node.js 均遵循）

这是事件循环的**黄金规则**，所有执行顺序问题都能通过此规则推导，务必牢记：

1. 先执行**全局同步代码**（属于宏任务的入口任务），执行过程中遇到的任务按类型入对应队列：
    
    1. 同步任务：立即执行，不进入队列；
        
    2. 宏任务：入**宏任务队列**（可存在多个，如定时器队列、I/O队列）；
        
    3. 微任务：入**微任务队列**（全局唯一，优先级最高）；
        
2. 全局同步代码执行完毕后，**立即清空微任务队列**：按FIFO顺序依次执行所有微任务，**如果微任务执行过程中产生新的微任务，继续加入当前微任务队列，直到队列完全为空**（微任务的“嵌套执行”）；
    
3. 微任务队列清空后，**执行一次UI渲染**（仅浏览器，Node.js无此步骤），渲染完成后进入下一次事件循环；
    
4. 下一次事件循环：从宏任务队列中取出**第一个宏任务**执行，执行完毕后，**再次清空微任务队列**（规则同步骤2）；
    
5. 重复步骤3-4，直到宏任务队列和微任务队列均为空。
    

**一句话总结**：**同步先执行，微任务永远在宏任务之后、UI渲染/下一个宏任务之前执行，且一次宏任务后必清空所有微任务**。

## 二、浏览器 vs Node.js 事件循环：核心差异

TS/JS 事件循环的**核心规则（宏/微任务优先级）**一致，但浏览器和Node.js的**任务队列结构**、**宏任务执行顺序**、**特殊API优先级**存在差异，这是开发中容易踩坑的点，TS开发时需根据运行时场景区分。

### 1. 浏览器事件循环（简单直观，前端主流）

- **队列结构**：**1个微任务队列 + 多个宏任务队列**（定时器、DOM事件、网络请求等），但浏览器不严格区分宏任务队列的优先级，按任务入队顺序取第一个执行；
    
- **核心步骤**：同步代码 → 清空微任务 → UI渲染 → 取一个宏任务 → 清空微任务 → UI渲染 → ...；
    
- **特殊点**：`requestAnimationFrame` 执行时机在**UI渲染之前**，属于宏任务但优先级高于普通定时器。
    

### 2. Node.js 事件循环（分阶段，更复杂）

Node.js 为了高效处理I/O操作，将事件循环划分为**6个明确的阶段**，每个阶段对应一个宏任务队列，按**固定顺序执行**，且微任务队列在**每个阶段执行完毕后**清空（而非仅在单个宏任务后）。

#### Node.js 事件循环的6个阶段（按执行顺序）

```plain
┌───────────────────────────┐
┌─>│        timers         │ 阶段1：执行setTimeout/setInterval的回调
│  └─────────────┬─────────┘
│  ┌─────────────┴─────────┐
│  │     pending callbacks │ 阶段2：执行上一轮循环未处理的I/O回调
│  └─────────────┬─────────┘
│  ┌─────────────┴─────────┐
│  │     idle, prepare     │ 阶段3：Node内部使用，开发者无需关注
│  └─────────────┬─────────┘      ┌───────────────┐
│  ┌─────────────┴─────────┐      │   incoming:   │
│  │         poll          │<─────┤  connections, │ 阶段4：核心阶段，处理网络/I/O文件回调，阻塞等待新任务
│  └─────────────┬─────────┘      │   data, etc.  │
│  ┌─────────────┴─────────┐      └───────────────┘
│  │        check          │ 阶段5：执行setImmediate的回调
│  └─────────────┬─────────┘
│  ┌─────────────┴─────────┐
└──┤    close callbacks    │ 阶段6：执行关闭回调（如socket.on('close', ...)）
   └───────────────────────┘
```

#### Node.js 事件循环核心规则

1. 按**上述6个阶段的固定顺序**执行，每个阶段执行完毕后，**立即清空微任务队列**（规则同浏览器：新产生的微任务继续执行，直到队列为空）；
    
2. 微任务队列分为**两个子队列**，优先级：`process.nextTick()` 队列 > Promise微任务队列（`then/catch/finally`）；
    
3. **特殊API优先级**（关键）：
    
    1. `process.nextTick()`：**所有微任务中优先级最高**，甚至在Node.js各阶段之间也会优先执行；
        
    2. `setImmediate`：在阶段5执行，**比setTimeout(fn, 0) 更稳定**（setTimeout(fn,0) 实际延迟约1-2ms，可能被poll阶段阻塞）；
        
4. 无UI渲染步骤，执行完微任务后直接进入下一个阶段。
    

### 3. 浏览器/Node.js 核心差异总结

|特性|浏览器|Node.js|
|---|---|---|
|宏任务执行方式|按入队顺序取第一个执行|按6个阶段固定顺序执行|
|微任务执行时机|单个宏任务执行后|每个阶段执行完毕后|
|微任务队列|全局唯一|分process.nextTick和Promise队列|
|特殊API|requestAnimationFrame/DOM事件|process.nextTick/setImmediate|
|额外步骤|微任务后执行UI渲染|无UI渲染步骤|

## 三、TypeScript 对事件循环的「类型加持」

TS 不改变事件循环的执行机制，但会通过**静态类型检查**，为事件循环相关的API（定时器、Promise、async/await等）提供**类型约束**，避免因参数类型错误导致的异步执行异常，让异步代码更健壮、可维护。

以下是TS开发中最常用的异步API类型约束示例，掌握这些可避免90%的类型错误。

### 1. 定时器API的类型约束

TS为`setTimeout/setInterval`提供了明确的参数类型，约束回调函数、延迟时间、参数的类型：

```typescript
// TS类型定义（简化版）
declare function setTimeout(
  callback: (...args: any[]) => void, // 回调函数，支持可选参数
  delay?: number, // 延迟时间，默认0，必须是数字
  ...args: any[] // 传递给回调的参数
): NodeJS.Timeout; // 返回定时器实例，用于clearTimeout

// 正确使用（TS无报错）
const timer: NodeJS.Timeout = setTimeout((name: string) => {
  console.log(`Hello ${name}`); // 类型匹配：string
}, 1000, "TypeScript");

clearTimeout(timer); // 传入正确的定时器实例

// 错误使用（TS直接报错，提前拦截）
setTimeout("console.log('hello')", 1000); // 报错：回调不能是字符串（JS中允许，TS中禁止）
setTimeout(() => {}, "1000"); // 报错：delay必须是数字
```

### 2. Promise 的类型约束

TS为Promise提供了**泛型支持**，可明确Promise成功值/失败原因的类型，让`then/catch`中的回调参数类型自动推导，避免类型模糊。

```typescript
// 泛型指定Promise成功值为string，失败原因为Error
const promise: Promise<string> = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("TS Event Loop"); // 必须传入string，否则TS报错
    // reject(new Error("请求失败")); // 失败原因必须是Error类型，符合约束
  }, 1000);
});

// then回调的res自动推导为string，catch的err自动推导为Error
promise
  .then((res) => {
    console.log(res.length); // TS提示：string具有length属性，无报错
  })
  .catch((err) => {
    console.log(err.message); // TS提示：Error具有message属性，无报错
  });
```

### 3. async/await 的类型推导

`async/await`是Promise的语法糖，TS能自动推导`await`后的值类型，以及`async`函数的返回值类型（必然是Promise），让异步代码的类型追踪更清晰。

```typescript
// 异步函数，返回值自动推导为Promise<number>
async function getNum(): Promise<number> {
  // await后的值自动推导为string（继承上方promise的类型）
  const res = await promise;
  return res.length; // 返回number，符合Promise<number>约束
}

// 调用时TS提示：返回值是Promise<number>
getNum().then((num) => {
  console.log(num + 10); // 无类型错误，类型匹配
});
```

### 4. queueMicrotask 的类型约束

TS为通用微任务API `queueMicrotask` 约束回调函数类型，确保回调无返回值、参数规范，避免非法回调导致的微任务执行异常。

```typescript
// TS类型定义
declare function queueMicrotask(callback: () => void): void;

// 正确使用
queueMicrotask(() => {
  console.log("微任务执行");
});

// 错误使用（TS报错）
queueMicrotask((a: number) => { // 报错：回调不能有参数，与类型定义冲突
  console.log(a);
});
```

**TS的核心价值**：在事件循环的执行逻辑之外，通过类型约束提前拦截参数类型、返回值类型错误，避免因类型问题导致的异步代码执行异常，提升代码可维护性和稳定性。

## 四、TS/JS 事件循环实战案例：执行顺序推导

结合核心规则和TS类型特性，通过3个由浅入深的案例，掌握事件循环的执行顺序推导，这是面试和开发的核心考点，同时验证TS类型约束下的代码执行一致性。

### 案例1：基础同步+宏任务+微任务（浏览器/Node.js 执行结果一致）

```typescript
// 严格模式+类型检查
"use strict";

// 同步任务1
console.log(1); // 立即执行

// 宏任务：setTimeout（延迟0ms，入宏任务队列）
const timer: NodeJS.Timeout = setTimeout(() => {
  console.log(2); // 宏任务回调
}, 0);

// 微任务1：Promise.then（new Promise内部是同步代码）
new Promise<number>((resolve) => {
  // 同步任务2
  console.log(3);
  resolve(4); // 改变Promise状态，then回调入微任务队列
}).then((res: number) => {
  // 微任务1回调
  console.log(res); // 4
});

// 微任务2：queueMicrotask
queueMicrotask(() => {
  console.log(5);
});

// 同步任务3
console.log(6);

// 清空定时器（不影响队列，仅取消执行）
clearTimeout(timer);
```

#### 执行顺序推导：

1. 执行同步任务1 → 输出 `1`；
    
2. 遇到`setTimeout`，入宏任务队列，暂不执行；
    
3. 执行`new Promise`内部同步代码 → 输出 `3`，`resolve(4)` 后，`then`回调入微任务队列；
    
4. 遇到`queueMicrotask`，回调入微任务队列；
    
5. 执行同步任务3 → 输出 `6`；
    
6. 全局同步代码执行完毕，**清空微任务队列**（FIFO）：
    
    1. 执行`Promise.then`回调 → 输出 `4`；
        
    2. 执行`queueMicrotask`回调 → 输出 `5`；
        
7. 微任务队列清空后，取宏任务队列第一个任务（`setTimeout`），但已被`clearTimeout`取消，不执行；
    
8. 所有队列为空，事件循环结束。
    

**最终输出**：`1 → 3 → 6 → 4 → 5`（TS无类型错误，执行结果与纯JS一致）。

### 案例2：微任务嵌套微任务（核心考点：微任务队列清空规则）

```typescript
"use strict";

console.log(1); // 同步

setTimeout(() => {
  console.log(2); // 宏任务1
}, 0);

Promise.resolve().then(() => {
  console.log(3); // 微任务1
  // 微任务1执行中产生新的微任务2
  Promise.resolve().then(() => {
    console.log(4); // 微任务2
    // 微任务2执行中产生新的微任务3
    queueMicrotask(() => {
      console.log(5); // 微任务3
    });
  });
});

queueMicrotask(() => {
  console.log(6); // 微任务4
});

console.log(7); // 同步
```

#### 执行顺序推导：

1. 同步任务 → 输出 `1 → 7`；
    
2. 全局同步代码执行完毕，清空微任务队列（初始队列：微任务1、微任务4）；
    
3. 执行微任务1 → 输出 `3`，产生微任务2，加入当前微任务队列；
    
4. 执行微任务4 → 输出 `6`；
    
5. 微任务队列未空，执行微任务2 → 输出 `4`，产生微任务3，加入当前微任务队列；
    
6. 微任务队列未空，执行微任务3 → 输出 `5`；
    
7. 微任务队列完全清空，执行宏任务1 → 输出 `2`；
    
8. 宏任务1执行完毕，微任务队列为空，事件循环结束。
    

**最终输出**：`1 → 7 → 3 → 6 → 4 → 5 → 2`。

### 案例3：async/await 与事件循环（await的本质是Promise.then）

```typescript
"use strict";

// 定义带类型的异步函数
async function asyncFn(): Promise<string> {
  console.log(1); // 同步（await之前的代码）
  // await 后面的代码会被包裹为Promise.then的微任务
  await Promise.resolve();
  console.log(2); // 微任务
  return "async done";
}

console.log(3); // 同步
asyncFn(); // 调用异步函数
console.log(4); // 同步

setTimeout(() => {
  console.log(5); // 宏任务
}, 0);

Promise.resolve().then(() => {
  console.log(6); // 微任务
});
```

#### 关键知识点：`await xxx` 的执行逻辑

- `await` 之前的代码：**同步执行**；
    
- `await xxx`：如果`xxx`是Promise，会等待其状态改变，并将`await`后面的代码加入微任务队列（等价于`Promise.then`）；
    
- 如果`xxx`不是Promise，会先包装为`Promise.resolve(xxx)`，再执行上述逻辑。
    

#### 执行顺序推导：

1. 同步任务 → 输出 `3`；
    
2. 调用`asyncFn`，执行内部同步代码 → 输出 `1`；
    
3. 遇到`await Promise.resolve()`，将`console.log(2)` 加入微任务队列；
    
4. 退出`asyncFn`，执行同步任务 → 输出 `4`；
    
5. 全局同步代码执行完毕，清空微任务队列（初始队列：`console.log(2)`、`console.log(6)`）；
    
6. 按FIFO执行微任务 → 输出 `2 → 6`；
    
7. 微任务队列清空，执行宏任务 → 输出 `5`；
    
8. 事件循环结束。
    

**最终输出**：`3 → 1 → 4 → 2 → 6 → 5`。

## 五、TS/JS 事件循环常见误区与避坑指南

开发中很多异步执行的“诡异问题”，本质是对事件循环的理解偏差导致的。结合TS开发特点，总结6个高频误区及避坑方法，帮助规避潜在问题。

### 误区1：`setTimeout(fn, 0)` 会立即执行

- **错误认知**：延迟0ms，回调会立即执行；
    
- **正确认知**：`setTimeout` 是宏任务，即使延迟0ms，也会等全局同步代码执行完毕、微任务队列清空后才执行（且浏览器中实际延迟至少1ms）；
    
- **TS避坑**：通过类型约束`delay`为数字，避免传入非数字导致的延迟异常。
    

### 误区2：微任务执行时不会产生新的微任务

- **错误认知**：微任务队列执行一次就结束；
    
- **正确认知**：微任务执行过程中产生的新微任务，会加入当前微任务队列，继续执行直到队列为空（微任务的“饥饿问题”：若无限产生新微任务，会阻塞宏任务和UI渲染）；
    
- **避坑**：避免在微任务中执行无限循环的异步逻辑。
    

### 误区3：浏览器和Node.js的事件循环完全一致

- **错误认知**：只要掌握通用规则，无需区分运行时；
    
- **正确认知**：Node.js的阶段化执行、`process.nextTick`优先级、`setImmediate`与`setTimeout`的顺序，与浏览器差异极大，TS跨端开发（前端+Node.js服务）时必须严格区分；
    
- **示例**：Node.js中`process.nextTick` 永远比Promise微任务先执行： `// Node.js执行输出：1 → 3 → 2，浏览器执行输出：1 → 2 → 3` `console.log(1);` `Promise.resolve().then(() => console.log(2));` `process.nextTick(() => console.log(3));`
    

### 误区4：`new Promise` 是微任务

- **错误认知**：Promise相关都是微任务；
    
- **正确认知**：`new Promise((executor) => {})` 中的**executor函数（执行器）是同步代码**，只有`then/catch/finally` 才是微任务；
    
- **避坑**：不要将同步逻辑误放入`new Promise`执行器，导致执行顺序混乱。
    

### 误区5：async/await 是同步的

- **错误认知**：`async`函数调用后会立即执行所有代码；
    
- **正确认知**：`async`函数本身是同步调用，但`await` 后面的代码是微任务，会异步执行；
    
- **TS避坑**：通过返回值类型（Promise）明确`async`函数的异步特性，避免直接赋值`await`的结果（需通过`then`或再次`await`获取）。
    

### 误区6：事件循环只处理异步任务

- **错误认知**：事件循环仅协调宏/微任务；
    
- **正确认知**：事件循环的核心是协调所有任务，包括全局同步代码（入口宏任务）、宏任务、微任务，同步代码是事件循环的执行起点。
    

## 六、总结

TypeScript 事件循环的核心是**JavaScript事件循环机制**，TS本身仅通过静态类型检查为异步API提供约束，不改变执行逻辑。核心知识点可归纳为3点：

1. **核心规则**：同步先执行 → 微任务永远在宏任务之后、下一个宏任务/UI渲染之前执行 → 一次宏任务后必清空所有微任务（微任务嵌套可继续执行）；
    
2. **任务分类**：同步任务阻塞主线程，宏任务粒度粗、按队列执行，微任务优先级高、宏任务后清空；
    
3. **运行时差异**：浏览器事件循环简单直观，Node.js分6个阶段执行，微任务优先级和执行时机有差异，TS开发需根据场景适配。
    

掌握以上内容，能轻松解决TS/JS开发中99%的异步执行顺序问题，结合TS的类型约束，可让异步代码更健壮、可维护，避开各类高频坑点。