<template>
  <div class="game-board">
    <!-- 棋盘网格 -->
    <div v-for="(row,rowIndex) in grid" :key="rowIndex" class="row">
      <div 
        v-for="(cell,colIndex) in row" 
        class="cell"
        :class="{isSelected : isCellSelected(rowIndex,colIndex)}"
        :key="colIndex"
        @click="selectCell(rowIndex,colIndex)"
      >
        <span v-if="!grid[rowIndex][colIndex]"></span>
        <img v-else :src="getImage(grid[rowIndex][colIndex])" class="cell-image" alt="">
        <div class="cell-halo" v-if="isCellSelected(rowIndex, colIndex)"></div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';

const grid = ref<number[][]>([])
const lastSelectedCell = ref<number[]>([])
const selectCell = (rowIndex: number, colIndex: number) => {
  // 上次选中了
  if(lastSelectedCell.value?.length) {
    if(canConnect(lastSelectedCell.value,[rowIndex,colIndex])) {
      console.log('可以连接~~');
      grid.value[lastSelectedCell.value[0]][lastSelectedCell.value[1]] = 0
      grid.value[rowIndex][colIndex] = 0
    }
    lastSelectedCell.value = []
  } else {
    lastSelectedCell.value = [rowIndex,colIndex]
  }
}
/**
 * 格子是否被选中
 */
const isCellSelected = (rowIndex: number, colIndex: number): boolean => {
  return lastSelectedCell.value[0] === rowIndex && lastSelectedCell.value[1] === colIndex;
};

/**
 * 检查两个格子是否可以连接
 */
const canConnect = (cell1: number[],cell2: number[]): boolean => {
  // 点击同一个格子
  if(cell1[0] === cell2[0] && cell1[1] === cell2[1]) {
    return false
  }
  // 是否值相同
  if(grid.value[cell1[0]][cell1[1]] !== grid.value[cell2[0]][cell2[1]]) {
    return false
  }
  // 路径是否通
  // console.log(isPathConnected(cell1,cell2));
  return isPathConnected(cell1,cell2)
}
/**
 * 使用 BFS 检查路径是否通
 */
const directions = [
  [-1, 0], // 上
  [1, 0],  // 下
  [0, -1], // 左
  [0, 1],  // 右
];
const isPathConnected = (startCell: number[], endCell: number[]): boolean => {
  const visited = new Set<string>();
  const queue: number[][] = [startCell];

  while (queue.length) {
    const [row, col] = queue.shift()!;
    
    // 检查四个方向的相邻格子
    for (const [dx, dy] of directions) {
      const newRow = row + dx;
      const newCol = col + dy;
      const key = `${newRow}-${newCol}`;
      
      if(endCell[0] === newRow && endCell[1] === newCol) return true
      

      if (
        newRow >= 0 &&
        newRow < grid.value.length &&
        newCol >= 0 &&
        newCol < grid.value[0].length &&
        !visited.has(key) &&
        grid.value[newRow][newCol] === 0
      ) {
        queue.push([newRow, newCol]);
        visited.add(key);
      }
    }
  }

  return false; // 没找到路径
};

/**
 * 随机生成grid，确保每一个值都是成对存在的
 */
const randomizeGrid = () => {
  const rows = 10;
  const cols = 10;
  const numPairs = (rows * cols) / 2; // 总共需要的值对数  
  const igrid:number[][] = Array(rows).fill(0).map(() => Array(cols).fill(0))
  const availableCells = []; // 存储可用格子的坐标

  // 初始化可用格子的坐标数组
  for (let i = 0; i < rows; i++) {
    for (let j = 0; j < cols; j++) {
      availableCells.push([i, j]);
    }
  }
  console.log(availableCells);
  

  // 随机生成成对的图案，并填充到grid中
  for (let pair = 1; pair <= numPairs; pair++) {
    // 随机选择一个可用格子
    const cellIndex = Math.floor(Math.random() * availableCells.length);
    const [row, col] = availableCells[cellIndex];

    // 随机生成一个图案（这里假设有4种不同的图案）
    const value = Math.floor(Math.random() *  4+ 1);
    igrid[row][col] = value;
    availableCells.splice(cellIndex, 1);
    
    // 随机选择一个可用格子，将相同图案填充到第二个位置
    const secondCellIndex = Math.floor(Math.random() * availableCells.length);
    const [secondRow, secondCol] = availableCells[secondCellIndex];
    igrid[secondRow][secondCol] = value;

    // 从可用格子数组中移除已经填充的格子
    availableCells.splice(secondCellIndex, 1);
  }
  grid.value = igrid
};

/**
 * 动态引入图片
 */
const getImage = (image:number) =>{
  return new URL(`./assets/mooncake_${image}.png`,import.meta.url).href
}
onMounted(() => {
  randomizeGrid()
})
</script>

<style scoped lang="less">
img {
  width: 100%;
  height: 100%;
}
.game-board {
  background: url('@/assets/bg.jpg') no-repeat;
  background-size: contain;
  .row {
    display: flex;
    .cell {
      width: 60px;
      height: 60px;
      margin: 5px;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      transition: all .3s;
    }
    .cell-image {
      position: relative;
      transition: transform 0.3s ease-in-out;
    }

    /* 定义选中时的动画效果 */
    .isSelected .cell-image {
      transform: scale(1.2); /* 放大 10% */
    }
  }
}
/* 定义光圈效果 */
.cell-halo {
  position: absolute;
  width: 90px;
  height: 90px;
  border: 2px solid rgba(255, 255, 255, 0.5); /* 白色半透明边框 */
  box-sizing: border-box;
  border-radius: 50%; /* 圆形光圈 */
  animation: pulse 1s infinite; /* 循环动画 */
  pointer-events: none; /* 防止光圈阻挡点击事件 */
  z-index: 1;
}

/* 定义光圈的脉冲动画 */
@keyframes pulse {
  0% {
    transform: scale(1);
    opacity: 1;
  }
  50% {
    transform: scale(1.1);
    opacity: 0.7;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}
</style>
