<template>

  <!-- Roller page -->
  <div class="dice-roller p-3" v-show="!showHistory">
    <!-- Add dice area-->
    <div class="d-flex justify-content-center mb-3">
      <div role="button" v-for="(dice, index) in diceTypes" :key="index" class="btn-dice m-1" v-bind:class="dice.colour"  @click="addDice(dice.colour, $event)">
        <span v-if="!showResults && countDice(dice.colour) > 0">{{countDice(dice.colour)}}</span>
        <span v-else-if="!rolling"><fa icon="plus" size="sm"/></span>
      </div>
    </div>
    <!-- Dice pool area-->
    <div class="d-flex flex-wrap justify-content-center mt-3">
      <div role="button" v-for="(dice, index) in diceRoll" :key="index" class="result-dice m-1" v-bind:class="[dice.colour, {rolling: rolling}]" @click="removeDice(index)">
        <div v-if="dice.result" class="face" v-bind:class="'face-'+dice.result"></div>
      </div>
    </div>
    <!-- Results area-->
    <div v-if="showResults" class="d-flex justify-content-center mt-3">
      <div v-for="(result, index) in rollResults" :key="index">
        <div v-if="result.count > 0" class="m-1 result" v-bind:class="'result-'+result.icon">
          {{result.count}}
        </div>
      </div>
    </div>
    <!-- Main buttons area-->
    <div class="fixed-bottom d-flex align-items-center justify-content-center mb-3">    
      <button type="button" class="btn btn-dark m-3" @click="resetRoll()" :disabled="!countPool || rolling"><fa icon="trash"/></button>
      <button type="button" class="btn btn-primary btn-lg" @click="rollAllDice()" :disabled="!countPool || rolling"><fa icon="dice" size="2x"/></button>
      <button type="button" class="btn btn-dark m-3" @click="toggleHistory()" :disabled="history.length < 1 || rolling"><fa icon="list"/></button>
    </div>
  </div>

  <!-- Roll history page -->
  <div id="roll-history" v-show="showHistory" class="roll-history modal-fullpage p-3">
    <div v-for="(r, index) in history" :key="index" class="d-flex justify-content-center bg-white my-3 rounded-pill">
      <span v-for="(resultDice, index) in r" :key="index">
        <span v-if="resultDice.count > 0" class="m-1 result" v-bind:class="'result-'+resultDice.icon">
          {{resultDice.count}}
        </span>
      </span>
    </div>
    <div class="fixed-bottom d-flex align-items-center justify-content-center mb-3">
      <button type="button" class="btn btn-dark m-2" @click="toggleHistory()"><fa icon="arrow-left"/></button>
    </div>
  </div>

</template>

<script>
export default {
  name: 'Roller',
  props: {
    msg: String
  },
  data: () => {
    return {
      diceTypes: [
        {colour: 'white', faces: {1: [], 2: [], 3: ['hit'], 4: ['hit'], 5: ['man'], 6: ['agi']}},
        {colour: 'blue', faces: {1: [], 2: [], 3: ['hit', 'man'], 4: ['hit', 'man'], 5: ['hit', 'man'], 6: ['agi']}},
        {colour: 'green', faces: {1: [], 2: [], 3: ['hit', 'agi'], 4: ['hit', 'agi'], 5: ['hit', 'agi'], 6: ['man']}},
        {colour: 'red', faces: {1: [], 2: [], 3: ['hit'], 4: ['hit', 'hit'], 5: ['hit', 'man'], 6: ['hit', 'agi']}},
        {colour: 'black', faces: {1: [], 2: [], 3: ['block'], 4: ['block'], 5: [], 6: []}},
      ],
      diceRoll: [],
      rollResults: [],
      showHistory: false,
      history: [],
      showResults: false,
      rolling: false,
    }
  },
  computed: {
    countPool() {
      return this.diceRoll.length;
    },
  },
  methods: {
    addDice(diceColour, e) {
      if (this.diceRoll.length > 0 && this.showResults) {
        this.resetRoll();
        this.showResults = false;
      }
      if (this.countDice(diceColour) < 15 && !this.rolling) {
        this.diceRoll.push({colour: diceColour, result: null});
        const diceButton = e.target.closest('.btn-dice');
        diceButton.classList.remove('animate');
        diceButton.classList.add('animate');
        setTimeout(function(){
          diceButton.classList.remove('animate');
        },200);
      }
    },
    removeDice(index) {
      if (!this.showResults && !this.rolling) {
        this.diceRoll.splice(index, 1);
      }
    },
    countDice(colour) {
      return this.diceRoll.filter(dice => dice.colour == colour).length;
    },
    rollDice(dice) {
      let count = 0;
      const timer = setInterval(() => {
        dice.result = Math.floor(Math.random() * 6) + 1;
        if (count >= 3) {
          clearInterval(timer);
        }
        count += 1;
      }, 400);
    },
    async rollDiceBasic(dice) {
      await new Promise(resolve => setTimeout(resolve, 1000));
      dice.result = Math.floor(Math.random() * 6) + 1;
    },
    async rollAllDice() {
      this.showResults = false;
      this.rolling = true;
      this.diceRoll.forEach(dice => this.rollDice(dice))
      await new Promise(resolve => setTimeout(resolve, 1600));
      this.rollResults = this.getRollResults();
      this.showResults = true;
      this.rolling = false;
      this.history.unshift(this.rollResults);
      if (this.history.length > 8) {
        this.history.pop();
      }
    },
    getRollResults() {
      let resultRaw = [];
      let result = [];
      this.diceRoll.forEach(
        roll =>  {
          if (roll.result) {
            this.getDiceResult(roll.colour, roll.result).forEach(diceResult => resultRaw.push(diceResult));
          }
        }
      );
      result.push({icon: 'hit', count: resultRaw.filter(i => i == 'hit').length});
      result.push({icon: 'block', count: resultRaw.filter(i => i == 'block').length});
      result.push({icon: 'agi', count: resultRaw.filter(i => i == 'agi').length});
      result.push({icon: 'man', count: resultRaw.filter(i => i == 'man').length});
      return result;
    },
    resetRoll() {
      this.diceRoll = [];
      this.result = [];
      this.showResults = false;
    },
    getDiceResult(colour, number) {
      return this.diceTypes.find(dice => dice.colour == colour).faces[number];
    },
    toggleHistory() {
      this.showHistory = !this.showHistory;
    }
  }
}
</script>

<style lang="scss" scoped>

</style>
