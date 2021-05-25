<template lang="pug">
.container.text-white
  .col-12.text-center
    h1.my-3 最佳路徑
    table.table.matrix.mx-auto
      tr(v-for="(m, i) in matrix" :key="i")
        td.border(
          v-for="(mm, j) in m" :key="j"
          :class="{blocked: mm == 1, start: mm == 2, end: mm == 3, path: mm == 4 && resultPath.show}"
          @click="block(i, j)"
        )
    p 路徑長度: {{ resultPath.length }}
    span.text-danger 紅色是開始位置
    br
    span.text-success 綠色是結束位置
    br
    span 點擊表格放置黑色無法通過的區塊
    hr
    .row
      .col-6
        h1 格子
        .row.my-3
          label.col-2.col-form-label(for="inputW") 寬度
          .col-10
            input.form-control#inputW(type="number" min="3" max="20" v-model="input.w")
        .row.my-3
          label.col-2.col-form-label(for="inputH") 高度
          .col-10
            input.form-control#inputH(type="number" min="3" max="20" v-model="input.h")
        input.btn.btn-primary(type="button" value="更新格子" @click="generateGrid")
      .col-6
        h1 設定
        .row.my-3
          label.col-2.col-form-label(for="inputS") 演算法
          .col-10
            select.form-select#inputS(v-model="heuristic" :disabled="heuristicDisabled")
              option(v-for="(h, i) in heuristicOptions" :key="i" :value="h.value") {{ h.name }}
        .row.my-3
          label.col-2.col-form-label(for="inputF") 搜尋法
          .col-10
            select.form-select#inputF(v-model="finder")
              option(v-for="(f, i) in finderOptions" :key="i" :value="f.value") {{ f.name }}
        input.btn.btn-primary(type="button" value="產生路徑" @click="getPath")
</template>

<script setup>
import { ref, reactive, computed } from 'vue'
import PF from 'pathfinding'

// 0 = walkable
// 1 = blocked
// 2 = start
// 3 = end
// 4 = path
const matrix = reactive([
    [2, 0, 0, 0, 0],
    [0, 0, 0, 0, 0],
    [0, 0, 0, 0, 3],
])

const heuristic = ref('manhattan')
const finder = ref('AStarFinder')

const heuristicOptions = reactive([
  { value: 'manhattan', name: '曼哈頓距離' },
  { value: 'chebyshev', name: '柴比雪夫距離' },
  { value: 'euclidean', name: '歐幾里得距離' },
  { value: 'octile', name: 'Octile 距離' }
])

const finderOptions = reactive([
  { value: 'AStarFinder', name: 'A*' },
  { value: 'BestFirstFinder', name: '最良優先搜尋' },
  { value: 'BreadthFirstFinder', name: '廣度優先搜尋' },
  { value: 'DijkstraFinder', name: 'dijkstra' },
  { value: 'JumpPointFinder', name: '跳點搜索' },
  { value: 'BiAStarFinder', name: '雙向 A*' },
  { value: 'BiBestFirstFinder', name: '雙向最良優先搜尋' },
  { value: 'BiBreadthFirstFinder', name: '雙向廣度優先搜尋' },
  { value: 'BiDijkstraFinder', name: '雙向 dijkstra' },
])

const resultPath = reactive({show: false, length: 0})

const input = reactive({w: 5, h: 3})

const block = (i, j) => {
  if (resultPath.show) {
    resultPath.show = false
    resultPath.length = 0
  }

  if (matrix[i][j] === 1) {
    matrix[i][j] = 0
  } else if (matrix[i][j] !== 2 && matrix[i][j] !== 3) {
    matrix[i][j] = 1
  }
}

const generateGrid = () => {
  matrix.splice(0, matrix.length)
  for(let i = 0; i<input.h; i++) {
    const arr = []
    for(let j = 0; j<input.w; j++) {
      arr.push(0)
    }
    matrix.push(arr)
  }
  matrix[0][0] = 2
  matrix[input.h-1][input.w-1] = 3
}

const getPath = () => {
  const start = {x: 0, y: 0}
  const end = {x: 0, y: 0}
  const matrixClone = JSON.parse(JSON.stringify(matrix))
  for (const i in  matrixClone) {
    for (const j in  matrixClone[i]) {
      if (matrixClone[i][j] == 2) {
        start.x = j
        start.y = i
        matrixClone[i][j] = 0
      } else if (matrixClone[i][j] == 3) {
        end.x = j
        end.y = i
        matrixClone[i][j] = 0
      } else if (matrixClone[i][j] == 4) {
        matrixClone[i][j] = 0
        matrix[i][j] = 0
      }
    }
  }
  const pfinder = new PF[finder.value]({
    heuristic: PF.Heuristic[heuristic.value]
  })
  const grid = new PF.Grid(matrixClone)
  // find path and remove start/end
  const path = pfinder.findPath(start.x, start.y, end.x, end.y, grid).slice(1, -1)
  for (const i in path) {
    matrix[path[i][1]][path[i][0]] = 4
  }
  resultPath.show = true
  resultPath.length = path.length
}

const heuristicDisabled = computed(() => {
  if(finder.value === 'AStarFinder' || finder.value === 'BestFirstFinder' ||
    finder.value === 'BiAStarFinder' || finder.value === 'BiBestFirstFinder'
  ) {
    return false
  }
  return true
})
</script>
