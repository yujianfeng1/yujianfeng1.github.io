
深度优先搜索（Depth-First Search，简称 DFS）是一种用于遍历或搜索树/图的算法。本文将通过图文结合的方式，深入浅出地讲解 DFS 算法的原理、实现和应用。

### DFS 算法的基本概念

深度优先搜索就像在走迷宫时采用的策略：沿着一条路一直走到底，当走不通时就回退一步换一条路继续走。这种"一条路走到黑"的特性使其非常适合解决迷宫类问题。

### 图文解释
#### 示例图结构

#### 遍历过程

#### 遍历顺序说明

#### 状态说明
#### 深度优先的关键特点
+ 优先选择一个方向深入探索，直到无法继续前进
+ 使用回溯的方式返回到最近的未完全探索的节点
+ 确保每个分支都被完全探索后才会转向其他分支
+ 通过递归或栈来实现回溯机制


### 代码实践
```go
package main

import "fmt"

// Node 表示图中的节点
type Node struct {
    Value     int
    Neighbors []*Node
    Visited   bool
}

// NewNode 创建新节点
func NewNode(value int) *Node {
    return &Node{
        Value:     value,
        Neighbors: make([]*Node, 0),
        Visited:   false,
    }
}

// AddEdge 添加边（无向图）
func (n *Node) AddEdge(neighbor *Node) {
    n.Neighbors = append(n.Neighbors, neighbor)
    neighbor.Neighbors = append(neighbor.Neighbors, n)
}

// DFS 递归实现深度优先搜索
func DFS(node *Node) {
    if node == nil || node.Visited {
        return
    }
    
    // 标记当前节点为已访问
    node.Visited = true
    fmt.Printf("访问节点: %d\n", node.Value)
    
    // 递归访问所有相邻节点
    for _, neighbor := range node.Neighbors {
        DFS(neighbor)
    }
}

// DFSIterative 迭代实现深度优先搜索
func DFSIterative(start *Node) {
    if start == nil {
        return
    }
    
    // 使用切片作为栈
    stack := []*Node{start}
    
    for len(stack) > 0 {
        // 弹出栈顶节点
        node := stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        
        if !node.Visited {
            node.Visited = true
            fmt.Printf("访问节点: %d\n", node.Value)
            
            // 将未访问的相邻节点压入栈中
            for i := len(node.Neighbors) - 1; i >= 0; i-- {
                neighbor := node.Neighbors[i]
                if !neighbor.Visited {
                    stack = append(stack, neighbor)
                }
            }
        }
    }
}

func main() {
    // 创建一个示例图
    node1 := NewNode(1)
    node2 := NewNode(2)
    node3 := NewNode(3)
    node4 := NewNode(4)
    node5 := NewNode(5)
    
    // 添加边
    node1.AddEdge(node2)
    node1.AddEdge(node3)
    node2.AddEdge(node4)
    node3.AddEdge(node4)
    node4.AddEdge(node5)
    
    fmt.Println("递归DFS遍历结果：")
    DFS(node1)
    
    // 重置visited标记
    node1.Visited = false
    node2.Visited = false
    node3.Visited = false
    node4.Visited = false
    node5.Visited = false
    
    fmt.Println("\n迭代DFS遍历结果：")
    DFSIterative(node1)
}

```

我实现了两种深度优先搜索的方式：

1. 递归实现（DFS函数）：
- 使用递归的方式实现最简单直观
- 利用系统栈来保存访问路径
- 代码更简洁，但对于深度很大的图可能会导致栈溢出

2. 迭代实现（DFSIterative函数）：
- 使用显式栈来模拟递归过程
- 避免了递归调用的栈溢出风险
- 可以更好地控制搜索过程

代码中包含了：
- 图节点的数据结构定义
- 添加边的方法
- 两种DFS实现
- 完整的示例，展示了如何构建和遍历一个简单的图

示例中构建的图结构如下：
```
1 -- 2
|    |
3 -- 4 -- 5
```

运行这段代码，你可以看到两种实现方式的遍历顺序。需要注意的是，对于同一个图，不同的实现方式可能会产生不同的遍历顺序，但都是深度优先的。
