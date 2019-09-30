HashMap 解析

默认参数：

```
// 类继承关系
HashMap<K,V> extends AbstractMap<K,V>  implements Map<K,V>, Cloneable, Serializable 

默认容量（capacity）： 1<<4 【16】
最大容量（maximum capacity）：1 << 30
载入（load factor）： 0.75f
连表转为树（threshold for using a tree）：8
树转化为连表（threshold for untreeifying a (split) bin）：6
全部是转化为树（The smallest table capacity for which bins may be treeified）：64


```



##### 解析数据结构 https://www.jianshu.com/p/c67284ef9e00

##### 红黑树的特点

```
红黑树也叫自平衡二叉查找树或者平衡二叉B树，
时间复杂度为O(log n)
高度h <= log2(n+1)

性质1 节点是红色或黑色。
性质2 根节点是黑色。
性质3 每个叶节点（NIL节点，空节点）是黑色的。
性质4 每个红色节点的两个子节点都是黑色。(从每个叶子到根的所有路径上不能有两个连续的红色节点)
性质5 从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点。
```

##### 讲解红黑树

```
1、要了解红黑树，先要知道avl树，要知道avl树，首先要知道二叉树，其实很简单，二叉树就是每个父节点下面有零个一个或两个子节点

2、avl树即平衡树，他对二叉树做了改进，在我们每插入一个节点的时候，必须保证每个节点对应的左子树和右子树的树高度差不超过1。如果超过了就对其进行调平衡，具体的调平衡操作就不在这里讲了，无非就是四个操作——左旋，左旋再右旋，右旋再左旋。最终可以是二叉树左右两边的树高相近，这样我们在查找的时候就可以按照二分查找来检索，也不会出现退化成链表的情况。

3、二三树：二三树与普通二叉树的不同点在于他有二节点和三节点。二节点下面有两个子节点，二节点里面可以容纳一个值，而三节点下面有三个子节点，三节点里面可以容纳两个值。

```



##### treeifyBin 分析    [连表转为树，满足 连表长度为8，且容量大于64]

```java
/*树形化*/
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        // 定义n:节点数组长度、index:hash对应的数组下标、e:用于循环的迭代变量,代表当前节点
        int n, index; Node<K,V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            // 若数组尚未初始化或者数组长度小于64,则直接扩容而不进行树形化
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {// 获取指定数组下标的头结点e
            TreeNode<K,V> hd = null, tl = null;// 定义head节点hd、尾节点tl
            do {// 循环,该循环主要是将原单向链表转化为双向链表
                TreeNode<K,V> p = replacementTreeNode(e, null);// 以e的hash、key、value,以及以null为后继元创建树形节点p
                if (tl == null)// 若尾节点为null表明首次循环,此时e为头结点、p为根节点,因此将p赋值给表示头结点的hd
                    hd = p;
                else {// 负责根节点已经产生过了此时tl尾节点指向上次循环创建的树形节点
                    p.prev = tl;// 此时p为上次循环的的后继元在本次循环为当前节点,产生当前节点与前驱元的prev链
                    tl.next = p;// 产生前驱元与当前节点的next链
                }
                tl = p;// 将tl指向当前节点
            } while ((e = e.next) != null);// e指向e的后继元
            if ((tab[index] = hd) != null)// 若指定的位置头结点不为空则进行树形化
                hd.treeify(tab);// 根据链表创建红黑树结构
        }
    }
```

