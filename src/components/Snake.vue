<template>
  <div :class="$style['tile-wrapper']" @keydown="keyPress">
    <template v-for="y in noTiles">
      <div
        v-for="x in noTiles"
        :key="[x, y]"
        :data-tile="[x, y]"
        :class="[
          $style['tile'],
          {
            [$style['tile--snake']]:
              (x === snakeCoords[0] && y === snakeCoords[1]) ||
              tileCoords(x, y),
          },
          {
            [$style['tile--apple']]:
              x === appleCoords[0] && y === appleCoords[1],
          },
        ]"
      ></div>
    </template>
  </div>
</template>

<script lang="ts">
export default {
  name: "Snake",
  data() {
    return {
      noTiles: 20,
      tile: [],
      velocity: [-1, 0],
      snakeCoords: [10, 10],
      appleCoords: [],
      path: []
    };
  },
  mounted() {
    window.addEventListener("keydown", e => {
      switch (e.keyCode) {
        case 37:
          if (this.velocity[0] === 1) return;
          this.velocity = [-1, 0];
          break;
        case 38:
          if (this.velocity[1] === 1) return;
          this.velocity = [0, -1];
          break;
        case 39:
          if (this.velocity[0] === -1) return;
          this.velocity = [1, 0];
          break;
        case 40:
          if (this.velocity[1] === -1) return;
          this.velocity = [0, 1];
          break;
      }
    });

    setInterval(() => {
      this.moveSnake();
    }, 200);

    this.randomAppleCoors();
  },
  methods: {
    moveSnake() {
      this.snakeCoords = [
        this.snakeCoords[0] + this.velocity[0],
        this.snakeCoords[1] + this.velocity[1]
      ];
      if (this.snakeCoords[0] === 0) {
        this.snakeCoords[0] = this.noTiles;
      } else if (this.snakeCoords[0] === 20) {
        this.snakeCoords[0] = 0;
      }

      if (this.snakeCoords[1] === 0) {
        this.snakeCoords[1] = this.noTiles;
      } else if (this.snakeCoords[1] === 20) {
        this.snakeCoords[1] = 0;
      }

      if (
        this.snakeCoords[0] === this.appleCoords[0] &&
        this.snakeCoords[1] === this.appleCoords[1]
      ) {
        this.tile.push(this.appleCoords);
        this.randomAppleCoors();
      }

      (this as any).path.unshift(this.snakeCoords);
      if (this.tile.length + 3 === this.path.length) {
        this.path.pop();
      }

      // if(this.tile.length){
      //   this.tile[0][0] = this.snakeCoords[0];
      //   this.tile[0][1] = this.snakeCoords[1];
      // }

      // console.log(this.tile[0], this.snakeCoords);
      this.tile.forEach((tile, index) => {
        tile[0] = this.path[index + 1][0];
        tile[1] = this.path[index + 1][1];
      });

      console.log(this.tile[0], this.snakeCoords);
    },
    randomAppleCoors() {
      (this.appleCoords as number[]) = [
        Math.floor(Math.random() * 20 + 1),
        Math.floor(Math.random() * 20 + 1)
      ];
    },
    tileCoords(x: number, y: number) {
      for (let i = 0; i < this.tile.length; i++) {
        if (this.tile[i][0] == x && this.tile[i][1] == y) {
          return true;
        }
      }
      return false;
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
