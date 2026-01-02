---
title: Clean Code Python 版 - 代码整洁之道
source: https://github.com/ryanmcdermott/clean-code-javascript
author: Robert C. Martin (原著) / Ryan McDermott (JS版) / Python 改编
date: 2025-12-29
category: programming
tags: [clean-code, python, best-practices, code-quality, solid]
---

# Clean Code Python 版 - 代码整洁之道

> 📖 原文：[clean-code-javascript](https://github.com/ryanmcdermott/clean-code-javascript)
> 📖 原著：Robert C. Martin《Clean Code》
> 📅 学习日期：2025-12-29
> 🏷️ 分类：编程技术

---

## 一句话总结

代码质量与其**整洁度成正比**，好代码应该是**可读、可复用、可重构**的。

---

## 核心要点

1. **命名即文档**：好的命名胜过注释
2. **函数单一职责**：一个函数只做一件事
3. **避免副作用**：函数式编程思想
4. **组合优于继承**：Python 更推崇组合
5. **SOLID 原则**：面向对象设计的基石

---

## 📝 变量命名

### 1. 使用有意义且可发音的变量名

```python
# ❌ Bad
yyyymmdstr = datetime.now().strftime("%Y/%m/%d")

# ✅ Good
current_date = datetime.now().strftime("%Y/%m/%d")
```

### 2. 同类变量使用相同词汇

```python
# ❌ Bad - 三种不同叫法
def get_user_info(): pass
def get_client_data(): pass
def get_customer_record(): pass

# ✅ Good - 统一命名
def get_user(): pass
```

### 3. 使用可搜索的名称

```python
# ❌ Bad - 86400 是什么？
time.sleep(86400)

# ✅ Good - 命名常量
SECONDS_PER_DAY = 60 * 60 * 24  # 86400
time.sleep(SECONDS_PER_DAY)
```

### 4. 使用解释性变量

```python
# ❌ Bad
address = "One Infinite Loop, Cupertino 95014"
match = re.match(r'^[^,]+[,\s]+(.+?)\s*(\d{5})?$', address)
save_city_zip_code(match.group(1), match.group(2))

# ✅ Good
address = "One Infinite Loop, Cupertino 95014"
city_zip_pattern = r'^[^,]+[,\s]+(.+?)\s*(\d{5})?$'
match = re.match(city_zip_pattern, address)
if match:
    city, zip_code = match.groups()
    save_city_zip_code(city, zip_code)
```

### 5. 避免心智映射

```python
# ❌ Bad - l 是什么？
locations = ["Austin", "New York", "San Francisco"]
for l in locations:
    do_stuff()
    dispatch(l)  # l 是什么？

# ✅ Good - 明确命名
locations = ["Austin", "New York", "San Francisco"]
for location in locations:
    do_stuff()
    dispatch(location)
```

### 6. 不要添加不必要的上下文

```python
# ❌ Bad - Car 类里不需要 car 前缀
class Car:
    car_make: str
    car_model: str
    car_color: str

# ✅ Good
class Car:
    make: str
    model: str
    color: str
```

### 7. 使用默认参数

```python
# ❌ Bad
def create_microbrewery(name=None):
    brewery_name = name or "Hipster Brew Co."

# ✅ Good
def create_microbrewery(name: str = "Hipster Brew Co."):
    brewery_name = name
```

---

## 🔧 函数设计

### 1. 函数参数最好不超过 2 个

```python
# ❌ Bad - 参数太多
def create_menu(title, body, button_text, cancellable):
    pass

# ✅ Good - 使用数据类或字典
from dataclasses import dataclass

@dataclass
class MenuConfig:
    title: str
    body: str
    button_text: str
    cancellable: bool = True

def create_menu(config: MenuConfig):
    pass

# 使用
create_menu(MenuConfig(
    title="Foo",
    body="Bar",
    button_text="Baz",
    cancellable=True
))
```

### 2. 函数只做一件事

```python
# ❌ Bad - 做了多件事
def email_clients(clients):
    for client in clients:
        client_record = database.lookup(client)
        if client_record.is_active():
            email(client)

# ✅ Good - 拆分职责
def get_active_clients(clients):
    return [c for c in clients if is_active_client(c)]

def is_active_client(client):
    client_record = database.lookup(client)
    return client_record.is_active()

def email_active_clients(clients):
    for client in get_active_clients(clients):
        email(client)
```

### 3. 函数名应说明其功能

```python
# ❌ Bad - 不清楚添加什么
def add_to_date(date, month):
    pass

# ✅ Good - 明确说明
def add_months_to_date(date, months):
    pass
```

### 4. 函数只应有一层抽象

```python
# ❌ Bad - 混合多层抽象
def parse_code(code):
    REGEXES = [...]
    statements = code.split(" ")
    tokens = []
    for regex in REGEXES:
        for statement in statements:
            # 词法分析...
            pass
    
    ast = []
    for token in tokens:
        # 语法分析...
        pass
    
    for node in ast:
        # 解析...
        pass

# ✅ Good - 每层抽象独立
def parse_code(code):
    tokens = tokenize(code)
    syntax_tree = parse(tokens)
    return interpret(syntax_tree)

def tokenize(code):
    # 词法分析
    pass

def parse(tokens):
    # 语法分析
    pass
```

### 5. 删除重复代码

```python
# ❌ Bad - 重复逻辑
def show_developer_list(developers):
    for dev in developers:
        expected_salary = dev.calculate_expected_salary()
        experience = dev.get_experience()
        github_link = dev.get_github_link()
        render({"salary": expected_salary, "experience": experience, "github": github_link})

def show_manager_list(managers):
    for mgr in managers:
        expected_salary = mgr.calculate_expected_salary()
        experience = mgr.get_experience()
        portfolio = mgr.get_mba_projects()
        render({"salary": expected_salary, "experience": experience, "portfolio": portfolio})

# ✅ Good - 抽象共同逻辑
def show_employee_list(employees):
    for employee in employees:
        data = {
            "salary": employee.calculate_expected_salary(),
            "experience": employee.get_experience(),
        }
        if isinstance(employee, Developer):
            data["github"] = employee.get_github_link()
        elif isinstance(employee, Manager):
            data["portfolio"] = employee.get_mba_projects()
        render(data)
```

### 6. 不要使用标志参数

```python
# ❌ Bad - 布尔标志说明函数做了多件事
def create_file(name, temp):
    if temp:
        with open(f"./temp/{name}", "w") as f:
            pass
    else:
        with open(name, "w") as f:
            pass

# ✅ Good - 拆分为两个函数
def create_file(name):
    with open(name, "w") as f:
        pass

def create_temp_file(name):
    create_file(f"./temp/{name}")
```

### 7. 避免副作用

```python
# ❌ Bad - 修改了全局变量
name = "Ryan McDermott"

def split_into_first_and_last_name():
    global name
    name = name.split(" ")

split_into_first_and_last_name()
print(name)  # ['Ryan', 'McDermott'] - 原始值被改变了！

# ✅ Good - 纯函数，无副作用
def split_into_first_and_last_name(name):
    return name.split(" ")

name = "Ryan McDermott"
new_name = split_into_first_and_last_name(name)
print(name)      # 'Ryan McDermott' - 原始值不变
print(new_name)  # ['Ryan', 'McDermott']
```

### 8. 避免修改可变参数

```python
# ❌ Bad - 修改了传入的列表
def add_item_to_cart(cart, item):
    cart.append({"item": item, "date": datetime.now()})

# ✅ Good - 返回新列表
def add_item_to_cart(cart, item):
    return [*cart, {"item": item, "date": datetime.now()}]
```

### 9. 函数式编程优于命令式

```python
# ❌ Bad - 命令式
programmer_output = [
    {"name": "Uncle Bobby", "lines_of_code": 500},
    {"name": "Suzie Q", "lines_of_code": 1500},
]

total_output = 0
for programmer in programmer_output:
    total_output += programmer["lines_of_code"]

# ✅ Good - 函数式
from functools import reduce

total_output = reduce(
    lambda total, p: total + p["lines_of_code"],
    programmer_output,
    0
)

# 或者更 Pythonic
total_output = sum(p["lines_of_code"] for p in programmer_output)
```

### 10. 封装条件判断

```python
# ❌ Bad
if fsm.state == "fetching" and is_empty(list_node):
    # ...

# ✅ Good
def should_show_spinner(fsm, list_node):
    return fsm.state == "fetching" and is_empty(list_node)

if should_show_spinner(fsm, list_node):
    # ...
```

### 11. 避免否定条件

```python
# ❌ Bad - 双重否定难理解
def is_dom_node_not_present(node):
    pass

if not is_dom_node_not_present(node):
    # ...

# ✅ Good
def is_dom_node_present(node):
    pass

if is_dom_node_present(node):
    # ...
```

---

## 🏛️ 类设计

### 1. 优先使用 dataclass

```python
# ❌ Bad - 冗长的 __init__
class Animal:
    def __init__(self, age):
        self.age = age
    
    def move(self):
        pass

# ✅ Good - 使用 dataclass
from dataclasses import dataclass

@dataclass
class Animal:
    age: int
    
    def move(self):
        pass
```

### 2. 使用方法链

```python
# ❌ Bad
class Car:
    def __init__(self, make, model, color):
        self.make = make
        self.model = model
        self.color = color
    
    def set_color(self, color):
        self.color = color

car = Car("Ford", "F-150", "red")
car.set_color("pink")
car.save()

# ✅ Good - 返回 self 支持链式调用
class Car:
    def __init__(self, make, model, color):
        self.make = make
        self.model = model
        self.color = color
    
    def set_color(self, color):
        self.color = color
        return self
    
    def save(self):
        print(f"Saved: {self.make} {self.model} {self.color}")
        return self

Car("Ford", "F-150", "red").set_color("pink").save()
```

### 3. 组合优于继承

```python
# ❌ Bad - 错误使用继承（EmployeeTaxData 不是 Employee）
class Employee:
    def __init__(self, name, email):
        self.name = name
        self.email = email

class EmployeeTaxData(Employee):  # 错误！税务数据不是员工
    def __init__(self, ssn, salary):
        super().__init__()
        self.ssn = ssn
        self.salary = salary

# ✅ Good - 使用组合
@dataclass
class EmployeeTaxData:
    ssn: str
    salary: float

@dataclass
class Employee:
    name: str
    email: str
    tax_data: EmployeeTaxData = None
    
    def set_tax_data(self, ssn, salary):
        self.tax_data = EmployeeTaxData(ssn, salary)
```

---

## 🔷 SOLID 原则

### S - 单一职责原则 (SRP)

```python
# ❌ Bad - 一个类做太多事
class UserSettings:
    def __init__(self, user):
        self.user = user
    
    def change_settings(self, settings):
        if self.verify_credentials():
            # 修改设置...
            pass
    
    def verify_credentials(self):
        # 验证逻辑...
        pass

# ✅ Good - 职责分离
class UserAuth:
    def __init__(self, user):
        self.user = user
    
    def verify_credentials(self):
        # 验证逻辑...
        pass

class UserSettings:
    def __init__(self, user):
        self.user = user
        self.auth = UserAuth(user)
    
    def change_settings(self, settings):
        if self.auth.verify_credentials():
            # 修改设置...
            pass
```

### O - 开闭原则 (OCP)

```python
# ❌ Bad - 需要修改现有代码来添加新适配器
class HttpRequester:
    def __init__(self, adapter):
        self.adapter = adapter
    
    def fetch(self, url):
        if self.adapter.name == "ajax_adapter":
            return make_ajax_call(url)
        elif self.adapter.name == "node_adapter":
            return make_http_call(url)

# ✅ Good - 对扩展开放，对修改关闭
from abc import ABC, abstractmethod

class Adapter(ABC):
    @abstractmethod
    def request(self, url):
        pass

class AjaxAdapter(Adapter):
    def request(self, url):
        return make_ajax_call(url)

class NodeAdapter(Adapter):
    def request(self, url):
        return make_http_call(url)

class HttpRequester:
    def __init__(self, adapter: Adapter):
        self.adapter = adapter
    
    def fetch(self, url):
        return self.adapter.request(url)
```

### L - 里氏替换原则 (LSP)

```python
# ❌ Bad - Square 不能正确替换 Rectangle
class Rectangle:
    def __init__(self):
        self.width = 0
        self.height = 0
    
    def set_width(self, width):
        self.width = width
    
    def set_height(self, height):
        self.height = height
    
    def get_area(self):
        return self.width * self.height

class Square(Rectangle):
    def set_width(self, width):
        self.width = width
        self.height = width  # 违反 LSP！
    
    def set_height(self, height):
        self.width = height
        self.height = height

# ✅ Good - 正确的抽象
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def get_area(self):
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def get_area(self):
        return self.width * self.height

class Square(Shape):
    def __init__(self, length):
        self.length = length
    
    def get_area(self):
        return self.length ** 2
```

### I - 接口隔离原则 (ISP)

```python
# ❌ Bad - 强制实现不需要的方法
class DOMTraverser:
    def __init__(self, settings):
        self.settings = settings
        self.setup()
    
    def setup(self):
        self.root_node = self.settings["root_node"]
        self.settings["animation_module"].setup()  # 不是所有情况都需要动画！

# ✅ Good - 可选配置
class DOMTraverser:
    def __init__(self, settings):
        self.settings = settings
        self.options = settings.get("options", {})
        self.setup()
    
    def setup(self):
        self.root_node = self.settings["root_node"]
        if animation := self.options.get("animation_module"):
            animation.setup()
```

### D - 依赖倒置原则 (DIP)

```python
# ❌ Bad - 高层模块依赖低层模块
class InventoryRequester:
    def __init__(self):
        self.req_methods = ["HTTP"]
    
    def request_item(self, item):
        # HTTP 请求...
        pass

class InventoryTracker:
    def __init__(self, items):
        self.items = items
        self.requester = InventoryRequester()  # 紧耦合！
    
    def request_items(self):
        for item in self.items:
            self.requester.request_item(item)

# ✅ Good - 依赖注入
from abc import ABC, abstractmethod

class Requester(ABC):
    @abstractmethod
    def request_item(self, item):
        pass

class HttpRequester(Requester):
    def request_item(self, item):
        # HTTP 请求...
        pass

class WebSocketRequester(Requester):
    def request_item(self, item):
        # WebSocket 请求...
        pass

class InventoryTracker:
    def __init__(self, items, requester: Requester):
        self.items = items
        self.requester = requester  # 依赖注入
    
    def request_items(self):
        for item in self.items:
            self.requester.request_item(item)

# 使用
tracker = InventoryTracker(["apples", "bananas"], WebSocketRequester())
```

---

## 🧪 测试

### 每个测试一个概念

```python
# ❌ Bad - 一个测试测多个概念
def test_moment_js():
    date = MomentJS("1/1/2015")
    date.add_days(30)
    assert str(date) == "1/31/2015"
    
    date = MomentJS("2/1/2016")
    date.add_days(28)
    assert str(date) == "02/29/2016"

# ✅ Good - 每个测试一个概念
def test_handles_30_day_months():
    date = MomentJS("1/1/2015")
    date.add_days(30)
    assert str(date) == "1/31/2015"

def test_handles_leap_year():
    date = MomentJS("2/1/2016")
    date.add_days(28)
    assert str(date) == "02/29/2016"

def test_handles_non_leap_year():
    date = MomentJS("2/1/2015")
    date.add_days(28)
    assert str(date) == "03/01/2015"
```

---

## ⚠️ 错误处理

### 不要忽略捕获的错误

```python
# ❌ Bad
try:
    function_that_might_throw()
except Exception as error:
    print(error)  # 仅打印，无实际处理

# ✅ Good
try:
    function_that_might_throw()
except Exception as error:
    # 选项 1：记录错误
    logger.error(error)
    # 选项 2：通知用户
    notify_user_of_error(error)
    # 选项 3：上报服务
    report_error_to_service(error)
    # 选项 4：重新抛出
    raise
```

---

## 📝 注释

### 只注释复杂的业务逻辑

```python
# ❌ Bad - 无意义的注释
def hash_it(data):
    # 哈希值
    hash_value = 0
    # 字符串长度
    length = len(data)
    # 遍历每个字符
    for i in range(length):
        # 获取字符编码
        char = ord(data[i])
        # 计算哈希
        hash_value = ((hash_value << 5) - hash_value) + char

# ✅ Good - 只注释复杂逻辑
def hash_it(data):
    hash_value = 0
    for char in data:
        char_code = ord(char)
        hash_value = ((hash_value << 5) - hash_value) + char_code
        # 转换为 32 位整数
        hash_value &= 0xFFFFFFFF
    return hash_value
```

### 不要保留注释掉的代码

```python
# ❌ Bad
do_stuff()
# do_other_stuff()
# do_some_more_stuff()

# ✅ Good - 版本控制会记录历史
do_stuff()
```

---

## 行动清单

- [ ] 检查项目中的变量命名是否清晰
- [ ] 重构超过 3 个参数的函数
- [ ] 确保每个函数只做一件事
- [ ] 应用 SOLID 原则重构类设计
- [ ] 删除项目中注释掉的代码
- [ ] 为复杂业务逻辑添加有意义的注释

---

## 个人思考

{留空，供后续补充}

---

## 延伸阅读

- 《Clean Code》Robert C. Martin
- 《Refactoring》Martin Fowler
- 《The Pragmatic Programmer》
- [PEP 8 - Python 代码风格指南](https://peps.python.org/pep-0008/)
