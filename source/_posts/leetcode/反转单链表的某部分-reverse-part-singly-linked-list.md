layout: title
title: 反转单链表的某部分
date: 2017-01-17 17:07:01
tags:
- leetcode
categories:
- algorithm
- leetcode
- linked list
---

反转单链表的某部分（思考过程及java源码）
<!-- more -->

### 反转单链表的某部分

Reverse Linked List II(reverse part)    
https://leetcode.com/problems/reverse-linked-list-ii/

#### 原题
    Reverse a linked list from position m to n. Do it in-place and in one-pass.

    For example:
    Given 1->2->3->4->5->NULL, m = 2 and n = 4,

    return 1->4->3->2->5->NULL.

    Note:
    Given m, n satisfy the following condition:
    1 ≤ m ≤ n ≤ length of list.

    Subscribe to see which companies asked this question

    `/**
     * Definition for singly-linked list.
     * public class ListNode {
     *     int val;
     *     ListNode next;
     *     ListNode(int x) { val = x; }
     * }
     */`

#### 题意
大概就是给我们一个单链表，然后让我们反转单链表的一部分（2个参数表示反转的第m个结点到第n个结点，m、n从1开始），反转单链表是要求反转全部，要求必须是一次遍历完成，且必须使用原地算法（使用变量是固定数量的、小的空间，不能现行增长，本地意思是不能复制链表）    
原地算法：https://zh.wikipedia.org/wiki/%E5%8E%9F%E5%9C%B0%E7%AE%97%E6%B3%95

#### 大致思路
不考虑特殊情况，就考虑大众情况，比方说让我反转1->2->3->4->5->NULL的结点2到结点4的部分（m=2，n=4），那我很容易想到就是遍历单链表，遍历的时候定义一个变量i，i==m到i==n中间结点就是我要反转的部分结点，这部分结点反转的逻辑就是反转某个单链表；接着就是这部分反转后的部分结点，需要和原链表按顺序连一块，本例就是1和4连，2和5连，那意思是我遍历的时需要记住待反转部分结点的前面一个结点和后一个结点，还需记住待反转部分反转后的首尾结点，然后返回结果是原链表的头结点，所以还需要记住原链表的头结点。
接着，我可以遍历一遍找到m、n对应结点，然后再从m遍历到n进行链表反转，这样子遍历不止一次，不符合题目要求，只能在一次遍历的时候同时反转

#### 具体过程   

  1、首先我先对head或者head.next == null的情况做下检查，head.next == null情况可能不太好想一开始，先排除：   

      public ListNode reverseBetween(ListNode head, int m, int n) {
          if(head == null || head.next == null) return head;
          ...
      }
  2、接着我按着刚才的思路，定义几个相关变量，并初始化下值：

      public ListNode reverseBetween(ListNode head, int m, int n) {
          if(head == null || head.next == null) return head;
          int i = 1;// 因为m、n从1开始，所以统一i也从1开始
          ListNode oldHead = head;// 原链表的头结点，赋值应该没啥异议
          ListNode reversePreNode = null;// 反转部分链表反转前其头结点的前一个结点
          ListNode reversedTail = null;// 反转部分链表反转后的尾结点，一开始不知道，为null
          ListNode reverseNextNode = null;// 反转部分链表反转前尾结点的下一个结点
          ListNode reversedNewHead = null;// 反转部分链表反转后的头结点，一开始不知道，为null
          ...
      }
  3、开始遍历，遍历时，我可以用while、for都行，此处用while，所以上步定义了变量i表示迭代的次数，遍历得有结束遍历的条件吧，毕竟只反转一部分，不需要遍历所有结点，所以我需要加个结束条件if(i>n)break;接着就是寻找第2步中定义的几个变量的值：

  - reversePreNode的含义是反转部分反转前头结点的前一个结点，反转部分反转前头结点其索引顺序为m，m的前一个结点索引为m－1，所以遍历时，若i=m-1，则该结点就是reversePreNode的值（反转部分反转前头结点的前一个结点）；
  - reversedTail含义为反转部门链表反转后的尾结点，那就是找反转前的头结点，头结点索引顺序为m，则若i＝＝m，则当前结点就是reversedTail的值
  因为要在一次循环里搞定，所以朦胧感觉可能需要在while里有一段代码做反转操作：      

        public ListNode reverseBetween(ListNode head, int m, int n) {
           if (head == null) return null;
           if (head.next == null) return head;
           int i = 1;
           ListNode reversedNewHead = null;// 反转部分链表反转后的头结点
           ListNode reversedTail = null;// 反转部分链表反转后的尾结点
           ListNode oldHead = head;// 原链表的头结点
           ListNode reversePreNode = null;// 反转部分链表反转前其头结点的前一个结点
           ListNode reverseNextNode = null;
           while (head != null) {
               if (i > n) {
                   break;
               }

               if (i == m - 1) {
                   reversePreNode = head;
               }
               if (i == m) {
                   reversedTail = head;
               }
               // 反转结点逻辑...
               i++;// 更新i，没啥异议吧
           }
           return xx;// 暂定，未知
        }

4、接着就是反转结点部分的逻辑:
  - 首先反转结点逻辑可参考反转单链表的几句关键逻辑（临时变量next保存当前结点的next结点，修改当前结点的next指向已️反转部分链表头结点，然后更新已反转链表头结点指向，更新待反转链表的头结点指向)，几个变量对应到步骤2定义的变量即可
  - 反转的结点是在m和n之间，所以反转逻辑应该在if(i>=m && i<=n) 条件内执行
  - 遍历时需要修改head指向为下一个结点，所以要在各个if else分支里都要执行head＝head.next


    public ListNode reverseBetween(ListNode head, int m, int n) {
       if (head == null) return null;
       if (head.next == null) return head;
       int i = 1;
       ListNode reversedNewHead = null;// 反转部分链表反转后的头结点
       ListNode reversedTail = null;// 反转部分链表反转后的尾结点
       ListNode oldHead = head;// 原链表的头结点
       ListNode reversePreNode = null;// 反转部分链表反转前其头结点的前一个结点
       ListNode reverseNextNode = null;
       while (head != null) {
           if (i > n) {
               break;
           }

           if (i == m - 1) {
               reversePreNode = head;
           }
           if (i >= m && i <= n) {
              if (i == m) {// 这部分移到if条件里
                  reversedTail = head;
              }

              reverseNextNode = head.next;// 反转前保存当前结点的next结点，反转结束时该值刚好是reverseNextNode代表的含义
              head.next = reversedNewHead;// 修改当前结点（待反转链表部分的头结点）指向已反转链表部分的头结点
              reversedNewHead = head;// 更改已反转链表的头结点的指向
              head = reverseNextNode;// 更新head指向为待反转链表的头结点
            } else {
                head = head.next;//  更新head指向为其下一个结点
            }
           i++;// 更新i，没啥异议吧
       }
       return xx;// 暂定，未知
    }
5、收尾与返回逻辑：
 - 在循环里执行反转后，紧接着需要在循环外执行更新reversedTail的next指向，根据几个变量含义，让其指向reverseNextNode即可。
 - 返回结果就需要考虑几种情况了，因为需要返回反转后新链表的头结点，但是m有可能>=2，此时reversePreNode肯定不为空，需要将reversePreNode的next指向反转部分新的头结点，然后直接返回oldHead即可；若m==1,则不能这么做，需要返回反转部分链表的新的头结点，即reversedNewHead：

       public ListNode reverseBetween(ListNode head, int m, int n) {
           if (head == null) return null;
           if (head.next == null) return head;
           int i = 1;
           ListNode reversedNewHead = null;// 反转部分链表反转后的头结点
           ListNode reversedTail = null;// 反转部分链表反转后的尾结点
           ListNode oldHead = head;// 原链表的头结点
           ListNode reversePreNode = null;// 反转部分链表反转前其头结点的前一个结点
           ListNode reverseNextNode = null;
           while (head != null) {
               if (i > n) {
                   break;
               }

               if (i == m - 1) {
                   reversePreNode = head;
               }

               if (i >= m && i <= n) {
                   if (i == m) {
                       reversedTail = head;
                   }
                   reverseNextNode = head.next;
                   head.next = reversedNewHead;
                   reversedNewHead = head;
                   head = reverseNextNode;
               } else {
                   head = head.next;
               }
               i++;
           }

           reversedTail.next = reverseNextNode;

           if (reversePreNode != null) {
               reversePreNode.next = reversedNewHead;
               return oldHead;
           } else {
               return reversedNewHead;
           }
       }
