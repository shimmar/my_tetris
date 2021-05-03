<template>
  <canvas id="canvas" width="200" height="400"></canvas>
</template>

<script>

  export default {
    name: 'GameField',
    data: function () {
      return {
        ctx: undefined,
        gridConditions: [], //10*21
        unfixedBlock: undefined, //落下中のブロックが占有するマス
        blockKind: [[[0,3], [0,4], [0,5], [0,6]], [[0,4], [0,5], [0,6], [-1,6]], [[0,6], [0,5], [0,4], [-1,4]], 
        [[-1,4], [-1,5], [0,5], [0,6]], [[0,4], [0,5], [-1,5], [-1,6]], [[-1,4], [0,4], [-1,5], [0,5]], [[-1,5], [0,5],[0,4],[0,6]]],
        //7種類のブロックそれぞれが占有するマス 0: 棒 1: L 2: 逆L 3: Z 4: 逆Z 5: 四角 6: 凸
        blockNumber: undefined, //どのブロックかを示す番号、0~6
        underBlock: undefined, //ブロックが次に落ちるマス
        rotateCount: 0, //これまでの回転の回数
        rotateNext: 
        [[[[-2,2], [-1,1], [0,0], [1,-1]], [[2,-2], [1,-1], [0,0], [-1,1]]], //0
        [[[-1,1], [0,0], [1,-1], [2,0]], [[1,1], [0,0], [-1,-1], [0,-2]], [[1,-1], [0,0], [-1,1], [-2,0]], [[-1,-1], [0,0], [1,1], [0,2]]], //1
        [[[1,-1], [0,0], [-1,1], [0,2]], [[-1,-1], [0,0], [1,1], [2,0]], [[-1,1], [0,0], [1,-1], [0,-2]], [[1,1], [0,0], [-1,-1], [-2,0]]], //2
        [[[0,2], [1,1], [0,0], [1,-1]], [[0,-2], [-1,-1], [0,0], [-1,1]]], //3
        [[[-1,1], [0,0], [1,1], [2,0]], [[1,-1], [0,0], [-1,-1], [-2,0]]], //4
        [[[0,0], [0,0], [0,0], [0,0]]], //5
        [[[1,1], [0,0], [-1,1], [1,-1]], [[1,-1], [0,0], [1,1], [-1,-1]], [[-1,-1], [0,0], [1,-1], [-1,1]], [[-1,1], [0,0], [-1,-1], [1,1]]] //6
        ], //ブロックの回転に伴う座標の変化を定義
        game: undefined
      }
    },
    mounted: function () {
      const canvas = document.getElementById('canvas');
      this.ctx = canvas.getContext('2d');
      for (let i = 0; i < 21; i++) { //横線描画
            this.ctx.beginPath();
            this.ctx.moveTo(0, 20 * i);
            this.ctx.lineTo(200, 20 * i);
            this.ctx.stroke();
          }
          for (let i = 0; i < 11; i++) { //縦線描画
            this.ctx.beginPath();
            this.ctx.moveTo(20 * i, 0);
            this.ctx.lineTo(20 * i, 400);
            this.ctx.stroke();
          }
      for (let i = 0; i < 20; i++) {
        this.gridConditions.push(Array(10).fill(false));
      }
      this.gridConditions.push(Array(10).fill(true));
      this.appearBlock();
      this.game = setInterval(this.fallBlock, 500);
      window.addEventListener("keydown", this.handleKeyDown);
    },
    methods: {
      appearBlock: function () { //ブロックの出現
        this.rotateCount = 0;
        this.blockNumber = Math.floor(Math.random() * 7);
        this.unfixedBlock = this.blockKind[this.blockNumber];
        this.rewriteUnder();
        let overChecker = false;
        for (let i = 0; i < 4; i++) {
          if (this.unfixedBlock[i][0] > -1 && this.gridConditions[this.unfixedBlock[i][0]][this.unfixedBlock[i][1]]) {
            overChecker = true;
            window.removeEventListener("keydown", this.handleKeyDown);
            clearTimeout(this.game);
            console.log('GAME OVER');
            break;
          }
        }
        if (!overChecker) {
          this.drawBlock();
        }
      },
      fallBlock: function () { //ブロックの落下
        if (this.underCheck() === 0) {
          this.clearBlock();
          let newUnfixed = [];
          for (let i = 0; i < 4; i++) {
            newUnfixed.push([this.unfixedBlock[i][0] + 1, this.unfixedBlock[i][1]])
          }
          this.unfixedBlock = newUnfixed;
          this.rewriteUnder();
          this.drawBlock();
        } else if (this.underCheck() === 1) {
          let blockY = [];
          for (let i = 0; i < 4; i++) {
            this.gridConditions[this.unfixedBlock[i][0]][this.unfixedBlock[i][1]] = true;
            blockY.push(this.unfixedBlock[i][0]);
          }
          const min = blockY.reduce(function(a, b) {
            return Math.min(a, b);
          })
          const max = blockY.reduce(function(a, b) {
            return Math.max(a, b);
          })
          this.clearLine(min, max);
          this.appearBlock();
        } else { //gameover
          window.removeEventListener("keydown", this.handleKeyDown);
          clearTimeout(this.game);
          console.log('GAME OVER');
        }
      },
      drawBlock: function () { //ブロックの描画
        this.ctx.fillStyle = 'black';
        for (let i = 0; i < 4; i++) {
          if (this.unfixedBlock[i][0] > -1) {
            this.ctx.fillRect(this.unfixedBlock[i][1] * 20, this.unfixedBlock[i][0] * 20, 20, 20);
          }
        }
      },
      clearBlock: function() { //ブロックの描画取り消し
        this.ctx.fillStyle = 'white';
        for (let i = 0; i < 4; i++) {
          if (this.unfixedBlock[i][0] > -1) {
            this.ctx.fillRect(this.unfixedBlock[i][1] * 20, this.unfixedBlock[i][0] * 20, 20, 20);
            this.ctx.strokeRect(this.unfixedBlock[i][1] * 20, this.unfixedBlock[i][0] * 20, 20, 20);
          }
        }
      },
      rewriteUnder: function () { //underBlockを書き換え
        this.underBlock = [];
        for (let i = 0; i < 4; i++) {
          this.underBlock.push([this.unfixedBlock[i][0] + 1, this.unfixedBlock[i][1]]);
        }
      },
      underCheck: function () { //ブロックの落ちる余地があるかどうか確認、あるなら0、無くて固定なら1、gameoverなら2
        let result = 0, minusExistance = false;
        for (let i = 0; i < 4; i++) {
          if (this.unfixedBlock[i][0] < 0) {
            minusExistance = true;
          } else if (this.gridConditions[this.underBlock[i][0]][this.underBlock[i][1]]) {
            result = 1;
          }
        }
        if (result === 1 && minusExistance) {
          result = 2;
        }
        return result;
      },
      moveBlock: function (dir) {
        let moveto = new Array(4);
        for (let i = 0; i < 4; i++) {
          moveto[i] = [this.unfixedBlock[i][0], this.unfixedBlock[i][1] + dir];
        }
        if (this.filledCheck(moveto) === false) {
          this.clearBlock();
          this.unfixedBlock = moveto;
          this.rewriteUnder();
          this.drawBlock();
        }
      },
      rotateBlock: function () {
        const lib = this.rotateNext[this.blockNumber];
        const difference = lib[this.rotateCount % lib.length];
        let next = new Array(4);
        for (let i = 0; i < 4; i++) {
          next[i] = [this.unfixedBlock[i][0] + difference[i][0], this.unfixedBlock[i][1] + difference[i][1]];
        }
        if (this.filledCheck(next)) {
          let leftRotated = new Array(4);
          for (let i = 0; i < 4; i++) {
            leftRotated[i] = [next[i][0], next[i][1] - 1];
          }
          if (this.filledCheck(leftRotated)) {
            let rightRotated = new Array(4);
            for (let i = 0; i < 4; i++) {
              rightRotated[i] = [next[i][0], next[i][1] + 1];
            }
            if (this.filledCheck(rightRotated) === false) {
              //右にずらして回転
              this.clearBlock();
              this.unfixedBlock = rightRotated;
              this.rotateCount++;
              this.rewriteUnder();
              this.drawBlock();
            }
          } else {
            //左にずらして回転
            this.clearBlock();
            this.unfixedBlock = leftRotated;
            this.rotateCount++;
            this.rewriteUnder();
            this.drawBlock();
          }
        } else {
          //そのまま回転
          this.clearBlock();
          this.unfixedBlock = next;
          this.rotateCount++;
          this.rewriteUnder();
          this.drawBlock();
        }
      },
      filledCheck: function (target) { //埋まってるか枠外ならtrue、空いてたらfalse
        for (let i = 0; i < 4; i++) {
          if (target[i][0] > 19 || target[i][1] < 0 || target[i][1] > 9) {
            return true;
          } else if (target[i][0] >= 0 && this.gridConditions[target[i][0]][target[i][1]]) {
            return true;
          }
        }
        return false;
      },
      clearLine: function (top, bottom) { //top, bottomは新たに固定されたブロックの縦の座標
        let cleared = new Set();
        for (let i = top; i <= bottom; i++) { //揃った行がどれか判定
          const torf = new Set(this.gridConditions[i]);
          if (torf.has(false) === false) {
            cleared.add(i);
          }
        }
        if (cleared.size > 0) { //一行以上消える時だけ以下の処理
          let newConditions = []; //新たなgridConditions
          for (let i = 0; i < cleared.size; i++) {
            newConditions.push(Array(10).fill(false));
          }
          for (let i = 0; i < 20; i++) {
            if (cleared.has(i) === false) {
              newConditions.push(this.gridConditions[i]);
            }
          }
          newConditions.push(Array(10).fill(true));
          cleared = Array.from(cleared);
          const lowest = cleared.reduce(function(a, b) {
            return Math.max(a, b);
          })
          this.ctx.fillStyle = 'white';
          this.ctx.fillRect(0, 0, 200, 20 * (lowest + 1)); //一旦lowestより上を消す
          for (let i = 0; i < (lowest + 1); i++) { //横線描画
            this.ctx.beginPath();
            this.ctx.moveTo(0, 20 + 20 * i);
            this.ctx.lineTo(200, 20 + 20 * i);
            this.ctx.stroke();
          }
          for (let i = 0; i < 11; i++) { //縦線描画
            this.ctx.beginPath();
            this.ctx.moveTo(20 * i, 0);
            this.ctx.lineTo(20 * i, 400);
            this.ctx.stroke();
          }
          this.ctx.fillStyle = 'black';
          for (let i = 0; i < (lowest + 1); i++) { //埋まったマス再描画
            for (let j = 0; j < 10; j++) {
              if (newConditions[i][j]) {
                this.ctx.fillRect(20 * j, 20 * i, 20, 20);
              }
            }
          }
          this.gridConditions = newConditions;
        }
      },
      handleKeyDown: function (event) {
        //up: 38, down: 40, left: 37, right: 39
        const keyCode = event.keyCode;
        if (keyCode == 40) {
          this.fallBlock();
        } else if (keyCode == 37) {
          this.moveBlock(-1);
        } else if (keyCode == 39) {
          this.moveBlock(1);
        } else if (keyCode == 38) {
          this.rotateBlock();
        }
      }
    }
  }
</script>