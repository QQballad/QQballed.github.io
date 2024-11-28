# 数据结构部分

## 线性表

### 顺序表

线性表的顺序**存储**结构，逻辑上相邻的数据元素，其物理位置也是相邻的。线性表中第 $i + 1$ 个数据元素的存储位置 $LOC(a_{i+1})$ 和第 $i$ 个数据元素的存储位置 $LOC(a_i)$ 之间满足下列关系：
$$
LOC(a_{i+1}) = LOC(a_i) + \text{sizeof}(a_i)
$$

一般来说，线性表的第 $i$ 个数据元素 $a_i$ 的存储位置为：
$$
LOC(a_i) = LOC(a_1) + (i-1) \times \text{sizeof}(a_i)
$$

只要确定了存储线性表的起始位置，线性表中任一数据元素都可随机存取，所以顺序表是一种随机**存取**结构。

???success "顺序表的功能实现"
    === "定义"
        ```cpp linenums="1"
        #define MAXSIZE 100
        typedef struct 
        {
            ElemType *elem; // (1)!
            int length;
        } SqList;
        ```

        1.  如果此处直接定义为 `ElemType data[MAXSIZE]`，则初始化时无需为 `elem` 数组分配空间。
    === "初始化"
        
        **算法步骤**        

        1. 为顺序表 `L` 动态分配一个预定义大小的数组空间，使 `elem` 指向这段空间的基地址；
        2. 将表长置为 $0$；

        ```cpp linenums="1"
        Status InitList(SqList &L) // (1)!
        {
            L.elem = new ElemType[MAXSIZE]; // (2)!
            if (!L.elem) return OVERFLOW;
            L.length = 0;
            return OK;
        }
        ```

        1.  如果传入的参数需要被修改，则需要加上 `&` 符号。
        2.  如果定义为 `ElemType data[MAXSIZE]`，则无需这一步。
    === "表长"
        
        ```cpp linenums="1"
        int ListLength(SqList L)
        {
            return L.length;
        }
        ```

    === "判空"
        
        ```cpp linenums="1"
        bool ListEmpty(SqList L)
        {
            return L.length == 0;
        }
        ```
    === "取值"
        
        **算法步骤**

        1. 判断指定位置序号 $i$ 是否合理 $(1 \leq i \leq$ `L.length`$)$，若不合理，则返回 `ERROR`；
        2. 若 $i$ 值合理，则将第 $i$ 个数据元素 `elem[i-1]`[^1] 赋给参数 `e`，通过 `e` 返回第 $i$ 个数据元素的传值。
        
        [^1]: 逻辑上顺序表下标从 $1$ 开始，实际存储位置从 $0$ 开始。

        ```cpp linenums="1"
        Status GetElem(SqList L, int i, ElemType &e)
        {
            if (i < 1 || i > L.length) return ERROR;
            e = L.elem[i - 1];
            return OK;
        }
        ```

        显然，时间复杂度为 $O(1)$。
    === "查找"
        
        **算法步骤**

        1. 从第一个元素起，依次将其值和 `e` 相比较，若找到值与 `e` 相等的元素 `elem[i]`，则返回其序号 $i+1$；
        2. 若遍历完整个表，仍未找到 `e`，则返回 `0`。
        
        ```cpp linenums="1"
        int LocateElem(SqList L, ElemType e)
        {
            for (int i = 0; i < L.length; i++)
                if (L.elem[i] == e) return i + 1;
            return 0;
        }
        ```

        为确定元素在顺序表中的位置，需和给定值进行比较的数据元素个数的期望值称为查找算法在查找成功时的**平均查找长度**（Average Search Length，**ASL**）。

        假设 $p_i$ 是查找第 $i$ 个元素的概率，$C_i$ 为找到表中其关键字与给定值相等的第 $i$ 个记录时，和给定值已进行过比较的关键字个数，则在长度为 $n$ 的线性表中，查找成功时的平均查找长度为：
        $$
        ASL = \sum_{i=1}^{n} p_i C_i
        $$

        一般情况下，$C_i=i$，假设每个元素的概率相等，即：
        $$
        p_i = \dfrac{1}{n}
        $$

        则有：
        $$
        ASL = \dfrac{1}{n} \sum_{i=1}^{n} i = \dfrac{n(n+1)}{2}
        $$

        故顺序表的查找操作的平均时间复杂度为 $O(n)$。

    === "插入"

        **算法步骤**

        1. 判断指定位置序号 $i$ 是否合理 $(1 \leq i \leq$ `L.length` $+1)$，若不合理，则返回 `ERROR`；
        2. 判断表是否已满，若满，则返回 `OVERFLOW`；
        3. 将下标范围 $[i,$ `L.length`$)$ 依次向后移动一位，空出位置 $i$，若 $i =$ `L.length` $+1$，则无需移动；
        4. 将新元素 `e` 存入位置 $i$；
        5. 表长加 $1$；
        
        ```cpp linenums="1"
        Status ListInsert(SqList &L, int i, ElemType e)
        {
            if (i < 1 || i > L.length + 1) return ERROR;
            if (L.length == MAXSIZE) return OVERFLOW;
            for (int j = L.length - 1; j >= i - 1; j--)
                L.elem[j + 1] = L.elem[j];
            L.elem[i - 1] = e;
            ++L.length;
            return OK;
        }
        ```

        假设 $p_i$ 是在第 $i$ 个元素之前插入一个元素的概率，$E_{\text{ins}}$ 为在长度为 $n$ 的线性表中插入一个元素时所需移动元素次数的期望值（平均次数），则有：
        $$
        E_{\text{ins}} = \sum_{i=1}^{n+1} p_i(n-i+1)
        $$

        假设每个元素的概率相等，即：
        $$
        p_i = \dfrac{1}{n+1}
        $$

        则有：
        $$
        E_{\text{ins}} = \dfrac{1}{n+1} \sum_{i=1}^{n+1} (n-i+1) = \dfrac{n}{2}
        $$  

        故顺序表的插入操作的平均时间复杂度为 $O(n)$。
    === "删除"

        **算法步骤**

        1. 判断指定位置序号 $i$ 是否合理 $(1 \leq i \leq$ `L.length`$)$，若不合理，则返回 `ERROR`；
        2. 将下标范围 $[i+1,$ `L.length`$)$ 依次向前移动一位，若 $i =$ `L.length`，则无需移动；
        3. 表长减 $1$；
        
        ```cpp linenums="1"
        Status ListDelete(SqList &L, int i)
        {
            if (i < 1 || i > L.length) return ERROR;
            for (int j = i; j < L.length; j++)
                L.elem[j - 1] = L.elem[j];
            --L.length;
            return OK;
        }
        ```

        假设 $p_i$ 是删除第 $i$ 个元素的概率，$E_{\text{del}}$ 为在长度为 $n$ 的线性表中删除一个元素时所需移动元素次数的期望值（平均次数），则有：
        $$
        E_{\text{del}} = \sum_{i=1}^{n} p_i(n-i)
        $$

        假设每个元素的概率相等，即：
        $$
        p_i = \dfrac{1}{n}
        $$

        则有：
        $$
        E_{\text{del}} = \dfrac{1}{n} \sum_{i=1}^{n} (n-i) = \dfrac{n-1}{2}
        $$  

        故顺序表的删除操作的平均时间复杂度为 $O(n)$。
    
???example "往年真题"

    ???question "2012 A"
        === "题目描述"
            **判断题，正确的打 √，错误的打 × 并改正**

            在顺序表中删除一个元素的时间复杂度为 $O(1)$。
        === "答案"
            **×**

            在顺序表中删除一个元素的时间复杂度为 $\boldsymbol{\underline{O(n)}}$。
    
    ???question "2013 A"
        === "题目描述"
            **简答题**

            阅读顺序表的算法，指出算法的功能并分析算法的时间复杂度。

            ```cpp linenums="1"
            int TTT(SqList *&L, int i, int &e)
            {
                int j;
                if (i < 1 || i > L->length) return 0;
                i--; e = L->data[i];
                for (j = i; j < L->length - 1; j++) L->data[j] = L->data[j + 1];
                L->length--; return 1;
            }
            ```
        === "答案"
            功能：删除指定位置的元素，通过 `e` 返回被删除的元素的值。返回值为是否删除成功。

            时间复杂度：删除操作的时间复杂度为 $O(n)$。

    ???question "2013 B"
        === "题目描述"
            **简答题**

            阅读顺序表的算法，指出算法的功能并分析算法的时间复杂度。

            ```cpp linenums="1"
            int BBB(SqList *&L, int a[], int n)
            {
                int i;
                L = (SqList*)malloc(sizeof(SqList));
                for (i = 0; i < n; i++) L->data[i] = a[i];
                L->length = n;
            }
            ```
        === "答案"
            功能：将数组 `a` 转化为顺序表 `L`。

            时间复杂度：遍历了一遍数组，所以时间复杂度为 $O(n)$。

    ???question "2014 A"
        === "题目描述"
            **简答题**

            简述顺序存储结构的特点。
        === "答案"
            顺序存储结构的特点是：逻辑上相邻的数据元素，其物理位置也是相邻的。借助元素在存储器中的相对位置来表示数据元素之间的逻辑关系的，通常借助程序设计语言的数组类型来描述。可以随机访存。
    
    ???question "2014 B"
        === "题目描述"
            **算法设计题**

            已知线性顺序表的形式化定义如下：

            ```cpp linenums="1"
            typedef struct
            {
                int data[ListSize];
                int length;
            } Sqlist;
            ```

            请根据该形式化定义设计算法，其功能是在有序顺序表 `L` 的合适位置上插入新数据 `x` 并保持有序且算法时间复杂度为 $O(n)$。要求简述算法思想并编程实现该算法对应的函数 `void func(Sqlist *&L, int x)`。
        === "答案"
            算法思想：

            1. 判断 `L` 是否已满，若满，则返回 `OVERFLOW`；
            2. 找到 `L` 中第一个大于等于 `x` 的元素的位置 $i$；
            3. 将 `L` 中 $[i,$ `L.length`$)$ 及之后的元素依次向后移动一位，空出位置 $i$，若 $i =$ `L.length` $+1$，则无需移动；
            4. 将新数据 `x` 存入位置 $i$；
            5. 表长加 $1$；

            编程实现：

            ```cpp linenums="1"
            void func(Sqlist *&L, int x)
            {
                int i;
                if (L->length == MAXSIZE) return OVERFLOW;
                for (i = 0; i < L->length; i++)
                    if (L->data[i] >= x) break;
                for (int j = L->length - 1; j >= i; j--)
                    L->data[j + 1] = L->data[j];
                L->data[i] = x;
                L->length++;
            }
            ```

    ???question "2017"
        === "题目描述"
            **简答题**

            现有一个线性表 $L = (a, c, a, d, b)$，求 `ListLength(L)`、`ListEmpty(L)`、`GetElem(L, 3, e)`、`LocateElem(L, a)`、`ListInsert(L, 4, f)` 和 `ListDelete(L, 3)` 等基本运算的执行结果。
        === "答案"
            | 基本运算 | 执行结果 |
            | :-: | :-: |
            | `ListLength(L)` | $5$ |
            | `ListEmpty(L)` | `false`$(0)$ |
            | `GetElem(L, 3, e)` | `e = a` |
            | `LocateElem(L, a)` | $1$ |
            | `ListInsert(L, 4, f)` | $L = (a, c, a, f, d, b)$ |
            | `ListDelete(L, 3)` | $L = (a, c, d, b)$ |

    ???question "2017"
        === "题目描述"
            **算法设计题**

            设计一个算法，将 `x` 插入一个有序（从小到大）的线性表（顺序存储结构）的适当位置上，并保持线性表的有序性。

            线性表描述如下：

            ```cpp linenums="1"
            typedef struct
            {
                ElemType data[MaxSize];
                int length;
            } Sqlist;

        === "答案"
            ```cpp linenums="1"
            void InsertSort(Sqlist &L, ElemType x)
            {
                int i;
                for (i = 0; i < L.length; i++)
                    if (x < L.data[i]) break;
                for (int j = L.length - 1; j >= i; j--)
                    L.data[j + 1] = L.data[j];
                L.data[i] = x;
                L.length++;
            }
            ```

    ???question "2018"
        === "题目描述"
            **简答题**

            有一个线性表 $L = (x, y, t, m, k, z, x)$，求 `ListLength(L)`、`ListEmpty(L)`、`GetElem(L, 4, e)`、`LocateElem(L, x)`、`ListInsert(L, 4, z)`、`ListDelete(L, 6)` 等基本运算的执行结果。
        === "答案"
            | 基本运算 | 执行结果 |
            | :-: | :-: |
            | `ListLength(L)` | $7$ |
            | `ListEmpty(L)` | `false`$(0)$ |
            | `GetElem(L, 4, e)` | `e = m` |
            | `LocateElem(L, x)` | $1$ |
            | `ListInsert(L, 4, z)` | $L = (x, y, t, z, m, k, z, x)$ |
            | `ListDelete(L, 6)` | $L = (x, y, t, m, k, x)$ |

    ???question "2020"
        === "题目描述"
            **算法设计题**
            
            用 C 语言写一算法，将顺序表 `L` 中的所有元素逆置，要求算法空间复杂度为 $O(1)$。
        === "答案"
            算法思想：

            1. 定义两个指针 `i` 和 `j`，`i` 指向下标 $0$，`j` 指向 `L.length` $- 1$；
            2. 当 `i <= j` 时，交换 `L.data[i]` 和 `L.data[j]`；
            3. 同时 `i` 和 `j` 向前移动一位；
            4. 直到 `i > j` 时，算法结束。

            编程实现：

            ```cpp linenums="1"
            void reverseList(Sqlist *&L)
            {
                for (int i = 0, j = L->length - 1; i <= j; i++, j--)
                {
                    int temp = L->data[i];
                    L->data[i] = L->data[j];
                    L->data[j] = temp;
                }
            }
            ```
            
            只遍历了一次数组，时间复杂度为 $O(n)$，只使用了常数额外空间，空间复杂度为 $O(1)$。

### 链表

线性表链式存储结构的特点是：用一组任意的存储单元存储线性表的数据元素（这组存储单元可以是连续的，也可以是不连续的）。

#### 单链表

???success "单链表的功能实现"
    
    === "定义"
        ```cpp linenums="1"
        typedef struct Node
        {
            ElemType data;
            struct Node *next;
        } LNode *LinkList;
        ```

    === "初始化"
        **算法步骤**

        1. 生成新节点作为头节点，用头指针 `L` 指向该节点[^2]；
        2. 头节点的指针域置空。

        [^2]: 一般链表头节点为空节点，不存放数据，头节点的下一节点为首元节点。

        ```cpp linenums="1"
        Status InitList(LinkList &L)
        {
            L = new LNode; // (1)!
            L->next = NULL;
            return OK;
        }
        ```

        1.  此处使用 C 语言为：`L = (LNode*)malloc(sizeof(LNode));`。

    === "取值"
        **算法步骤**

        1. 用指针 `p` 指向首元节点，用 $j$ 做计数器初值赋为 $1$；
        2. 从首元节点开始依次顺着链域 `next` 向下访问，只要指向当前节点的指针 `p` 不为空，并且没有到达序号为 $i$ 的节点，则循环执行以下操作：
            - `p` 指向下一个节点；
            - 计数器 $j$ 加 $1$；
        3. 退出循环时，若 `p` 为空，或者计数器 $j > i$，说明指定的序号 $i$ 不合法（$i > n$ 或 $i \leq 0$）[^3]，取值失败返回 `ERROR`；否则取值成功，此时 $j=i$，`p` 所指的节点就是要找的第 $i$ 个节点，将其数据值赋给 `e`，返回 `OK`。
        
        [^3]: 对于单链表，$n$ 为链表长度。

        ```cpp linenums="1"
        Status GetElem(LinkList L, int i, ElemType &e)
        {
            LNode *p = L->next;
            int j = 1;
            while (p && j < i)
            {
                p = p->next;
                j++;
            }
            if (!p || j > i) return ERROR;
            e = p->data;
            return OK;
        }
        ```

        该算法的基本操作是比较 $j$ 和 $i$ 的大小并后移指针 `p`，`while` 循环体中的语句频度与位置 $i$ 有关。若 $1 \leq i \leq n$，则频度为 $i − 1$，一定能取值成功；若 $i > n$，则频度为 $n$，取值失败。因此算法的最坏时间复杂度为 $O(n)$。

        假设每个位置上元素的取值概率相等，即：
        $$
        p_i = \dfrac 1 n
        $$
        则：
        $$
        ASL = \dfrac 1 n \sum_{i=1}^n(i-1)=\dfrac{n-1}2
        $$

        故单链表的取值操作的平均时间复杂度为 $O(n)$。

    === "查找"
        **算法步骤**
        
        1. 用指针 `p` 指向首元节点；
        2. 从首元节点开始依次顺着链域 `next` 向下查找，只要指向当前节点的指针 `p` 不为空，并且 `p` 所指节点的数据域不等于给定值 `e`，则循环执行以下操作：`p` 指向下一个节点；
        3. 返回 `p`。若查找成功，`p` 此时指向节点的地址值，若查找失败，则 `p` 的值为空。

        ```cpp linenums="1"
        LNode *LocateElem(LinkList L, ElemType e)
        {
            LNode *p = L->next;
            while (p && p->data!= e) p = p->next;
            return p;
        }
        ```

        该算法的执行时间与待查找的值 `e` 相关，其平均时间复杂度分析类似于单链表的取值操作，也为 $O(n)$。
    === "插入"
        **算法步骤**

        1. 查找节点 $a_{i−1}$ 并由指针 `p` 指向该节点；
        2. 生成一个新节点 `*s`；
        3. 将新节点 `*s` 的数据域置为 `e`；
        4. 将新节点 `*s` 的指针域指向节点 $a_i$；
        5. 将节点 `*p` 的指针域指向新节点 `*s`。

        ```cpp linenums="1"
        Status ListInsert(LinkList &L, int i, ElemType e)
        {
            LNode *p = L->next;
            int j = 0;
            while (p && j < i - 1)
            {
                p = p->next;
                j++;
            }
            if (!p || j > i - 1) return ERROR;
            LNode *s = new LNode;
            s->data = e;
            s->next = p->next;
            p->next = s;
            return OK;
        }
        ```

        该算法的执行时间与待插入位置 $i$ 相关，其平均时间复杂度分析类似于单链表的取值操作，也为 $O(n)$。

    === "删除"
        **算法步骤**

        1. 查找节点 $a_{i−1}$ 并由指针 `p` 指向该节点；
        2. 临时保存待删除节点 $a_i$ 的地址在 `q` 中，以备释放；
        3. 将节点 `*p` 的指针域指向 $a_i$ 的直接后继节点；
        4. 释放节点 $a_i$ 的空间。

        ```cpp linenums="1"
        Status ListDelete(LinkList &L, int i)
        {
            LNode *p = L;
            int j = 0;
            while (p->next && j < i - 1)
            {
                p = p->next;
                j++;
            }
            if (!p->next || j > i - 1) return ERROR;
            LNode *q = p->next;
            p->next = q->next;
            delete q; // (1)!
            return OK;
        }
        ```

        1.   此处使用 C 语言为：`free(q);`。

        该算法的执行时间与待删除位置 $i$ 相关，其平均时间复杂度分析类似于单链表的取值操作，也为 $O(n)$。

    === "创建单链表"
        
        === "前插法（头插法）"

            前插法是通过将新节点逐个插入链表的头部（头节点之后）来创建链表，每次申请一个新节点，读入相应的数据元素值，然后将新节点插入到头节点之后。因为每次插入在链表的头部，所以输入顺序和线性表中的逻辑顺序是相反的。
            
            **算法步骤**

            1. 创建一个只有头节点的空链表。
            2. 根据待创建链表包括的元素个数 $n$，循环 $n$ 次执行以下操作：
                - 生成一个新节点 `*p`；
                - 输入元素值赋给新节点 `*p` 的数据域；
                - 将新节点 `*p` 插入到头节点之后。

            ```cpp linenums="1"
            void CreateList_H(LinkList &L, int n)
            {
                L = new LNode;
                L->next = NULL;
                for (int i = 0; i < n; i++)
                {
                    LNode *p = new LNode;
                    cin >> p->data;
                    p->next = L->next;
                    L->next = p;
                }
            }
            ```

            该算法的执行时间与待创建元素个数 $n$ 相关，其平均时间复杂度分析为 $O(n)$。

        === "后插法（尾插法）"

            后插法是通过将新节点逐个插入链表的尾部来创建链表。同前插法一样，每次申请一个新节点，读入相应的数据元素值。不同的是，为了使新节点能够插入表尾，需要增加一个尾指针 `r` 指向链表的尾节点。

            **算法步骤**

            1. 创建一个只有头节点的空链表。
            2. 尾指针 `r` 初始化，指向头节点。
            3. 根据创建链表包括的元素个数 $n$，循环 $n$ 次执行以下操作：
                - 生成一个新节点 `*p`；
                - 输入元素值赋给新节点 `*p` 的数据域；
                - 将新节点 `*p` 插入尾节点 `*r` 之后；
                - 尾指针 `r` 指向新的尾节点 *p`。

            ```cpp linenums="1"
            void CreateList_T(LinkList &L, int n)
            {
                L = new LNode;
                L->next = NULL;
                LNode *r = L;
                for (int i = 0; i < n; i++)
                {
                    LNode *p = new LNode;
                    cin >> p->data;
                    p->next = NULL;
                    r->next = p;
                    r = p;
                }
            }
            ```

            该算法的执行时间与待创建元素个数 $n$ 相关，其平均时间复杂度分析为 $O(n)$。

#### 双向链表

在单链表中，查找直接后继的执行时间为 $O(1)$，而查找直接前驱的执行时间为 $O(n)$。为克服单链表这种单向性的缺点，可利用**双向链表**（Double Linked List）。

???success "双向链表的功能实现"
    
    === "定义"
        ```cpp linenums="1"
        typedef struct Node
        {
            ElemType data;
            struct Node *prior;
            struct Node *next;
        } DuLNode, *DuLinkList;
        ```

    === "插入"
        ```cpp linenums="1"
        Status ListInsert_DuL(DuLinkList &L, int i, ElemType e)
        {
            if (!(p=GetElem_DuL(L, i))) return ERROR;
            DuLNode *s = new DuLNode;
            s->data = e;
            s->prior = p->prior;
            p->prior->next = s;
            s->next = p;
            p->prior = s;
            return OK;
        }
        ```

        该算法的执行时间与待插入位置 $i$ 相关，其平均时间复杂度分析为 $O(n)$。

    === "删除"
        ```cpp linenums="1"
        Status ListDelete_DuL(DuLinkList &L, int i)
        {
            if (!(p=GetElem_DuL(L, i))) return ERROR;
            p->prior->next = p->next;
            p->next->prior = p->prior;
            delete p;
            return OK;
        }
        ```

        该算法的执行时间与待删除位置 $i$ 相关，其平均时间复杂度分析为 $O(n)$。

### 顺序表与链表的比较

#### 时间性能比较

| 操作 | 顺序表 | 链表 |
| :-: | :-: | :-: |
| 存取元素 | $\boldsymbol{O(1)}$ | $O(n)$ |
| 插入元素 | $O(n)$ | $\boldsymbol{O(1)}$ |
| 删除元素 | $O(n)$ | $\boldsymbol{O(1)}$ |

因此，若线性表的主要操作是和元素位置紧密相关的一类取值操作，很少做插入或删除时，宜采用顺序表作为存储结构；而对于频繁进行插入或删除操作的线性表，宜采用链表作为存储结构。

#### 空间性能比较

| 结构 | 顺序表 | 链表 |
| :-: | :-: | :-: |
| 存储空间 | 连续存储 | 离散存储 |
| 内存分配 | 需预先分配 | 动态分配 |
| 存储密度[^4] | **高** | 低 |

[^4]: 存储密度是指数据元素本身所占用的存储量和整个节点结构所占用的存储量之比，即：$\text{存储密度}=\dfrac {\text{数据元素本身占用的存储量}} {\text{节点结构占用的存储量}}$

因此，当线性表的长度变化较大，难以预估存储规模时，宜采用链表作为存储结构；当线性表的长度变化不大，易于事先确定其大小时，为了节约存储空间，宜采用顺序表作为存储结构。

???example "往年真题"
    ???question "2012 A"
        === "题目描述"
            **判断题，正确的打 √，错误的打 × 并改正**

            链式存储结构是一种顺序存取结构。
        === "答案"
            **√**

    ???question "2012 A"
        === "题目描述"
            **算法设计题**

            设计一个函数 `ListInsert` 实现在带头节点的有序单链表上实现插入新元素并保持有序。单链表的节点形式化定义为：

            ```cpp linenums="1"
            typedef struct node
            {
                int data;
                struct node *next;
            } linklist;
            ```

            `ListInsert` 函数的声明为：

            ```cpp linenums="1"
            void ListInsert(linklist *L, int x);
            ```

        === "答案"
            ```cpp linenums="1"
            void ListInsert(linklist *L, int x)
            { 
                linklist *q = (linklist*)malloc(sizeof(linklist)), *p = L;
                q->next = NULL;
                q->data = x;
                while (p->next && p->next->data <= x)
                    p = p->next;
                q->next = p->next;
                p->next = q;
            }
            ```
    ???question "2013 A"
        === "题目描述"
            **简答题**

            阅读单链表的算法，指出算法的功能并分析该算法的效率能否提高到 $O(1)$。

            ```cpp linenums="1"
            int AAA(LinkList *L)
            {
                LinkList *p = L;
                int i = 0;
                while (p->next != NULL)
                {
                    i++;
                    p = p->next;
                }
                return i;
            }
            ```
        === "答案"
            该算法功能为计算单链表的长度，可以在插入时将长度保存到头节点中，从而提高效率到 $O(1)$。

    ???question "2013 B"
        === "题目描述"
            **简答题**

            阅读单链表的算法，指出算法的功能并分析算法的时间复杂度。

            ```cpp linenums="1"
            int TTT(LinkList *L, int e)
            {
                LinkList *p = L->next;
                int n = 1;
                while (p != NULL && p->data != e)
                {
                    p = p->next;
                    n++;
                }
                if (p == NULL) return 0;
                else return n;
            }
            ```
        === "答案"
            该算法功能为查找单链表中第 $e$ 个元素的位置，时间复杂度为 $O(n)$。

    ???question "2013 B"
        === "题目描述"
            **算法设计题**

            已知长度为 $n$ 的线性表，数据为整型，设计算法删除线性表中所有值为 `item` 的数据元素，要求：

            **(1)** 设计合适的存储结构，描述算法的实现思路，并分析算法的时间复杂度。

            **(2)** 用 C 语言实现该算法。

        === "答案"
            要频繁删除，可以使用链表，定义为：

            ```cpp linenums="1"
            typedef struct node
            {
                int data;
                struct node *next;
            } Linklist;
            ```

            **算法思路**

            遍历链表，若当前节点的下一节点不为空且数据等于 `item`，则删除该节点；否则，移动到下一个节点。

            ```cpp linenums="1"
            void DeleteItem(Linklist *L, int item)
            {
                if (L == NULL) return;
                Linklist *p = L, *q;
                while (p->next != NULL)
                {
                    if (p->data == item)
                    {
                        q = p->next;
                        p->next = q->next;
                        free(q);
                    }
                    else p = p->next;
                }
            }
            ```

            **时间复杂度分析**

            遍历链表一次，单点删除次数至多 $n$ 次，时间复杂度为 $O(n)$。

    ???question "2014 A"
        === "题目描述"
            **算法设计题**

            已知线性链表的形式化定义如下：

            ```cpp linenums="1"
            typedef struct node
            {
                int data;
                struct node *next;
            } listnode;
            typedef listnode *linklist;
            ```

            请根据该形式化定义设计算法，其功能是在递增有序单链表 $L$ 的合适位置上插入新数据 `x` 并保持递增有序且算法时间复杂度为 $O(n)$。要求简述算法思想并编程实现该算法对应的函数 `void func(linklist L, int x)`。

        === "答案"
            **算法思路**

            遍历链表，若当前节点的下一节点不为空且数据大于等于 `x`，则插入新节点；否则，移动到下一个节点。

            ```cpp linenums="1"
            void func(linklist L, int x)
            {
                if (L == NULL) return;
                linklist p = L, q;
                while (p->next != NULL && p->next->data < x)
                    p = p->next;
                q = (linklist)malloc(sizeof(listnode));
                q->data = x;
                q->next = p->next;
                p->next = q;
            }
            ```

            时间复杂度分析：遍历链表一次，单点插入次数至多一次，时间复杂度为 $O(n)$。

    ???question "2015 A"
        === "题目描述"
            **算法设计题**

            单链表的定义如下：

            ```cpp linenums="1"
            typedef struct LNode
            {
                int data;
                struct LNode *next;
            } LinkList;
            ```

            有一个递增的带头节点的单链表（允许出现重复的值），设计一个算法删除值相同的节点（相同值只保留一个）。要求：

            **(1)** 描述算法的思路；

            **(2)** 补充完成算法 `void Dels(LinkList *Head) {}`。

        === "答案"
            **算法思路**

            遍历链表，若当前节点的下一节点不为空且数据等于当前节点的数据，则删除该节点；否则，移动到下一个节点。

            ```cpp linenums="1"
            void Dels(LinkList *Head)
            {
                if (Head == NULL) return;
                LinkList *p = Head, *q;
                while (p->next != NULL)
                {
                    if (p->data == p->next->data)
                    {
                        q = p->next;
                        p->next = q->next;
                        free(q);
                    }
                    else p = p->next;
                }
            }
            ```

            时间复杂度分析：遍历链表一次，单点删除次数至多 $n-1$ 次，时间复杂度为 $O(n)$。

    ???question "2016 B / 2018 / 2021"
        === "题目描述"
            **简答题**

            （2016 B / 2018）线性表的两种存储结构各有哪些优缺点？（2021）各有何特点？

        === "答案"
            
            | | 顺序表 | 链表 |
            | :-: | :-- | :-- |
            | 特点 | 逻辑上相邻的数据元素，其物理位置也是相邻的。| 用一组任意的存储单元存储线性表的数据元素（这组存储单元可以是连续的，也可以是不连续的）。|
            | 优点 | 存取元素时间复杂度为 $O(1)$，适合频繁访问的元素；<br>存储密度高，适合大量数据的存储；| 插入和删除操作只需修改指针，时间复杂度为 $O(1)$；<br>存储密度低，适合存储离散数据。|
            | 缺点 | 插入和删除操作需要移动大量元素，时间复杂度为 $O(n)$；<br>占用大量内存，不适合存储离散数据。| 存取元素时间复杂度为 $O(n)$，不适合频繁访问的元素；<br>占用内存空间多，不适合大量数据的存储。|

    ???question "2016 B"
        === "题目描述"
            **简答题**

            单链表的定义如下：

            ```cpp linenums="1"
            typedef struct LNode
            {
                ElemType data;
                struct LNode *next;
            } LinkList;
            ```

            阅读下面的代码：

            ```cpp linenums="1" hl_lines="8 9"
            LinkList mynote(LinkList L) // L 是不带头节点的单链表的头指针
            {
                if (L && L->next)
                {
                    LinkList q = L;
                    L = L->next;
                    LinkList p = L;
            S1:     while (p->next) p = p->next;
            S2:     p->next = q; q->next = NULL;
                }
                return L;
            }
            ```

            请回答下列问题：

            **(1)** 说明语句 $S_1$ 的功能；

            **(2)** 说明语句组 $S_2$ 的功能；

            **(3)** 设链表表示的线性表为 $(a_1, a_2, \cdots, a_n)$，写出算法执行后的返回值所表示的线性表。

        === "答案"
            **(1)** 语句 $S_1$ 用于找到单链表的最后一个节点。

            **(2)** 语句组 $S_2$ 用于将原第一个节点变成最后一个节点。

            **(3)** 算法执行后的返回值所表示的线性表为 $(a_2, a_3, \cdots, a_n, a_1)$。

    ???question "2016 B / 2019"
        === "题目描述"
            **算法设计题**

            设计算法统计出单链表 `HL` 中节点的值等于给定值 `x` 的节点数，单链表的定义如下：

            ```cpp linenums="1"
            typedef struct LNode
            {
                ElemType data;
                struct LNode *next;
            } LinkNode;
            ```

        === "答案"
            ```cpp linenums="1"
            int Count(LinkNode *HL, ElemType x)
            {
                int count = 0;
                LinkNode *p = HL;
                while (p != NULL)
                {
                    if (p->data == x) count++;
                    p = p->next;
                }
                return count;
            }
            ```

    ???question "2018"
        === "题目描述"
            **算法设计题**

            请设计一个算法，用于判断单链表中的元素是否是递增的。

        === "答案"
            ```cpp linenums="1"
            bool IsIncr(LinkList *L)
            {
                if (L == NULL || L->next == NULL) return true;
                LinkList *p = L->next;
                while (p != NULL)
                {
                    if (p->data < L->data) return false;
                    L = p;
                    p = p->next;
                }
                return true;
            }
            ```

    ???question "2022"
        === "题目描述"
            **简答题**

            假设一个含有 $n$ 个元素的线性表采用带头节点的循环双链表存储，请给出以下运算所需要的时间复杂度。

            **(1)** 查找序号为 $i(1 \leq i \leq n)$ 的元素；

            **(2)** 查找第一个值为 $x$ 的元素；

            **(3)** 插入新元素作为第一个元素；

            **(4)** 插入新元素作为最后一个元素；

            **(5)** 插入第 $i(2 \leq i \leq n)$ 个元素；

            **(6)** 删除第一个元素；

            **(7)** 删除最后一个元素；

            **(8)** 删除第 $i(2 \leq i \leq n)$ 个元素。

        === "答案"
            | 运算 | 时间复杂度 |
            | :-: | :-: |
            | 查找序号为 $i$ 的元素 | $O(n)$ |
            | 查找第一个值为 $x$ 的元素 | $O(n)$ |
            | 插入新元素作为第一个元素 | $O(1)$ |
            | 插入新元素作为最后一个元素 | $O(1)$ |
            | 插入第 $i$ 个元素 | $O(n)$ |
            | 删除第一个元素 | $O(1)$ |
            | 删除最后一个元素 | $O(1)$ |
            | 删除第 $i$ 个元素 | $O(n)$ |