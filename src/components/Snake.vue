<template>
  <controls
    :show-button="runGame"
    @update:start="onStartGame"
  />
  <game-lost v-if="gameOver" />

  <div
    v-if="runGame"
    :class="$style['tile-wrapper']"
  >
    <template v-for="y in noTiles">
      <div
        v-for="x in noTiles"
        :key="[x, y]"
        :class="$style['tile']"
      >
        <div
          v-if="isSnake(x, y)"
          :data-direction="isSnake(x, y, false, true) && velocity"
          :class="[
            $style['tile--snake'],
            {
              [$style['tile--snake-head']]: isSnake(x, y, false, 'head'),
              [$style['tile--snake-tail']]: isSnake(x, y, false, 'tail')
            }
          ]"
        />
        <div
          v-if="x === appleCoords[0] && y === appleCoords[1]"
          :class="$style['tile--apple']"
        />
      </div>
    </template>
  </div>
</template>

<script lang="ts">
import Controls from './Controls.vue';
import GameLost from './GameLost.vue';

export default {
  name: "Snake",
  components: {
    Controls, GameLost
  },
  data() {
    return {
      noTiles: 20,
      velocity: [-1, 0],
      prevVelocity: [],
      snakeCoords: [[10, 10]],
      appleCoords: [],
      path: [],
      isAppleEaten: false,
      runGame: false,
      gameOver: false,
      gameTickTimout: null,
      headDirection: null
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

    this.randomAppleCoors();
  },
  methods: {
    startGame() {
      const gameSpeed = 200;
      let tick;

      this.gameTickTimout = setTimeout(tick = () => {
        if(this.runGame){
          this.moveSnake(this.velocity);
          this.prevVelocity = this.velocity;
          this.gameTickTimout = setTimeout(tick, gameSpeed);
        }
      }, gameSpeed);
    },
    stopGame(){
      clearInterval(this.gameTickTimout);
    },
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

      this.path.unshift(this.snakeCoords[0]);
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
        this.stopGame();
        this.runGame = false;
        this.gameOver = true;
        this.velocity = [0, 0];
        this.path = [];
        this.snakeCoords = [];
      }
    },
    randomAppleCoors() {
      (this.appleCoords as number[]) = [
        Math.floor(Math.random() * 20 + 1),
        Math.floor(Math.random() * 20 + 1)
      ];
    },
    isSnake(x: number, y: number, doNotCountFirst = false, returnPart = '') {
      const len = this.snakeCoords.length;
      for (let i = 0; i < len; i++) {
        let index = i;

        if(returnPart === 'head'){
          index = 0;
        } else if(returnPart === 'tail'){
          index = len - 1;
        }

        if (doNotCountFirst && i === 0) {
          continue;
        }

        if (this.snakeCoords[index][0] == x && this.snakeCoords[index][1] == y) {
          return true;
        }
      }
      return false;
    },
    onStartGame() {
      this.startGame();
      this.runGame = true;
      this.gameOver = false;
      this.velocity = [-1, 0];
      this.path = [];
      this.snakeCoords = [[10, 10]];
    }
  }
};
</script>

<style lang="scss" module>
$tileSize: 20px;
$tileSpacing: 1px;
$radiusSize: $tileSize * 2;
$canvasSize: ($tileSize + $tileSpacing) * 20;

.tile {
  background-color: grey;
  width: $tileSize;
  height: $tileSize;
  display: inline-block;
  margin-right: $tileSpacing;
  margin-bottom: $tileSpacing;

  &--snake {
    width: $tileSize;
    height: $tileSize;
    background-color: green;

    &-head{
      position: relative;
      background-color: darkgreen;

      &:after,
      &:before{
        content: '';
        position: absolute;
        width: $tileSize * .3;
        height: $tileSize * .3;
        background-color: #fff;
        border-radius: 50%;
      }

      //TOP-BOTTOM
      &[data-direction="0,-1"],
      &[data-direction="0,1"] {
        &:after{
          left: 3px;
        }

        &:before{
          left: 11px;
        }
      }

      //TOP
      &[data-direction="0,-1"] {
        border-radius: $radiusSize $radiusSize 0 0;

        &:after,
        &:before{
          top: 5px;
        }
      }

      //BOTTOM
      &[data-direction="0,1"] {
        border-radius: 0 0 $radiusSize $radiusSize;

        &:after,
        &:before{
          top: 7px;
        }
      }

      //LEFT-RIGHT
      &[data-direction="-1,0"],
      &[data-direction="1,0"] {
        &:after{
          top: 3px;
        }

        &:before{
          top: 11px;
        }
      }

      //LEFT
      &[data-direction="-1,0"] {
        border-radius: $radiusSize 0 0 $radiusSize;

        &:after,
        &:before{
          left: 5px;
        }
      }

      //RIGHT
      &[data-direction="1,0"] {
        border-radius:  0 $radiusSize $radiusSize 0;

        &:after,
        &:before{
          left: 7px;
        }
      }
    }

    &-tail{
      background-color: lightgreen
    }
  }

  &--apple {
    position: relative;

    &:before,
    &:after{
      content: '';
      position: absolute;
    }

    &:after{
      top: 2px;
      width: 2px;
      height: 5px;
      left: 50%;
      background-color: brown;
      transform: rotate(20deg);
    }

    &:before{
      top: $tileSize * .25;
      left: $tileSize * .05;
      width: $tileSize * .9;
      height: $tileSize * .7;
      background-color: red;
      border-radius: 50%;
    }
  }

  &-wrapper {
    width: $canvasSize;
    line-height: 0;
  }
}
</style>
