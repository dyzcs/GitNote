# 猫场面试题第 1 套

以下为我为大家整理的猫场面试题第一套，均为笔者自己参加面试或者一些读者分享给我的题目，保证真实和准确性。

![阿里巴巴](https://dyz-picbed.obs.cn-south-1.myhuaweicloud.com/04_alibaba/00_alibaba.jpg)

## 1 框架部分

### 1.1 Spark 提交 job 流程

所谓提交流程，其实就是我们开发人员根据需求写的应用程序通过 Spark 客户端提交给 Spark 运行环境执行计算的流程。在不同的部署环境中，这个提交过程基本相同，但又有细微的区别。

![Spark 提交作业](https://dyz-picbed.obs.cn-south-1.myhuaweicloud.com/04_alibaba/01_sparksubmit.png)

在国内的工作环境中，将 Spark 引用部署到 Yarn 环境中会更多一些，Spark 应用程序提交到 Yarn 环境中执行的时候，一般会有两种部署执行的方式：**Client** 和 **Cluster**。两种模式，主要的区别在于：**Driver 程序的运行节点**。

#### 1.1.1 Yarn Client 模式

Client 模式将用于监控和调度的 Driver 模块在客户端执行，而不是 Yarn 中，所以一般用于测试。

![Yarn client模式](https://dyz-picbed.obs.cn-south-1.myhuaweicloud.com/04_alibaba/02_yarnclient.png)

1. Driver 在任务提交的本地机器上运行。
2. Driver 启动后会和 ResourceManager 通讯申请启动 ApplicationMaster。
3. ResourceManager 分配 container，在合适的 NodeManager 上启动 ApplicationMaster，负责向 ResourceManager 申请 Executor 内存。
4. ResourceManager 接到 ApplicationMaster 的资源申请后会分配 container，然后 ApplicationMaster 在资源分配指定的 NodeManager 上启动 Executor 进程。
5. Executor 进程启动后会向 Driver 反向注册，Executor 全部注册完成后 Driver 开始执行 main 函数。
6. 之后执行 Action 算子时，触发一个 job，并根据宽依赖开始划分 stage，每个 stage 生成对应的 TaskSet，之后将 task 分发到各个 Executor 上执行。

#### 1.1.2 Yarn Cluster 模式

Cluster 模式将用于监控和调度的 Driver 模块启动在 Yarn 集群资源中执行。一般应用于实际生产环境。

![03_yarncluster](https://dyz-picbed.obs.cn-south-1.myhuaweicloud.com/04_alibaba/03_yarncluster.png)

1. 在 Yarn cluster 模式下，任务提交后会和 ResourceManager 通讯申请启动 ApplicationMaster。
2. 随后 ResourceManager 分配 container，在合适的 NodeManager 上启动 ApplicationMaster，此时的 ApplicationMaster 就是 Driver。
3. Driver 启动后向 ResourceManager 申请 Executor 内存，ResourceManager 接到 ApplicationMaster 的资源申请后会分配 container，然后在合适的 NodeManager 上启动 Executor 进程。
4. Executor 进程启动后会向 Driver 反向注册，Executor 全部注册完成后 Driver 开始执行 main 函数。
5. 之后执行到 Action 算子时，触发一个 job，并根据宽依赖开始划分 stage，每个 stage 生成对应的 TaskSet，之后将 task 分发到各个 Executor 上执行。

### 1.2 提交脚本中 -jar 是什么意思

使用 spark-submit 时，应用程序的jar包以及通过 —jars 选项包含的任意 jar 文件都会被自动传到集群中。如果代码依赖于其它项目，将这些资源集成打包，在执行 bin/spark-submit 脚本时就可以传递这些 jar 包了。

### 1.3 Executor 怎么获取 Task

### 1.4 详解 Hadoop 的 WordCount

![image-20201012202552209](https://dyz-picbed.obs.cn-south-1.myhuaweicloud.com/04_alibaba/07_wordcount.png)

1. 将文件拆分成 splits，由于测试用的文件较小，所以每个文件为一个 split，并将文件按行分割形成 <key,value> 对。这一步由MapReduce框架自动完成，其中偏移量（即 key 值）包括了回车所占的字符数（Windows 和 Linux 环境会不同）。

    <img src="https://dyz-picbed.obs.cn-south-1.myhuaweicloud.com/04_alibaba/08_wc01.png" alt="word count 01" style="zoom: 67%;" />

2. 将分割好的 <key,value> 对交给用户定义的 map 方法进行处理，生成新的 <key,value> 对。

    <img src="https://dyz-picbed.obs.cn-south-1.myhuaweicloud.com/04_alibaba/09_wc02.png" alt="word count 02" style="zoom:67%;" />

3. 得到 map 方法输出的 <key,value> 对后，Mapper 会将它们按照 key 值进行排序，并执行 Combine 过程，将 key 至相同 value 值累加，得到 Mapper 的最终输出结果。

    <img src="https://dyz-picbed.obs.cn-south-1.myhuaweicloud.com/04_alibaba/10_wc03.png" alt="word count 03" style="zoom:67%;" />

4. Reducer 先对从 Mapper 接收的数据进行排序，再交由用户自定义的 reduce 方法进行处理，得到新的 <key,value> 对，并作为 WordCount 的输出结果。

    <img src="https://dyz-picbed.obs.cn-south-1.myhuaweicloud.com/04_alibaba/11_wc04.png" alt="word count 03" style="zoom:67%;" />

### 1.5 Spark 做过哪些优化

**优化说完会问你为什么？原理是什么？**

**Spark 优化细则较多，以后我会把我整理的笔记逐渐上传**

1. 常规性能调优
2. 算子调优
3. Shuffle 调优
4. JVM调优

### 1.6 Spark 内存管理

**有关 Spark 的内存管理请看我的下一章** *Spark 内存管理*

## 2 算法部分

### 2.1 单向链表翻转

[剑指 Offer 24. 反转链表](https://leetcode-cn.com/problems/fan-zhuan-lian-biao-lcof/)

> 定义一个函数，输入一个链表的头节点，反转该链表并输出反转后链表的头节点。
>
> **示例:**
>
> ```tex
> 输入: 1->2->3->4->5->NULL
> 输出: 5->4->3->2->1->NULL
> ```
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode reverseList(ListNode head) {
        //新链表
        ListNode newHead = null;
        while (head != null) {
            // 先保存访问的节点的下一个节点，保存起来
            // 留着下一步访问的
            ListNode temp = head.next;
            // 每次访问的原链表节点都会成为新链表的头结点，
            // 其实就是把新链表挂到访问的原链表节点的
            // 后面就行了
            head.next = newHead;
            // 更新新链表
            newHead = head;
            // 重新赋值，继续访问
            head = temp;
        }
        // 返回新链表
        return newHead;
    }
}
```

[LeetCode 92. 反转链表 II](https://leetcode-cn.com/problems/reverse-linked-list-ii/)

> 反转从位置 m 到 n 的链表。请使用一趟扫描完成反转。
>
> 说明:
> 1 ≤ m ≤ n ≤ 链表长度。
>
> 示例:
>
> ```tex
> 输入: 1->2->3->4->5->NULL, m = 2, n = 4
> 输出: 1->4->3->2->5->NULL
> ```
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode reverseBetween(ListNode head, int m, int n) {
        ListNode dmy = new ListNode(0);
        dmy.next = head;
        int delta = n-m;
        ListNode pre = dmy,tail = head;
        //先定位出m节点和m之前的节点
        while(m>1){
            pre = tail;
            tail = tail.next;
            m--;
        }
        while(delta > 0){
            ListNode next = tail.next;
            tail.next = next.next;//tail一直不变，只要修改指针到next.next
            next.next = pre.next;//next.next指向pre的next，也就是最新的第m个位置
            pre.next = next;//更新next为最新的第m个位置
            delta --;
        }
        return dmy.next;
    }
}
```

### 2.2 实现堆栈 Push Pop Min 复杂度 O(1)

[剑指 Offer 30. 包含min函数的栈](https://leetcode-cn.com/problems/bao-han-minhan-shu-de-zhan-lcof/)

> 定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的 min 函数在该栈中，调用 min、push 及 pop 的时间复杂度都是 O(1)。
>
> 示例:
>
> ```tex
> MinStack minStack = new MinStack();
> minStack.push(-2);
> minStack.push(0);
> minStack.push(-3);
> minStack.min();   --> 返回 -3.
> minStack.pop();
> minStack.top();      --> 返回 0.
> minStack.min();   --> 返回 -2.
> ```
```
class MinStack {
	Stack<Integer> A, B;
    public MinStack() {
        A = new Stack<>();
        B = new Stack<>();
    }
    public void push(int x) {
        A.add(x);
        if(B.empty() || B.peek() >= x)
            B.add(x);
    }
    public void pop() {
        if(A.pop().equals(B.peek()))
            B.pop();
    }
    public int top() {
        return A.peek();
    }
    public int min() {
        return B.peek();
    }
}
/**
 * Your MinStack object will be instantiated and called as such:
 * MinStack obj = new MinStack();
 * obj.push(x);
 * obj.pop();
 * int param_3 = obj.top();
 * int param_4 = obj.min();
 */
```

## 3 技术部分

### 第一题

题目：**亿级的交易订单量，每笔都有金额，快速找出 top1000，要求不是简单的排序然后求出 top1000 ，代码要有健壮性；提示注意是 top1000 不是 top10。**

### 第二题

题目：**有两个约 1000 万行记录的 4 到 5G 文件，JVM 只有 32M，在内存不溢出的情况下，找出相似的条数并打印出来。**

答案：**布隆过滤器**

布隆过滤器（Bloom Filter）是 1970 年由布隆提出的。它实际上是一个很长的二进制向量和一系列随机映射函数。布隆过滤器可以用于检索一个元素是否在一个集合中。**它的优点是空间效率和查询时间都比一般的算法要好的多，缺点是有一定的误识别率和删除困难**。

**当一个元素加入布隆过滤器中的时候，会进行如下操作：**

1. 使用布隆过滤器中的哈希函数对元素值进行计算，得到哈希值（有几个哈希函数得到几个哈希值）。
2. 根据得到的哈希值，在位数组中把对应下标的值置为 1。

![布隆过滤器 01](https://dyz-picbed.obs.cn-south-1.myhuaweicloud.com/04_alibaba/05_bloom01.png)

![布隆过滤器 02](https://dyz-picbed.obs.cn-south-1.myhuaweicloud.com/04_alibaba/06_bloom02.png)

**当我们需要判断一个元素是否存在于布隆过滤器的时候，会进行如下操作：**

1. 对给定元素再次进行相同的哈希计算；
2. 得到值之后判断位数组中的每个元素是否都为 1，如果值都为 1，那么说明这个值在布隆过滤器中，如果存在一个值不为 1，说明该元素不在布隆过滤器中。

### 第三题

题目：**有一个双十一的天猫场景，我要做实时和离线两种分析方向，从数据建模、计算性能、元数据管理、数据质量上讲一讲基本架构设计成什么样子。**

推荐大家两本书《大数据之路 - 阿里巴巴大数据实践》、《数仓建模工具箱》。

## 4 思维部分

### 第一题

题目：**岛上有100个囚犯，他们都是蓝眼睛，但是他们都只能看到别人眼睛的颜色，并不能知道自己的眼睛颜色，而且他们之间不能去谈论眼睛颜色的话题，规定每天晚上都可以有一个人去找守卫说出自己的眼睛颜色，如果错了被杀死，如果对了被释放。但是大家在没有十足的把握前都不敢去找守卫，有一天，一个医生对他们说你们之中至少有一个蓝眼睛，然后N天，这些人都获救了，为什么？这句话对他们有什么影响？**

[知乎链接](https://www.zhihu.com/question/21262930)

### 第二题

题目：**有 100 层楼梯，从其中一层摔下鸡蛋的时候鸡蛋会碎，并且次层之上的都会碎，次层之下的都不会碎，如果你有一个鸡蛋、两个鸡蛋、三个鸡蛋，你会怎么去找出这个楼层，最多要试多少次？**

笔者看法：如果只有一个鸡蛋，那只能从第一层开始逐渐尝试；有两个鸡蛋其中一个可以选择从 30 层或者 50 层开始，如果碎了那就将第二个鸡蛋从 0 开始，如果没碎那就减少了一半或者三分之一。如果 n 个鸡蛋可以进行 n - 1 次尝试，减少逐层尝试的次数。 