<template>
  <h1>Minesweeper</h1>
  <div class="minesweeper">

    <!-- Settings form for user input -->
    <div v-if="!gameStarted" class="settings">
      <h2>Game Settings</h2>
      <label for="rows">Rows:</label>
      <input type="number" id="rows" v-model.number="rows" min="5" max="20" />

      <label for="cols">   Columns:</label>
      <input type="number" id="cols" v-model.number="cols" min="5" max="20" />

      <label for="mines">   Mines:</label>
      <input type="number" id="mines" v-model.number="mines" min="1" :max="maxMines" />
      <br>
      <button @click="startGame">Start Game</button>
    </div>

    <!-- Game Grid -->
    <div v-if="gameStarted" >
      <div v-if="gameOver" class="game-over">
        <p>{{ gameOverMessage }}</p>
        <button @click="resetGame">Play Again</button>
      </div>

      <div class="grid" :style="{ gridTemplateColumns: `repeat(${cols}, 40px)` }">
        <div
            v-for="(cell, index) in grid"
            :key="index"
            :class="['cell', cell.state]"
            @click="revealCell(index)"
            @contextmenu.prevent="flagCell(index)"
            :style="getCellStyle(cell)"
        >
          <span v-if="cell.state === 'revealed'">{{ cell.value }}</span>
          <span v-if="cell.state === 'flagged'">🚩</span>
        </div>
      </div>

      <div class="stats">
        <p>Flags left: {{ flagsLeft }}</p>
        <p v-if="!gameOver">Mines: {{ mines }}</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      rows: 10, // Default rows
      cols: 10, // Default columns
      mines: 10, // Default number of mines
      maxMines: 0, // Max number of mines (based on grid size)
      grid: [], // The grid array
      gameStarted: false, // Flag to check if the game has started
      gameOver: false, // Flag for game over
      gameOverMessage: '', // Game over message
      flagsLeft: 0, // Flags remaining
    };
  },
  methods: {
    // Start the game with the user-defined settings
    startGame() {
      if (this.mines > this.rows * this.cols) {
        alert('Number of mines cannot exceed the number of cells!');
        return;
      }

      this.maxMines = Math.floor(this.rows * this.cols * 0.2); // Limit max mines to 20% of grid size
      this.flagsLeft = this.mines;
      this.gameStarted = true;
      this.gameOver = false;
      this.gameOverMessage = '';
      this.grid = [];
      this.createGrid();
    },

    // Create a new grid with mines placed
    createGrid() {
      let grid = [];
      // Initialize grid with empty cells
      for (let i = 0; i < this.rows * this.cols; i++) {
        grid.push({
          state: 'hidden', // 'hidden', 'revealed', 'flagged'
          value: 0, // Will store mine count or 'M' for a mine
        });
      }

      // Place mines randomly
      let minesPlaced = 0;
      while (minesPlaced < this.mines) {
        const randomIndex = Math.floor(Math.random() * grid.length);
        if (grid[randomIndex].value !== 'M') {
          grid[randomIndex].value = 'M';
          minesPlaced++;
        }
      }

      // Calculate the number of adjacent mines for each non-mine cell
      for (let i = 0; i < grid.length; i++) {
        if (grid[i].value === 'M') continue;
        let surroundingMines = 0;
        const neighbors = this.getNeighbors(i);
        neighbors.forEach((neighborIndex) => {
          if (grid[neighborIndex]?.value === 'M') surroundingMines++;
        });
        grid[i].value = surroundingMines;
      }

      this.grid = grid;
    },

    // Get neighbors of a given cell index
    getNeighbors(index) {
      const row = Math.floor(index / this.cols);
      const col = index % this.cols;
      const neighbors = [];

      // Check 8 possible neighbors
      for (let r = -1; r <= 1; r++) {
        for (let c = -1; c <= 1; c++) {
          const neighborRow = row + r;
          const neighborCol = col + c;
          if (neighborRow >= 0 && neighborRow < this.rows && neighborCol >= 0 && neighborCol < this.cols) {
            const neighborIndex = (neighborRow * this.cols) + neighborCol;
            if (neighborIndex !== index) neighbors.push(neighborIndex);
          }
        }
      }
      return neighbors;
    },

    // Reveal a cell when clicked
    revealCell(index) {
      const cell = this.grid[index];

      if (cell.state === 'flagged' || this.gameOver) return; // Ignore flagged or game over cells

      cell.state = 'revealed';

      if (cell.value === 'M') {
        this.gameOver = true;
        this.gameOverMessage = 'Game Over! You hit a mine!';
        return;
      }

      // If the cell is empty (no adjacent mines), reveal surrounding cells
      if (cell.value === 0) {
        const neighbors = this.getNeighbors(index);
        neighbors.forEach((neighborIndex) => {
          const neighborCell = this.grid[neighborIndex];
          if (neighborCell.state === 'hidden') {
            this.revealCell(neighborIndex);
          }
        });
      }

      // Check if the player has won (all non-mine cells revealed)
      this.checkWin();
    },

    // Flag a cell by right-clicking
    flagCell(index) {
      const cell = this.grid[index];
      if (cell.state === 'revealed' || this.gameOver) return;

      if (cell.state === 'hidden') {
        cell.state = 'flagged';
        this.flagsLeft--;
      } else if (cell.state === 'flagged') {
        cell.state = 'hidden';
        this.flagsLeft++;
      }
    },

    // Check if the player has won
    checkWin() {
      const revealedCells = this.grid.filter(cell => cell.state === 'revealed');
      const nonMineCells = this.grid.filter(cell => cell.value !== 'M');
      if (revealedCells.length === nonMineCells.length) {
        this.gameOver = true;
        this.gameOverMessage = 'You Win!';
      }
    },

    // Reset the game
    resetGame() {
      this.gameStarted = false;
      this.grid = [];
      this.rows = 10;
      this.cols = 10;
      this.mines = 10;
    },

    // Get dynamic styles for text color based on the cell's value
    getCellStyle(cell) {
      if (cell.state !== 'revealed') return {};

      let textColor = '#000000'; // Default text color (black)

      if (cell.value === 0) {
        textColor = '#808080'; // Gray for empty cells
      } else if (cell.value === 1) {
        textColor = '#1E90FF'; // DodgerBlue for 1
      } else if (cell.value === 2) {
        textColor = '#a85ca7'; // Blue for 2
      } else if (cell.value === 3) {
        textColor = '#c5465c'; // DarkBlue for 3
      } else if (cell.value >= 4) {
        textColor = '#971504'; // MidnightBlue for 4+
      }

      return { color: textColor };
    }
  },
};
</script>

<style scoped>
.minesweeper {
  display: flex;
  justify-content: center;
  height: 100vh; /* Makes the container fill the full height of the viewport */
  padding: 20px;
  text-align: center;
}

.settings {
  margin-bottom: 20px;
}

label {
  margin-right: 10px;
}

input[type="number"] {
  width: 60px;
  padding: 5px;
  margin: 10px 0;
}

button {
  padding: 10px 20px;
  background-color: #4CAF50;
  color: white;
  border: none;
  cursor: pointer;
  font-size: 1rem;
  border-radius: 4px;
}

button:hover {
  background-color: #45a049;
}

.grid {
  display: grid;
  gap: 2px;
  justify-items: center;
  margin: 20px auto;
}

.cell {
  width: 40px;
  height: 40px;
  background-color: #d3d3d3;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 18px;
  cursor: pointer;
  border-radius: 4px;
}

.cell:hover {
  background-color: #b0b0b0;
}

.cell.revealed {
  background-color: #f0f0f0;
  cursor: default;
}

.cell.flagged {
  background-color: #f0f0f0;
  color: red;
}

.game-over {
  margin-top: 20px;
}

.stats {
  margin-top: 20px;
}
</style>