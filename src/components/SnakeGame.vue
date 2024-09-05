<script setup>
import { ref, watch, onMounted } from 'vue';

// Define Emits
const emit = defineEmits(['score-updated']); //this allows you to define emits from script setup
//Define Props
const props = defineProps(['settings']);

// Variable Declarations
const gameBoard = ref(null);
const ctx = ref(null);
const resetBtn = ref(null);

//updatable Settings
const gameWidth = ref(props.settings.width);
const gameHeight = ref(props.settings.height);
const boardBackground = ref(props.settings.bgColor); //colors of things in game
const snakeColor = ref(props.settings.snakeColor);
const snakeBorder = ref(props.settings.snakeBorderColor);
const foodColor = ref(props.settings.foodColor);
const tickSpeed = ref(props.settings.tickSpeed);

//watch for props changes, and update refs for all settings
watch(
  () => props.settings,
  (newValue) => {
    gameWidth.value = `${newValue.width}`;
    gameHeight.value = `${newValue.height}`;
    boardBackground.value = `${newValue.bgColor}`;
    snakeColor.value = `${newValue.snakeColor}`;
    snakeBorder.value = `${newValue.snakeBorderColor}`;
    foodColor.value = `${newValue.foodColor}`;
    tickSpeed.value = `${newValue.tickSpeed}`;

    //update gameOver size
    gameOverFontSize = 0.08 * gameWidth.value;
  }
);

const unitSize = props.settings.unitSize; //size of everything in the game - keeping this constant
let gameOverFontSize = 40; //size of gameOver text //updated depending on width of game
//variables that update during game runtime
const running = ref(false);
let paused = ref(false);
let xVelocity = unitSize; //move one unit every game tick
let yVelocity = 0; //not starting moving vertically
let inputXVelo = unitSize; //the player's input velocity before it has been updated by the tick
let inputYVelo = 0;
let foodX; //coordinates of the snake food
let foodY;
let score = 0;
//the snake is an array of objects - each object is a coordinate pair of where that body part is
let snake = [
  //make sure to always scale by unitSize
  { x: unitSize * 4, y: 0 },
  { x: unitSize * 3, y: 0 },
  { x: unitSize * 2, y: 0 },
  { x: unitSize, y: 0 },
  { x: 0, y: 0 },
]; //will initialize snake onMounted based on refs to gameSize

//after mounting, this is runtime execution
onMounted(() => {
  gameBoard.value = document.querySelector('#gameBoard');
  ctx.value = gameBoard.value.getContext('2d');
  resetBtn.value = document.querySelector('#resetBtn');

  createFood();
  drawFood();
  clearBoard();
  drawFood();
  drawSnake();
});

//METHODS
const gameStart = () => {
  running.value = true;

  createFood();
  drawFood();
  nextTick();
};

//this is our game loop function
const nextTick = () => {
  if (running.value && !paused.value) {
    setTimeout(() => {
      clearBoard();
      drawFood();
      moveSnake();
      drawSnake();
      checkGameOver();
      nextTick();
    }, tickSpeed.value); //the number here is the amount of time before a new tick refresh
  } else if (!running.value) {
    displayGameOver();
  }
};

const clearBoard = () => {
  ctx.value.fillStyle = boardBackground.value;
  ctx.value.fillRect(0, 0, gameWidth.value, gameHeight.value);
};

//create a food at a random location on the board
const createFood = () => {
  /* //generate a random number for a food coordinate
  const randomFood = (min, max) => {
    const randNum =
      Math.round((Math.random() * (max - min + min)) / unitSize) * unitSize;
    return randNum;
  };
  //assign the value of the food's coordinates
  foodX = randomFood(0, gameWidth.value - unitSize);
  foodY = randomFood(0, gameHeight.value - unitSize); */
  foodNotInBodyHelper(0);
};

const foodNotInBodyHelper = (retries) => {
  const randomFood = (min, max) => {
    const randNum =
      Math.round((Math.random() * (max - min + min)) / unitSize) * unitSize;
    return randNum;
  };
  //assign the value of the food's coordinates
  foodX = randomFood(0, gameWidth.value - unitSize);
  foodY = randomFood(0, gameHeight.value - unitSize);

  if (retries <= 20) {
    snake.forEach((bodyPiece) => {
      if (bodyPiece.x == foodX && bodyPiece.y == foodY) {
        foodNotInBodyHelper(retries++);
      }
    });
  }
};

const drawFood = () => {
  console.log('food drawn at: ' + foodX + ' : ' + foodY);
  ctx.value.fillStyle = foodColor.value;
  ctx.value.fillRect(foodX, foodY, unitSize, unitSize);
  ctx.value.strokeRect(foodX, foodY, unitSize, unitSize); //border
};

const moveSnake = () => {
  //set velocities to the final input from that tick
  xVelocity = inputXVelo;
  yVelocity = inputYVelo;
  const head = {
    x: snake[0].x + xVelocity, //new body piece for moving forward: front piece shifted by velocity
    y: snake[0].y + yVelocity,
  };
  snake.unshift(head); //unshift adds the new head body piece to front of array

  //if food is eaten
  if (foodIsEaten()) {
    //increase score
    score += 1;
    //send the score to parent
    emit('score-updated', score);
    createFood();
  } else {
    snake.pop(); //eliminate the tail
  }
};

//helper to check if food has been eaten
const foodIsEaten = () => {
  return snake[0].x == foodX && snake[0].y == foodY;
};

const drawSnake = () => {
  ctx.value.fillStyle = snakeColor.value;
  ctx.value.strokeStyle = snakeBorder.value;
  snake.forEach((snakePart) => {
    ctx.value.fillRect(snakePart.x, snakePart.y, unitSize, unitSize);
    ctx.value.strokeRect(snakePart.x, snakePart.y, unitSize, unitSize); //border
  });
};

const checkGameOver = () => {
  //check if hit a border of container
  switch (
    true //switch of true b/c we're not checking any input
  ) {
    //if run past left border
    case snake[0].x < 0:
      running.value = false;
      break;
    //if run past right border
    case snake[0].x >= gameWidth.value:
      running.value = false;
      break;
    //if run past top border
    case snake[0].y < 0:
      running.value = false;
      break;
    //if run past right border
    case snake[0].y >= gameHeight.value:
      running.value = false;
      break;
  }

  //check if ran into self
  for (let i = 1; i < snake.length; i++) {
    //start loop from index 1 to exclude head
    //if head position is same as position of one of the body parts
    if (snake[i].x == snake[0].x && snake[i].y == snake[0].y) {
      running.value = false;
    }
  }
};

const displayGameOver = () => {
  ctx.value.font = `${gameOverFontSize}px "Press Start 2P"`;
  ctx.value.fillStyle = 'black';
  ctx.value.textAlign = 'center';
  ctx.value.fillText('GAME OVER!', gameWidth.value / 2, gameHeight.value / 2);
  running.value = false;
};

//change direction on keypress from event listener
const changeDirection = (event) => {
  const keyPressed = event.keyCode;
  //numerical values corresponding to keys returned by keypress event
  const LEFT = 37;
  const RIGHT = 39;
  const UP = 38;
  const DOWN = 40;
  const ESCAPE = 27; //for pausing game

  //define some boolean checks to see which way we are going
  const goingUp = yVelocity == -unitSize;
  const goingDown = yVelocity == unitSize;
  const goingRight = xVelocity == unitSize;
  const goingLeft = xVelocity == -unitSize;

  switch (true) {
    //if going left and not going right
    case keyPressed == LEFT && !goingRight:
      inputXVelo = -unitSize; //assign to input velo and NEVER actual velo
      inputYVelo = 0; //this is because true velo should only update once per tick
      break;
    case keyPressed == UP && !goingDown:
      inputXVelo = 0;
      inputYVelo = -unitSize;
      break;
    case keyPressed == RIGHT && !goingLeft:
      inputXVelo = unitSize;
      inputYVelo = 0;
      break;
    case keyPressed == DOWN && !goingUp:
      inputXVelo = 0;
      inputYVelo = unitSize;
      break;
  }

  //check if 'escape' pressed
  if (keyPressed == ESCAPE) {
    console.log('called');
    handlePause();
  }
};

//event listener for keypresses
window.addEventListener('keydown', (event) => changeDirection(event));

//handle button press
const pushButton = () => {
  if (running.value) {
    handlePause();
  } else {
    resetGame();
  }
};

//helper to handle reset game
const resetGame = () => {
  score = 0;
  emit('score-updated', score);
  paused.value = false;
  xVelocity = unitSize;
  yVelocity = 0;
  inputXVelo = unitSize;
  inputYVelo = 0;

  snake = [
    //make sure to always scale by unitSize
    { x: unitSize * 4, y: 0 },
    { x: unitSize * 3, y: 0 },
    { x: unitSize * 2, y: 0 },
    { x: unitSize, y: 0 },
    { x: 0, y: 0 },
  ];

  gameStart();
};

const handlePause = () => {
  if (!paused.value) {
    paused.value = true;
  } else {
    paused.value = false;
    nextTick(); //start game again
  }
};
</script>

<template>
  <div id="gameContainer">
    <!-- : in front o fwidth and height binds them to the values of the field-->
    <canvas id="gameBoard" :width="gameWidth" :height="gameHeight"></canvas>
    <div class="justify-content-center game-btn-row">
      <button id="resetBtn" class="reset-btn" @click="pushButton">
        <!-- don't use .value for refs in template - automatically unwrapped by vue-->
        <span v-if="running && !paused">Pause</span>
        <span v-if="running && paused">Resume</span>
        <span v-if="!running">Play</span>
      </button>
    </div>
  </div>
</template>
