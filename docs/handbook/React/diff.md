---
title: React——diff
date: 2023-08-13
author: 倒数
tags:
  - react
categories:
  - react
isShowComments: true
subSidebar: auto
---

### 对比规则

因为对比 2 颗树的过于复杂 O(n3), 所以 react 采取的规则是：

1. 同级比较
2. type 不同则销毁老节点及其子树
3. 通过 key 提供复用的线索

### 单节点 diff

前提：新 vNode 是单节点，currentFiber 和 新的 vNode 对比

1. 如果 key 和 type 一样，则可以复用，通过 useFiber 复用当前 fiber
2. key 不同：删掉当前 fiber 节点
   1. 与旧 fiber 的 sibling 对比，重复步骤 1
3. key 相同 type 不同，删除该节点和兄弟节点，然后新创建

### 多节点 diff

前提：newChildren 是数组

结合 reconcileChildrenArray 分析，得出规则

1. key 相同：
   1. type 相同， 复用
   2. type 不同，新建，oldFiber 标记删除
2. key 不同：
   1. 进入第 3 个 for 循环，记录 oldFibers 的 map
   2. 在老节点中找
      1. 找到了，则复用老节点
         1. 如果 old 的 index>lastPlacedIndex，位置不动，lastPlacedIndex=index
         2. old 的 index<lastPlacedIndex, 标记插入
      2. 找不到，新建 fiber，标记插入

```js
举个例子：
old: 1 2 3 4 5
new: 2 1 4 7
进入第3个for
第一次循环 旧map中找到2 lastPlacedIndex = 1   2不动
第二次循环 旧map中找到1 lastPlacedIndex > oldIndex 1插入
第三次循环 找到4  4不动
2 4 满足index>lastPlacedIndex，则2 4不动
1 标记插入
7 找不到 也标记插入
```

```js
function reconcileChildrenArray(returnFiber, currentFirstChild, newChildren, lanes) {
  var resultingFirstChild = null; // 对比出来结果的第一个节点
  var previousNewFiber = null; // 对比过的节点
  var oldFiber = currentFirstChild;// 老节点

  //第3个for中用到，遍历newNodeList时，找到旧节点的索引oldIndex，
	// oldIndex > lastPlacedIndex 新节点不动，
	// oldIndex < lastPlacedIndex 新节点标记插入
	// oldIndex 不存在，新节点标记插入
  var lastPlacedIndex = 0;
  var newIdx = 0; // newChildren[newIdx]= 当前newNode
  var nextOldFiber = null

  // 第一个for
  // 非mount，update时有旧节点 命中
   for (; oldFiber !== null && newIdx < newChildren.length; newIdx++) {
      if (oldFiber.index > newIdx) {
        nextOldFiber = oldFiber;
        oldFiber = null;
      } else {
        nextOldFiber = oldFiber.sibling;
      }

     // 1. key type同：复用
     // 2. key同，type不同：新建, oldFiber标记删除
     // 3. key不同，不看type： 返回null ——> break，进入第3个for循环
      var newFiber = updateSlot(returnFiber, oldFiber, newChildren[newIdx], lanes);
      if (newFiber === null) {
        ...
        break;
      }

     // 标记删除老节点、 新节点插入的操作
     ...
   }

   if (newIdx === newChildren.length) {
      // 删除多余的 比如 old:abc new: ab  删除c
      // We've reached the end of the new children. We can delete the rest.
      deleteRemainingChildren(returnFiber, oldFiber);
      return resultingFirstChild;
    }

    // 第二个for：新增，2种情况
    // 1. mount阶段，没有old,新增2个新节点
    // 2. old: a b  new: a b c key同，那么进入这里，为新增c
    if (oldFiber === null) {
      for (; newIdx < newChildren.length; newIdx++) {
        var _newFiber = createChild(returnFiber, newChildren[newIdx], lanes);
        if (_newFiber === null) {
          continue;
        }
        lastPlacedIndex = placeChild(_newFiber, lastPlacedIndex, newIdx);
      }

      return resultingFirstChild;
    }

   // 旧节点集合map
    var existingChildren = mapRemainingChildren(returnFiber, oldFiber);
    for (; newIdx < newChildren.length; newIdx++) {
      // 在map中找或创建_newFiber2
      var _newFiber2 = updateFromMap(existingChildren, returnFiber, newIdx, newChildren[newIdx], lanes);

      if (_newFiber2 !== null) {
        if (shouldTrackSideEffects) {
           // 找到了 旧节点存在
          if (_newFiber2.alternate !== null) {
      			// 删除map中找到的
            existingChildren.delete(_newFiber2.key === null ? newIdx : _newFiber2.key);
          }
        }
        // 更新lastPlacedIndex和 插入标记
        lastPlacedIndex = placeChild(_newFiber2, lastPlacedIndex, newIdx);
      }
    }

    if (shouldTrackSideEffects) {
     // map中还存在的标记删除
      existingChildren.forEach(function (child) {
        return deleteChild(returnFiber, child);
      });
    }
}
```

###
