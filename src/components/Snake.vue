<template>
  <button @click="onStartGame()" v-if="!hideButton">Start Game</button>
  <div :class="$style['tile-wrapper']" @keydown="keyPress" v-if="startGame">
    <template v-for="y in noTiles">
      <div
        v-for="x in noTiles"
        :key="[x, y]"
        :data-tile="[x, y]"
        :class="[
          $style['tile'],
          {
            [$style['tile--snake']]: isSnake(x, y),
          },
          {
            [$style['tile--apple']]:
              x === appleCoords[0] && y === appleCoords[1],
          },
        ]"
      ></div>
    </template>
  </div>
  <div v-if="gameOver">Game OVER loseeerr</div>
</template>

<script lang="ts">
export default {
  name: "Snake",
  data() {
    return {
      noTiles: 20,
      velocity: [-1, 0],
      prevVelocity: [],
      snakeCoords: [[10, 10]],
      appleCoords: [],
      path: [],
      isAppleEaten: false,
      startGame: false,
      gameOver: false,
      hideButton: false
    };
  },
  mounted() {
    this.velocity = [-1, 0];
    window.addEventListener("keydown", e => {
      switch (e.keyCode) {
        case 37:
          if (this.prevVelocity[0] === 1) return;
          this.velocity = [-1, 0];
          break;
        case 38:
          if (this.prevVelocity[1] === 1) return;
          this.velocity = [0, -1];
          break;
        case 39:
          if (this.prevVelocity[0] === -1) return;
          this.velocity = [1, 0];
          break;
        case 40:
          if (this.prevVelocity[1] === -1) return;
          this.velocity = [0, 1];
          break;
      }
    });

    setInterval(() => {
      if (this.startGame) {
        this.moveSnake(this.velocity);
        (this as any).prevVelocity = this.velocity;
      }
    }, 200);
    this.randomAppleCoors();
  },
  methods: {
    moveSnake(velocity: number[]) {
      this.snakeCoords[0] = [
        this.snakeCoords[0][0] + velocity[0],
        this.snakeCoords[0][1] + velocity[1]
      ];

      if (this.snakeCoords[0][0] === 0) {
        this.snakeCoords[0][0] = this.noTiles;
      } else if (this.snakeCoords[0][0] === this.noTiles + 1) {
        this.snakeCoords[0][0] = 1;
      }

      if (this.snakeCoords[0][1] === 0) {
        this.snakeCoords[0][1] = this.noTiles;
      } else if (this.snakeCoords[0][1] === this.noTiles + 1) {
        this.snakeCoords[0][1] = 1;
      }

      (this as any).path.unshift(this.snakeCoords[0]);
      if (this.snakeCoords.length + 2 === this.path.length) {
        this.path.pop();
      }

      if (this.isAppleEaten) {
        this.snakeCoords.push(this.path[this.path.length - 1]);
        this.isAppleEaten = false;
      }

      if (
        this.snakeCoords[0][0] === this.appleCoords[0] &&
        this.snakeCoords[0][1] === this.appleCoords[1]
      ) {
        this.isAppleEaten = true;
        this.randomAppleCoors();
      }

      this.snakeCoords.forEach((dot, index) => {
        dot[0] = this.path[index][0];
        dot[1] = this.path[index][1];
      });

      if (this.isSnake(this.snakeCoords[0][0], this.snakeCoords[0][1], true)) {
        this.startGame = false;
        this.gameOver = true;
        this.velocity = [0, 0];
        this.path = [];
        this.snakeCoords = [];
        this.hideButton = false;
      }
    },
    randomAppleCoors() {
      (this.appleCoords as number[]) = [
        Math.floor(Math.random() * 20 + 1),
        Math.floor(Math.random() * 20 + 1)
      ];
    },
    isSnake(x: number, y: number, doNotCountFirst = false) {
      for (let i = 0; i < this.snakeCoords.length; i++) {
        if (doNotCountFirst && i === 0) {
          continue;
        }
        if (this.snakeCoords[i][0] == x && this.snakeCoords[i][1] == y) {
          return true;
        }
      }
      return false;
    },
    onStartGame() {
      this.startGame = true;
      this.gameOver = false;
      this.velocity = [-1, 0];
      this.snakeCoords = [[10, 10]];
      this.hideButton = true;
    }
  }
};
</script>

<style lang="scss" module>
.tile {
  background-color: black;
  width: 15px;
  height: 15px;
  display: inline-block;
  margin: 1px;

  &--snake {
    background-color: red;
  }

  &--apple {
    background-color: green;
  }

  &-wrapper {
    width: 340px;
    line-height: 0;
  }
}
</style>
