<script setup>
import { ref } from 'vue';
import SnakeGame from './SnakeGame.vue';
import Popup from './Popup.vue';
</script>

<template>
  <div class="navbar">
    <div class="navbar-brand">Snake</div>
    <div class="navbar-links">
      <button
        type="button"
        class="btn bar-button"
        @click="showPopup(buttonClicked.INSTRUCTIONS)"
      >
        <i class="fa-solid fa-circle-info"></i> Instructions
      </button>
      <button
        type="button"
        class="btn bar-button"
        @click="showPopup(buttonClicked.SETTINGS)"
      >
        <i class="fa-solid fa-gear"></i> Settings
      </button>
    </div>
  </div>
  <div class="score-div">Score: {{ score }}</div>
  <Popup
    v-if="isPopupVisible"
    :popupTypeInput="popupChoice"
    :savedValues="formOutput"
    @close="closePopup"
    @close-modal="closePopup"
  />
  <div class="snake-game">
    <SnakeGame :settings="settings" @score-updated="updateScore" />
  </div>
</template>

<script>
export default {
  data() {
    return {
      score: 0,
      isPopupVisible: false,
      buttonClicked: {
        INSTRUCTIONS: 'instructions',
        SETTINGS: 'settings',
      },
      popupChoice: '',

      //initialized settings to default
      settings: {
        width: 500,
        height: 500,
        bgColor: 'white',
        snakeColor: 'lightgreen',
        snakeBorderColor: 'black',
        foodColor: 'red',
        unitSize: 25,
        tickSpeed: 75,
      },
      formOutput: {
        //initialized to default values
        theme: 'Classic',
        difficulty: 'Medium',
        width: 500,
        height: 500,
      },
      //enum to represent possible themes
      themes: {
        CLASSIC: 'Classic',
        ARMY: 'ARMY',
        SPOOKY: 'Spooky',
        BEACH: 'Beach',
      },
      //enum to represent possible difficulties
      difficulties: {
        EASY: 'Easy',
        MEDIUM: 'Medium',
        HARD: 'Hard',
        EXTREME: 'Extreme',
      },
    };
  },
  components: {
    SnakeGame,
    Popup,
  },
  methods: {
    updateScore(score) {
      this.score = score;
    },
    showPopup(popupChoice) {
      this.isPopupVisible = true;
      this.popupChoice = popupChoice;
    },
    closePopup(output) {
      this.isPopupVisible = false;
      this.formOutput.theme = output.theme;
      this.formOutput.difficulty = output.difficulty;
      this.formOutput.width = output.width;
      this.formOutput.height = output.height;

      this.updateSettings();
    },
    //NEEDS MODIFICATION
    updateSettings() {
      this.settings = {
        //must run at least once updating settigns this way so that it updates
        ...this.settings,
        width: this.formOutput.width,
        height: this.formOutput.height,
      };

      switch (this.formOutput.difficulty) {
        case this.difficulties.EASY:
          this.settings.tickSpeed = 100;
          break;
        case this.difficulties.MEDIUM:
          this.settings.tickSpeed = 75;
          break;
        case this.difficulties.HARD:
          this.settings.tickSpeed = 50;
          break;
        case this.difficulties.EXTREME:
          this.settings.tickSpeed = 25;
          break;
      }

      switch (this.formOutput.theme) {
        case this.themes.CLASSIC:
          this.settings.bgColor = 'white';
          this.settings.snakeColor = 'lightgreen';
          this.settings.snakeBorderColor = 'black';
          this.settings.foodColor = 'red';
          break;
        case this.themes.ARMY:
          this.settings.bgColor = '#c8d1da';
          this.settings.snakeColor = '#8b9b74';
          this.settings.snakeBorderColor = 'black';
          this.settings.foodColor = '#3f6c29';
          break;
        case this.themes.SPOOKY:
          this.settings.bgColor = '#2b1d4f';
          this.settings.snakeColor = 'white';
          this.settings.snakeBorderColor = 'black';
          this.settings.foodColor = 'orange';
          break;
        case this.themes.BEACH:
          this.settings.bgColor = 'yellow';
          this.settings.snakeColor = '#ff59c2';
          this.settings.snakeBorderColor = 'black';
          this.settings.foodColor = '#46e5fa';
          break;
      }
    },
  },
};
</script>
