---
title: LRU cache的简单实现
date: 2017-06-25 15:54:07
tags: 算法
categories:
- 算法
---

## LRU是什么

说白了就是一种替换策略，在固定容量的缓存中，当存储满了的时候，新的数据插入，该采用何种策略，LRU就是其中一种，LRU(least recently used)，顾名思义就是最近最少使用。当新的插入时，我们要把最近最少使用的给从缓存里拿掉。

## 实现思路

链表+哈希表

java中可以使用LinkedHashMap来实现，但我这里只给个简单的实现。

<!-- more -->

```python

class Node(object):
    def __init__(self):
        self.pre=None
        self.next=None
        self.key=None
        self.value=None
        
class LRUCache(object):

    def __init__(self, capacity):
        """
        :type capacity: int
        """
        self.cache={}
        self.volume=capacity
        self.head=Node()
        self.tail=Node()
        self.head.next=self.tail
        self.tail.pre=self.head


    def get(self, key):
        """
        :type key: int
        :rtype: int
        """
        try:
            node=self.cache[key]
        except:
            return -1
        node.pre.next=node.next
        node.next.pre=node.pre
        self.insertNode(node)
        return node.value

    def put(self, key, value):
        """
        :type key: int
        :type value: int
        :rtype: void
        """
        node=self.cache.get(key)
        if node:
            self.deleteNode(node)
        node=Node()
        node.key=key
        node.value=value
        self.insertNode(node)
        self.cache[key]=node
        if len(self.cache)>self.volume:
            self.deleteNode(self.head.next)

    def deleteNode(self,node):
        node.pre.next=node.next
        node.next.pre=node.pre
        del self.cache[node.key]
        del node

    def insertNode(self,node):
        last=self.tail.pre
        last.next=node
        node.pre=last
        node.next=self.tail
        self.tail.pre=node
```

