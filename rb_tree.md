# 快速理解红黑树

### 红黑树性质
1. 节点是红色或黑色。
2. 根是黑色。
3. 所有叶子都是黑色（叶子是NIL节点）。
4. 每个红色节点必须有两个黑色的子节点。（从每个叶子到根的所有路径上不能有两个连续的红色节点。）
5. 从任一节点到其每个叶子的所有简单路径都包含相同数目的黑色节点。

本文只介绍如何理解红黑树
### 2-3-4树介绍

2-3-4树是四阶的 B树(Balance Tree)

1. 所有叶子节点都拥有相同的深度。
2. 节点只能是 2-节点、3-节点、4-节点之一。
    * 2-节点：包含 1 个元素的节点，有 2 个子节点；
    * 3-节点：包含 2 个元素的节点，有 3 个子节点；
    * 4-节点：包含 3 个元素的节点，有 4 个子节点；
3. 元素始终保持排序顺序，整体上保持二叉查找树的性质，即父结点大于左子结点，小于右子结点；而且结点有多个元素时，每个元素必须大于它左边的和它的左子树中元素。

![1](https://s.im5i.com/2021/04/18/GPLL3.png)


一棵典型的2-3-4树

![2](https://s.im5i.com/2021/04/18/GPitM.png)


任何一棵红黑树都可以与2-3-4树互相等价转换。例如

![3](https://s.im5i.com/2021/04/18/GPrJY.png)

可以随意找个红黑树实验下，只需将指向红色节点的连接折平，结合为一个3-节点或4-节点。

#### 2-3-4树的插入

因为是排序树所以可以很快找到插入位置，从底部插入。

1. 插入节点为2节点，直接插入

![4](https://s.im5i.com/2021/04/18/GP04w.png)

2. 插入节点为3节点，直接插入

![5](https://s.im5i.com/2021/04/18/GPM0F.png)

3. 插入节点为4节点，
    * 先提取3-节点的中间值，并分裂为两个2-节点，再根据要插入的值，选择插入其中一个2-节点。
    * 再将中间值插入到上层节点，重复插入步骤。最终如果到达根节点，且根节点为4节点，插入值则树的高度增加一。

插入4-节点，且父节点为2-节点

![6](https://s.im5i.com/2021/04/18/GPj2d.png)

插入3-节点，且父节点为4-节点

![7](https://s.im5i.com/2021/04/18/GPTtK.png)



#### 如果用红黑树模拟2-3-4树插入



直观的，3节点对应两种结构，
1. 红色节点在左边
2. 红色节点在右边

预备知识
* 左旋: 结构2变为结构1
* 右旋: 结构1变为结构2


下图是对应的3，4节点：


![8](https://s.im5i.com/2021/04/18/GPhYC.png)


* 插入2-节点(a)
    2-节点是黑色，根据key与a,b的大小，直接插入
* 插入3-节点(a < b)
    3-节点是一黑一红，可以有两种结构，
    * key < a
        * 3-节点表示为第一种结构时，先变为第二种结构，再插入
        * 3-节点表示为第二种结构时，直接插入
    * a < key < b
        * 3-节点表示为第一种结构时，先(a,key)左旋，此时再按上面处理
        * 3-节点表示为第二种结构时，先(a,b)变为第一种结构，再按上面处理
    * b < key
        * 3-节点表示为第一种结构时，直接插入
        * 3-节点表示为第二种结构时，变为第一种结构，再插入

* 插入4节点(a< b < c)
    不能直接插入，此时要处理的情况已经非常多了。 
    * 该4节点父亲是2-节点
    * 该4节点父亲是3-节点
    * 该4节点父亲是4-节点

根据不同的父节点，还要区分处理key与a，b，c之间的关系，并且再根据关系，使用旋转和变色进行调整，情况就变的更加复杂起来。
显然按照这种思路去实现时，不仅情况复杂，且不易于实现。此时我们再介绍一种2-3-4树的等价树2-3树。

### 2-3树介绍

1. 所有叶子节点都拥有相同的深度。
2. 节点只能是 2-节点、3-节点之一。
    * 2-节点：包含 1 个元素的节点，有 2 个子节点；
    * 3-节点：包含 2 个元素的节点，有 3 个子节点；
3. 元素始终保持排序顺序，整体上保持二叉查找树的性质，即父结点大于左子结点，小于右子结点；而且结点有多个元素时，每个元素必须大于它左边的和它的左子树中元素。


我们将一个2-3-4树中的4-节点，当做一种已经往3节点插入值需要处理的节点。将4-节点(a,b,c)的中间值b提出，分裂为(a),(c)两个子节点。再将b插入其父亲节点。所以与2-3-4树完全等价，即与红黑树完全等价。如下图，将11提出，分裂为(10)和(12)；递归的，再将11插入变为(7，9，11)，；递归的，提出9，将9插入根节点，变为(5，9).

![9](https://s.im5i.com/2021/04/18/GPkFH.png)



#### 2-3树的插入与模拟

1. 插入节点为2节点(a)，直接插入
2. 插入节点为3节点(a b)
    * 先提取(a,b,key)的中间值，并分裂为两个2-节点。
    * 再将中间值插入到上层节点，重复插入步骤。最终如果到达根节点，且根节点为3节点，插入值则树的高度增加一。

以下图为例子

![10](https://s.im5i.com/2021/04/18/GPCOa.png)

插入节点1，过程如下

![11](https://s.im5i.com/2021/04/18/GPqDT.png)


**我们规定3节点的左边的数为红色**，用红黑树模拟，此时情况变少了

1. 插入2-节点(a)
    * key < a ，直接插入
    * key > a ，插入，左旋并改变颜色(红色)；右子树不能为红色
2. 插入3节点(a,b)，取出(a,b,key)的中间值x，其余值分裂为俩2-节点
    * 父节点是2-节点
        * 按1步骤，将x插入到父节点中
    * 父节点是3-节点,如(a1,b1)
        * x < a1 ，右旋，变色 ，递归处理 
        * a1 < x < b1 ，左旋，右旋，变色，递归处理
        * b1 < x ， 变色，递归处理
    * 没有父节点，即根节点
        * 将x作为新的根节点，颜色为黑色

上图对应红黑树如下

![12](https://s.im5i.com/2021/04/18/GP8hA.png)


插入节点key < a ,如插入1，过程如下
1. 1应该插入2的左边，但出现连续红色，(2,3)右旋，2获得3的颜色，3变红色。

![13](https://s.im5i.com/2021/04/18/GPFJS.png)

2. 此时2的左右子树均为红色，分裂为(1)(3)，即1，3变黑色，2变红色。

![14](https://s.im5i.com/2021/04/18/GPVA0.png)

3. 递归向上，6的左子树2，4出现连续红色，同样将(6,4)右旋，4获得6的颜色，6变为红色。 假设6原本的颜色为红色，那么4根节点也有可能就变为红，所以最后要将根节点置为黑色 。

![15](https://s.im5i.com/2021/04/18/GP50B.png)


同样，如果插入节点为a < key < b,如在树中插入2，只需将(1,2)左旋，又回到步骤1。

![16](https://s.im5i.com/2021/04/18/GPG2z.png)

如果key > b,插入3，那么又回到了步骤2

![17](https://s.im5i.com/2021/04/18/GPbvs.png)


### 从红黑树，到从2-3-4树，到2-3树，再到红黑树

为什么不直接从2-3树开始？
> 因为红黑树中，黑节点下面可以有两个红节点，并不容易直接看成2-3树，但可以直接看为2-3-4树。无论是2-3树还是2-3-4树，所有叶子节点都拥有相同的深度，并且升高时只会在根节点，矮胖且修改时，不会过多影响其他子树。

![18](https://s.im5i.com/2021/04/18/GP2Yo.png)

### 实践代码

红黑树节点
```
struct rb_node 
{
    enum Color : int{
        red,
        black
    };
    
    int key;
    int val;
    Color  color;
    rb_node* left;
    rb_node* right;
public:
    rb_node(int k,int v) : key(k),val(v),color(red),left(nullptr),right(nullptr){};
    
};
```
#### 红黑树的插入
```
struct rb_tree
{
    //判断颜色
    bool is_red(){
        if(node == nullptr) return false; //叶节点为黑色
        return (node->color == rb_node::red);
    }

    //左旋
    rb_node* rotate_left(rb_node* node){

        rb_node* x = node->right;
        rb_node* x_left = x->left;
        node->right = x_left;
        x->left = node;
        x->color = node->color; //将node节点颜色保留
        node->color =rb_node:: red; 
    }

    //右旋
    rb_node* rotate_right(rb_node* node){

        rb_node* x = node->left;
        rb_node* x_right = x->right;
        node->left = x_right;
        x->right = node;
        x->color = node->color; // 将node节点颜色保留
        node->color = rb_node::red;
    }

    //变色
    void flip_color(rb_node* node){
        node->color = rb_node::red;
        node->left->color = rb_node::black;
        node->right->color = rb_node::black;
    }

    //插入
    void put(int key,int val){
        put(root,key,val);
        root->color = rb_node::black; //总是将根节点变黑
    }

    rb_node* put(rb_node* node, int key,int val){
        if(node == nullptr){        
            node = new rb_node(key,val);
            return node;
        }        

        //找到适当位置，从叶子节点，递归向上处理
        if(key < node->key){  
            node->left = put(node->left,key,val);
        }else if(key > node->key){
            node->right = put(node->right,key,val);
        }else{
            node->val = val;
        }

        
        //选择要操作的node        
        if(is_red(node->right) && !is_red(node->left)) node = rotate_left(node);  //右子树为红色，且左子树不为红色，左旋
        if(is_red(node->left) && is_red(node->left->left)) node = rotate_right(node);//左子树，左子树的子树为红，即左子树连续两个红色节点，将该节点右旋。
        if(is_red(node->right) && is_red(node->left)) flip_color(node); //当左右子树都为红色时，变色

        return node;
    }

private:
    rb_node* root;
}
```



[参考资料:算法（第4版）作者: 塞奇威克 (Robert Sedgewick) / 韦恩 (Kevin Wayne)](https://book.douban.com/subject/19952400/)