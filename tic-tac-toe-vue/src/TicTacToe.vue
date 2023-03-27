<template>
    <div class="tictactoe-board">
      <div v-for="(n, i) in 3">
        <div v-for="(n, j) in 3">
          <cell @click="performMove(j, i)" :value="board.cells[j][i]"></cell>
        </div>
      </div>
  
      <div class="game-over-text" v-if="gameOver">
        {{ gameOverText }}
      </div>
    </div>
  </template>
  <script>
  import cell from './Cell.vue'
    import Board from "./Board";
    export default {
      data() { return {
        gameOver: false,
        gameOverText: '',
        board: new Board()
      } },
      components:{
        cell
      },
      methods: {
        performMove(x, y) {
          if (this.gameOver) {
            return;
          }
          if (! this.board.doMove(x, y, 'x')) {
            // Invalid move.
            return;
          }
          this.$forceUpdate();
          if (this.board.isGameOver()) {
            this.gameOver = true;
            this.gameOverText = this.board.playerHas3InARow('x') ? 'You win!' : 'Draw';
            return;
          }
          let aiMove = this.minimax(this.board.clone(), 'o');
          this.board.doMove(aiMove.move.x, aiMove.move.y, 'o');
          if (this.board.isGameOver()) {
            this.gameOver = true;
            this.gameOverText = this.board.playerHas3InARow('o') ? 'You lose!' : 'Draw';
          }
          this.$forceUpdate();
        },
        minimax(board, player, depth = 1) {
          // If the board has 3 in a row return the score of the board.
          if (board.isGameOver()) {
            return {
              score: board.getScore() + depth,
              move: null
            }
          }
          // The 'o' player wants to maximize its score, the 'x' player wants to
          // minimize its score.
          let bestScore = player === 'o' ? -10000 : 10000;
          let bestMove = null;
          let moves = board.getPossibleMoves();
          for (let i = 0; i < moves.length; i++) {
            let move = moves[i];
            let newBoard = board.clone();
            newBoard.doMove(move.x, move.y, player);
            // Recursively call the minimax function for the new board.
            const score = this.minimax(newBoard, player === 'x' ? 'o' : 'x', depth + 1).score;
            // If the score is better than the best saved score update the best saved
            // score to this move.
            if ((player === 'o' && score > bestScore) || (player === 'x' && score < bestScore)) {
              bestScore = score;
              bestMove = move;
            }
          }
          // Return the best found score & move!
          return {
            score: bestScore,
            move: bestMove
          }
        }
      }
    }
  </script>
  <style>
    .tictactoe-board {
      display: flex;
      flex-wrap: wrap;
      width: 204px;
      height: 204px;
    }
  </style>