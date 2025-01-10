<script setup lang="ts">
import { onKeyStroke, useSwipe } from '@vueuse/core'
const RefBoard = ref()

const bgColor = shallowRef({
  2: '#EEE4DA',
  4: '#EFE0C9',
  8: '#F3B27A',
  16: '#F69664',
  32: '#F77C5F',
  64: '#F75E3B',
  128: '#EDD073',
  256: '#EDCB62',
  512: '#EDC950',
})

const color = shallowRef({
  2: '#776E65',
  4: '#776E65',
})

const isGameOver = ref(false)
const isWin = ref(false)
const score = ref(0)
const bestScore = ref(0)
const boards = ref<any>([
  [null, null, null, null],
  [null, null, null, null],
  [null, null, null, null],
  [null, null, null, null],
])

const positionNewVal = ref<any>({
  i: null,
  j: null,
})

const getColor = computed(() => {
  return (val) => color.value[val] ?? '#F8F6F2'
})

const getBgColor = computed(() => {
  return (val) => bgColor.value[val] ?? '#EDC22F'
})

const getFadeIn = computed(() => {
  return ({ i, j }) => {
    if (i === positionNewVal.value.i && j === positionNewVal.value.j) {
      return 'fade-in'
    }
    return ''
  }
})

function saveLocal() {
  localStorage.setItem(
    '2048',
    JSON.stringify({
      boards: boards.value,
      isGameOver: isGameOver.value,
      score: score.value,
      bestScore: bestScore.value,
    }),
  )
}
function randomBoard() {
  const vals = [2, 4]
  const max = boards.value.length - 1
  const min = 0

  let i = Math.floor(Math.random() * (max - min + 1)) + min
  let j = Math.floor(Math.random() * (max - min + 1)) + min
  boards.value[i][j] = vals[Math.floor(Math.random() * (1 - min + 1)) + min]
  positionNewVal.value = { i, j }

  let flag = true
  while (flag) {
    i = Math.floor(Math.random() * (max - min + 1)) + min
    j = Math.floor(Math.random() * (max - min + 1)) + min
    if (boards.value[i][j] === null) {
      boards.value[i][j] = vals[Math.floor(Math.random() * (1 - min + 1)) + min]
      positionNewVal.value = { i, j }
      flag = false
    }
  }
  saveLocal()
}

function gameOver() {
  let flag = true
  for (let i = 0; i <= boards.value.length - 1; i++) {
    for (let j = 0; j <= boards.value.length - 1; j++) {
      if (boards.value[i][j]) {
        const up = i + 1
        const down = i - 1
        const right = j + 1
        const left = j - 1
        const length = boards.value.length - 1

        if (up >= 0 && up <= length) {
          if (boards.value[i][j] === boards.value[up][j]) {
            flag = false
            break
          }
        }
        if (down >= 0 && down <= length) {
          if (boards.value[i][j] === boards.value[down][j]) {
            flag = false
            break
          }
        }
        if (right >= 0 && right <= length) {
          if (boards.value[i][j] === boards.value[i][right]) {
            flag = false
            break
          }
        }
        if (left >= 0 && left <= length) {
          if (boards.value[i][j] === boards.value[i][left]) {
            flag = false
            break
          }
        }
      } else {
        flag = false
        break
      }
    }
    if (!flag) break
  }
  isGameOver.value = flag
  saveLocal()
}

function addVal(boards: any) {
  const positions: any = []
  boards.forEach((rows, i) => {
    rows.forEach((val, j) => {
      if (val === null) positions.push({ i, j })
    })
  })

  if (positions.length === 0) return boards

  const max = positions.length - 1
  const min = 0
  const position = Math.floor(Math.random() * (max - min + 1)) + min
  const { i, j } = positions[position]
  positionNewVal.value = { i, j }

  const vals = [2, 4]
  boards[i][j] = vals[Math.floor(Math.random() * (1 - min + 1)) + min]
  return boards
}

function newGame() {
  boards.value = [
    [null, null, null, null],
    [null, null, null, null],
    [null, null, null, null],
    [null, null, null, null],
  ]
  isGameOver.value = false
  isWin.value = false
  score.value = 0
  nextTick(() => {
    randomBoard()
  })
}

function upAction() {
  if (isGameOver.value || isWin.value) return false
  const _boards = JSON.parse(JSON.stringify(boards.value))
  let VAL = null
  let FLAG_NEW_VAL = false
  for (let j = 0; j <= _boards.length - 1; j++) {
    for (let i = 0; i <= _boards.length - 1; i++) {
      //
      for (let down = i + 1; down <= _boards.length - 1; down++) {
        if (_boards[down][j]) {
          if (_boards[i][j] === _boards[down][j]) {
            _boards[i][j] += _boards[down][j]
            VAL = _boards[i][j]
            score.value += _boards[i][j]
            if (score.value > bestScore.value) bestScore.value = score.value
            _boards[down][j] = null
            FLAG_NEW_VAL = true
          }
          break
        }
      }

      if (_boards[i][j]) {
        let moveI: null | number = null
        for (let up = i - 1; up >= 0; up--) {
          if (_boards[up][j]) break
          if (_boards[up][j] === null) moveI = up
        }
        if (moveI !== null) {
          FLAG_NEW_VAL = true
          _boards[moveI][j] = _boards[i][j]
          _boards[i][j] = null
        }
      }
    }
  }

  if (FLAG_NEW_VAL) {
    if (VAL === 2048) isWin.value = true
    boards.value = addVal(_boards)
  } else boards.value = _boards
  gameOver()
}

function rightAction() {
  if (isGameOver.value || isWin.value) return false
  const _boards = JSON.parse(JSON.stringify(boards.value))
  let VAL = null
  let FLAG_NEW_VAL = false
  for (let i = 0; i <= _boards.length - 1; i++) {
    for (let j = _boards.length - 1; j >= 0; j--) {
      //
      for (let left = j - 1; left >= 0; left--) {
        if (_boards[i][left]) {
          if (_boards[i][left] === _boards[i][j]) {
            _boards[i][j] += _boards[i][left]
            VAL = _boards[i][j]
            score.value += _boards[i][j]
            if (score.value > bestScore.value) bestScore.value = score.value
            _boards[i][left] = null
            FLAG_NEW_VAL = true
          }
          break
        }
      }

      if (_boards[i][j]) {
        let moveJ: null | number = null
        for (let right = j + 1; right <= _boards.length - 1; right++) {
          if (_boards[i][right]) break
          if (_boards[i][right] === null) moveJ = right
        }
        if (moveJ !== null) {
          _boards[i][moveJ] = _boards[i][j]
          _boards[i][j] = null
          FLAG_NEW_VAL = true
        }
      }
    }
  }

  if (FLAG_NEW_VAL) {
    if (VAL === 2048) isWin.value = true
    boards.value = addVal(_boards)
  } else boards.value = _boards
  gameOver()
}

function downAction() {
  if (isGameOver.value || isWin.value) return false
  const _boards = JSON.parse(JSON.stringify(boards.value))
  let VAL = null
  let FLAG_NEW_VAL = false
  for (let j = 0; j <= _boards.length - 1; j++) {
    for (let i = _boards.length - 1; i >= 0; i--) {
      //
      for (let up = i - 1; up >= 0; up--) {
        if (_boards[up][j]) {
          if (_boards[i][j] === _boards[up][j]) {
            _boards[i][j] += _boards[up][j]
            VAL = _boards[i][j]
            score.value += _boards[i][j]
            if (score.value > bestScore.value) bestScore.value = score.value
            _boards[up][j] = null
            FLAG_NEW_VAL = true
          }
          break
        }
      }

      if (_boards[i][j]) {
        let moveI: null | number = null
        for (let down = i + 1; down <= _boards.length - 1; down++) {
          if (_boards[down][j]) break
          if (_boards[down][j] === null) moveI = down
        }

        if (moveI !== null) {
          _boards[moveI][j] = _boards[i][j]
          _boards[i][j] = null
          FLAG_NEW_VAL = true
        }
      }
    }
  }

  if (FLAG_NEW_VAL) {
    if (VAL === 2048) isWin.value = true
    boards.value = addVal(_boards)
  } else boards.value = _boards
  gameOver()
}

function leftAction() {
  if (isGameOver.value || isWin.value) return false
  const _boards = JSON.parse(JSON.stringify(boards.value))
  let VAL = null
  let FLAG_NEW_VAL = false
  for (let i = 0; i <= _boards.length - 1; i++) {
    for (let j = 0; j <= _boards.length - 1; j++) {
      //
      for (let right = j + 1; right <= _boards.length - 1; right++) {
        if (_boards[i][right]) {
          if (_boards[i][right] === _boards[i][j]) {
            _boards[i][j] += _boards[i][right]
            VAL = _boards[i][j]
            score.value += _boards[i][j]
            if (score.value > bestScore.value) bestScore.value = score.value
            _boards[i][right] = null
            FLAG_NEW_VAL = true
          }
          break
        }
      }

      if (_boards[i][j]) {
        let moveJ: null | number = null
        for (let left = j - 1; left >= 0; left--) {
          if (_boards[i][left]) break
          if (_boards[i][left] === null) moveJ = left
        }

        if (moveJ !== null) {
          _boards[i][moveJ] = _boards[i][j]
          _boards[i][j] = null
          FLAG_NEW_VAL = true
        }
      }
    }
  }

  if (FLAG_NEW_VAL) {
    if (VAL === 2048) isWin.value = true
    boards.value = addVal(_boards)
  } else boards.value = _boards
  gameOver()
}

onKeyStroke('ArrowUp', (e) => {
  e.preventDefault()
  upAction()
})

onKeyStroke('ArrowDown', (e) => {
  e.preventDefault()
  downAction()
})

onKeyStroke('ArrowRight', (e) => {
  e.preventDefault()
  rightAction()
})

onKeyStroke('ArrowLeft', (e) => {
  e.preventDefault()
  leftAction()
})

const { isSwiping, direction: swipeDirection } = useSwipe(RefBoard)

watch(swipeDirection, () => {
  if (!isSwiping) return
  if (swipeDirection.value === 'up') return upAction()
  if (swipeDirection.value === 'down') return downAction()
  if (swipeDirection.value === 'left') return leftAction()
  if (swipeDirection.value === 'right') return rightAction()
})
</script>

<template>
  <div class="game-container">
    <div class="game">
      <div class="game__header">
        <div class="game__header__title">2048</div>
        <div class="game__header__score-board">
          <div class="score-board-item">
            <div>Score</div>
            <div>{{ score }}</div>
          </div>
          <div class="score-board-item">
            <div>Best Score</div>
            <div>{{ bestScore }}</div>
          </div>
        </div>
      </div>

      <div class="game__control">
        <div v-if="isWin" class="btn btn-keep-going" @click="isWin = false">
          Keep going
        </div>
        <div class="btn btn-new-game" @click="newGame()">New Game</div>
      </div>

      <table ref="RefBoard">
        <tr v-for="(rows, i) in boards" :key="i">
          <td
            v-for="(val, j) in rows"
            :key="j"
            :class="getFadeIn({ i, j })"
            :style="{
              color: val ? getColor(val) : '',
              'background-color': val ? getBgColor(val) : '',
            }"
          >
            {{ val }}
          </td>
        </tr>
      </table>
      <div v-if="isGameOver" class="game__game-over">GAME OVER</div>
    </div>
  </div>
</template>

<style scoped lang="postcss">
.btn {
  @apply h-[30px] py-0 px-[10px];
  @apply inline-block flex-shrink-0 leading-[31px];
  @apply cursor-pointer;
  @apply rounded-[3px];
  text-align: right;
}
.btn-new-game {
  @apply bg-[#8f7a66] text-[#f9f6f2];
}
.btn-keep-going {
  @apply bg-[#edc950] text-[#f8f6f2];
}

.game {
  &-container {
    @apply text-center p-5;
  }
  @apply w-[313px];
  @apply mx-auto;

  &__header {
    @apply mt-2 mb-10;
    @apply flex justify-between;

    &__title {
      @apply flex items-center px-5;
      @apply text-3xl font-bold;
    }
    &__score-board {
      @apply flex gap-2;
      .score-board-item {
        @apply bg-[#baada0];
        @apply text-white font-bold;
        @apply py-[0.2em] px-[0.6em];
        @apply rounded-md;
      }
    }
  }
  &__control {
    @apply mb-2;
  }
  &__game-over {
    @apply font-bold text-[#dc3545];
  }
}

table {
  @apply inline-block;
}
table tr td {
  @apply size-[80px];
  @apply border-[0.5em] border-[#baada0];
  @apply text-center text-white text-lg font-bold;
  @apply bg-[#cdc0b4];
}

.fade-in {
  animation: fadeIn ease 0.5s;
  -webkit-animation: fadeIn ease 0.5s;
  -moz-animation: fadeIn ease 0.5s;
  -o-animation: fadeIn ease 0.5s;
  -ms-animation: fadeIn ease 0.5s;
}
@keyframes fadeIn {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}

@-moz-keyframes fadeIn {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}

@-webkit-keyframes fadeIn {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}

@-o-keyframes fadeIn {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}

@-ms-keyframes fadeIn {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}
</style>
