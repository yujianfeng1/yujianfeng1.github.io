

## 1. 哈希表简介

哈希表（也称散列表）是一种基于键值对存储数据的数据结构，它通过哈希函数将键映射到数组的某个位置来存储和查找数据，实现了近似 O(1) 的查找效率。

### 1.1 核心概念

- 键（Key）：用于标识数据的唯一值
- 值（Value）：与键关联的数据
- 哈希函数（Hash Function）：将键转换为数组索引的函数
- 桶（Bucket）：数组中存储数据的位置

## 2. 哈希函数

### 2.1 好的哈希函数特征

1. 计算速度快
2. 散列均匀
3. 冲突少
4. 满足确定性

### 2.2 常见的哈希函数

1. 除法散列法
```python
def hash_div(key, table_size):
    return key % table_size
```

2. 乘法散列法
```python
def hash_mul(key, table_size):
    A = 0.6180339887 # 黄金分割比
    return int(table_size * ((key * A) % 1))
```

3. 通用字符串哈希
```python
def hash_string(s, table_size):
    hash_val = 0
    for char in s:
        hash_val = (hash_val * 31 + ord(char)) % table_size
    return hash_val
```

## 3. 冲突解决

### 3.1 开放寻址法

1. 线性探测（Linear Probing）
   - 当发生冲突时，顺序查找下一个空位
   - 优点：实现简单，缓存友好
   - 缺点：容易产生聚集现象

2. 二次探测（Quadratic Probing）
   - 冲突时按二次函数步长寻找：hash(key) + 1², hash(key) + 2², ...
   - 缓解了聚集问题

3. 双重散列（Double Hashing）
   - 使用第二个哈希函数计算步长
   - 分布更均匀

### 3.2 链地址法（拉链法）

```python
class Node:
    def __init__(self, key, value):
        self.key = key
        self.value = value
        self.next = None

class HashTable:
    def __init__(self, size=997):
        self.size = size
        self.table = [None] * size
        self.count = 0
    
    def _hash(self, key):
        if isinstance(key, str):
            return hash_string(key, self.size)
        return hash_div(key, self.size)
    
    def put(self, key, value):
        index = self._hash(key)
        if self.table[index] is None:
            self.table[index] = Node(key, value)
        else:
            current = self.table[index]
            while current:
                if current.key == key:
                    current.value = value
                    return
                if current.next is None:
                    break
                current = current.next
            current.next = Node(key, value)
        self.count += 1
```

## 4. 动态扩容

### 4.1 负载因子

负载因子 = 元素个数 / 表大小
- 通常在负载因子超过 0.75 时进行扩容
- 扩容一般将表大小扩大为原来的 2 倍

```python
def resize(self):
    if self.count / self.size < 0.75:
        return
    
    old_table = self.table
    self.size *= 2
    self.table = [None] * self.size
    self.count = 0
    
    for node in old_table:
        current = node
        while current:
            self.put(current.key, current.value)
            current = current.next
```

## 5. 实际应用

### 5.1 常见应用场景

1. 缓存系统
   - LRU Cache
   - 数据库缓存
   - Web缓存

2. 数据去重
   - 集合实现
   - 重复数据检测

3. 索引
   - 数据库索引
   - 搜索引擎

### 5.2 性能分析

| 操作 | 平均时间复杂度 | 最坏时间复杂度 |
|------|----------------|----------------|
| 插入 | O(1)           | O(n)           |
| 查找 | O(1)           | O(n)           |
| 删除 | O(1)           | O(n)           |

## 6. 优化技巧

1. 选择合适的初始容量
2. 使用素数作为表的大小
3. 根据数据特征选择合适的哈希函数
4. 及时清理已删除的元素
5. 动态调整负载因子阈值

## 7. 注意事项

1. 哈希函数的选择要考虑数据特征
2. 警惕哈希攻击（DOS攻击）
3. 考虑线程安全性
4. 注意内存使用效率
5. 定期维护和优化

## 8. 总结

哈希表是一种极其重要的数据结构，它在各种场景下都有广泛应用。掌握哈希表的原理和实现，对于提升系统性能和解决实际问题都有重要意义。在实际使用中，需要根据具体场景选择合适的实现方式和优化策略。