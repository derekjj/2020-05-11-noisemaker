<template lang="pug">
  #app
    .container
      .row
        .col-12.p-4
          h3.p-4 Derek Johnston's
          b Mine Sweeper
      .row.pb-2
        .col-12 {{mines.marked}} of {{mines.total}} mines marked
      .row.pb-2
        .col-12
          b-button(variant="danger" @click="initWorld()") Reset
    .container
      .row(v-for="(row, iRow) in world")
        div(v-for="(cell, iCol) in row"
            :key="iCol"
            @click="fire(iRow,iCol)"
            @click.right="toggle(iRow,iCol)"
            oncontextmenu="return false;")
          div.border(style="width:45px; height:45px;"
            :class="{'bg-warning' : cell.marked,'bg-secondary' : !cell.marked, 'bg-light' : cell.cleared, 'bg-danger' : cell.hit}")
            div.p-2(v-if="cell.revealedMines") {{cell.revealedMines}}
    div.box.stack-top.bg-secondary.m-0(v-if="gameover")
      table.vh-100.w-100
        tbody
          tr
            td
              div.text-light.p-3
                h1 Game Over
                br
                | {{mines.marked}} of {{mines.total}} mines marked
                div
                  b-button(size="lg" variant="primary" @click="initWorld()") Reset
</template>

<script>
export default {
  data() {
    return {
      world: [],
      gameover: false,
      mines: { marked: 0, total: 0 }
    };
  },
  created() {
    this.initWorld();
  },
  methods: {
    initWorld(rowSize = 25, colSize = 25, mines = 300) {
      this.gameover = false;
      this.world = [];
      for (let rows = 0; rows < rowSize; rows++) {
        const row = [];
        for (let cols = 0; cols < colSize; cols++) {
          const isMine = mines > 0 ? Math.random() >= 0.8 : false;
          mines = isMine ? mines - 1 : mines;
          this.mines.total = isMine ? this.mines.total + 1 : this.mines.total;
          row.push({
            mine: isMine,
            marked: false,
            revealedMines: 0,
            hit: false,
            cleared: false
          });
        }
        this.world.push(row);
      }
    },
    toggle(row, col) {
      //todo: make computed based of the count of marked only cells
      if (!this.gameover && !this.world[row][col].marked && !this.world[row][col].cleared) {
        this.mines.marked++;
        let tempWorld = this.world.slice();
        tempWorld[row][col].marked = !this.world[row][col].marked;
        this.world = tempWorld.slice();
      }
    },
    fire(row, col) {
      var cell = this.world[row][col];
      if (this.world[row][col].mine) {
        this.hit(cell);
      } else {
        this.miss(cell, row, col);
      }
    },
    hit(cell) {
      var audio = new Audio(require("../assets/Explosion3.mp3"));
      audio.play();

      cell.hit = true;
      this.gameover = true;
    },
    miss(cell, row, col) {
      var audio = new Audio(require("../assets/Miss.mp3"));
      const numNeighbours = this.getNumNeighbours(row, col);
      audio.play();
      cell.revealedMines = numNeighbours;
      cell.cleared = true;
      if (numNeighbours === 0) {
        this.clearClearing(row, col);
      }
    },
    clearClearing(cellRow, cellCol) {
      // todo: if zero clear all connecting no Neighbours
      console.log("cellRow", cellRow);
      console.log("cellCol", cellCol);
    },
    getNumNeighbours(cellRow, cellCol) {
      const startRow = cellRow - 1;
      const startCol = cellCol - 1;

      const endRow = cellRow + 1;
      const endCol = cellCol + 1;

      let numNeighbours = 0;

      for (var row = startRow; row <= endRow; row++) {
        if (row < 0 || row >= this.world.length) {
          continue;
        }
        for (var col = startCol; col <= endCol; col++) {
          if (col < 0 || col >= this.world[0].length) {
            continue;
          }
          if (row === cellRow && col === cellCol) {
            continue;
          }
          if (this.world[row][col].mine) {
            numNeighbours++;
          }
        }
      }
      return numNeighbours;
    }
  }
};
</script>
<style scoped>
.box {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  opacity: 0.8; /* for demo purpose */
}

.stack-top {
  z-index: 9;
}
</style>