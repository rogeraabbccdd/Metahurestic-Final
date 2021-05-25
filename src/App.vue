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
          span(v-if="mm == 2") 開始
          span(v-else-if="mm == 3") 結束
    p 路徑長度: {{ resultPath.length }}
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
            select.form-select#inputS(v-model="heuristic")
              option(v-for="(h, i) in heuristicOptions" :key="i" :value="h.value") {{ h.name }}
        input.btn.btn-primary(type="button" value="產生路徑" @click="getPath")
</template>

<script setup>
import { ref, reactive } from 'vue'
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

const heuristicOptions = reactive([
  { value: 'manhattan', name: '曼哈頓距離' },
  { value: 'chebyshev', name: '柴比雪夫距離' },
  { value: 'euclidean', name: '歐幾里得距離' },
  { value: 'octile', name: 'Octile 距離' }
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
  const finder = new PF.AStarFinder({
    heuristic: PF.Heuristic[heuristic.value]
  })
  const grid = new PF.Grid(matrixClone)
  // find path and remove start/end
  const path = finder.findPath(start.x, start.y, end.x, end.y, grid).slice(1, -1)
  for (const i in path) {
    matrix[path[i][1]][path[i][0]] = 4
  }
  resultPath.show = true
  resultPath.length = path.length
}

</script>
