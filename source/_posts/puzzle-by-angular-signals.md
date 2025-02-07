---
title: 使用Angular最新的signals实现拼图游戏
date: 2025-01-12 20:00:51
tags: [Angular, 前端, signals, puzzle, 拼图游戏]
categories:
- 技术博客
- 原创
---

Angular自v16引入signals后到目前已经升级迭代到Angular19，其signals特性经过发展优化，现在基本功能完备，具备在生产项目中使用。今天我们用一个简单的拼图游戏，来实践一下signals的使用。

<!-- more -->

## 拼图游戏效果预览
![](https://s2.loli.net/2025/01/12/n17GxlBWOpHzLCr.gif)

游戏玩法很简单，在一个 `3x3` 的棋盘上，总共9个位置，其中一个留空，用于其他位置移动的空位，剩余8个位置分别被1~8数字标记，点击空白位置附近的格子，可以将标有数字的格子与空白格子交换位置，当所有格子都有数字标记，游戏完成并结束。

由于Angular signals的响应式特性，要实现该拼图游戏，可以说简单不少，这篇博客我们就将其一一拆解，看看如何使用Angular signals实现该拼图游戏。

## 拼图游戏的实现

### 数据模型

首先，我们创建一个数据模型，用来管理拼图游戏的数据，包括棋盘数据，空白格子的位置，以及棋盘上的格子的数字标记等。

首先思考，棋盘上棋子如何移动，以及如何判断最终的输赢。

棋子的移动，在UI上展现出来是移动，但在数据模型上，数据并不一定要移动，只需要对数据进行修改就好了。如何判断最终拼图完成呢，当然也是根据每个棋子是否回到了其“应该在”的位置上。

**应该在的位置**，就得需要两个指标来确定，**棋子目前所在的位置**以及**棋子应该在的位置**。

所以，这里我们采用一个二维数组，来存储每个棋子的数据，包括当前位置，以及应该在的位置。

大致结构如下：

```json
const cells: Cell[][] = [
  [
    { id: 1, value: 1 },
    { id: 2, value: 2 },
    { id: 3, value: 3 }
  ],
  [
    { id: 4, value: 4 },
    { id: 5, value: 5 },
    { id: 6, value: 6 }
  ],
  [
    { id: 7, value: 7 },
    { id: 8, value: 8 },
    { id: 0, value: 0 }
  ]
];
```

结构非常简单，id表示每个棋子应该在的位置，value为棋子在UI上显示的数字，也即其当前所在的位置。
每次在移动棋子时，我们只需要更新当前棋子的value值，与空白棋子的value值进行交换即可。

上面就是初始的数据，1~8为可移动棋子，0为空白格子。

所以，对应在UI上，我们选择对应数据模型的二维数组，使用两个for循环来渲染棋子，采用flex布局即可。

```html
@for (cs of cells(); track $index) {
<div class="w-full flex justify-center items-center">
    @for (cell of cs; track $index) {
    <div class="w-32 h-32 border border-solid border-gray-600 m-[1px] flex justify-center items-center"
    [ngClass]="{ 'bg-orange-300 dark:bg-orange-800 cursor-pointer': cell.value !== 0}" (click)="clickCell(cell)">
    <span class="text-3xl font-bold text-gray-800 dark:text-gray-200">
        {{ cell.value === 0 ? '' : cell.value }}
    </span>
    </div>
    }
</div>
}
```

### 数据逻辑
对于数据模型以及对应的UI布局方式，我们采用了简单的二维数组模式，但如何将具体的操作逻辑实现在数据模型上，是非常重要的。

这里面有几个状态，都是响应式的，根据用户操作拼图，数据状态是联动响应的，比如棋盘上UI的展现，当前用户已操作的步骤数，以及最终的输赢状态等等，这些状态的响应式变化，在Angular的signals中的实现，可以说是轻而易举的。

首先，上面的二维数组模型，我们将其使用signals来定义，这样，其他的状态，都可以通过这个二维数组的变化来同步响应实现。

```ts
const cells: Cell[][] = [
  [
    { id: 1, value: 1 },
    { id: 2, value: 2 },
    { id: 3, value: 3 }
  ],
  [
    { id: 4, value: 4 },
    { id: 5, value: 5 },
    { id: 6, value: 6 }
  ],
  [
    { id: 7, value: 7 },
    { id: 8, value: 8 },
    { id: 0, value: 0 }
  ]
];

// ...
private _cells = signal<Cell[][]>([]);

// ...
this._cells.set(cells);

// ...
finished = computed(() => {
    let finished = true;
    const cells = this.cells();
    for (let i = 0; i < cells.length; i++) {
        for (let j = 0; j < cells[i].length; j++) {
        if (cells[i][j].value !== cells[i][j].id) {
            finished = false;
        }
        }
    }
    return finished;
});
```

处理点击事件（或者监听键盘事件，采用方向键来进行移动棋子操作）时，我们需要找到空白格子的坐标，以及当前点击格子的坐标，先判断点击的格子与空白格式是否相邻，只有两者相邻，才能进行交换操作。

交换操作也很简单，只需要将空白格子与点击的格子的value值进行交换即可。

```ts
clickCell(cell: Cell) {

    // 空白格子不可点击
    if (cell.value === 0) {
      return;
    }

    // 空白格子的坐标
    const zeroCoord = this.boardStore.getZeroCoord();
    // 当前点击格子的坐标
    const { row, col } = this.boardStore.getCoordByValue(cell.value);
    // 检查是否相邻，只有相邻的格子才能交换
    const isAdjacent = Math.abs(zeroCoord.row - row) + Math.abs(zeroCoord.col - col) === 1;
    if (!isAdjacent) {
      return;
    }

    // 交换
    this.move(cell.value, 0);

}

getZeroCoord() {
    for (let i = 0; i < this._cells().length; i++) {
      for (let j = 0; j < this._cells()[i].length; j++) {
        if (this._cells()[i][j].value === 0) {
          return { row: i, col: j };
        }
      }
    }
    return { row: 0, col: 0 };
}

getCoordByValue(value: number) {
    let row = 0, col = 0;
    for (let i = 0; i < this._cells().length; i++) {
      for (let j = 0; j < this._cells()[0].length; j++) {
        if (this.cells()[i][j].value === value) {
          row = i;
          col = j;
        }
      }
    }
    return { row, col };
}

move(fromValue: number, toValue: number) {
    // 移动
    const from = this.boardStore.getCoordByValue(fromValue);
    const to = this.boardStore.getCoordByValue(toValue);
    const cells = this.boardStore.cells().map(row => row.map(cell => ({ ...cell })));
    const temp = cells[from.row][from.col];
    cells[from.row][from.col].value = toValue;
    cells[to.row][to.col].value = fromValue;
    this.boardStore.updateCells(cells);
}
```

简易的拼图游戏，通过以上几个方法即可实现，因为采用了signals，所以几个状态都是响应式的，在用户操作棋子时，不用刻意去计算其他状态，对应的状态会联动发生变化。

## 总结
通过采用signals，我们可以发现，其强大之处便在于，定义的数据状态都是联动的，响应式的，根据原始的signals派生出一些派生状态，这些派生状态会跟随着原始signal联动变化，这无疑简化了我们在实际开发时的复杂度，因为我们不用再去计算其他的状态，只需要关注当前的状态，就可以实现我们的需求了。

> 本博客中的代码并非全部代码，仅仅摘录出核心方法的核心代码，如果有兴趣，可以从[这里](https://github.com/coolcao/my-puzzle)下载完整的代码，一起学习吧！



