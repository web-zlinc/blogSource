---
title: 数据结构与算法（上）
layout: post
date: 2021-12-16 22:16
comments: true
tags: 
	- javaScript
key: "1"
---

### 栈(FILO, first in last out)

> 是一种受限的数据结构，插入与删除元素都在栈顶进行操作

**模拟实现方式**

- 用数组来模拟
- 用链表来模拟

<!--more-->

```javaScript
  class Stack{
    constructor(props=[]){
      // 存储元素容器
      this.items = [...props]
    }
    // 压栈
    push(element){
      this.items.push(element);
    }
    // 弹栈
    pop(){
      return this.items.pop();
    }
    // 查看栈顶元素
    peek(){
      return this.items.slice(this.items.length - 1);
    }
    //  查看栈里的元数个数
    size(){
      return this.items.length;
    }
    // 将栈元素以字符串形式输出
    toString(){
      let result = ''
      this.items.forEach(item => result+= ' ' + item)
      return result;
    }
    // 是否为空栈
    isEmpty(){
      return this.items.length === 0;
    }
  }

  const stack = new Stack([2, 4, 6, 8, 10, 12]);

```

#### 十进制转二进制（栈应用）

```javaScript
  const dec2bin = (decnumber) => {
    // 定义栈对象
    const stack = new Stack();
    // 循环操作
    while(decnumber){
      // 获取余数并放入栈中
      stack.push(decnumber % 2);
      // 获取整除后的结果作为下一次运行数字
      decnumber = Math.floor(decnumber/2);
    }
    // 弹栈获取转换后的二进制数字
    let binaryString = '';
    while(!stack.isEmpty()){
      binaryString+= stack.pop();
    }
    return binaryString;
  }

  console.log(dec2bin(100)) // 1100100

```

### 队列（FIFO, first in first out）

**基于数组模拟**

```javascript
class Queue {
  constructor(props = []) {
    this.items = props
  }

  enQueue(element) {
    this.items.push(element)
  }

  deQueue() {
    this.items.shift()
  }

  front() {
    return this.items[0]
  }

  ieEmpty() {
    return this.items.length === 0
  }

  size() {
    return this.items.length
  }

  toString() {
    return this.items.join(',')
  }
}
```

#### 击鼓传花（队列的应用）

> **规则：** 类似一群人围成一个圈，传递手绢，听到击鼓声就停下，此时拿到手绢的人就从游戏中推出，直到只剩一人为胜出者

```javascript
const passGame = (nameList = [], num) => {
  const queue = new Queue(nameList)

  while (queue.size() > 1) {
    for (let i = 0; i < num - 1; i++) {
      queue.enQueue(queue.deQueue())
    }
  }
  console.log(queue.front())
  return queue.items.indexOf(queue.front())
}

const nameList = ['lily', 'lucy', 'tom', 'lilei', 'why']
console.log(passGame(nameList, 3))
```

### 优先级队列

> 一个元素除了包含内容外，还有指定这个元素的一个优先级

```typescript
  interface Item {
    content: string;
    priority: number
  }

  class priorityQueue {
    constructor(){
      this.items!:Item[];
    }
    // 优先级小的排在前面
    enQueue(element: Item){
      if(!this.items.length) {
        this.items.push(element);
      }else {
        let flag = false;
        for(let i=0; i<this.items.length; i++){
          if(element.priority > this.items[i].priority){
            this.items.splice(i, 0, element);
            flag = true
            break;
          }
        }
        if (!flag) this.items.push(element)
      }
    }
  }
```

### 链表

> 与数组类型，相比数组在插入和删除元素时性能会更佳，但是在查找元素时对比数组性能会比较差

```javascript
  class Node {
    constructor(data){
      this.data = data;
      this.next = null;
    }
  }

  class LinkedList {
    constructor(){
      this.length = 0;
      this.head = null;
    }

    append(data){// 往链表里追加数据
      const newNode = new Node(data);
      if(this.length === 0){
        this.head = newNode
      }else {
        let current = this.head;
        while(current.next){
          current = current.next
        }
        current.next = newNode;
      }
      this.length += 1;
    }

    toString(){ // 以字符串返回链表数据
      let current = this.head;
      let linkSting = '';
      while(current){
        linkString += current.data + ' '
        current = current.next;
      }
      return linkString;
    }

    insert(position, data){// 在指定位置插入元素
      if(position < 0 || position > this.length) return false;
      const newNode = new Node(data);
      if(position === 0){
        newNode.next = this.head;
        this.head = newNode;
      }else {
        let current = this.head;
        let previous = null
        let index = 0;
        while(index++ < position) {
          previous = current;
          current = current.next;
        }
        newNode.next = current;
        previous.next = newNode;
      }
      this.length += 1;
      return true;
    }

    get(position){ // 获取指定索引对应的元素
      if(position < 0 || position >= this.length) return false;
      let current = this.head;
      let idx = 0;
      while(idx++ < position){
        current = current.next;
      }
      return current.data;
    }

    size(){ // 查询链表长度
      return this.length;
    }

    indexOf(data){ // 根据指定元素返回其对应索引
      let current = this.head;
      let idx = 0;
      let dataIdx = false;
      while(idx < this.length){
        if(current.data === data) {
          dataIdx = idx;
          break;
        }else {
          current = current.next;
          idx +=1;
        }
      }
      return dataIdx;
    }

    removeAt(position){ // 根据索引删除元素
      if(position < 0 || position >= this.length) return null;
      if(position === 0) {
        this.head = this.head.next;
      }else {
        let current = this.head;
        let previous = null;
        let idx = 0;
        while(idx++ < position) {
          previous = current;
          current = current.next;
        }
        previous.next = current.next;
      }
      this.length -=1;
      return current.data;
    }

    remove(data){ // 删除指定元素
      const position = this.indexOf(data);
      return this.removeAt(position);
    }
  }

  ieEmpty() { // 判断链表是否为空
    return this.length === 0;
  }
```
### 双向链表
> 上面讲的链表是单向链表，其缺点就是在查找某个节点的前一个节点时，必选从前往后遍历链表才能才能找到该节点，而双向链表在处理这种情况时可以直接通过当前节点获取前一个接节点。下面我们来实现一下双向链表方法。
```javascript
  class Node {
    constructor(data){
      this.data = data;
      this.next = null;
      this.prev =  null;
    }
  }
   class DoubleLinkedList {
    constructor(){
      this.length = 0;
      this.head = null;
      this.tail = null; //标记最后一个节点
    }

    append(data){// 往链表里追加数据
      const newNode = new Node(data);
      if(this.length === 0){
        this.head = newNode;
        this.tail = newNode;
      } 
      else {
        newNode.prev = this.tail;
        this.tail.next = newNode;
        this.tail = newNode;
      }
      this.length += 1;
    }

    toString(){ // 以字符串返回链表数据
      return this.backwardString();
    }

    insert(position, data){// 在指定位置插入元素
      if(position < 0 || position > this.length) return false;
      const newNode = new Node(data);
      if(this.length === 0){
        newNode.next = this.head;
        this.head.prev = newNode;
        this.head = this.tail = newNode;
      } else if (position === 0) {
        newNode.next = this.head;
        this.head.prev = newNode
        this.head = newNode;
      } 
       else if (position === this.length) {
        newNode.prev = this.tail;
        this.tail.next = newNode;
        this.tail = newNode;
      } else {
        let current = this.head;
        let index = 0;
        while(index++ < position) {
          current = current.next;
        }
        newNode.next = current;
        newNode.prev = current.prev;
        current.prev.next = newNode;
        current.prev = newNode;
      }
      this.length += 1;
      return true;
    }

    get(position){ // 获取指定索引对应的元素
      if(position < 0 || position >= this.length) return false;
      let current = this.head;
      const flag = (this.length / 2 >= position); // 根据position的大小决定从前还是从后遍历
      let idx = 0;
      if(flag) {
        while(idx++ < position) {
          current = current.next;
        }
      } else {
        idx = this.length
        current = this.tail;
        while(idx-- > position) {
          current = current.prev;
        }
      }
      
      return current.data;
    }

    size(){ // 查询链表长度
      return this.length;
    }

    indexOf(data){ // 根据指定元素返回其对应索引
      let current = this.head;
      let idx = 0;
      while(current){
        if(current.data === data) {
          return idx;
        }
        current = current.next;
        idx +=1;
      }
      return -1;
    }

    update(position, data){
      if(position < 0 || position >= this.length) return false;
      let current = null;
      let idx = null;
      const flag = (this.length / 2 > position);
      if(flag) {
        current = this.head;
        idx = 0;
        while(i++ < position) {
          current = current.next;
        }
      } else {
        current = this.tail;
        idx = this.length;
        while(idx-- > position){
          current = current.prev;
        }
      }
      current.data = data;
      return true;
    }

    removeAt(position){ // 根据索引删除元素
      if(position < 0 || position >= this.length) return null;
      let current = this.head;
      if(this.length ===1){
        this.head = this.tail = null
      }else {
        if(position === 0) {
          this.head.next.prev = null;
          this.head = this.head.next;
        } else if(position === this.length-1){
          current = this.tail;
          this.tail.prev.next = null;
          this.tail = this.tail.prev;
        } else {
          let idx = 0;
          while(idx++ < position) {
            current = current.next;
          }
          current.next.prev = current.prev;
          current.prev.next = current.next;
        }
      }
      this.length -=1;
      return current.data;
    }

    remove(data){ // 删除指定元素
      const position = this.indexOf(data);
      return this.removeAt(position);
    }
  }

  ieEmpty() { // 判断链表是否为空
    return this.length === 0;
  }

  forwardString() { // 从后向前遍历链表
    let current = this.tail;
    let resultString = '';
    while(current){
      resultString += current.data + ' ';
      current = current.prev;
    }
    return resultString;
  }

  backwardString() { // 从前向后遍历链表
    let current = this.head;
    let resultString = '';
    while(current){
      resultString += current.data + ' ';
      current = current.next;
    }
    return resultString;
  }

  getHead(){
    return this.head.data;
  }

  getTail(){
    return this.tail.data;
  }
}
```

### 集合
> 相当与特殊的数组，没有顺序，且元素不能重复，相关实现如下：
```javascript
  class Set {
    constructor(props={}){
      this.items = {};
    }
    add(value){
      if(!this.has(value)) return false;
      this.items[value] = value;
    }
    has(value){
      return this.items.hasOwnProperty(value);
    }
    clear(){
      this.items = {}
    }
    delete(value){
       if(!this.has(value)) return false;
       delete this.items[value];
       return true;
    }
    values(){
      return Object.keys(this.items);
    }

    size(){
      return Object.keys(this.items).length;
    }

    // 集合之间的操作
    union(otherSet) { // 两个集合的交集封装
      const unions = new Set();
      let values = this.values();
      for(let i=0; i<values.length; i++) {
        unions.add(values[i]);
      }
      values = otherSet.values();
      for(let i=0; i<values.length; i++) {
        unions.add(values[i]);
      }

      return unions;
    }

    intersection(otherSet) { // 交集
      const intersectionSet = new Set();
      let values = this.values();
      let values1 = otherSet.values();
      for(let i=0; i<values; i++){
        if(values1.has(values[i])){
          intersectionSet.add(values[i])
        }
      }
      return intersectionSet;
    }
    diff(otherSet) { // 差集
      const diffSet = new Set();
      let values = this.values();
      let values1 = otherSet.values();
      for(let i=0; i<values; i++){
        if(!values1.has(values[i])){
          diffSet.add(values[i])
        }
      }
      return diffSet;
    }

    subSet(otherSet){ // 子集
      const values = this.values();
      for (let i = 0; i<values.length; i++){
        if(!otherSet.has(values[i])) {
          return false;
        }
      }
      return true;
    }
  }
```