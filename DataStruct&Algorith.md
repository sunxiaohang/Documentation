##  *DataStructure-Algorithm*

##### 二分查找（binary_search）
`tip`:Arrays为查找数组 result为查找结果 length为数组长度
- 时间复杂度
- 最好结果：1
- 最坏结果：LOG(N)

```
/*静态查找（字典）
 * 动态查找（文件结构）
 * */
int binary_search(int arrays[],int result,int length){
    int begin=0,end=length-1;
    int mid=0;
    while(begin<=end){
        mid=(begin+end)/2;
        if(arrays[mid]==result)break;
        else if(arrays[mid]<result)begin=mid+1;
        else if(arrays[mid]>result)end=mid-1;
    }
    return mid;
}
```

##### 最大子序列和（maxSubSeqSum）
- 时间复杂度:T(N)=O(N3)

```
int MaxSubSeqSum(int arrays[],int length){
    int i,j,k,thisSum=0,maxSum=0;
    for(i=0;i<length;i++){
        for(j=i;j<length;j++){
            thisSum=0;
            for(k=i;k<=j;k++){
                thisSum+=arrays[k];
            }
            if(thisSum>maxSum)maxSum=thisSum;
        };
    }
    return maxSum;
}
```

##### 最大子序列和改进1（maxSubSeqSum）
- 时间复杂度:T(N)=O(N2)

```
int MaxSubSeqSum(int arrays[],int length){
    int i,j,thisSum=0,maxSum=0;
    for(i=0;i<length;i++){//i是子列左端
        thisSum=0;//从arrays[i]到arrays[j]的子序列和
        for(j=i;j<length;j++){//j是子序列右端
            thisSum+=arrays[j];
            if(thisSum>maxSum)maxSum=thisSum;//如果本次求和大于最终结果，更新maxSum
        };
    }
    return maxSum;
}
```

##### 最大子序列和（maxSubSeqSum）--分治法
- 算法复杂度:T(N)=O(NlgN)

```
int MaxSubSeqSum(int arrays[], int left, int right) {
    int sum = 0;
    if (left == right) {
        if (arrays[left] > 0)sum=arrays[left];
        else sum = 0;
    } else {
        int middle = (left + right) / 2;
        int leftSum = MaxSubSeqSum(arrays, left, middle);
        int rightSum = MaxSubSeqSum(arrays, middle+1, right);
        int finalLeftSum = 0, thisLeftSum = 0;
        for (int i = middle; i >=left; i--) {
            thisLeftSum += arrays[i];
            if (thisLeftSum > finalLeftSum)finalLeftSum = thisLeftSum;
        }
        int finalRightSum = 0, thisRightSum = 0;
        for (int j = middle+1; j <=right; j++) {
            thisRightSum += arrays[j];
            if (thisRightSum > finalRightSum)finalRightSum = thisRightSum;
        }
        sum = finalLeftSum + finalRightSum;
        printf("left sum is %d,right sum is %d\n",finalLeftSum,finalRightSum);
        if (sum < leftSum)sum = leftSum;
        if (sum < rightSum)sum = rightSum;
    }
    return sum;
}
```

##### 线性表的顺序存储

```
typedef struct LinkedList{
    int data[MAXSIZE];//存储数组
    int Last;//游标指针
}List;
List * MakeEmpty(){
    List *Ptrl;
    Ptrl=(List *)malloc(sizeof(List));
    Ptrl->Last=-1;
    return Ptrl;
}
/**T(n)=O(N)*/
int FindElement(int result,List* Ptrl){
    int i=0;
    while(i<=Ptrl->Last&&Ptrl->data[i]!=result)
        i++;
    if(i>Ptrl->Last)return -1;//如果没有找到返回-1
    else return i;
}
void Insert(int result,int i,List *Ptrl){
    int j;
    if(Ptrl->Last==MAXSIZE-1){
        printf("表满");
        return;
    }
    if(i<1||i>Ptrl->Last+2){
        printf("插入位置不合法");
        return;
    }
    for(j=Ptrl->Last;j>=i-1;j--)//倒序移动（注意j>=i-1）
        Ptrl->data[j+1]=Ptrl->data[j];
    Ptrl->data[i]=result;//插入数据
    Ptrl->Last++;//Last指针++
    return;
}
/*T(N)=O(N)*/
void DeleteElement(int i,List* Ptrl){
    int j;
    if(i<1||i>Ptrl->Last+1){//判断位置的合法性（注：链表下标是从1开始的）
        printf("第%d个元素不存在",i);
        return;
    }
    for(j=i;j<=Ptrl->Last;j++)//所有元素前移
        Ptrl->data[j-1]=Ptrl->data[j];
    Ptrl->Last--;
    return;
}
```

##### 线性表的链式存储

```
typedef struct LinkedList {
    int data;
    struct LinkedList *next;
} List;

//求链表长度
int Length(List *Ptrl) {
    List *p = Ptrl;
    int j = 0;
    while (p) {
        p = p->next;
        j++;
    }
    return j;
}

//按序号查找
List *FindKth(int k, List *Ptrl) {
    List *p = Ptrl;//不改变头节点，返回移动指针p
    int i = 1;
    while (p != NULL && i < k) {
        p = p->next;
        i++;
    }
    if (i == k)return p;
    else return NULL;
}

//按值查找
List *Find(int result, List *Ptrl) {
    List *p = Ptrl;
    while (p != NULL && p->data != result)
        p = p->next;
    return p;
}

//构造新的结点，申请空间
//找到链表的低i-1个结点
//修改指针插入结点
List *Insert(int result, int i, List *Ptrl) {
    List *p, *s;
    if (i == 1) {//判断链表为空的时候
        s = (List *) malloc(sizeof(List));
        s->data = result;
        s->next = Ptrl;
        return s;
    }
    p = FindKth(i - 1, Ptrl);//查找第i-1个节点
    if (p == NULL) {
        printf("参数错误");
        return NULL;
    } else {
        s = (List *) malloc(sizeof(List));
        s->data = result;
        s->next = p->next;
        p->next = s;
        return Ptrl;
    }
}

List *Delete(int i, List *Ptrl) {
    List *p, *s;
    if (i == 1) {//检查要删除的是不是第一个结点
        s = Ptrl;//s指向第一个结点
        if (Ptrl != NULL)Ptrl = Ptrl->next;//从链表中删除
        else return NULL;
        free(s);//释放空间
        return Ptrl;
    }
    p = FindKth(i, Ptrl);
    if (p == NULL) {//后项前移
        printf("输入错误");
        return NULL;
    } else if (p->next != NULL) {
        s=p->next;
        p->next = s->next;
        free(s);//清理无用空间
        return Ptrl;
    }
}
```


##### 广义表

`tip`:广义表是线性表的推广
 - 对于线性表而言，n的元素都是基本单元素
 - 广义表中，这些元素不仅可以是单元素也可以是另一个广义表

```
typedef struct GNode{
    int Tag;//标识域：0表示结点是单元素，1表示结点是广义表
    union {//子表指针域Sublist与单元素数据域Data复用，即共用存储空间
        int data;
        struct GNode *SubList;
    }URegion;
    struct GNode *next;
}GList;
```

 - 多重链表中节点的指针与会有多个，但包含两个指针域的链表并不一定是多重链表，比如双向链表
 - 多重链表有广泛的用途，（树，图等相对复杂的数据结构都可以用多重链表存储）

*矩阵可以使用二维数组表示，但二维数组表示优缺点*
 - 1.数组的大小需要事先确定 
 - 2.对于稀疏矩阵将造成大量的存储空间浪费
  
*采用一种典型的多重链表-十字链表来存储稀疏矩阵*
-  只存储矩阵非0元素项（结点数据域：行坐标Row，列坐标Col，数值Value）
- 每个结点通过两个指针域，吧同行、同列串起来；
- 行指针（右指针）Right
- 列指针（下指针）Down
 
##### 堆栈（顺序存储）数组方式

```
typedef struct{
    int Data[MAXSIZE];
    int Top;
}Stack;
void Push(Stack *stack,int value){
    if(stack->Top==MAXSIZE-1){//数组有界
        printf("堆栈满");
    }else{
        stack->Data[++(stack->Top)]=value;
        return;
    }
}
int Pop(Stack *stack){
    if(stack->Top==-1){//为空检查
        printf("堆栈为空");
        return ERROR;
    } else
    return stack->Data[stack->Top--];
}
```

#####  一个有界数组存储两个堆栈
`tip`:一个有界数组存储两个堆栈，如果数组有空间则执行入栈操作，（一个向右增长，一个向左增长）

```
#define MAXSIZE 50
   typedef struct DStack{
       int data[MAXSIZE];
       int Top1;
       int Top2;
   }Stacks;
   void Push(Stacks *stacks,int value,int Tag){
       if(stacks->Top2-stacks->Top1==1){
           printf("堆栈满");return;
       }
       if(Tag==1)
           stacks->data[++stacks->Top1]=value;
       else
           stacks->data[--stacks->Top2]=value;
   }
   int Pop(Stacks *stacks,int Tag){
       if(Tag==1){
           if(stacks->Top1==-1){
               printf("堆栈1空");
               return NULL;
           }else {
               return stacks->data[stacks->Top1--];
           }
       }else{
           if(stacks->Top2==MAXSIZE){
               printf("堆栈2空");
               return NULL;
           }else{
               return stacks->data[stacks->Top2++];
           }
       }
   }
   int main() {
       Stacks *stacks;
       stacks->Top1=-1;
       stacks->Top2=MAXSIZE;//初始化两个堆栈头指针
       return 0;
   }
```

##### 堆栈（链式存储）

```
/*用单向链表表示栈时候，栈Top结点一定是链头结点
 * */
typedef struct Node{
    int value;
    struct Node *next;
}LinkedStack;
LinkedStack * CreateLinkedStack(){
    LinkedStack *stack;
    stack=(LinkedStack *)malloc(sizeof(LinkedStack));
    stack->next=NULL;
    return stack;
};
int isEmpty(LinkedStack *stack){//注意Top结点没有值，只有一个单链表的头指针
    return (stack->next==NULL);
}
void Push(LinkedStack *stack,int value){
    LinkedStack *insertElement;
    insertElement=malloc(sizeof(LinkedStack));//分配内存空间
    insertElement->value=value;//插入的值赋值给结点
    insertElement->next=stack->next;//将已存在链表链接到插入的结点
    stack->next=insertElement;//改变Top结点
}
int Pop(LinkedStack *stack){
    int result;
    LinkedStack *popElement;
    if(isEmpty(stack)){
        printf("链表为空");
        return ERROR;
    }else{
        popElement=stack->next;
        result=popElement->value;
        stack->next=popElement->next;
        free(popElement);//记得释放无用内存空间
        return result;
    }
}
```

*中缀表达式如何转换为后缀表达式 从头到尾读取中缀表达式的每一个对象 1.运算数：直接输出 2.左括号：压入堆栈 3.右括号：将栈顶的运算符弹出并输出，直到遇到左括号（出栈不输出） 4.运算符： 若优先级大于栈顶运算符，则压栈 若优先级小于等于栈顶运算符，将栈顶运算符弹出兵书vhu，再比较新的栈顶运算符，直到改运算符大于栈顶运算符优先级为止，然后压栈 5.若个对象处理完毕，则把堆栈中存留的运算符一并输出 堆栈用途： 函数调用及递归实现 深度优先搜索 回溯算法*

##### 队列（顺序存储）

```
#define MAXSIZE 50
typedef struct {
    int value[MAXSIZE];
    int rear;
    int front;
}Queue;
Queue *CreateQueue(){
    Queue *queue;
    queue=(Queue *)malloc(sizeof(Queue));
    queue->front=0;
    queue->rear=0;
    return queue;
}
void AddQueue(Queue *queue,int value){
    if((queue->rear+1)%MAXSIZE==queue->front){
        printf("queue is full");
        return;
    }else {
        queue->rear=(queue->rear+1)%MAXSIZE;
        queue->value[queue->rear] = value;
    }
}
int IsEmpty(Queue *queue){
    if(queue->rear==queue->front)return 1;
    else return 0;
}
void OutQueue(Queue* queue,int *value){
    if(IsEmpty(queue)){
        printf("Queue is empty");
        return;
    }else{
        queue->front=(queue->front+1)%MAXSIZE;
        *value=queue->value[queue->front];
    }
}
```
##### 队列（链式存储）注意
```
typedef struct Node {
    int value;
    struct Node *next;
} QNode;
typedef struct {
    QNode *rear;
    QNode *front;
} Queue;

void InitQueue(Queue **queue) {
    QNode *p;
    p = (QNode *) malloc(sizeof(QNode));
    p->next = NULL;
    (*queue)->front = p;
    (*queue)->rear = p;
}

void InQueue(Queue *queue, int value) {
    QNode *InElement;
    InElement = (QNode *) malloc(sizeof(QNode));
    InElement->value = value;
    InElement->next = NULL;
    queue->rear->next = InElement;
    queue->rear = InElement;
}

int IsEmpty(Queue *queue) {
    if (queue->rear = queue->front)return 1;
    else return 0;
}

void OutQueue(Queue *queue, int *value) {
    QNode *OutElement;
    if (IsEmpty(queue)) {
        printf("queue is empty");
        return;
    } else {
        OutElement = queue->front->next;
        queue->front->next=OutElement->next;//指针头结点指向下一个结点(pspspspsps)
        *value=OutElement->value;
        free(OutElement);
        if(IsEmpty(queue)){//出队列后如果队列为空则置为空队列（有必要么？）
            queue->front=queue->rear;
        }
    }
}
```
##### 树
`tip`:判定树 每个结点需要查找的次数刚好为该结点所在的层数 查找成功时查找次数不会超过判定树的深度 n个结点的判定树的深度为[LgN]+1 平均查找长度ASL（Average Search Length） ASL=sum(层数*个数)/各层总个数 n(n>=0)个结点构成的有限集合当n=0时称为空树 对于任何一棵非空树（n>0），它具备以下性质 树中会有一个root特殊结点用r表示 其余结点可分为m(m>0)个互不相交的有限集，其中每个集合本身又是一棵树，称为原来树的子树 树的特点： 子树是不相交的 出了根节点外，每隔结点有且仅有一个父结点 一棵N个结点的树有N-1条边（树是保证结点连通的最少边链接方式） 树的一些基本术语： 结点的度：结点的子树个数（满二叉树单个结点的度为2） 树的度：树的所有结点中最大的度数 叶结点：结点度为0的结点 父节点，子节点，兄弟结点 路径和路径长度；祖先节点：沿树根到某一结点路径上的所有节点都是这个结点的祖先结点 子孙结点通； 结点的层次：规定根结点在一层，其他层次随子节点+1 树的深度：树中结点的最大层次就是这棵树的深度

儿子兄弟表示法可以将所有的树转化为二叉树 特殊二叉树： 斜二叉树：只有左儿子或只有右结点 完美二叉树：满二叉树 完全二叉树：结点编号与满二叉树结点编号相同（编号不间断） 一个二叉树第i层的最大节点数为：2(i-1),i>=1 深度为k的二叉树有最大节点总数为2k-1,k>=1; 对于任何非空二叉树T，若N0表示叶子结点的个数，N2是度为2的非叶结点个数，那么两者满足关系：N0=N2+1; (即叶子结点个数-1=度数为2的结点个数)

二叉树的抽象数据类型定义 数据对象集：一个有穷的结点集合 若不为空，则由根节点和其左、右二叉树组成 操作集：判断树是否为空，遍历，创建二叉树

常用的遍历方法有： 先序遍历（根左右），中序遍历（左根右），后序遍历（左右根），层次遍历（从上到下，从左到右）

在二叉树中，我们知道叶结点总数n0与有两个儿子的结点总数n2之间的关系是：n0=n2+1. 那么类似关系是否可以推广到m叉树中？也就是，如果在m叉树中，叶结点总数是n0， 有一个儿子的结点总数是n1，有2个儿子的结点总数是n2，有3个儿子的结点总数是n3， ...。那么，ni之间存在什么关系？ 完全二叉树，非根节点的父节点序号是[i/2] 结点的左孩子结点序号是2i，若2i<=n，否则没有左孩子结点 结点的右孩子结点序号是2i+1，（若2i+1<=n,否则没有右孩子）

```
 typedef struct BT{
     int value;
     struct BT *leftchild;
     struct BT *rightchild;
 }BinTree;
 //二叉树的每个结点遍历都会遇到三次，第一次遇到就打印的为先序遍历，第二次遇到就打印的为中序遍历，第三次遇到就打印的为后序遍历
 //先序遍历(递归遍历)
 void PreOrderTraversal(BinTree *BT){
     if(BT){
     if(!BT->leftchild&&!BT->rightchild)
         printf("%d\n",BT->value);
         PreOrderTraversal(BT->leftchild);
         PreOrderTraversal(BT->rightchild);
     }
 }
 //中序遍历(递归遍历)
 void InOrderTraversal(BinTree *BT){
     if(BT){
     if(!BT->leftchild&&!BT->rightchild)
         InOrderTraversal(BT->leftchild);
         printf("%d\n",BT->value);
         InOrderTraversal(BT->rightchild);
     }
 }
 //后序遍历(递归遍历)
 void PostOrderTraversal(BinTree *BT){
     if(BT){
     if(!BT->leftchild&&!BT->rightchild)
         PostOrderTraversal(BT->leftchild);
         PostOrderTraversal(BT->rightchild);
         printf("%d\n",BT->value);
     }
 }
 //二叉树遍历的本质是将二维序列转换为一维序列
 //使用队列进行二叉树的层级访问（遍历根节点，将左右儿子节点入队列）
 void LevelOrderTraversal(BinTree BT){
     Queue *queue;
     BinTree *T;
     queue=CreateQueue();
     AddQueue(queue,BT);
     while(!IsEmptyQueue(queue)){
         T=DeleteQueue(queue);
         printf("%d\n",T->value);
         if(T->leftchild)AddQueue(queue,T->leftchild);
         if(T->rightchild)AddQueue(queue,T->rightchild);
     }
 }
 //给定前中序遍历结果或中后序遍历结果可以唯一确定一棵二叉树，给定前后序遍历结果不能唯一确定二叉树
 //非递归实现（中序遍历）
 void InOrderTraversal(BinTree *BT){
     BinTree *T=BT;
     LinkedStack *stack=CreateLinkedStack();//创建并初始化堆栈
     while(T||!isEmpty(stack)){
         while(T){//一直向左将沿途结点压入堆栈
             Push(stack,T);
             T=T->leftchild;//转向左子树
         }
         if(!isEmpty(stack)){
             T=Pop(stack);//结点弹出堆栈
             printf("%5d",T->value);//打印结点
             T=T->rightchild;//转向右子树
         }
     }
 }
 //非递归实现（先序遍历）
 void PreOrderTraversal(BinTree *BT){
     BinTree *T=BT;
     LinkedStack *stack=CreateLinkedStack();//创建并初始化堆栈
     while(T||!isEmpty(stack)){
         while(T){//一直向左将沿途结点压入堆栈
             printf("%5d",T->value);//打印结点
             Push(stack,T);
             T=T->leftchild;//转向左子树
         }
         if(!isEmpty(stack)){
             T=Pop(stack);//结点弹出堆栈
             T=T->rightchild;//转向右子树
         }
     }
 }
二叉搜索树：BST(binary search tree)
也称二叉排序树或二叉查找树 二叉搜索树条件 1.非空左子树的所有键值小于其根节点的键值 2.非空右子树的所有键值大于其根节点的键值 3.左，右子树都是二叉搜索树

//递归方式实现
Position Find(BinTree *binTree,int result){
    if(!binTree)return NULL;
    if(result>binTree->value)return Find(binTree->rightchild,result);
    else if(result<binTree->value)return Find(binTree,result);
    else return binTree;//查找成功，返回结点地址(return尾递归)
}
//非递归方式实现
Position IterFind(BinTree *binTree,int value){
    while(binTree){
        if(result>binTree->value)
            binTree=binTree->rightchild;
        else if(result<binTree->value)
            binTree=binTree->leftchild;
        else 
            return binTree;
    }
    return NULL;
}
//寻找最小值
Position FindMin(BinTree *binTree){
    if(!binTree)return NULL;
    else if(!binTree->leftchild)
        return binTree;
    else
        return FindMin(binTree->leftchild);
}
//寻找最大值
Position FindMax(BinTree *binTree){
    if(binTree){
        while(binTree->rightchild)
            binTree=binTree->rightchild;
    }
    return binTree;
}
//结点插入
BinTree * Insert(BinTree *binTree, int value) {
    if(!binTree){
        binTree=malloc(sizeof(BinTree));
        binTree->value=value;
        binTree->leftchild=binTree->rightchild=NULL;
    }else{
        if(value<binTree->value)
            binTree->leftchild=Insert(binTree->leftchild,value);
        else if(value>binTree->value)
            binTree->rightchild=Insert(binTree->rightchild,value);
    }
    return binTree;
}
//删除结点
BinTree *Delete(BinTree *binTree,int value){
    (Position)BinTree *Temp;
    if(!binTree)printf("要删除的元素未找到");
        //左子树递归删除
    else if(value<binTree->value)binTree->leftchild=Delete(binTree,value);
        //右子树递归删除
    else if(value>binTree->value)binTree->rightchild=Delete(binTree->rightchild,value);
    else //找到要删除的结点
        if(binTree->leftchild&&binTree->rightchild){//被删除结点有左右量子子节点
            Temp=FindMin(binTree->rightchild);//在右子树中招最小的元素填充删除结点
            binTree->value=Temp->value;
            binTree->rightchild=Delete(binTree->rightchild,binTree->value);
        }else{//被删除的结点有一个或无子结点
            Temp=binTree;
            if(!binTree->leftchild)binTree=binTree->rightchild;
            else if(!binTree->rightchild)binTree=binTree->leftchild;
            free(Temp);
        }
    return binTree;
}
```

##### 平衡二叉树(Balanced Binary Tree)(AVL树)
- 空树或任一结点左右子树高度差不超不过1|BF(T)|<=1

- 平衡因子(Balance Factor 简称BF：BF(T)=Hl-Hr)

- 其中hl和hr分别为T的左右子树高度

- 高度=层数-1

- 完全二叉树高度为log2N(平衡二叉树)

- Nh是高度为h的平衡二叉树的最小结点树
 Nh=F(h+2)-1

```
 #define MaxData 10000
 typedef struct HeapStruct{
     int *value;//存储对元素的数组
     int length;//堆的当前元素个数
     int capacity;//堆的最大容量
 }Heap;
```
 
##### 优先队列（PriorityQueue）

`tip`:取出元素的先后顺序是按照元素的优先权（关键字）大小，而不是元素进入队列的先后顺序

最大堆和最小堆都必须满足完全二叉树（切根节点最大或最小） 最大堆的建立 建立最大堆：将已经存在的N个元素按最大堆的要求存放在要给一维数组中 方法一：通过插入操作，将N个元素一个个相继插入到一个初始为空的堆中去，其时间代价最大为O（NlogN） 方法二：在线性时间复杂度下建立最大堆 (1)将N个元素按输入顺序存入，先满足完全二叉树的结构特性 (2)调整各结点位置以满足最大堆的有序特性 建堆时，最坏的情况下需要挪动的元素次数等于树中各节点的高度和

对由同样n个整数构成的二叉搜索树（查找树）和最小堆：有以下结论 二叉搜索树高度大于等于最小堆高度 对该二叉搜索树进行中序遍历可得到从小到大的序列 从最小堆根节点到起任何叶结点的路径上的结点值构成从小到大的序列

```
Heap * Create(int MaxSize){
   Heap *heap=malloc(sizeof(Heap));
   heap->value=malloc((MaxSize+1)*sizeof(int));
   heap->length=0;
   heap->capacity=MaxSize;
   heap->value[0]=MaxData;//定义哨兵，便于操作
   return heap;
}
void Insert(Heap *heap,int value){
   int i;
   if(IsFull(heap)){
       printf("最大堆已经满了");
       return;
   }
   i=++heap->length;
   for(;heap->value[i/2]<value;i/=2)
       heap->value[i]=heap->value[i/2];
   heap->value[i]=value;
}
int DeleteMax(Heap *heap){
   int parent,child;
   int maxValue,temp;
   if(IsEmpty(heap)){
       printf("最大堆已空");
       return 0;
   }
   maxValue=heap->value[1];
   //用最大堆中最后一个元素从根节点开始过滤下层结点
   temp=heap->value[heap->length--];
   for(parent=1;parent*2<=heap->length;parent=child){
       child=parent*2;
       //左儿子和右儿子节点比较取较大者
       if((child!=heap->length)&&(heap->value[child]<heap->value[child+1]))
           child++;
       if(temp>=heap->value[child])break;
       else
           heap->value[parent]=heap->value[child];
   }
   heap->value[parent]=temp;
   return maxValue;
}
int IsEmpty(Heap *heap){
   return heap->length==0;
}
int IsFull(Heap *heap){
   return heap->length==heap->capacity;
}

typedef struct TreeNode{
   int weight;
   struct TreeNode *left,*right;
}HuffmanTree;
```
##### 哈夫曼树（HuffmanTree）
`tip`:查找效率，查找次数乘查找概率 带权路径长度（WPL）：设二叉树有n个叶子结点，每隔叶子结点带有权值Wk，从根节点到每隔叶子结点的长度是Lk，则每隔叶子结点的带全路径长度之和WPL=（nEk=1）WkLk 最优二叉树或哈夫曼树：WPL最小的二叉树 哈夫曼树的特点 没有度为1的结点 n个叶子结点的HuffmanTree有2n-1个结点 HuffmanTree的任意非叶结点的左右子树交换后仍是HuffmanTree 对于一组权值，可能有不同构的两棵HuffmanTree

```
HuffmanTree *Huffman(Heap *heap){
    //假设heap->length权值已经存在heap->value[]->weight里面
    int i;HuffmanTree *huffmanTree;
    BuildHeap(heap);//将heap->value[]按权值调整为最小堆
    for(i=1;i<heap->length;i++){
        huffmanTree=malloc(sizeof(HuffmanTree));//建立新结点
        huffmanTree->left=DeleteMin(heap);//从最小堆中删除一个结点，作为新huffmanTree的左子结点
        huffmanTree->right=DeleteMin(heap);//从最小堆中删除一个结点，作为新huffmanTree的右子结点
        huffmanTree->weight=huffmanTree->weight+huffmanTree->right->weight;//计算新// 权值
        Insert(heap,huffmanTree);
    }
    huffmanTree=DeleteMin(heap);
    return huffmanTree;
}
```
*二叉树用于编码*
- 当被编码字母全部在二叉树的叶子结点的时候（即，编码字母不会出现在有子节点的结点中）便可以保证字符编码没有二义性
