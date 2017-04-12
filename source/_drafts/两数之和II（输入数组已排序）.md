---
title: 两数之和II（输入数组已排序）
tags:
---

### 笨方法
    public class Solution {
        public int[] twoSum(int[] numbers, int target) {
            int len = numbers.length;
            int[] res = new int[2];
            for(int i = 0; i < len; i++){
                for(int j = i + 1;j<len;j++){
                    int sum = numbers[i] + numbers[j];
                    if(sum < target){
                        continue;
                    }else if(sum == target ){
                        res[0] = i+1;
                        res[1] = j+1;
                        return res;
                    }else {
                        break;
                    }
                }
            }
            return null;
        }
    }

### Stack+StringBuilder
    public class Solution {
       public int[] twoSum(int[] numbers, int target) {
           int len = numbers.length;
           int otherIndex;
           for(int i =0 ;i<len;i++){
               int other = target - numbers[i];
               if(numbers[i] == other){
                   otherIndex=i+1;
               }else{
                otherIndex = search(numbers, 0, len-1, other);}
               if(otherIndex == -1) continue;
               return new int[]{Math.min(i+1, otherIndex+1),Math.max(i+1, otherIndex +1)};
           }
           return null;
       }

       private int search(int[] numbers, int left, int right, int target){
           if(left > right) return -1;
           int mid = (left + right)/2;
           if(numbers[mid] == target){
               return mid;
           }
           if(numbers[mid] > target){
               return search(numbers, left, mid-1,target);
           }
           if(numbers[mid] < target){
               return search(numbers, mid+1, right, target);
           }
           return -1;
       }
    }

### 另一种
    public class Solution {
       public int[] twoSum(int[] numbers, int target) {
           int len = numbers.length;
           for(int i =0 ;i<len;i++){
               int other = target - numbers[i];
               int otherIndex = -1;
               if(numbers[i] <= other){
                    otherIndex = search(numbers, i+1, len-1, other);
               }else{
                    otherIndex = search(numbers, 0, i-1, other);
               }
               if(otherIndex == -1) continue;
               return new int[]{Math.min(i+1, otherIndex+1),Math.max(i+1, otherIndex +1)};
           }
           return null;
       }

       private int search(int[] numbers, int left, int right, int target){
           if(left > right) return -1;
           int mid = (left + right)/2;
           if(numbers[mid] == target){
               return mid;
           }
           if(numbers[mid] > target){
               return search(numbers, left, mid-1,target);
           }
           if(numbers[mid] < target){
               return search(numbers, mid+1, right, target);
           }
           return -1;
       }
   }
