---
title: Python 设计模式完全指南
source: https://github.com/faif/python-patterns
author: faif (参考 iluwatar/java-design-patterns 补充)
date: 2025-12-29
category: programming
tags: [design-patterns, python, gof, creational, structural, behavioral]
---

# Python 设计模式完全指南

> 📖 原文：[faif/python-patterns](https://github.com/faif/python-patterns)
> 📖 参考：[iluwatar/java-design-patterns](https://github.com/iluwatar/java-design-patterns)
> 📅 学习日期：2025-12-29
> 🏷️ 分类：编程技术

---

## 一句话总结

设计模式是解决软件设计中常见问题的**最佳实践方案**，每种模式都有其**权衡取舍**，选择时要关注**为什么用**而不仅是**怎么实现**。

---

## 核心要点

1. **设计模式三大类**：创建型、结构型、行为型
2. **Python 特色**：模块天然单例、鸭子类型简化接口、装饰器语法糖
3. **反模式警惕**：避免单例滥用、上帝对象、过度继承
4. **组合优于继承**：Python 更推崇组合和委托
5. **简单优先**：不要为了用模式而用模式

---

## 设计模式分类总览

### 🏗️ 创建型模式 (Creational Patterns)

> 关注**对象创建机制**，将对象创建与使用分离

| 模式 | 描述 | Python 实现要点 | 使用场景 |
|------|------|-----------------|----------|
| **Abstract Factory** | 提供创建相关对象家族的接口 | 使用工厂函数 + 字典映射 | 跨平台 UI 组件 |
| **Builder** | 分步构建复杂对象 | 链式调用，返回 self | 构建复杂配置对象 |
| **Factory Method** | 委托子类决定实例化哪个类 | 简单函数即可实现 | 日志记录器创建 |
| **Prototype** | 通过克隆创建对象 | `copy.deepcopy()` | 对象创建开销大时 |
| **Singleton/Borg** | 确保类只有一个实例 | **Borg 模式**：共享状态 | 配置管理、连接池 |
| **Lazy Evaluation** | 延迟计算属性 | `@property` + 缓存 | 昂贵计算的属性 |
| **Object Pool** | 预创建并复用对象 | 队列管理对象池 | 数据库连接池 |

#### 💡 Python 特色：Borg vs Singleton

```python
# Borg 模式 - Python 推荐方式（共享状态而非单例）
class Borg:
    _shared_state = {}
    
    def __init__(self):
        self.__dict__ = self._shared_state

# 使用
b1 = Borg()
b2 = Borg()
b1.state = "shared"
print(b2.state)  # "shared" - 状态共享！
```

---

### 🔧 结构型模式 (Structural Patterns)

> 关注**类和对象的组合**，形成更大的结构

| 模式 | 描述 | Python 实现要点 | 使用场景 |
|------|------|-----------------|----------|
| **Adapter** | 转换接口使不兼容类协作 | 包装器类 | 集成第三方库 |
| **Bridge** | 分离抽象与实现 | 组合替代继承 | 跨平台抽象 |
| **Composite** | 树形结构统一处理 | 递归组合 | 文件系统、菜单 |
| **Decorator** | 动态添加职责 | `@decorator` 语法糖 | 日志、缓存、权限 |
| **Facade** | 简化复杂子系统接口 | 统一入口类 | API 封装 |
| **Flyweight** | 共享细粒度对象 | 工厂 + 缓存字典 | 大量相似对象 |
| **Proxy** | 控制对象访问 | `__getattr__` 代理 | 延迟加载、访问控制 |
| **MVC** | 模型-视图-控制器 | 分层架构 | Web 应用 |

#### 💡 Python 装饰器模式

```python
# Python 内置装饰器语法
from functools import wraps

def log_calls(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"Finished {func.__name__}")
        return result
    return wrapper

@log_calls
def process_data(data):
    return data.upper()
```

---

### 🎭 行为型模式 (Behavioral Patterns)

> 关注**对象间的通信**和**职责分配**

| 模式 | 描述 | Python 实现要点 | 使用场景 |
|------|------|-----------------|----------|
| **Chain of Responsibility** | 请求沿链传递 | 链表结构处理器 | 日志级别、审批流程 |
| **Command** | 封装请求为对象 | 可调用对象/闭包 | 撤销/重做、任务队列 |
| **Iterator** | 顺序访问集合元素 | `__iter__`, `__next__` | 自定义集合遍历 |
| **Mediator** | 中介者协调对象交互 | 事件总线 | 聊天室、UI 组件通信 |
| **Memento** | 保存和恢复对象状态 | 状态快照 | 撤销功能 |
| **Observer** | 一对多依赖通知 | 回调列表 | 事件系统、数据绑定 |
| **State** | 状态改变行为 | 状态类 + 上下文 | 有限状态机 |
| **Strategy** | 算法族可互换 | 函数作为参数 | 排序策略、支付方式 |
| **Template Method** | 定义算法骨架 | 抽象基类 + 钩子 | 框架扩展点 |
| **Visitor** | 分离算法与对象结构 | 双重分派 | AST 处理 |

#### 💡 Python 策略模式（函数式）

```python
# Python 中函数是一等公民，策略模式极简实现
def strategy_add(a, b):
    return a + b

def strategy_multiply(a, b):
    return a * b

def execute(strategy, a, b):
    return strategy(a, b)

# 使用
result = execute(strategy_add, 5, 3)  # 8
result = execute(strategy_multiply, 5, 3)  # 15
```

---

### 🔬 其他模式

| 模式 | 描述 | 使用场景 |
|------|------|----------|
| **Dependency Injection** | 依赖注入 | 解耦、测试 |
| **Delegation** | 委托模式 | 组合复用 |
| **Blackboard** | 黑板模式 | AI 知识系统 |
| **HSM** | 层次状态机 | 复杂状态管理 |

---

## 🚫 Python 反模式（需要避免）

### 1. Singleton 滥用
- **问题**：Python 模块本身就是单例
- **替代**：使用模块级变量或 Borg 模式

### 2. God Object（上帝对象）
- **问题**：一个类承担太多职责
- **替代**：拆分为多个小型、内聚的类

### 3. 过度继承
- **问题**：深层继承树难以维护
- **替代**：组合优于继承

---

## 设计原则速记

| 原则 | 含义 | 记忆口诀 |
|------|------|----------|
| **KISS** | Keep It Simple, Stupid | 简单就是美 |
| **DRY** | Don't Repeat Yourself | 不要重复 |
| **YAGNI** | You Aren't Gonna Need It | 不要过度设计 |
| **组合优于继承** | Favor Composition | 能组合不继承 |

---

## 常用模式 Python 代码示例

### 工厂模式

```python
class Dog:
    def speak(self):
        return "Woof!"

class Cat:
    def speak(self):
        return "Meow!"

def get_pet(pet_type):
    pets = {"dog": Dog, "cat": Cat}
    return pets.get(pet_type, Dog)()

# 使用
pet = get_pet("cat")
print(pet.speak())  # Meow!
```

### 观察者模式

```python
class Subject:
    def __init__(self):
        self._observers = []
    
    def attach(self, observer):
        self._observers.append(observer)
    
    def notify(self, message):
        for observer in self._observers:
            observer.update(message)

class Observer:
    def update(self, message):
        print(f"Received: {message}")

# 使用
subject = Subject()
subject.attach(Observer())
subject.notify("Hello!")  # Received: Hello!
```

### 单例模式（模块方式）

```python
# config.py - Python 最简单的单例
class _Config:
    def __init__(self):
        self.setting = "default"

config = _Config()  # 模块级实例

# 使用
from config import config
config.setting = "custom"
```

---

## 行动清单

- [ ] 掌握 Python 装饰器模式的 `@decorator` 语法
- [ ] 理解 Borg 模式与传统 Singleton 的区别
- [ ] 练习使用工厂函数替代复杂的类继承
- [ ] 学习 `__iter__` 和 `__next__` 实现迭代器
- [ ] 了解 `functools.lru_cache` 作为缓存装饰器

---

## 📝 金句摘录

> "设计模式是解决软件设计中常见问题的最佳实践方案，每种模式都有其权衡取舍。"

> "Python 特色：模块天然单例、鸭子类型简化接口、装饰器语法糖。"

> "不要为了用模式而用模式——简单优先。"

> "能组合不继承——组合优于继承是 Python 的哲学。"

---

## 个人思考

{留空，供后续补充}

---

## 📚 延伸阅读

- [Refactoring Guru - 设计模式图解](https://refactoringguru.cn/design-patterns) - 可视化学习设计模式
- [faif/python-patterns GitHub](https://github.com/faif/python-patterns) - Python 设计模式集合
- [iluwatar/java-design-patterns](https://github.com/iluwatar/java-design-patterns) - 更详细的模式说明
- [《Head First 设计模式》](https://www.oreilly.com/library/view/head-first-design/0596007124/) - 入门经典
- [《Python Cookbook》](https://www.oreilly.com/library/view/python-cookbook-3rd/9781449357337/) - 第 8 章 类与对象
