<template>
  <canvas id="canvas" width="200" height="400"></canvas>
</template>

<script>

  export default {
    name: 'TetrisField',
    data: function () {
      return {
        ctx: undefined,
        gridConditions: [], //10*21
        unfixedBlock: undefined, //落下中のブロックが占有するマス
        blockKind: [[[0,3], [0,4], [0,5], [0,6]], [[1,4], [1,5], [0,6], [1,6]], [[0,4], [1,4], [1,5], [1,6]], 
        [[0,4], [0,5], [1,5], [1,6]], [[1,4], [0,5], [1,5], [1,6]], [[0,4], [1,4], [0,5], [1,5]], [[1,4], [0,5],[1,5],[1,6]]],
        //7種類のブロックそれぞれが占有するマス 0: 棒 1: L 2: 逆L 3: Z 4: 逆Z 5: 四角 6: 凸
        underBlock: undefined, //ブロックが次に落ちるマス
        game: undefined
      }
    },
    mounted: function () {
      const canvas = document.getElementById('canvas');
      this.ctx = canvas.getContext('2d');
      for (let i = 0; i < 20; i++) { //グリッドを描画
        for (let j = 0; j < 10; j++) {
          this.ctx.strokeRect(j * 20, i * 20, 20, 20);
        }
      }
      for (let i = 0; i < 20; i++) {
        this.gridConditions.push(Array(10).fill(false));
      }
      this.gridConditions.push(Array(10).fill(true));
      this.appearBlock();
      this.game = setInterval(this.fallBlock, 200);
    },
    methods: {
      appearBlock: function () { //ブロックの出現
        const kind = Math.floor(Math.random() * 7);
        this.unfixedBlock = this.blockKind[kind];
        this.rewriteUnder();
        for (let i = 0; i < 4; i++) {
          if (this.gridConditions[this.unfixedBlock[i][0]][this.unfixedBlock[i][1]]) {
            clearTimeout(this.game);
            throw 'game over';
          }
        }
        this.drawBlock();
      },
      fallBlock: function () { //ブロックの落下 setInterval?
        if (this.underCheck()) {
          this.clearBlock();
          let newUnfixed = [];
          for (let i = 0; i < 4; i++) {
            newUnfixed.push([this.unfixedBlock[i][0] + 1, this.unfixedBlock[i][1]])
          }
          this.unfixedBlock = newUnfixed;
          this.rewriteUnder();
          this.drawBlock();
        } else {
          for (let i = 0; i < 4; i++) {
            this.gridConditions[this.unfixedBlock[i][0]][this.unfixedBlock[i][1]] = true;
          }
          this.appearBlock();
        }
      },
      drawBlock: function () { //ブロックの描画
        this.ctx.fillStyle = 'black';
        for (let i = 0; i < 4; i++) {
          this.ctx.fillRect(this.unfixedBlock[i][1] * 20, this.unfixedBlock[i][0] * 20, 20, 20);
        }
      },
      clearBlock: function() { //ブロックの描画取り消し
        this.ctx.fillStyle = 'white';
        for (let i = 0; i < 4; i++) {
          this.ctx.fillRect(this.unfixedBlock[i][1] * 20, this.unfixedBlock[i][0] * 20, 20, 20);
          this.ctx.strokeRect(this.unfixedBlock[i][1] * 20, this.unfixedBlock[i][0] * 20, 20, 20);
        }
      },
      rewriteUnder: function () { //underBlockを書き換え
        this.underBlock = [];
        for (let i = 0; i < 4; i++) {
          this.underBlock.push([this.unfixedBlock[i][0] + 1, this.unfixedBlock[i][1]]);
        }
      },
      underCheck: function () { //ブロックの落ちる余地があるかどうか確認、あるならreturn true
        let bool = true;
        for (let i = 0; i < 4; i++) {
          if (this.gridConditions[this.underBlock[i][0]][this.underBlock[i][1]]) {
            bool = false;
            break;
          }
        }
        return bool;
      }
    }
  }
</script>