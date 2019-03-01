layout: title
title: 反转单链表
date: 2017-01-12 17:22:08
tags:
- leetcode
categories:
- algorithm
- leetcode
- linked list
---

反转单链表（思考过程及java源码）
<!-- more -->

### 反转单链表

Reverse a singly linked list.   
https://leetcode.com/problems/reverse-linked-list/

#### 迭代方式思路
  已知条件：假定给定一个正常单链表a->b->c->d，ListNode l1 = a;
  推导：反转无非是把已知的链表转化为d->c->b->a，即将原链表的结点指向全部反向。那么，开始思考：   

  1、首先我定义一个变量指向a，表示待反转链表的头结点ListNode oldHead=a，接着第一步很自然想到把a的next指向null，但是一旦我这么改的话，我就没法得到b结点，进而没法修改b的next指向。故我需要在修改a的指向前，提前定义一个额外的变量来临时保存a的next指向，那么我定义ListNode nextNode = oldHead.next，此时nextNode指向b结点，链表还未做任何改动，即a(oldHead)->b(nextNode)->c->d;  

  2、定义了nextNode变量，那么我修改a的指向便无后顾之忧，所以我执行oldHead.next = null,即使a的next指向null，此时原链表变为了2段，一段是a(oldHead)->null（称作已反转链表部分），另一段是b(nextNode)->c->d（称作未反转链表部分），因为oldHead的含义理解为当前需要反转的结点，因为需要将oldHead指向nextNode(b)，并将nextNode指向nextNode.next(c)，修改oldHead指向的话，需要记住已分割部分的头结点，只能再定义个结点newHead=oldHead，执行顺序为newHead=oldHead，oldHead=nextNode，nextNode=nextNode.next，此时链表的2段为a(newHead)->null, b(oldHead)->c(nextNode)->d->null。

  3、此时我接着要把未反转链表部分接着反转，把头结点b的next指向a（即把未反转链表的头结点指向已反转链表的头结点），此时链表分两段为b（oldHead）->a（newHead）->null，c(nextNode)->d，接着我需要更新几个变量的指向（使其oldHead指向c，newHead就需要指向b，nextNode指向d，2段链表分别为b(newHead)->a->null,c(oldHead)->d(nextNode)->null。

  4、继续重复过程3，修改c的next使其指向b，那么我得到c（oldHead）—>b（newHead）->a->null，d（nextNode）->null，更新几个变量指向后，2段链表为c(newHead)->b->a->null，d(oldHead)-null(nextNode)

  5、再重复过程3，修改d的next=newHead,2段链表分别为d(oldHead)->c(newHead)->b->a-null，null(nextNode)，此时已经反转完成，然后迭代结束条件就是nextNode==null

  6、总的来说，需要newHead、oldHead、nextNode3个变量，在原链表还是一段的时候，我们的oldHead指向a，nextNode=a.next，newHead则等于null,三个变量的表示意思为：oldHead表示待反转链表的头结点，newHead表示已反转链表的头结点（所以一开始初始值为null），nextNode表示带反转链表的头结点的下一个结点（所以初始值为oldHead.next)。

#### 迭代（循环）实现
  ```
    `/**
     * Definition for singly-linked list.
     * public class ListNode {
     *     int val;
     *     ListNode next;
     *     ListNode(int x) { val = x; }
     * }
     */`
    public class Solution {
        public ListNode reverseList(ListNode head) {
           if(head == null) return null;
           ListNode newHead = null;
           ListNode oldHead = head;
           ListNode nextNode = head.next;// nextNode定义为待反转链表头结点的next结点，初始化为head的next结点
           while(nextNode != null){//  待反转链表头结点的next结点不为null作为条件
              oldHead.next = newHead;
              newHead =oldHead;
              oldHead = nextNode;
              nextNode = nextNode.next;
           }
           oldHead.next = newHead;// 需要手动更改待反转链表的最后一个头结点的指向
           newHead = oldHead;// 该步可以省略，直接返回oldHead，为了更清晰。
           return newHeadNode;
        }
    }
  ```


  为什么while循环要以nextNode！= null作为结束条件？考虑以下实现，只是不同的初始化方式：
  ```
  
    `/**
     * Definition for singly-linked list.
     * public class ListNode {
     *     int val;
     *     ListNode next;
     *     ListNode(int x) { val = x; }
     * }
     */`
    public class Solution {
        public ListNode reverseList(ListNode head) {
            if(head == null) return null;
            ListNode newHead = null;// 定义为已反转链表头结点
            ListNode oldHead = head;// 定义为待反转链表头结点
            ListNode next = null;// 定义为待反转链表的头结点的next结点，初始化为null
            while(oldHead != null){//迭代结束条件是待反转链表的头结点不为null，为null说明反转结束
                next = oldHead.next;// 保存next指向
                oldHead.next = newHead;
                newHead = oldHead;
                oldHead = next;
            }
            return newHead;// newHead结点即为新链表的头结点
        }
    }
```
#### 递归方式思路
前面分析的过程可以概括为一个已反转链表（初始为null）部分，一个未反转链表部分（初始为待反转链表），每次都是从待反转链表拿出头结点，作为已反转链表部分的头结点，不断重复这个过程，直到待反转链表部分只剩下null结点。这个过程和递归很类似，递归就是在函数里调用函数本身，重复调用函数本身说明解决的问题结构很类似，刚才的分析过程中一直存在的结构或变量有2个，一个是已反转链表、一个未反转链表，那递归函数是不是可以这么定义？

  ```
      private ListNode reverse(ListNode oldHead, ListNode newHead)
  ```
我在这个函数里要调用函数自身进行递归，所以：    
```
        private ListNode reverse(ListNode oldHead, ListNode newHead){
          return reverse(oldHead, newHead);
        }
```
  貌似需要在递归里做些处理，回想我们在每一次的从待反转链表结点里拿出头结点oldHead（拿出前需要先记住oldHead的next结点，否则修改oldHead的next指向后，便找不到原next结点了），将其next指向已反转链表的头结点，这样未反转链表和已反转链表的头结点都发生了变化，我们需要更新oldHead和newHead的指向。
```
        private ListNode reverse(ListNode oldHead, ListNode newHead){
          ListNode next = oldHead.next;// 记住待反转链表头结点的next结点
          oldHead.next = newHead;// 将oldHead结点从待反转链表结点拿出，使其成为已反转链表结点的头结点
          newHead = oldHead;// 更新newHead结点指向
          oldHead = next;// 更新oldHead结点指向，指向此时待反转链表结点的头结点（为先前保存的next指向的结点）
          return reverse(oldHead, newHead);
        }
  ```
  貌似递归还缺少递归出口，拿什么作为出口？回想刚才分析过程，最终是以待反转链表的头结点未null，结束，此时已反转链表的头结点就是反转后新链表的头结点。
  
  ```
     private ListNode reverse(ListNode oldHead, ListNode newHead){
       if(oldHead == null) return newHead;
       ListNode next = oldHead.next;// 记住待反转链表头结点的next结点
       oldHead.next = newHead;// 将oldHead结点从待反转链表结点拿出，使其成为已反转链表结点的头结点
       newHead = oldHead;// 更新newHead结点指向
       oldHead = next;// 更新oldHead结点指向，指向此时待反转链表结点的头结点（为先前保存的next指向的结点）
       return reverse(oldHead, newHead);
     }
  ```
#### 递归方式实现

  ```
      public ListNode reverseList(ListNode head) {
        if(head == null) return null;
        return reverse(head, null);// 一开始已反转链表为null，因此头结点未null
    }
    private ListNode reverse(ListNode oldHead, ListNode newHead){
      if(oldHead == null) return newHead;
      ListNode next = oldHead.next;// 记住待反转链表头结点的next结点
      oldHead.next = newHead;// 将oldHead结点从待反转链表结点拿出，使其成为已反转链表结点的头结点

      // 以下2个更新可以省略，为了清晰
      newHead = oldHead;// 更新newHead结点指向
      oldHead = next;// 更新oldHead结点指向，指向此时待反转链表结点的头结点（为先前保存的next指向的结点）
      return reverse(oldHead, newHead);
    }
  ```