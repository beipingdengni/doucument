# B+树

B+树与B树类似，不过有一些关键的不同点：

1. B+树的所有值都在叶子节点上，并且叶子节点之间按顺序相连，这使得范围查询变得非常高效。
2. B+树的内部节点（非叶子节点）仅存储键值，并不真正存储数据，这使得内部节点可以有更多的子节点，从而拥有更宽的树和更低的树高。

1. **搜索(Search)**: 从根节点开始，按照键值递归下降到叶子节点，如果找到对应的键，则返回关联的值。
2. **插入(Insert)**: 插入操作涉及向叶子节点添加键值对，如果叶子节点满了，则需要递归地分裂节点，并可能更新根节点。
3. **分裂(Split)**: 当一个节点的键值对数量超过了最大限定，需要将节点分裂成两个，并将中间的键提升到父节点中。
4. **删除(Delete)**: 删除操作涉及从叶子节点中移除键值对，如果删除后节点过小，则可能需要节点合并或重新分配键值对。

```java
class BPlusTree {
    // 定义B+树的节点
    abstract class Node {
        // ...
    }

    // 定义内部节点
    class InternalNode extends Node {
        List<Integer> keys;
        List<Node> children;
        // ...
    }

    // 定义叶子节点
    class LeafNode extends Node {
        List<Integer> keys;
        List<Integer> values;
        LeafNode next;
        // ...
    }

    // B+树的根节点
    private Node root;
    // 树的阶数
    private int order;
    
    // 查找操作
    public Integer search(int key) {
        // ...
    }
    
    // 插入操作
    public void insert(int key, int value) {
        // ...
    }
    
    // 分裂节点
    private Node split(Node node) {
        // ...
    }
    
    // 删除操作
    public void delete(int key) {
        // ...
    }
    
    // ...
}
```

