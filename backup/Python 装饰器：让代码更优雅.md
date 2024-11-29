# Python 装饰器：让代码更优雅

在 Python 中，装饰器是一个非常强大且常用的功能。它可以帮助我们在不修改函数代码的前提下，动态地扩展函数的功能。本文将介绍什么是装饰器、如何使用装饰器以及 `@` 语法的工作原理。

## 什么是装饰器？

装饰器本质上是一个函数，它接收另一个函数作为参数，并返回一个新的函数。新的函数通常在原有函数的基础上，增加了某些功能。简单来说，装饰器就是对现有函数进行“装饰”或“包装”的工具。

### 装饰器的基本形式

我们先看一个最简单的装饰器例子：

```python
# 定义一个装饰器
def decorator(func):
    def wrapper():
        print("函数执行前")
        func()
        print("函数执行后")
    return wrapper

# 使用装饰器
@decorator
def hello():
    print("Hello, world!")

hello()
```

### 解释

1. **定义装饰器**：`decorator` 是一个函数，它接受一个函数 `func` 作为参数。
2. **包装函数**：在装饰器内部，我们定义了一个新函数 `wrapper`，它在执行原函数 `func` 前后打印了一些额外的信息。
3. **返回新的函数**：装饰器最终返回这个 `wrapper` 函数，这样 `hello()` 就会被这个包装函数取代。
4. **使用装饰器**：通过 `@decorator` 语法，我们将 `hello` 函数传递给了 `decorator` 装饰器。

### 输出结果

```plaintext
函数执行前
Hello, world!
函数执行后
```

通过装饰器，我们成功地在不修改 `hello` 函数代码的情况下，增加了额外的功能。

## 为什么可以使用 `@` 语法？

在 Python 中，函数是第一类对象（first-class objects），这意味着函数可以像数据一样进行操作。函数不仅可以赋值给变量，还可以作为参数传递给其他函数，甚至作为返回值从其他函数中返回。

装饰器本质上是一个接收一个函数作为参数并返回一个新的函数的函数。因此，装饰器可以通过函数作为参数来扩展或修改原函数的功能。

### 使用 `@` 语法的原理

Python 提供了 `@` 符号作为**装饰器语法糖**，使得我们可以简洁地应用装饰器，而无需手动进行函数赋值。具体来说，`@decorator` 语法等同于以下操作：

```python
foo = decorator(foo)
```

也就是说，`@decorator` 语法会将被装饰的函数传递给 `decorator`，并返回一个新的函数，这个新的函数会替代原函数。

#### 例子

```python
def decorator(func):
    def wrapper():
        print("装饰器添加的功能")
        func()
    return wrapper

@decorator  # 使用 @ 语法应用装饰器
def foo():
    print("原始函数")

foo()  # 调用 foo
```

### 装饰器的执行流程

- Python 会先执行 `decorator(func)`，即 `decorator` 函数会接收 `foo` 作为参数。
- `decorator` 函数返回一个包装函数 `wrapper`，并将它赋值给 `foo`。
- 当我们调用 `foo()` 时，实际上是调用 `wrapper()`，而 `wrapper` 内部会执行 `foo()`。

### 总结

通过 `@` 语法，我们可以更加简洁地为函数添加额外的功能，同时保持代码的清晰和易读性。

## 装饰器的使用场景

### 1. **日志记录**

一个常见的应用场景是对函数调用进行日志记录。通过装饰器，我们可以轻松地为每个函数调用添加日志功能。

```python
def log(func):
    def wrapper(*args, **kwargs):
        print(f"调用函数: {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@log
def add(a, b):
    return a + b

add(5, 3)
```

### 解释

- `log` 装饰器会在调用 `add` 函数之前，先输出一条日志，记录函数的名称。
- `*args` 和 `**kwargs` 允许装饰器处理任何数量的位置参数和关键字参数。

### 输出结果

```plaintext
调用函数: add
```

### 2. **性能计时**

另一个常见用途是计时函数的执行时间，特别是在性能调优时，装饰器非常有用。

```python
import time

def timer(func):
    def wrapper(*args, **kwargs):
        start_time = time.time()  # 记录开始时间
        result = func(*args, **kwargs)
        end_time = time.time()    # 记录结束时间
        print(f"{func.__name__} 执行时间: {end_time - start_time:.4f}秒")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(2)

slow_function()
```

### 解释

- `timer` 装饰器会记录函数执行的开始时间和结束时间，并输出执行的耗时。

### 输出结果

```plaintext
slow_function 执行时间: 2.0001秒
```

### 3. **权限检查**

装饰器还可以用于权限检查，尤其在 Web 开发中非常常见。

```python
def requires_permission(permission):
    def decorator(func):
        def wrapper(*args, **kwargs):
            user_permissions = ['read', 'write']  # 模拟当前用户的权限
            if permission not in user_permissions:
                raise PermissionError("权限不足")
            return func(*args, **kwargs)
        return wrapper
    return decorator

@requires_permission('admin')
def delete_file():
    print("文件已删除")

delete_file()
```

### 解释

- `requires_permission` 是一个接收权限名称作为参数的装饰器，它检查当前用户是否有执行某个操作的权限。
- 如果权限不足，就抛出 `PermissionError`。

### 输出结果

```plaintext
PermissionError: 权限不足
```

## 装饰器的嵌套

装饰器也可以嵌套使用。即一个函数可以同时使用多个装饰器，装饰器会按照从下到上的顺序依次执行。

```python
def decorator1(func):
    def wrapper():
        print("装饰器1开始")
        func()
        print("装饰器1结束")
    return wrapper

def decorator2(func):
    def wrapper():
        print("装饰器2开始")
        func()
        print("装饰器2结束")
    return wrapper

@decorator1
@decorator2
def greet():
    print("你好，Python!")

greet()
```

### 输出结果

```plaintext
装饰器1开始
装饰器2开始
你好，Python!
装饰器2结束
装饰器1结束
```

### 解释

- `greet` 函数先被 `decorator2` 装饰，然后再被 `decorator1` 装饰。
- 装饰器的执行顺序是从内到外的，即先执行 `decorator2`，再执行 `decorator1`。

## 总结

装饰器是 Python 中一个非常强大的工具，可以让我们在不修改原始函数代码的情况下，添加额外的功能。常见的应用场景包括日志记录、性能计时、权限检查等。通过装饰器，我们可以使代码更简洁、更优雅，同时保持功能的扩展性。

### 小结

1. **装饰器的基本形式**：一个函数返回一个新的函数，包装了原始函数的功能。
2. **应用场景**：日志记录、性能计时、权限检查等。
3. **嵌套装饰器**：多个装饰器可以组合使用，按顺序执行。
4. **`@` 语法**：使得装饰器的应用更加简洁，等同于 `foo = decorator(foo)`。