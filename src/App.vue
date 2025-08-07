<script setup lang="ts">
import { ref } from 'vue';

let headValue: number | null = null;
const field = ref();
let win = ref();
let gameover = ref();
let moveTo = ref();
let moveToLast = null;

let newGameReload = () => {
  gameover.value = false;
  win.value = false;
  moveTo.value = 'up';
  headValue = 4;
  moveToLast = moveTo.value;
  field.value = [
    [0, 0, 0],
    [0, 4, 0],
    [0, 3, 0],
    [0, 2, 0],
    [0, 1, 0],
    [0, 0, 0]
  ];
};

newGameReload();

const cl = (...m) => console.log(...m);

let checkWin = () => {
  let allElements = field.value.length * field.value[0].length;
  if (allElements == headValue) win.value = true;
};

let createApple = () => {
  if (gameover.value == true || win.value == true) return;

  let countApples = 0;
  let zero: {i: number, j: number}[] = [];

  for (let i = 0; i < field.value.length; i = i + 1) {
    for (let j = 0; j < field.value[i].length; j = j + 1) {
      if (field.value[i][j] == 'a') {
        countApples = countApples + 1;
        if (countApples >= 2) return;
      }

      if (field.value[i][j] == 0) {
        zero.push({ i, j });
      }
    }
  }

  let { i, j } = zero[Math.floor(Math.random() * zero.length)];
  field.value[i][j] = 'a';
};

let moveSegment = (bodyLineCoordinate, bodyColumnCoordinate, i, j, v) => {
  if (field.value[bodyLineCoordinate]?.[bodyColumnCoordinate] == v - 1) {
    field.value[i][j] = v - 1;
    field.value[bodyLineCoordinate][bodyColumnCoordinate] = 0;
    if (v - 1 > 1) {
      moveBody(bodyLineCoordinate, bodyColumnCoordinate, v - 1);
    }
    return;
  }
};

let moveBody = (i, j, v) => {
  moveSegment(i + 1, j, i, j, v);
  moveSegment(i, j + 1, i, j, v);
  moveSegment(i, j - 1, i, j, v);
  moveSegment(i - 1, j, i, j, v);
};

document.addEventListener('keydown', (event) => {
  if (event.code == 'ArrowLeft') {
    if (moveToLast != 'right') moveTo.value = 'left';
  }

  if (event.code == 'ArrowRight') {
    if (moveToLast != 'left') moveTo.value = 'right';
  }

  if (event.code == 'ArrowUp') {
    if (moveToLast != 'down') moveTo.value = 'up';
  }

  if (event.code == 'ArrowDown') {
    if (moveToLast != 'up') moveTo.value = 'down';
  }
});

let moveHeadStep = (headLineCoordinate, headColumnCoordinate, i, j) => {
  moveToLast = moveTo.value;

  if (
    headLineCoordinate < 0 ||
    headColumnCoordinate < 0 ||
    headColumnCoordinate >= field.value[i].length ||
    headLineCoordinate >= field.value.length
  ) {
    gameover.value = true;
    return;
  }

  if (field.value[headLineCoordinate][headColumnCoordinate] == 'a') {
    field.value[headLineCoordinate][headColumnCoordinate] = (headValue ?? 0) + 1;
    headValue = (headValue ?? 0) + 1;
    checkWin();
    return;
  }

  if (field.value[headLineCoordinate][headColumnCoordinate] != 0) {
    gameover.value = true;
    return;
  }

  field.value[headLineCoordinate][headColumnCoordinate] = headValue;
  field.value[i][j] = 0;
  moveBody(i, j, headValue);
};

let move = () => {
  if (gameover.value == true || win.value == true) {
    return;
  }

  for (let i = 0; i < field.value.length; i = i + 1) {
    for (let j = 0; j < field.value[i].length; j = j + 1) {
      if (field.value[i][j] == headValue) {
        if (moveTo.value == 'up') {
          moveHeadStep(i - 1, j, i, j);
        }

        if (moveTo.value == 'left') {
          moveHeadStep(i, j - 1, i, j);
        }

        if (moveTo.value == 'right') {
          moveHeadStep(i, j + 1, i, j);
        }

        if (moveTo.value == 'down') {
          moveHeadStep(i + 1, j, i, j);
        }

        return;
      }
    }
  }
};

setInterval(move, 1000);
setInterval(createApple, 2000);
</script>

<template>
  <div class="centering">
    <div class="gameover" v-if="win">Вы выиграли.</div>
    <div class="gameover" v-if="gameover">Ваша игра окончена.</div>
    <div class="newgame" v-if="gameover" @click="newGameReload()">Новая игра</div>
    <table>
      <tr v-for="(row, i) in field" :key="i">
        <td
          v-for="(value, j) in row"
          :class="['square', value == 'a' && 'red', value != '0' && 'black']"
          :key="j"
        >
          <template v-if="value != 0 && value != 'a'">
            {{ value }}
          </template>
          <template v-if="value == headValue">
            <div
              :class="[
                'moveHead',
                moveTo == 'right' && 'moveRight',
                moveTo == 'left' && 'moveLeft',
                moveTo == 'down' && 'moveDown'
              ]"
            >
              <div :class="['eyes', 'eyeleft']"></div>
              <div :class="['eyes', 'eyeright']"></div>
            </div>
          </template>
        </td>
      </tr>
    </table>
  </div>
</template>

<style scoped>
.gameover {
  color: darkred;
  font-size: 32px;
}

.newgame {
  background-color: rgb(201, 201, 201);
  /* border: 10px dotted rgb(255, 255, 255); */
  box-shadow: 12px 12px 2px 1px rgba(0, 0, 255, 0.2);
  border-radius: 5px;
  color: darkred;
  font-size: 32px;
  cursor: pointer;
  margin-bottom: 21px;
  padding: 6px 18px;
}
.newgame:active {
  box-shadow: unset;
}

.newgame:hover {
  background-color: rgb(226, 224, 241);
}

.buttons {
  display: flex;
  align-items: center;
  justify-content: center;
}

.centering {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;

  height: 100vh;
}
.grey {
  color: grey;
}

.green {
  color: rgb(16, 203, 16);
}

.brown {
  color: rgb(188, 83, 14);
}

.purple {
  color: blueviolet;
}

.td {
  border: 1px solid black;
}

.square {
  width: 50px;
  height: 50px;
  background-color: rgb(242, 242, 242);
  border: 5px solid rgb(179, 179, 179);
  border-radius: 16px;
  margin: 6px;

  text-align: center;
  font-size: 30px;
  position: relative;
}

.black {
  background-color: rgb(0, 0, 0);
  border: 5px solid rgb(118, 118, 118);
}
.red {
  background-color: red;
  border: 5px solid rgb(205, 23, 23);
}

.eyes {
  position: absolute;
  width: 8px;
  height: 8px;
  background-color: rgb(255, 255, 255);
  border-radius: 30%;
  top: 20%;
}

.eyeleft {
  left: 15%;
}
.eyeright {
  right: 15%;
}

.moveHead {
  top: 0;
  position: absolute;
  width: 100%;
  height: 100%;
}

.moveRight {
  transform: rotate(90deg);
}

.moveLeft {
  transform: rotate(270deg);
}

.moveDown {
  transform: rotate(180deg);
}

.openedstyle {
  background-color: rgb(255, 255, 255);
}
</style>
