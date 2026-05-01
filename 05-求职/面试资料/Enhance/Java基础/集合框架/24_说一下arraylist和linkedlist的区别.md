**底层数据结构**：arrayList是动态数组（连续内存空间，元素按索引顺序存储），LinkedList底层基于双向链表实现（非连续内存，节点通过指针连接）；

**访问特性**：对于随机访问，arrayList的时间复杂度是O(1)（直接通过数组下标访问），linkedList是O(n)（需从头 / 尾节点遍历到目标位置）;

**元素插入特性**：arrayList中间或头部插入时间复杂度为O(N)，尾部插入为O(1)，linkedList头尾插入（addFirst/addLast）为 O(1)

**适用场景**：linkedlist适用于头尾操作频繁的场景

ArrayList 适用于随机读取比较多的情况

Ps：两者都可以存 null