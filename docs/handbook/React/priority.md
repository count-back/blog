---
title: React——优先级
date: 2023-08-29
author: 倒数
tags:
  - react
categories:
  - react
isShowComments: true
subSidebar: auto
---

# 产生更新

当调用 setState 时，意味着组件对应的 fiber 节点产生了一个更新。setState 实际上是生成一个 update 对象，调用 enqueueSetState，将这个 update 对象连接到 fiber 节点的 updateQueue 链表中.

```javascript
Component.prototype.setState = function (partialState, callback) {
  this.updater.enqueueSetState(this, partialState, callback, "setState");
};
```

enqueueSetState 的职责是创建 update 对象，将它入队 fiber 节点的 update 链表（updateQueue），然后发起调度。

```javascript
  enqueueSetState(inst, payload, callback) {
    // 获取当前触发更新的fiber节点。inst是组件实例
    const fiber = getInstance(inst);
    // eventTime是当前触发更新的时间戳
    const eventTime = requestEventTime();
    const suspenseConfig = requestCurrentSuspenseConfig();

    // 获取本次update的优先级
    const lane = requestUpdateLane(fiber, suspenseConfig);

    // 创建update对象
    const update = createUpdate(eventTime, lane, suspenseConfig);

    // payload就是setState的参数，回调函数或者是对象的形式。
    // 处理更新时参与计算新状态的过程
    update.payload = payload;

    // 将update放入fiber的updateQueue
    enqueueUpdate(fiber, update);

    // 开始进行调度
    scheduleUpdateOnFiber(fiber, lane, eventTime);
  }
```

梳理一下 enqueueSetState 中具体做的事情：

**找到 fiber**

首先获取产生更新的组件所对应的 fiber 节点，因为产生的 update 对象需要放到 fiber 节点的 updateQueue 上。然后获取当前这个 update 产生的时间，这与更新的饥饿问题相关，暂且不考虑，而且下一步的 suspenseConfig 可以先忽略。

**计算优先级**

之后比较重要的是计算当前这个更新它的优先级 lane：

```javascript
const lane = requestUpdateLane(fiber, suspenseConfig);
```

计算这个优先级的时候，是如何决定根据什么东西去计算呢？这还得从 React 的合成事件说起。

事件触发时，合成事件机制调用 scheduler 中的 runWithPriority 函数，目的是以该交互事件对应的事件优先级去派发真正的事件流程。runWithPriority 会将事件优先级转化为 scheduler 内部的优先级并记录下来。当调用 requestUpdateLane 计算 lane 的时候，会去获取 scheduler 中的优先级，以此作为 lane 计算的依据。

**创建 update 对象， 入队 updateQueue**

根据 lane 和 eventTime 还有 suspenseConfig，去创建一个 update 对象，结构如下：

```javascript
const update: Update<*> = {
  eventTime,
  lane,
  suspenseConfig,
  tag: UpdateState,
  payload: null,
  callback: null,
  next: null,
};
```

- eventTime：更新的产生时间
- lane：表示优先级
- suspenseConfig：任务挂起相关
- tag：表示更新是哪种类型（UpdateState，ReplaceState，ForceUpdate，CaptureUpdate）
- payload：更新所携带的状态。
  - 在类组件中，有两种可能，对象（{}），和函数（(prevState, nextProps):newState => {}）
  - 根组件中，为 React.element，即 ReactDOM.render 的第一个参数
- callback：可理解为 setState 的回调
- next：指向下一个 update 的指针

再之后就是去调用 React 任务执行的入口函数：`scheduleUpdateOnFiber`去调度执行更新任务了。

现在知道了，产生更新的 fiber 节点上会有一个 updateQueue，它包含了刚刚产生的 update。下面该进入`scheduleUpdateOnFiber`了，开始进入真正的调度流程。通过调用`scheduleUpdateOnFiber`，render 阶段的构建 workInProgress 树的任务会被调度执行，这个过程中，fiber 上的 updateQueue 会被处理。

# 调度准备

React 的更新入口是`scheduleUpdateOnFiber`，它区分 update 的 lane，将同步更新和异步更新分流，让二者进入各自的流程。但在此之前，它会做几个比较重要的工作：

- 检查是否是无限更新，例如在 render 函数中调用了 setState。
- 从产生更新的节点开始，往上一直循环到 root，目的是将 fiber.lanes 一直向上收集，收集到父级节点的 childLanes 中，childLanes 是识别这个 fiber 子树是否需要更新的关键。
- 在 root 上标记更新，也就是将 update 的 lane 放到 root.pendingLanes 中，每次渲染的优先级基准：renderLanes 就是取自 root.pendingLanes 中最紧急的那一部分 lanes。
  这三步可以视为更新执行前的准备工作。

第 1 个可以防止死循环卡死的情况。

第 2 个，如果 fiber.lanes 不为空，则说明该 fiber 节点有更新，而 fiber.childLanes 是判断当前子树是否有更新的重要依据，若有更新，则继续向下构建，否则直接复用已有的 fiber 树，就不往下循环了，可以屏蔽掉那些无需更新的 fiber 节点。

第 3 个是将当前 update 对象的 lane 加入到 root.pendingLanes 中，保证真正开始做更新任务的时候，获取到 update 的 lane，从而作为本次更新的渲染优先级（renderLanes），去更新。

> 实际上在更新时候获取到的 renderLanes，并不一定包含 update 对象的 lane，因为有可能它只是一个较低优先级的更新，有可能在它前面有高优先级的更新

梳理完`scheduleUpdateOnFiber`的大致逻辑之后，来看一下它的源码：

```javascript
export function scheduleUpdateOnFiber(
  fiber: Fiber,
  lane: Lane,
  eventTime: number,
) {
  // 第一步，检查是否有无限更新
  checkForNestedUpdates();

  ...
  // 第二步，向上收集fiber.childLanes
  const root = markUpdateLaneFromFiberToRoot(fiber, lane);

  ...

  // 第三步，在root上标记更新，将update的lane放到root.pendingLanes
  markRootUpdated(root, lane, eventTime);

  ...

  // 根据Scheduler的优先级获取到对应的React优先级
  const priorityLevel = getCurrentPriorityLevel();

  if (lane === SyncLane) {
    // 本次更新是同步的，例如传统的同步渲染模式
    if (
      (executionContext & LegacyUnbatchedContext) !== NoContext &&
      (executionContext & (RenderContext | CommitContext)) === NoContext
    ) {
      // 如果是本次更新是同步的，并且当前还未渲染，意味着主线程空闲，并没有React的
      // 更新任务在执行，那么调用performSyncWorkOnRoot开始执行同步任务

      ...

      performSyncWorkOnRoot(root);
    } else {
      // 如果是本次更新是同步的，不过当前有React更新任务正在进行，
      // 而且因为无法打断，所以调用ensureRootIsScheduled
      // 目的是去复用已经在更新的任务，让这个已有的任务
      // 把这次更新顺便做了
      ensureRootIsScheduled(root, eventTime);
      ...
    }
  } else {

    ...

    // Schedule other updates after in case the callback is sync.
    // 如果是更新是异步的，调用ensureRootIsScheduled去进入异步调度
    ensureRootIsScheduled(root, eventTime);
    schedulePendingInteractions(root, lane);
  }

  ...
}
```

经过了前面的准备工作后，`scheduleUpdateOnFiber`最终会调用`ensureRootIsScheduled`，来让 React 任务被调度，这是一个非常重要的函数，它关乎**同等或较低任务的收敛**、
**高优先级任务插队**和**任务饥饿问题**，下面详细讲解它。

# 开始调度

在开始讲解`ensureRootIsScheduled`之前，有必要弄清楚 React 的更新任务的本质。

## React 任务的本质

一个 update 的产生最终会使 React 在内存中根据现有的 fiber 树构建一棵新的 fiber 树，新的 state 的计算、diff 操作、以及一些生命周期的调用，都会在这个构建过程中进行。这个整体的构建工作被称为 render 阶段，这个 render 阶段整体就是一个完整的 React 更新任务，更新任务可以看作执行一个函数，这个函数在 concurrent 模式下就是`performConcurrentWorkOnRoot`，更新任务的调度可以看成是这个函数被 scheduler 按照任务优先级安排它何时执行。

> Scheduler 的调度和 React 的调度是两个完全不同的概念，React 的调度是协调任务进入哪种 Scheduler 的调度模式，它的调度并不涉及任务的执行，而 Scheduler 是调度机制的真正核心，它是实打实地去执行任务，没有它，React 的任务再重要也无法执行，希望读者加以区分这两种概念。

当一个任务被调度之后，scheduler 就会生成一个任务对象（task），它的结构如下所示，除了 callback 之外暂时不需要关心其他字段的含义。

```javascript
var newTask = {
  id: taskIdCounter++,
  // 任务函数，也就是 performConcurrentWorkOnRoot
  callback,
  // 任务调度优先级，由即将讲到的任务优先级转化而来
  priorityLevel,
  // 任务开始执行的时间点
  startTime,
  // 任务的过期时间
  expirationTime,
  // 在小顶堆任务队列中排序的依据
  sortIndex: -1,
};
```

每当生成了一个这样的任务，它就会被挂载到 root 节点的`callbackNode`属性上，以表示当前已经有任务被调度了，同时会将任务优先级存储到 root 的`callbackPriority`上，
表示如果有新的任务进来，必须用它的任务优先级和已有任务的优先级（root.callbackPriority）比较，来决定是否有必要取消已经有的任务。

所以在调度任务的时候，任务优先级是不可或缺的一个重要角色。

## 任务优先级

任务本身是由更新产生的，因此任务优先级本质上是和 update 的优先级，即 update.lane 有关（只是有关，不一定是由它而来）。得出的任务优先级属于 lanePriority，它不是 update 的 lane，而且与 scheduler 内部的调度优先级是两个概念

在 **调度准备** 的最后提到过，update.lane 会被放入 root.pendingLanes，随后会获取 root.pendingLanes 中最优先级的那些 lanes 作为 renderLanes。任务优先级的生成就发生在计算 renderLanes 的阶段，**任务优先级其实就是 renderLanes 对应的 lanePriority**。因为 renderLanes 是本次更新的优先级基准，所以它对应的 lanePriority 被作为任务优先级来衡量本次更新任务的优先级权重理所应当。

> root.pendingLanes，包含了当前 fiber 树中所有待处理的 update 的 lane。

任务优先级有三类：

- 同步优先级：React 传统的同步渲染模式产生的更新任务所持有的优先级
- 同步批量优先级：同步模式到 concurrent 模式过渡模式：blocking 模式产生的更新任务所持有的优先级
- concurrent 模式下的优先级：concurrent 模式产生的更新持有的优先级

最右面的两个 lane 分别为同步优先级和同步批量优先级，剩下左边的 lane 几乎所有都和 concurrent 模式有关。

```
export const SyncLane: Lane = /*                        */ 0b0000000000000000000000000000001;
export const SyncBatchedLane: Lane = /*                 */ 0b0000000000000000000000000000010;

concurrent模式下的lanes：/*                               */ 0b1111111111111111111111111111100;
```

计算 renderLanes 的函数是`getNextLanes`，生成任务优先级的函数是`getHighestPriorityLanes`

> 任务优先级决定着任务在 React 中被如何调度，而由任务优先级转化成的任务调度优先级（上面给出的 scheduler 的 task 结构中的 priorityLevel），
> 决定着 Scheduler 何时去处理这个任务。

## 任务调度协调 - ensureRootIsScheduled

目前为止了解了任务和任务优先级的本质，下面正式进入任务的调度过程。React 这边对任务的调度本质上其实是以任务优先级为基准，去操作多个或单个任务。

多个任务的情况，相对于新任务，会对现有任务进行**或复用，或取消**的操作，单个任务的情况，对任务进行**或同步，或异步，或批量同步（暂时不需要关注）** 的调度决策，
这种行为可以看成是一种任务调度协调机制，这种协调通过`ensureRootIsScheduled`去实现。

看一看`ensureRootIsScheduled`函数做的事情，先是准备本次任务调度协调所需要的 lanes 和任务优先级，然后判断是否真的需要调度

- 获取 root.callbackNode，即旧任务
- 检查任务是否过期，将过期任务放入 root.expiredLanes，目的是让过期任务能够以同步优先级去进入调度（立即执行）
- 获取 renderLanes（优先从 root.expiredLanes 获取），如果 renderLanes 是空的，说明不需要调度，直接 return 掉
- 获取本次任务，即新任务的优先级：newCallbackPriority
  接下来是协调任务调度的过程：
- 首先判断是否有必要发起一次新调度，方法是通过比较新任务的优先级和旧任务的优先级是否相等：
  - 相等，则说明无需再次发起一次调度，直接复用旧任务即可，让旧任务在处理更新的时候顺便把新任务给做了。
  - 不相等，则说明新任务的优先级一定高于旧任务，这种情况就是**高优先级任务插队**，需要把旧任务取消掉。
- 真正发起调度，看新任务的任务优先级：
  - 同步优先级：调用 scheduleSyncCallback 去同步执行任务。
  - 同步批量执行：调用 scheduleCallback 将任务以立即执行的优先级去加入调度。
  - 属于 concurrent 模式的优先级：调用 scheduleCallback 将任务以上面获取到的新任务优先级去加入调度。

这里有两点需要说明：

1. 为什么新旧任务的优先级如果不相等，那么新任务的优先级一定高于旧任务？
   这是因为每次调度去获取任务优先级的时候，都只获取 root.pendingLanes 中最紧急的那部分 lanes 对应的优先级，低优先级的 update 持有的 lane 对应的优先级是无法被获取到的。通过这种办法，可以将来自同一事件中的多个更新收敛到一个任务中去执行，言外之意就是同一个事件触发的多次更新的优先级是一样的，没必要发起多次任务调度。例如在一个事件中多次调用 setState：

```javascript
class Demo extends React.Component {
  state = {
    count: 0,
  };

  onClick = () => {
    this.setState({ count: 1 });
    this.setState({ count: 2 });
  };

  render() {
    return <button onClick={onClick}>{this.state.count}</button>;
  }
}
```

页面上会直接显示出 2，虽然 onClick 事件调用了两次 setState，但只会引起一次调度，设置 count 为 2 的那次调度被因为优先级与设置 count 为 1 的那次任务的优先级相同，
所以没有去再次发起调度，而是复用了已有任务。这是 React17 对于多次 setState 优化实现的改变，之前是通过 batchingUpdate 这种机制实现的。

2. 三种任务优先级的调度模式有何区别，行为表现上如何？

- 同步优先级：传统的 React 同步渲染模式和过期任务的调度。通过 React 提供的`scheduleSyncCallback`函数将任务函数**performSyncWorkOnRoot**加入到 React 自己的同步队列（syncQueue）中，之后以 ImmediateSchedulerPriority 的优先级将循环执行 syncQueue 的函数加入到 scheduler 中，目的是让任务在下一次事件循环中被执行掉。但是因为 React 的控制，这种模式下的时间片会在任务都执行完之后再去检查，表现为没有时间片。
- 同步批量执行：同步渲染模式到 concurrent 渲染模式的过渡模式 blocking 模式，会将任务函数**performSyncWorkOnRoot**以 ImmediateSchedulerPriority 的优先级加入到 scheduler 中，也是让任务在下一次事件循环中被执行掉，也不会有时间片的表现。
- 属于 concurrent 模式的优先级：将任务函数**performConcurrentWorkOnRoot**以任务自己的优先级加入到 scheduler 中，scheduler 内部的会通过这个优先级控制该任务在 scheduler 内部任务队列中的排序，从而决定任务合适被执行，而且任务真正执行时会有时间片的表现，可以发挥出 scheduler 异步可中断调度的真正威力。

> 要注意一点，用来做新旧任务比较的优先级与这里将任务加入到 scheduler 中传入的优先级不是一个，后者可由前者通过`lanePriorityToSchedulerPriority`转化而来。

经过以上的分析，相信大家已经对`ensureRootIsScheduled`的运行机制比较清晰了，现在看一下它的实现：

```javascript
function ensureRootIsScheduled(root: FiberRoot, currentTime: number) {
  // 获取旧任务
  const existingCallbackNode = root.callbackNode;

  // 记录任务的过期时间，检查是否有过期任务，有则立即将它放到root.expiredLanes，
  // 便于接下来将这个任务以同步模式立即调度
  markStarvedLanesAsExpired(root, currentTime);

  // 获取renderLanes
  const nextLanes = getNextLanes(
    root,
    root === workInProgressRoot ? workInProgressRootRenderLanes : NoLanes
  );

  // 获取renderLanes对应的任务优先级
  const newCallbackPriority = returnNextLanesPriority();

  if (nextLanes === NoLanes) {
    // 如果渲染优先级为空，则不需要调度
    if (existingCallbackNode !== null) {
      cancelCallback(existingCallbackNode);
      root.callbackNode = null;
      root.callbackPriority = NoLanePriority;
    }
    return;
  }

  // 如果存在旧任务，那么看一下能否复用
  if (existingCallbackNode !== null) {
    // 获取旧任务的优先级
    const existingCallbackPriority = root.callbackPriority;

    // 如果新旧任务的优先级相同，则无需调度
    if (existingCallbackPriority === newCallbackPriority) {
      return;
    }
    // 代码执行到这里说明新任务的优先级高于旧任务的优先级
    // 取消掉旧任务，实现高优先级任务插队
    cancelCallback(existingCallbackNode);
  }

  // 调度一个新任务
  let newCallbackNode;
  if (newCallbackPriority === SyncLanePriority) {
    // 若新任务的优先级为同步优先级，则同步调度，传统的同步渲染和过期任务会走这里
    newCallbackNode = scheduleSyncCallback(
      performSyncWorkOnRoot.bind(null, root)
    );
  } else if (newCallbackPriority === SyncBatchedLanePriority) {
    // 同步模式到concurrent模式的过渡模式：blocking模式会走这里
    newCallbackNode = scheduleCallback(
      ImmediateSchedulerPriority,
      performSyncWorkOnRoot.bind(null, root)
    );
  } else {
    // concurrent模式的渲染会走这里

    // 根据任务优先级获取Scheduler的调度优先级
    const schedulerPriorityLevel =
      lanePriorityToSchedulerPriority(newCallbackPriority);

    // 计算出调度优先级之后，开始让Scheduler调度React的更新任务
    newCallbackNode = scheduleCallback(
      schedulerPriorityLevel,
      performConcurrentWorkOnRoot.bind(null, root)
    );
  }

  // 更新root上的任务优先级和任务，以便下次发起调度时候可以获取到
  root.callbackPriority = newCallbackPriority;
  root.callbackNode = newCallbackNode;
}
```

`ensureRootIsScheduled`实际上是在任务调度层面整合了高优先级任务的插队和任务饥饿问题的关键逻辑，这只是宏观层面的决策，决策背后的原因是 React 处理更新时
对于不同优先级的 update 的取舍以及对 root.pendingLanes 的标记操作，这需要下沉到执行更新任务的过程中。

# 处理更新

一旦有更新产生，update 对象就会被放入 updateQueue 并挂载到 fiber 节点上。构建 fiber 树时，会带着 renderLanes 去处理 updateQueue，在 beginWork 阶段，对于类组件
会调用`processUpdateQueue`函数，逐个处理这个链表上的每个 update 对象，计算新的状态，一旦 update 持有的优先级不够，那么就会跳过这个 update 的处理，并把这个被跳过的 update 的 lane 放到 fiber.lanes 中，好在 completeWork 阶段收集起来。
关于优先级的部分比较好理解，就是只处理优先级足够的 update，跳过那些优先级不足的 update，并且将这些 update 的 lane 放到 fiber.lanes 中。直接来看一下实现：

```javascript
function processUpdateQueue<State>(
  workInProgress: Fiber,
  props: any,
  instance: any,
  renderLanes: Lanes,
): void {

  ...

  if (firstBaseUpdate !== null) {
    let update = firstBaseUpdate;
    do {
      const updateLane = update.lane;
      // isSubsetOfLanes函数的意义是，判断当前更新的优先级（updateLane）
      // 是否在渲染优先级（renderLanes）中如果不在，那么就说明优先级不足
      if (!isSubsetOfLanes(renderLanes, updateLane)) {

        ...

        /*
        *
        * newLanes会在最后被赋值到workInProgress.lanes上，而它又最终
        * 会被收集到root.pendingLanes。
        *
        * 再次更新时会从root上的pendingLanes中找出应该在本次中更新的优先
        * 级（renderLanes），renderLanes含有本次跳过的优先级，再次进入，
        * processUpdateQueue wip的优先级符合要求，被更新掉，低优先级任务
        * 因此被重做
        * */
        newLanes = mergeLanes(newLanes, updateLane);
      } else {

        // 优先级足够，去计算state
        ...

      }
    } while (true);

    // 将newLanes赋值给workInProgress.lanes，
    // 就是将被跳过的update的lane放到fiber.lanes
    workInProgress.lanes = newLanes;

  }
}

```

只处理优先级足够的 update 是让高优先级任务被执行掉的最本质原因，在循环了一次 updateQueue 之后，那些被跳过的 update 的 lane 又被放入了 fiber.lanes，现在，只需要将它放到 root.pendingLanes 中，就能表示在本轮更新后，仍然有任务未被处理，从而实现低优先级任务被重新调度。所以接下来的过程就是 fiber 节点的完成阶段：completeWork 阶段去收集这些 lanes。

# 收集未被处理的 lane

在 completeUnitOfWork 的时候，fiber.lanes 和 childLanes 被一层一层收集到父级 fiber 的 childLanes 中，该过程发生在`completeUnitOfWork`函数中调用的`resetChildLanes`，它循环 fiber 节点的子树，将子节点及其兄弟节点中的 lanes 和 childLanes 收集到当前正在 complete 阶段的 fiber 节点上的 childLanes。

假设第 3 层中的`<List/>`和`<Table/>`组件都分别有 update 因为优先级不够而被跳过，那么在它们父级的 div fiber 节点 completeUnitOfWork 的时候，会调用`resetChildLanes`
把它俩的 lanes 收集到 div fiber.childLanes 中，最终把所有的 lanes 收集到 root.pendingLanes.

```
                                    root（pendingLanes: 0b01110）
                                     |
  1                                  App
                                     |
                                     |
  2 compeleteUnitOfWork-----------> div （childLanes: 0b01110）
                                     /
                                    /
  3                              <List/> ---------> <Table/> --------> p
                            （lanes: 0b00010）   （lanes: 0b00100）
                         （childLanes: 0b01000）       /
                                 /                   /
                                /                   /
  4                            p                   ul
                                                  /
                                                 /
                                                li ------> li
```

在每一次往上循环的时候，都会调用 resetChildLanes，目的是将 fiber.childLanes 层层收集。

```javascript
function completeUnitOfWork(unitOfWork: Fiber): void {
  // 已经结束beginWork阶段的fiber节点被称为completedWork
  let completedWork = unitOfWork;

  do {
    // 向上一直循环到root的过程
    ...

    // fiber节点的.flags上没有Incomplete，说明是正常完成了工作
    if ((completedWork.flags & Incomplete) === NoFlags) {

      ...
      // 调用resetChildLanes去收集lanes
      resetChildLanes(completedWork);

      ...

    } else {/*...*/}

    ...

  } while (completedWork !== null);

  ...

}
```

resetChildLanes 中只收集当前正在 complete 的 fiber 节点的子节点和兄弟节点的 lanes 以及 childLanes：

```javascript
function resetChildLanes(completedWork: Fiber) {

  ...

  let newChildLanes = NoLanes;

  if (enableProfilerTimer && (completedWork.mode & ProfileMode) !== NoMode) {
    // profile相关，无需关注
  } else {
    // 循环子节点和兄弟节点，收集lanes
    let child = completedWork.child;
    while (child !== null) {
      // 收集过程
      newChildLanes = mergeLanes(
        newChildLanes,
        mergeLanes(child.lanes, child.childLanes),
      );
      child = child.sibling;
    }
  }
  // 将收集到的lanes放到该fiber节点的childLanes中
  completedWork.childLanes = newChildLanes;
}
```

最后将这些收集到的 childLanes 放到 root.pendingLanes 的过程，是发生在本次更新的 commit 阶段中，因为 render 阶段的渲染优先级来自 root.pendingLanes，不能随意地修改它。所以要在 render 阶段之后的 commit 阶段去修改。看一下 commitRootImpl 中这个过程的实现：

```javascript
function commitRootImpl(root, renderPriorityLevel) {

  // 将收集到的childLanes，连同root自己的lanes，一并赋值给remainingLanes
  let remainingLanes = mergeLanes(finishedWork.lanes, finishedWork.childLanes);
  // markRootFinished中会将remainingLanes赋值给remainingLanes
  markRootFinished(root, remainingLanes);

  ...

}
```

# 重新发起调度

至此，低优先级任务的 lane 重新收集到了 root.pendingLanes 中，这时只需要再发起一次调度就可以了，通过在 commit 阶段再次调用`ensureRootIsScheduled`去实现，这样就又会走一遍调度的流程，低优先级任务被执行。

```javascript
function commitRootImpl(root, renderPriorityLevel) {

  // 将收集到的childLanes，连同root自己的lanes，一并赋值给remainingLanes
  let remainingLanes = mergeLanes(finishedWork.lanes, finishedWork.childLanes);
  // markRootFinished中会将remainingLanes赋值给remainingLanes
  markRootFinished(root, remainingLanes);

  ...

  // 在每次所有更新完成的时候都会调用这个ensureRootIsScheduled
  // 以保证root上任何的pendingLanes都能被处理
  ensureRootIsScheduled(root, now());

}
```
