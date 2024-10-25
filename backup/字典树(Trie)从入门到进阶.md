

## 1. 什么是字典树？

字典树，也称为前缀树(Prefix Tree)或Trie树，是一种树形数据结构，特别适用于处理字符串集合。它的主要特点是：
- 根节点不包含字符，除根节点外的每个节点都包含一个字符
- 从根节点到某一节点，路径上经过的字符连接起来，为该节点对应的字符串
- 每个节点的所有子节点包含的字符都不相同

## 2. 字典树长什么样？

让我们用一个具体的例子：存储 "cat"、"car"、"dog" 这三个单词。

```
        根
     /     \
    c       d
    |       |
    a       o
   / \      |
  t   r     g
```

观察这个树：
1. 第一层分叉：c 和 d 开头的词分开存
2. 共同前缀只存一次："ca" 只存了一次
3. 不同结尾分开存："cat" 和 "car" 在最后一个字母处分道扬镳

## 3. 字典树解决什么问题？

### 3.1 输入法联想功能
当你输入"h"时，输入法会提示"hello"、"hi"、"how"等。这就是字典树的典型应用！

### 3.2 拼写检查
Word文档中输入错误时会有红色波浪线提示，背后可能就是字典树在工作。

### 3.3 搜索引擎的自动补全
你在搜索框输入时，会弹出相关的热门搜索，这也是字典树的应用。

## 4. Go语言实现

让我们用简单的代码实现一个基础的字典树：

```go
// 树的节点
type TrieNode struct {
    // children存储子节点，比如 'a' -> 节点A
    children map[rune]*TrieNode
    // 标记这里是否是一个完整的词
    isWord   bool
}

// 字典树
type Trie struct {
    root *TrieNode
}

// 创建新的字典树
func NewTrie() *Trie {
    return &Trie{
        root: &TrieNode{
            children: make(map[rune]*TrieNode),
        },
    }
}

// 往字典树中添加一个词
func (t *Trie) AddWord(word string) {
    node := t.root
    // 遍历单词的每个字母
    for _, char := range word {
        // 如果这个字母不存在，就创建新节点
        if node.children[char] == nil {
            node.children[char] = &TrieNode{
                children: make(map[rune]*TrieNode),
            }
        }
        // 移动到子节点
        node = node.children[char]
    }
    // 标记这是一个完整的词
    node.isWord = true
}

// 查找一个词是否在字典树中
func (t *Trie) SearchWord(word string) bool {
    node := t.root
    // 顺着路径走
    for _, char := range word {
        if node.children[char] == nil {
            return false  // 路走不通了，说明没有这个词
        }
        node = node.children[char]
    }
    return node.isWord  // 看看这里是否标记为一个完整的词
}

// 查找有哪些词是以prefix开头的
func (t *Trie) GetWordsWithPrefix(prefix string) []string {
    result := []string{}
    node := t.root
    
    // 先找到前缀的最后一个节点
    for _, char := range prefix {
        if node.children[char] == nil {
            return result
        }
        node = node.children[char]
    }
    
    // 从这个节点开始，搜集所有的词
    var collect func(node *TrieNode, word string)
    collect = func(node *TrieNode, word string) {
        if node.isWord {
            result = append(result, word)
        }
        for char, child := range node.children {
            collect(child, word + string(char))
        }
    }
    
    collect(node, prefix)
    return result
}
```

## 5. 实际使用例子

```go
func main() {
    // 创建一个新的字典树
    trie := NewTrie()
    
    // 添加一些词
    words := []string{"hello", "hi", "hey", "world"}
    for _, word := range words {
        trie.AddWord(word)
    }
    
    // 查找单词
    fmt.Println(trie.SearchWord("hello"))  // true
    fmt.Println(trie.SearchWord("hel"))    // false
    
    // 查找前缀为"he"的所有词
    fmt.Println(trie.GetWordsWithPrefix("he"))  // [hello hey]
}
```

## 6. 字典树的优缺点

### 优点：
1. **快速查找**: 找词的速度只和词的长度有关，和词典中有多少词无关
2. **节省空间**: 相同前缀的词只存一次，比如"hello"和"hey"的"he"只存一次
3. **支持前缀查找**: 可以很容易地找到所有以某个前缀开头的词

### 缺点：
1. **占内存**: 需要存储很多指向子节点的指针
2. **不适合精确查找**: 如果只是想知道一个词在不在词典里，用哈希表更好

## 7. 使用建议

1. 当你需要处理以下场景时，考虑使用字典树：
   - 需要自动补全功能
   - 需要前缀匹配功能
   - 有很多相似前缀的词需要存储

2. 不建议使用字典树的场景：
   - 仅需要精确查找词是否存在
   - 内存非常有限的环境
   - 词的前缀很少重复


 

