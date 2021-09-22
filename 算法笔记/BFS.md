## BFS广度优先搜索

适用于求最小路径

### 框架

```java
// 计算从起点 start 到终点 target 的最近距离
int BFS(Node start, Node target) {
    Queue<Node> q; // 核心数据结构
    Set<Node> visited; // 避免走回头路

    q.offer(start); // 将起点加入队列
    visited.add(start);
    int step = 0; // 记录扩散的步数

    while (q not empty) {
        int sz = q.size();
        /* 将当前队列中的所有节点向四周扩散 */
        for (int i = 0; i < sz; i++) {
            Node cur = q.poll();
            /* 划重点：这里判断是否到达终点 */
            if (cur is target)
                return step;
            /* 将 cur 的相邻节点加入队列 */
            for (Node x : cur.adj())
                if (x not in visited) {
                    q.offer(x);
                    visited.add(x);
                }
        }
        /* 划重点：更新步数在这里 */
        step++;
    }
}
```

### 例：111 二叉树的最小深度

```java
class Solution {
    public int minDepth(TreeNode root) {
        if (root == null){
            return 0;
        }
        Queue<TreeNode> q = new LinkedList<>();
        q.offer(root);
        int res = 1;

        while (!q.isEmpty()){
            int n = q.size();
            //将队列中所有节点向四周扩散，相当于探测这一层是否有达到叶子的，没有就把下一层所有节点加入队列
            for (int i = 0; i < n; i++) {
                TreeNode cur = q.poll();
                if (cur.left == null && cur.right == null){
                    return res;
                }
                if (cur.left != null){
                    q.offer(cur.left);
                }
                if (cur.right != null){
                    q.offer(cur.right);
                }
            }
            //一层探测结束后，没有到达终点，则高度加1
            res++;
        }
        return res;
    }
}
```