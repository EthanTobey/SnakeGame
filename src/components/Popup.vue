<script setup></script>

<template>
  <div class="modal" tabindex="-1" role="dialog" style="display: block">
    <div class="modal-dialog" role="document">
      <div class="modal-content">
        <div class="modal-header">
          <h5
            v-if="this.popupType == this.popupTypes.INSTRUCTIONS"
            class="modal-title text-center w-100"
          >
            Instructions
          </h5>
          <h5
            v-if="this.popupType == this.popupTypes.SETTINGS"
            class="modal-title text-center w-100"
          >
            Settings
          </h5>
          <button type="button" class="modal-close-btn" @click="closeModal">
            <i class="fa-solid fa-x"></i>
          </button>
        </div>
        <!--Popup for Controls-->
        <div
          v-if="this.popupType == this.popupTypes.INSTRUCTIONS"
          class="modal-body px-4"
        >
          <div class="text-entry mb-4">
            <h3 class="modal-h3">Welcome to Snake!</h3>
            <p class="modal-text mb-4 ml">
              Welcome to the classic video game Snake! Slither around and gobble
              up tasty treats to grow your snake. But watch out! If you run into
              walls or your own tail, it's game over. How long can you make your
              snake before it's game over?
            </p>
            <hr />
          </div>
          <div class="text-entry">
            <h3 class="modal-h3">Controls</h3>
            <span class="modal-text mb-3 ml">
              <ol class="custom-list">
                <li>Press "Play" Button To Play The Game</li>
                <li>Use Arrow Keys To Control The Snake</li>
                <li>Slither Over The Apples To Get Points</li>
                <li>
                  Press "Pause" Button Or "Escape" Key <br />To Pause The Game
                </li>
                <li>Don't Run Into Your Tail!</li>
              </ol>
            </span>
          </div>
        </div>
        <!--popup for Settings-->
        <div
          v-if="this.popupType == this.popupTypes.SETTINGS"
          class="modal-body px-4"
        >
          <form @submit.prevent="handleSubmit">
            <!-- Theme Dropdown-->
            <div class="mb-3 row align-items-center form-padding">
              <label for="dropdown" class="col-sm-3 col-form-label modal-h3"
                >Theme:</label
              >
              <div class="col-sm-8">
                <select
                  id="dropdown1"
                  v-model="formOutput.theme"
                  class="form-select form-dropdown"
                >
                  <option disabled value="">Choose an option</option>
                  <option value="Classic">Classic</option>
                  <option value="ARMY">Army</option>
                  <option value="Spooky">Spooky</option>
                  <option value="Beach">Beach</option>
                </select>
              </div>
            </div>
            <!-- Difficulty Dropdown-->
            <div class="mb-3 row align-items-center form-padding">
              <label for="dropdown" class="col-sm-5 col-form-label modal-h3"
                >Difficulty:</label
              >
              <div class="col-sm-6">
                <select
                  id="dropdown2"
                  v-model="formOutput.difficulty"
                  class="form-select form-dropdown"
                >
                  <option disabled value="">Choose an option</option>
                  <option value="Easy">Easy</option>
                  <option value="Medium">Medium</option>
                  <option value="Hard">Hard</option>
                  <option value="Extreme">Extreme</option>
                </select>
              </div>
            </div>
            <!-- Game Width Slider-->
            <div class="mb-3 form-padding">
              <label for="slider" class="form-label">
                <span class="modal-h3"
                  >Board Width:{{ this.formOutput.width }}</span
                >
              </label>
              <div class="row mt-1">
                <div class="col-sm-11">
                  <input
                    type="range"
                    id="slider1"
                    v-model="formOutput.width"
                    class="form-range form-slider"
                    min="250"
                    :max="windowWidth"
                    step="25"
                  />
                  <!-- WARNING - hard coded step - if game grid size changes, this needs to as well-->
                </div>
              </div>
            </div>
            <!-- Game Height Slider-->
            <div class="mb-3 form-padding">
              <label for="slider" class="form-label modal-h3"
                >Board Height:{{ this.formOutput.height }}</label
              >
              <div class="row mt-1">
                <div class="col-sm-11">
                  <input
                    type="range"
                    id="slider2"
                    v-model="formOutput.height"
                    class="form-range form-slider"
                    min="250"
                    :max="windowHeight"
                    step="25"
                  />
                  <!-- WARNING - hard coded step - if game grid size changes, this needs to as well-->
                </div>
              </div>
            </div>
          </form>
        </div>
        <div class="modal-footer d-flex justify-content-center">
          <span class="small">
            Game and Website Developed by
            <a
              href="https://ethantobey.github.io/"
              target="_blank"
              class="link-muted"
              >Ethan Tobey</a
            ></span
          >
        </div>
      </div>
    </div>
  </div>
  <div class="modal-backdrop fade show"></div>
</template>

<script>
export default {
  props: {
    popupTypeInput: String,
    savedValues: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      popupTypes: {
        INSTRUCTIONS: 'instructions',
        SETTINGS: 'settings',
      },
      popupType: this.popupTypeInput,
      formOutput: {
        theme: this.savedValues.theme,
        difficulty: this.savedValues.difficulty,
        width: this.savedValues.width,
        height: this.savedValues.height,
      },
      windowWidth:
        window.innerWidth > 475
          ? Math.floor(window.innerWidth / 25) * 25 - 200
          : 275,
      windowHeight:
        window.innerHeight > 575
          ? Math.floor(window.innerHeight / 25) * 25 - 300
          : 275,
    };
  },
  methods: {
    closeModal() {
      this.$emit('close-modal', this.formOutput);
    },
  },
};
</script>
