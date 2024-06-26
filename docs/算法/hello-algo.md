## 栈与队列

### 队列

在python中，一般将双向队列`deque`当做队列使用，虽然`deque.Queue()`是纯正的队列类，当时不太好用，不推荐使用。

- `deque.pop()`：从队列右侧（末尾）移除并获取元素；
- `deque.popleft()`：从队列左侧（开头）移除并获取元素；

???+ note "一个巧妙方法"
    使用数据实现队列时，在数组中删除首元素的时间复杂度是O(n)（因为删除首元素后后面的元素要向前移动），这会导致出队操作效率较低。我们可以采用一个巧妙的方法避免这个问题。

    使用一个变量`front`指向队首元素的索引，并维护一个变量`size`记录变量的长度。定义`rear = front + size`，这个公式计算出的`rear`指向队尾元素之后的下一个位置。

    基于此设计，数组中包含元素的有效区间为`[front, rear - 1]`。

    - 入队操作：将输出元素赋值给`rear`索引处，并将`size`增加1
    - 出队操作：将`front`增加1，并将`size`减少1

    你可能会发现一个问题：在不断进入入队和出队的过程中，`front`和`rear`都在向右移动，当他们到达数组尾部时就无法继续移动了。为了解决此问题，可以将数组视为首尾相接的“环形数组”，这种周期性规律可以通过取余实现。

    - 入队时：`rear: int = (self._front + self._size) % self.capacity()`
    - 出队时：`front: int = (self._front + 1) % self.capacity()`

