## Java 二叉树，前序、中序、后序遍历实现

- `preorderTraversal()` 方法实现了**前序遍历**，访问节点顺序为：根节点 -> 左子树 -> 右子树。
- `inorderTraversal()` 方法实现了**中序遍历**，访问节点顺序为：左子树 -> 根节点 -> 右子树。
- `postorderTraversal()` 方法实现了**后序遍历**，访问节点顺序为：左子树 -> 右子树 -> 根节点。

```java
class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;

    TreeNode(int x) {
        val = x;
    }
}

public class BinaryTreeTraversal {

    // 前序遍历
    public void preorderTraversal(TreeNode root) {
        if (root != null) {
            System.out.print(root.val + " ");
            preorderTraversal(root.left);
            preorderTraversal(root.right);
        }
    }

    // 中序遍历
    public void inorderTraversal(TreeNode root) {
        if (root != null) {
            inorderTraversal(root.left);
            System.out.print(root.val + " ");
            inorderTraversal(root.right);
        }
    }

    // 后序遍历
    public void postorderTraversal(TreeNode root) {
        if (root != null) {
            postorderTraversal(root.left);
            postorderTraversal(root.right);
            System.out.print(root.val + " ");
        }
    }
 
   // 使用前序遍历进行搜索（数据搜索返回）
    public TreeNode search(TreeNode root, int val) {
        if (root == null) {
            return null;
        }

        if (root.val == val) {
            return root;
        }

        // 首先在左子树中搜索
        TreeNode leftSearchResult = search(root.left, val);
        if (leftSearchResult != null) {
            return leftSearchResult;
        }

        // 如果左子树中没有找到，则在右子树中搜索
        return search(root.right, val);
    }

    public static void main(String[] args) {
        TreeNode root = new TreeNode(1);
        root.left = new TreeNode(2);
        root.right = new TreeNode(3);
        root.left.left = new TreeNode(4);
        root.left.right = new TreeNode(5);

        BinaryTreeTraversal btt = new BinaryTreeTraversal();

        System.out.print("前序遍历: ");
        btt.preorderTraversal(root);
        System.out.println();

        System.out.print("中序遍历: ");
        btt.inorderTraversal(root);
        System.out.println();

        System.out.print("后序遍历: ");
        btt.postorderTraversal(root);
        System.out.println();
    }
}
```

