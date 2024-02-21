

这个简化的B树实现中，我们定义了`BTreeNode`类作为B树的节点，并在其中实现了`insertNonFull`和`splitChild`方法。`BTree`类代表了整个B树，其中包含了`insert`方法来处理插入操作。

> 请注意，这个代码只是一个框架，没有包括完整的错误处理和所有的B树操作，如删除和完整的查找操作。B树的完整实现会更加复杂，包含更多的边界条件的处理。在实际应用中，B树的实现需要仔细处理节点的分裂和合并，以及与磁盘交互时的数据读写操作。

```java
import java.util.ArrayList;

class BTreeNode {
    int t;  // 最小度数（定义了树的分支数范围）
    ArrayList<Integer> keys; // 键的列表
    ArrayList<BTreeNode> children; // 子节点列表
    boolean leaf; // 是否为叶节点的标志

    public BTreeNode(int t, boolean leaf) {
        this.t = t;
        this.leaf = leaf;
        this.keys = new ArrayList<>();
        this.children = new ArrayList<>();
    }

    // 插入非满节点
    public void insertNonFull(int key) {
        int i = keys.size() - 1;

        if (leaf) {
            // 如果是叶节点，在合适位置插入新键
            keys.add(0); // 添加一个空位
            while (i >= 0 && keys.get(i) > key) {
                keys.set(i + 1, keys.get(i));
                i--;
            }
            keys.set(i + 1, key);
        } else {
            // 如果不是叶节点，找到子节点并插入递归
            while (i >= 0 && keys.get(i) > key) {
                i--;
            }
            BTreeNode child = children.get(i + 1);
            if (child.keys.size() == 2 * t - 1) {
                // 如果子节点已满，则先分裂
                splitChild(i + 1, child);
                if (keys.get(i + 1) < key) {
                    i++;
                }
            }
            children.get(i + 1).insertNonFull(key);
        }
    }

    // 分裂满子节点
    public void splitChild(int i, BTreeNode y) {
        BTreeNode z = new BTreeNode(y.t, y.leaf);
        for (int j = 0; j < t - 1; j++) {
            z.keys.add(y.keys.get(j + t));
        }
        if (!y.leaf) {
            for (int j = 0; j < t; j++) {
                z.children.add(y.children.get(j + t));
            }
        }

        children.add(i + 1, z);
        keys.add(i, y.keys.get(t - 1));
        y.keys.subList(t - 1, y.keys.size()).clear();
        if (!y.leaf) {
            y.children.subList(t, y.children.size()).clear();
        }
    }

    // ... 其他方法，如遍历、删除等
}

public class BTree {
    private BTreeNode root;
    private int t;  // 最小度数

    public BTree(int t) {
        this.root = null;
        this.t = t;
    }

    // 插入操作
    public void insert(int key) {
        if (root == null) {
            root = new BTreeNode(t, true);
            root.keys.add(key);
        } else {
            // 如果根节点已满，则树长高
            if (root.keys.size() == 2 * t - 1) {
                BTreeNode newRoot = new BTreeNode(t, false);
                newRoot.children.add(root);
                newRoot.splitChild(0, root);
                int i = 0;
                if (newRoot.keys.get(0) < key) {
                    i++;
                }
                newRoot.children.get(i).insertNonFull(key);
                root = newRoot;
            } else {
                root.insertNonFull(key);
            }
        }
    }

    // ... 其他方法，如遍历、删除等
}

```

