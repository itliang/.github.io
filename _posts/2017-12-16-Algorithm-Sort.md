---
layout: post
title:  "常用的排序算法实现"
date:   2017-12-16 15:23:05
categories: Algorithm
excerpt: 排序算法练习。
---

* content
{:toc}

## Quicksort 思想
Quicksort是分治法(divide and conquer)的一种，也是对冒泡排序(Bubblesort)的一种改进，通过一趟排序将要排序的数据分割成独立的两个部分，其中一部分的所有数据都比另外一部分的数据都要小，然后再对这两部分分别采用快速排序。整个排序过程采用递归的方法实现。

快速排序是个不稳定的排序算法。

### Quicksort 实现

    void quicksort(int a[], int low, int high){
      int i = low;
      int j = high;
      if (high <= low) return;
      int guard = a[low];
      while (i < j){		
        while (j>i && a[j] >= guard) {
          j--;			//从后往前寻找比哨兵值小的数
        }
        if (a[j] < guard){ //可能会造成算法的不稳定
          a[i] = a[j];	//将比哨兵值小的数存入a[i],因为a[i]的值已经存在哨兵当中
        }

        while (i < j && a[i] <= guard){
          i++;			//从前往后寻找比哨兵值大的数
        }		
        if (a[i] > guard){
          a[j] = a[i];	//比哨兵值大的数保存到a[j]
        }
        a[i] = guard;		//a[i]保留哨兵的值，继续下一次循环
      }
      quicksort(a, low, i-1);	//将原来的数组分成了两部分，继续递归调用
      quicksort(a, i+1, high);
    }


## Heapsort 思想

堆是具有以下性质的完全二叉树，每个节点的值都大于或等于左右孩子节点的值(最大堆)或者每个节点的值都小于或者等于左右孩子节点的值(最小堆)。

堆排序利用了大根堆/小根堆堆顶记录的关键字最大/最小的特征和数组快速定位指定索引的元素的特点。它是一种树行选择排序，在当前无序区中选择关键字最大或者最小的记录。


### Heapsort 实现

    void heap_adjust(int a[], int id, int size){
    	int lchild = 2 * id + 1;
    	int rchild = 2 * id + 2;
    	int tempId = id;
    	if (id < size / 2){
    		if (lchild < size && a[tempId] < a[lchild]){//找出最大的节点ID
    			tempId = lchild;
    		}
    		if (rchild < size && a[tempId] < a[rchild]){
    			tempId = rchild;
    		}
    		if (tempId != id){    //生成最大堆
    			int temp = a[id];
    			a[id] = a[tempId];
    			a[tempId] = temp;
    			heap_adjust(a, tempId, size);  //递归调整堆
    		}
    	}
    }

    void make_heap(int a[], int size){
    	int i;
    	for (i = size / 2; i >= 0; i--){
    		heap_adjust(a, i, size);
    	}
    }

    void heapsort(int a[], int n){
    	int i = 0;
    	int k;
    	make_heap(a, n);	//1.开始是无序的数组a[1..n]建成一个大根堆，此时a[0]是数组的最大值    
    	for (i = n - 1; i >= 0; i--){
    		int temp = a[0];	//2.将最大堆的a[0]和a[i]交换，得到新的无序数组a[1..i]
    		a[0] = a[i];
    		a[i] = temp;
    		heap_adjust(a, 0, i-1);	//3.交换后的a[0]可能违反堆的性质，所以应该调整现有的堆a[0..i]
    	}
    }

## Bucketsort 思想

桶排序假设输入数据均匀分布，然后将输入数据均匀地分配到有限数量的桶中，然后再对每个桶再分别排序，最后将每个桶中的数据有序结合起来。

### Bucketsort 实现

    typedef struct node {
      int val;
      struct node *next;
    } Node;

    void bucketsort(int array[], int arrsize, int bucketsize) {
      int i = 0;
      Node **bucket = (Node **)malloc(bucketsize * sizeof(Node *)); //每个节点里面存储了指向Node的指针
      for (i; i < bucketsize; i++){
        Node *node = (Node *)malloc(sizeof(Node));
        node->val = i;
        node->next = nullptr;
        bucket[i] = node;
      }

      for (i=0; i < arrsize; i++) {
        Node *temp = (Node *)malloc(sizeof(Node));
        temp->val = array[i];
        temp->next = nullptr;

        int index = array[i] / bucketsize;
        Node *node = bucket[index];

        Node *prv = node;		//当前节点的前置节点
        Node *cur = prv->next;	//当前节点
        if (cur == nullptr) {
          prv->next = temp;
          cur = temp;			
        }
        else {
          while (cur != nullptr && cur->val < temp->val){	 //寻找当前的节点			
            prv = cur;
            cur = cur->next;
          }				
          prv->next = temp;
          temp->next = cur;
          prv = temp;			
        }
      }
    }
