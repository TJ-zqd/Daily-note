# 红黑树



### 红黑树的性质：

#### 1.每个结点都是红的或者黑的

#### 2.根节点是黑的

#### 3.每个叶子结点是黑的

#### 4.如果一个结点是红的，则它的两个儿子都是黑的

#### 5.对每个节点，从该结点到其子孙结点的所有路径上的包含相同数目的黑结点 

### 左旋与右旋

![](.\img\01.png)



```cpp
#define RED 0
#define BLACK 1
typedef int KEY_TYPE;

#define RBTREE_ENTRY(name,type)       \
    struct name {                     \
        struct type *right;   \
        struct type *left;    \
        struct type *parent;  \
        unsigned char color;          \
    }


typedef struct _rbtree_node {   //

    KEY_TYPE key;
    void *value;
#if 1
    struct _rbtree_node *right;
    struct _rbtree_node *left;

    struct _rbtree_node *parent;

    unsigned char color;
#else

    // RBTREE_ENTRY(,thread) ready;

#endif
} rbtree_node;

typedef struct _rbtree {

    struct _rbtree_node *root;
    struct _rbtree_node *nil;

} rbtree;

// 红黑树的左旋
void rbtree_left_rotate(rbtree *T,rbtree_node *x) {

    rbtree_node *y = x->right;

    x->right = y->left;
    if (y->left != T->nil) {
        y->left->parent = x;
    }

    y->parent = x->parent;
    if (x->parent == T->nil) {
        T->root = y;
    } else if (x == x->parent->left) {
        x->parent->left = y;
    } else {
        x->parent->right = y;
    }

    y->left = x;
    x->parent = y; 
}


// 右旋
void rbtree_right_rotate(rbtree *T,rbtree_node *y) {
    
    rbtree_node *x = y->left;
    y->left = x->right;
    if (x->right != T->nil) {
        x->right->parent = y;
    }

    x->parent = y->parent;
    if (y->parent == T->nil) {
        T->root = x;
    } else if (y == y->parent->right) {
        y->parent->right = x;
    } else {
        y->parent->left = x;
    }

    x->left = y;
    y->parent = x; 
}


void rbtree_insert_fixup(rbtree *T,rbtree_node *z) {
    // 前提是Z==RED
    while (z->parent->color == RED) {
        if (z->parent == z->parent->parent->left) {
            rbtree_node *y = z->parent->parent->right;
            if (y->color == RED) {
                z->parent->color = BLACK;
                y->color = BLACK;
                z->parent->parent->color = RED;

                z = z->parent->parent; // z 一直是红色的
            } else {    // y==BLACK
                if (z == z->parent->right) {

                    z = z->parent;
                    rbtree_left_rotate(T,z);
                }
                z->parent->color = BLACK;
                z->parent->parent->color = RED;

                rbtree_right_rotate(T,z->parent->parent);
            }
        } else {

        }
    }
}


// 插入
void rbtree_insert(rbtree *T,rbtree_node *z) {

    rbtree_node *y = T->nil;
    rbtree_node *x = T->root;

    while (x != T->nil) {
        y = x;
        if (z->key < x->key) {
            x = x->left;
        } else if (z->key > x->key) {
            x = x->right;
        } else {  // exist
            return;
        }
    }


    if (y == T->nil) {
        T->root = z;
    } else {
        if (y->key > z->key) {
            y->left = z;
        } else {
            y->right = z;
        }
    }
    
    z->parent = y;

    z->left = T->nil;
    z->right = T->nil;

    z->color = RED;

    rbtree_insert_fixup(T,z);
}

```



## 一、 红黑树（RB-Tree）到底用在哪？

1. **HashMap (Java/C++等)**：
   - **作用**：解决哈希冲突。当哈希表里某个桶（bucket）下的链表太长（Java里是超过8个），查找就慢了。
   - **原理**：把链表转成红黑树，把查找复杂度从 $O(n)$ 降到 $O(\log n)$，防止性能抖动。
2. **CFS (Linux进程调度)**：
   - **作用**：内核得决定下一个运行哪个进程。
   - **逻辑**：红黑树按进程的“虚拟运行时间”排序。左边的节点运行时间最少，内核直接取最左边的节点来运行，保证“公平”。
3. **Epoll (网络模型)**：
   - **作用**：管理海量的文件描述符（fd）。
   - **逻辑**：当你监听一万个连接时，内核用红黑树存这些 fd。增删改查都是 $O(\log n)$，比原来的线性扫描快得多。
4. **定时器 (Timer)**：
   - **作用**：管理各种超时任务。
   - **逻辑**：按触发时间排序，红黑树能很快找到最近要到期的任务。
5. **Nginx**：
   - **应用**：Nginx 内部也用红黑树来管理缓存、定时器以及配置解析后的快速查找。

## 二、 二叉树的核心属性：Key-Value 与顺序

图中间提到的 `key, value` 和 `顺序`，是红黑树作为**关联容器**的灵魂：

- **有序性**：红黑树是二叉搜索树（BST）的变种，它是有序的。你可以轻松地做范围查找（比如找所有 Key 在 10 到 100 之间的值）。
- **自平衡**：普通的二叉树如果插入是有序的，会退化成一个“长链表”。红黑树通过“染色”和“旋转”强行保持平衡，保证树的高度不会太高。

## 三、 几种“强查找”结构的横向对比

图中右侧列出了四种主流方案，面试和架构设计经常考：

1. **RBTree (红黑树)**：
   - **特点**：全内存操作，增删查均衡。
   - **缺点**：不适合大规模磁盘存储，因为树太高，I/O 次数多。
2. **Hash (哈希表)**：
   - **特点**：快到极致（$O(1)$），只要不冲突就是秒找。
   - **缺点**：**无序**。不支持范围查找，只能搜“等于”不能搜“大于/小于”。
3. **B/B+Tree**：
   - **特点**：**矮胖**。一个节点存成百上千个 Key。
   - **应用**：数据库索引（MySQL）。因为它矮，所以去硬盘找数据的次数少。
4. **SkipList (跳表)**：
   - **特点**：用“空间换时间”。在链表上加多层索引。
   - **应用**：**Redis** 的 ZSET。实现起来比红黑树简单，而且并发写的时候锁的粒度更友好。







