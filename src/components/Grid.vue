<template>
    <div>
        <div class="gameStatus" :class="gameStatusColor"> 
            {{ gameStatusMessage }}
        </div>
        
        <table class="grid">
            <tr>
                <cell name="1"></cell>
                <cell name="2"></cell>
                <cell name="3"></cell>
            </tr>
            <tr>
                <cell name="4"></cell>
                <cell name="5"></cell>
                <cell name="6"></cell>
            </tr>
            <tr>
                <cell name="7"></cell>
                <cell name="8"></cell>
                <cell name="9"></cell>
            </tr>
        </table>
    </div>
    
</template>

<script>
import Cell from './Cell.vue';

export default {
  components: {
    Cell
  },

  data(){
      return {
          activePlayer: 'X',
          // maintain status of game; turn or win or draw
          gameStatus: 'turn',
          gameStatusMessage: `X's turn`,
          gameStatusColor: 'statusTurn',
          // no of moves by player; max=9
          moves: 0,
          cells: {
              1: '', 2: '', 3: '',
              4: '', 5: '', 6: '',
              7: '', 8: '', 9: ''
          },
          // win conditions contain all possible win conditions
          winConditions: [
              [1, 2, 3], [4, 5, 6], [7, 8, 9], // rows
              [1, 4, 7], [2, 5, 6], [3, 6, 9], // columns
              [1, 5, 9], [3, 5, 7] // diagonals
          ]
      }
  },

  methods: {
      created(){

          // listens for a strike made by player on cell
          // called by the cell component
          Event.$on('strike', (cellNumber)=>{
              // Sets either X or O in the cell that is clicked  
              this.cells[cellNumber] = this.activePlayer;

              // increments the number of moves
              this.moves++;

              // store game status by calling changeGameStatus()
              this.gameStatus = this.changeGameStatus();
              this.changePlayer();
          });

          // listen for the reset button click event
          // data of grid component is re-initialized
          Event.$on('gridReset', () => {
                Object.assign(this.$data, this.$options.data());
          });
      },

      changePlayer(){
          // changes the active player to a non-active player
          this.activePlayer = this.nonActivePlayer;
      },

      changeGameStatus(){
          if(this.checkForWin){
              return this.gameIsWon();
              // check if game is not won and all cells are filled
          } else {
              if(this.moves === 9){
                  return 'draw';
              }
          }

          return `${this.activePlayer}'s turn`;
      },

      checkForWin(){
          for(let i = 0; i < this.winConditions.length; i ++){
                let wc = this.winConditions[i];
                let cells = this.cells;

                // compare 3 cell values based on the cells in condition
                if(this.areEqual(cells[wc[0]], cells[wc[1]], cells[wc[2]])){
                    return true;
                }
          }
          return false;
      },

      areEqual(){
          // helper function to compare cell values
          var len = arguments.length;

          // loop through each cell value and compare with emptry string or inequality
          for(var i = 0; i < len; i ++){
              if(arguments[i] === '' || arguments[i] !== arguments[i - 1]){
                 return false;
              }
          }
          return true;
      },

      gameIsWon(){
          // fires event for App component to change score
          Event.$emit('win', this.activePlayer);

          // set the gameStatus message
          this.gameStatusMessage = `${this.activePlayer} + ' wins!'`;

          // fire event for cell to freeze
          Event.$emit('freeze');

          return 'win';
      }
  },

  computed: {
      // helper property to get the non-active player
      nonActivePlayer(){
          if(this.activePlayer === 'O'){
                return 'X';
          }

          return 'O';
      }
  },

  watch: {
      // watches for change in the game status, status message and color
      gameStatus(){
          if(this.gameStatus === 'win'){
              this.gameStatusColor = 'statusWin';
              return;
            //   this.gameStatusMessage = `${this.activePlayer} + 'wins!'`;
            //   return;
          } else if (this.gameStatus === 'draw'){
              this.gameStatusColor = 'statusDraw';
              this.gameStatusMessage = 'Draw!';
              return;
          }

          this.gameStatusMessage = `${this.activePlayer}'s turn`;
      }
  }
}
</script>

<style>

.grid {
    background-color: #34495e;
    color: #fff;
    width: 100%;
    border-collapse: collapse;
}

.gameStatus {
    margin: 0px;
    padding: 15px;
    border-top-left-radius: 20px;
    border-top-right-radius: 20px;
    background-color: #f1c40f;
    color: #fff;
    font-size: 1.4em;
    font-weight: bold;
}

.statusTurn {
    background-color: #f1c40f;
}

.statusWin {
    background-color: #2ecc71;
}

.statusDraw {
    background-color: #9b59b6;
}
</style>
