---
layout: post
title:  "Dynamic Program 算法练习"
date:   2017-08-30 17:00:05
categories: Algorithm
excerpt: DP算法练习。
---

* content
{:toc}

## R-B Tree基本概念及思想

Red-Black Tree,又称红黑树，它是一种自平衡的二叉搜索树，树中的每个节点都有一位来存储节点的颜色，这个颜色是用来确保插入或者删除操作时树大致的平衡。

红黑树的应用比较广泛，主要用来存储有序的数据，它的时间复杂度是O(logN),例如Java中的TreeSet和TreeMap,C++ STL中的set, map及Linux虚拟内存管理。

处理红黑树的核心思想：将红色的节点移动到根节点，然后将根节点设为黑书。也就是将破坏红黑树特性的红色节点上移（向根的方向移动）。

### R-B Tree的特性
- 每个节点是黑色，或者是红色。
- 根节点是黑色。
- 每个叶子节点是黑色。 [注意：这里叶子节点，是指为空的叶子节点！]
- 如果一个节点是红色的，则它的子节点必须是黑色的。
- 从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。

### R-B Tree的基本操作

添加或者删除节点以后，红黑树就发生了变化，不能满足红黑树的上述特性，需要通过左旋或者右旋来使这棵树重新成为红黑树。

#### R-B Tree初始化
为了便于处理红黑树的边界情况，使用一个哨兵nil节点。

	typedef enum{
		RED,
		BLACK
	}COLOR;
	typedef struct rbt_node{
		int key;
		COLOR color;
		struct rbt_node *p;
		struct rbt_node *left;
		struct rbt_node *right;
	}RBT_NODE;
	typedef struct rbt_tree{
		RBT_NODE *root;
		RBT_NODE *nil; //为了便于处理红黑树的边界情况，使用一个哨兵来代替所有的NIL结点
	}RBT_TREE;

	RBT_TREE * init_rbt(){
		RBT_TREE * tree = (RBT_TREE *)malloc(sizeof(RBT_TREE));
		RBT_NODE * nil = (RBT_NODE *)malloc(sizeof(RBT_NODE));
		nil->color = BLACK;
		nil->left = nil->right = nil->p = NULL;

		tree->nil = nil;
		tree->root = nil;
		return tree;
	}


#### 左旋

#### 右旋

#### R-B Tree插入节点

#### R-B Tree删除节点
