<template>
  <div id="app">
    <div class="simon-wrap">
      <div class="simon-circle">
        <div class="simon-row">
          <div class="simon-col">
            <div :class="`simon-btn simon-green ${ lights[0].status && 'light' }`" @click="makeMove(0)"></div>
          </div>
          <div class="simon-col">
            <div :class="`simon-btn simon-red ${ lights[1].status && 'light' }`" @click="makeMove(1)"></div>
          </div>
        </div>
        <div class="simon-row">
          <div class="simon-col">
            <div :class="`simon-btn simon-yellow ${ lights[2].status && 'light' }`" @click="makeMove(2)"></div>
          </div>
          <div class="simon-col">
            <div :class="`simon-btn simon-blue ${ lights[3].status && 'light' }`" @click="makeMove(3)"></div>
          </div>
        </div>
      </div>
      <div class="simon-actions">
        <div v-if="!agree">
          <button @click="agreed">Press to activate</button>
        </div>
        <div v-else>
          <button @click="start">{{ gameStarted ? 'Restart' : 'Start' }}</button>
          <select :selected="diff" v-model="diff" :disabled="gameStarted">
            <option value="0">Easy</option>
            <option value="1">Normal</option>
            <option value="2">Hard</option>
          </select>
          <button @click="demo">Demo</button>
          Round: {{ round }} <span v-if="over">Game Over!</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import * as Tone from "tone";

  const delay = t => {
    return new Promise(resolve => setTimeout(resolve, t));
  }

  export default {
    name: 'App',
    components: {
    },
    data() {
      return {
        agree: false,
        lock: false,
        gameStarted: false,
        synth: new Tone.Synth().toDestination(),
        round: 0,
        steps: [],
        playerSteps: [],
        lights: [
          { color: "green", status: false, key: "C" },
          { color: "red", status: false, key: "D" },
          { color: "yellow", status: false, key: "E" },
          { color: "blue", status: false, key: "F" },
        ],
        delayTime: [
                1500,
                1000,
                400
        ],
        over: false,
        diff: 1,
      }
    },
    methods: {
      agreed() {
        this.agree = true;
        Tone.start();
      },
      playNote(n) {
        this.synth.triggerAttackRelease(`${this.lights[n].key}4`, "8n");
      },
      makeMove(m) {
        if (!this.agree || this.lock) {
          return;
        }
          this.playNote(m);
        if (this.gameStarted) {
          this.playerSteps.push(m);
          let currentStep = this.playerSteps.length - 1;
          if(this.playerSteps[currentStep] === this.steps[currentStep]) {
            if(this.playerSteps.length === this.steps.length) {
              this.continue();
            }
          } else {
            this.gameOver();
          }
        }
      },
      async demo() {
        if (!this.agree || this.lock) {
          return;
        }
          this.lock = true;
          for (let [i, light] of this.lights.entries()) {
            this.playNote(i);
            light.status = true;
            await delay(this.delayTime[this.diff]);
            light.status = false;
          }
          this.lock = false;
      },
      addStep() {
        this.steps.push(Math.floor(Math.random() * 4));
      },
      async start() {
        this.over = false;
        this.gameStarted = true;
        this.steps = [];
        this.playerSteps = [];
        this.addStep();
        this.round = this.steps.length;
        this.playNote(this.steps[0]);
        this.lights[this.steps[0]].status = true;
        await delay(this.delayTime[this.diff]);
        this.lights[this.steps[0]].status = false;
      },
      async continue() {
        this.playerSteps = [];
        await delay(this.delayTime[this.diff]);
        this.addStep();
        this.round = this.steps.length;
        this.lock = true;
        for (const step of this.steps) {
          this.playNote(step);
          this.lights[step].status = true;
          await delay(this.delayTime[this.diff]);
          this.lights[step].status = false;
        }
        this.lock = false;
      },
      gameOver() {
        this.over = true;
        this.gameStarted = false;
        this.steps = [];
        this.playerSteps = [];
      }
    },
  }
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
}
  .simon-wrap {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .simon-circle {
    display: flex;
    flex-direction: column;
  }

  .simon-actions {
    padding: 16px;
  }

  .simon-row {
    display: flex;
    flex-direction: row;
  }

  .simon-col {
    display: flex;
    flex-direction: column;
  }

  .simon-btn {
    width: 128px;
    height: 128px;
    cursor: pointer;
  }

  .simon-green {
    background-color: #00ac00;
    border-radius: 100% 0 0 0;

    &:hover, &.light {
      background-color: #00ff00;
    }
  }

  .simon-red {
    background-color: #bf0000;
    border-radius: 0 100% 0 0;

    &:hover, &.light {
      background-color: #ff0000;
    }
  }

  .simon-yellow {
    background-color: #9aa500;
    border-radius: 0 0 0 100%;

    &:hover, &.light {
      background-color: #eeff00;
    }
  }

  .simon-blue {
    background-color: #002590;
    border-radius: 0 0 100% 0;

    &:hover, &.light {
      background-color: #005eff;
    }
  }
</style>
