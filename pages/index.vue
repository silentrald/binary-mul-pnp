<template>
  <div id="index">
    <div class="input-container">
      <h1>Sequential Circuit Binary Multiplier</h1>
      <label for="m-text">
        M (Multiplicand)
      </label>
      <input
        id="m-text"
        v-model="mText"
        type="text"
        name="m-text"
        maxlength="16"
        autocomplete="false"
        autofocus
        :readonly="show || help"
      >
      <div v-if="errors.m" class="error">
        {{ errors.m }}
      </div>

      <label for="q-text">
        Q (Multiplier)
      </label>
      <input
        id="q-text"
        v-model="qText"
        type="text"
        maxlength="16"
        autocomplete="false"
        name="q-text"
        :readonly="show || help"
      >
      <div v-if="errors.q" class="error">
        {{ errors.q }}
      </div>

      <div class="limitations">
        Input: Binary(0/1) of max 16-bits
      </div>

      <div class="button-container no-select mb-8">
        <button :disabled="show || help" @click="multiply()">
          SUBMIT (↵)
        </button>
        <button :disabled="show || help" @click="showHelp()">
          <u>H</u>ELP
        </button>
      </div>
      <div class="button-container no-select mb-8">
        <button :disabled="show || help" @click="toggleShow()">
          Show Se<u>t</u>ting: {{ showSettings }}
        </button>
      </div>
    </div>

    <div v-if="show || help" id="dim" />

    <transition name="maximize">
      <div v-if="show" class="results-table">
        <div class="mb-8">
          <div class="mb-8">
            <button @click="save()">
              <u>S</u>AVE TXT
            </button>
            <button class="float-right" @click="hideShow()">
              X
            </button>
          </div>
          <div class="mb-8">
            <button @click="toggleShow()">
              Show Se<u>t</u>ting: {{ showSettings }}
            </button>
          </div>
        </div>

        <div class="mb-8">
          +M: {{ mTextUsed }} <br>
          -M: {{ mNegText }}
        </div>

        <div v-if="showSettings === 'ALL'">
          <table id="table" class="mb-8">
            <tr>
              <td>STEP</td>
              <td>A</td>
              <td>Q</td>
              <td>Q-1</td>
              <td>ACTION</td>
            </tr>
            <tr
              v-for="(step, i) in steps"
              :key="i"
            >
              <td v-if="step.action && step.action !== 'SHIFT'">
                {{ step.step }}
              </td>
              <td v-else-if="step.step === 0">
                INIT
              </td>
              <td v-else />
              <td>
                {{ step.a }}
              </td>
              <td> {{ step.q }} </td>
              <td> {{ step.q1 }} </td>
              <td> {{ step.action }} </td>
            </tr>
          </table>
          <div class="mb-8">
            ANSWER: {{ dec2bin(ans).padStart(size * 2, '0') }}
          </div>
        </div>
        <div v-else-if="showSettings === 'STEPS'">
          <table id="table" class="mb-8">
            <tr>
              <td />
              <td>STEP</td>
              <td>A</td>
              <td>Q</td>
              <td>Q-1</td>
              <td>ACTION</td>
            </tr>
            <tr
              v-for="(step, i) in currentSteps"
              :key="i"
            >
              <td v-if="index - 1 === i">
                &gt;
              </td>
              <td v-else />
              <td v-if="step.action && step.action !== 'SHIFT'">
                {{ step.step }}
              </td>
              <td v-else-if="step.step === 0">
                INIT
              </td>
              <td v-else />
              <td> {{ step.a }} </td>
              <td> {{ step.q }} </td>
              <td> {{ step.q1 }} </td>
              <td> {{ step.action }} </td>
            </tr>
          </table>

          <div v-if="index === steps.length" class="mb-8">
            ANSWER: {{ dec2bin(ans).padStart(size * 2, '0') }}
          </div>

          <div class="mb-8" :style="{ width: '100%' }">
            <button
              v-if="index < steps.length"
              :style="{ width: '100%' }"
              @click="index = size * 2 + 1"
            >
              SHOW ALL
            </button>
          </div>
          <div>
            <button v-if="index > 1" @click="changeIndex(-1)">
              &lt;- <u>P</u>REV
            </button>
            <button
              v-if="index < steps.length"
              class="float-right"
              @click="changeIndex(1)"
            >
              <u>N</u>EXT -&gt;
            </button>
          </div>
        </div>
      </div>
    </transition>

    <transition name="maximize">
      <help v-if="help" @close="help = false" />
    </transition>
  </div>
</template>

<script>
import { saveAs } from 'file-saver'

const BINARY_REGEX = /^(0|1)*$/

const LABELS = {
  m: 'Multiplicand',
  q: 'Multiplier'
}

export default {
  data () {
    return {
      show: false,
      mText: '0',
      mTextUsed: '',
      qText: '0',
      mNegText: '0',
      m: 0,
      a: 0,
      q: 0,
      ans: 0,
      size: 0,
      steps: [],
      index: 0,
      showSettings: 'ALL',
      help: false,
      saving: false,
      errors: {}
    }
  },

  computed: {
    currentSteps () {
      return this.show && this.steps.length > 0
        ? this.steps.slice(0, this.index)
        : []
    }
  },

  watch: {
    currentSteps () {
      // Repositions the scrolly to the current step
      if (document && document.getElementById('table')) {
        const { offsetTop } = document.getElementById('table').lastElementChild
        window.scrollTo(0, offsetTop)
      }
    }
  },

  mounted () {
    // listens to key presses
    document.addEventListener('keydown', (event) => {
      switch (event.keyCode) {
        case 37: // Left Arrow Key
        case 80: // P key
          if (this.show && this.showSettings === 'STEPS') { this.changeIndex(-1) }
          break
        case 39: // Right Arrow Key
        case 78: // N key
          if (this.show && this.showSettings === 'STEPS') { this.changeIndex(1) }
          break
        case 27: // ESC Key
        case 88: // X key
          this.show = false
          this.help = false
          break
        case 13: // Enter Key
          if (!this.show) { this.multiply() }
          break
        case 83: // S key
          if (this.show && this.steps.length > 0) { this.save() }
          break
        case 84: // T key
          this.toggleShow()
          break
        case 65:
          this.toggleAlgo()
          break
        case 72:
          if (!this.show || !this.help) { this.help = true }
          break
      }
    })
  },

  methods: {
    shiftLeft (num, shift) {
      return shift > 14
        ? (num << 14) * Math.pow(2, shift - 14)
        : num << shift
    },

    // Saves the steps into a text file and can be downloaded
    save () {
      if (this.steps.length === 0) { return }
      // Convert steps to string
      // Set Given
      const empty = ''.padStart(this.size, '=')
      let text = `Sequential Circuit Binary Multiplier Results

+M = ${this.mTextUsed}
-M = ${this.mNegText}

+======+=${empty}=+=${empty}=+=====+========+
| STEP | ${'A'.padStart(this.size, ' ')} | ${'Q'.padStart(this.size, ' ')} | Q-1 | ACTION |
|======+=${empty}=+=${empty}=|=====|========+
`
      for (const index in this.steps) {
        const step = this.steps[index]
        if (index === '0') {
          text += `| INIT | ${step.a} | ${step.q} |  ${step.q1}  |        |\n`
        } else {
          if (step.action && !step.action.startsWith('SHIFT')) {
            text += `| ${step.step.toString().padStart(4, ' ')} | `
          } else {
            text += '|      | '
          }
          text += `${step.a} | ${step.q} |  ${step.q1}  |  ${step.action} |\n`
        }
      }
      text += `+======+=${empty}=+=${empty}=+=====+========+\n\n`

      text += `ANS = ${this.dec2bin(this.ans, this.size * 2)}`
      const blob = new Blob([text], { type: 'text/plain;charset=utf-8' })
      saveAs(blob, 'results.txt')
    },

    // Updates the step counter
    changeIndex (val) {
      this.index = Math.min(this.steps.length, Math.max(1, this.index + val))
    },

    // Toggles show settings
    toggleShow () {
      this.showSettings = this.showSettings === 'ALL' ? 'STEPS' : 'ALL'
    },

    // Converts a decimal number to a string binary
    dec2bin (dec, size) {
      const bin = (dec >>> 0).toString(2)
      return bin.length < size
        ? bin.padStart(size, '0')
        : bin.substring(bin.length - size)
    },

    // Checks whether the string number is binary
    validateBinary (num, field) {
      let validate = true

      if (!num) {
        validate = false
        this.$set(this.errors, field, `${LABELS[field]} should not be empty`)
      } else if (typeof (num) !== 'string') {
        validate = false
        this.$set(this.errors, field, `${LABELS[field]} invalid type`)
      } else if (num.length > 16) {
        validate = false
        this.$set(this.errors, field, `${LABELS[field]} should be up to 16 bits`)
      } else if (!BINARY_REGEX.test(num)) {
        validate = false
        this.$set(this.errors, field, `${LABELS[field]} should be a binary number (0 or 1)`)
      }

      return validate
    },

    // Validates the mText and qText field if they are binary
    validate () {
      // sanitize spaces
      this.mText = this.mText.trim()
      this.qText = this.qText.trim()

      let validate = true

      // validate mText and qText
      if (!this.validateBinary(this.mText, 'm')) {
        validate = false
      }

      if (!this.validateBinary(this.qText, 'q')) {
        validate = false
      }

      return validate
    },

    // Binary Multiplication Logic
    multiply () {
      this.errors = {}
      if (!this.validate()) {
        return
      }

      this.show = true

      // Reset values
      this.steps = []
      this.index = 1

      // Get the greater length between the two
      this.size = Math.max(this.mText.length, this.qText.length)

      // Set the decimal representation
      this.m = parseInt(this.mText, 2)
      this.q = parseInt(this.qText, 2)
      this.a = 0

      // Pad zero
      this.mTextUsed = this.mText.padStart(this.size, '0')

      this.algorithm()
    },

    algorithm () {
      const inc = this.shiftLeft(1, this.size)
      const mNeg = inc - this.m

      // Get -M
      this.mNegText = this.dec2bin(mNeg, this.size)

      // 0 is SUB M
      // 1 is NO OP
      // 2 is ADD M
      const add = [mNeg, 0, this.m]
      const actions = ['SUB M', 'NO OP', 'ADD M']

      let q1 = 0

      // Add initialize step
      this.steps.push({
        step: 0,
        a: ''.padStart(this.size, '0'),
        q: this.qText.padStart(this.size, '0'),
        q1: 0
      })

      let temp, aText, qText
      for (let i = 0; i < this.size; i++) {
        // ADD, SUB, OR NO OP
        temp = q1 - (this.q & 1) + 1
        this.a = (this.a + add[temp]) % inc

        aText = this.dec2bin(this.a, this.size)
        qText = this.dec2bin(this.q, this.size)
        this.steps.push({
          step: i + 1,
          a: aText,
          q: qText,
          q1,
          action: actions[temp]
        })

        // SHIFT
        q1 = this.q & 1

        if (this.a & 1) {
          this.q += inc
        }
        this.q >>= 1

        if (aText.charAt(0) === '1') {
          this.a += inc
        }
        this.a >>= 1

        aText = this.dec2bin(this.a, this.size)
        qText = this.dec2bin(this.q, this.size)
        this.steps.push({
          step: i + 1,
          a: aText,
          q: qText,
          q1,
          action: 'SHIFT'
        })
      }

      this.ans = this.shiftLeft(this.a, this.size) + this.q
    },

    // Hides the result table
    hideShow () {
      this.show = false
    },

    showHelp () {
      this.help = true
    }
  },

  head: {
    title: 'Sequential Circuit Binary Multiplier'
  }
}
</script>

<style scoped>
@font-face {
  font-family: Commodore;
  src: url('~assets/fonts/Commodore Pixelized v1.2.ttf');
}

#index {
  font-family: Commodore;
}

#dim {
  position: fixed;
  top: 0;
  margin: 0;
  padding: 0;

  width: 100vw;
  height: 100vh;

  background-color: #000000df;
}

h1 {
  margin: 0;
  padding: 0;
  text-align: center;
}

label {
  margin-top: 8px;
  font-size: 20px;
}

input[type="text"] {
  border: 4px solid black;
  padding: 4px;
  font-family: Commodore;
  font-size: 20px;
  text-align: right;
}

button {
  color: black;
  background-color: white;

  border: 2px solid black;
  font-family: Commodore;
  outline: none;
}

button:hover {
  background-color: black;
  color: white;
}

button:disabled, button:disabled:hover {
  background-color: white;
  color: black;
}

.input-container {
  position: fixed;
  margin: 20vh calc(50% - 285px);

  width: 500px;
  padding: 1vh 1vw;

  display: flex;
  flex-direction: column;

  background-color: white;
  border: 10px solid black;

  outline: none;
}

.limitations {
  margin: 8px 0;
}

.results-table {
  position: relative;
  margin: 10vh auto;
  padding: 1vh 1vw;
  width: max-content;

  display: flex;
  flex-direction: column;

  border: 4px solid black;
  background-color: white;
}

.float-right {
  float: right;
}

.mb-8 {
  margin-bottom: 8px;
}

.error {
  color: red;
}

table {
  border-collapse: collapse;
}

tr > td {
  border: 2px solid black;
  background-color: white;
}

td {
  vertical-align: bottom;
  text-align: right;
}

/* Selections */
td::selection,
div::selection,
h1::selection,
label::selection,
input::selection {
  color: white;
  background-color: black;
}

.no-select {
  -webkit-touch-callout: none; /* iOS Safari */
    -webkit-user-select: none; /* Safari */
     -khtml-user-select: none; /* Konqueror HTML */
       -moz-user-select: none; /* Old versions of Firefox */
        -ms-user-select: none; /* Internet Explorer/Edge */
            user-select: none; /* Non-prefixed version, currently
                                  supported by Chrome, Edge, Opera and Firefox */
}

/* Animations */
.maximize-enter-active {
  animation: maximize steps(8, end) 1s;
}
.maximize-leave-active {
  animation: maximize steps(8, end) 1s reverse;
}

@keyframes maximize {
  0% {
    transform: scale(0, 0.1);
  }
  50% {
    transform: scale(1, 0.1);
  }
  100% {
    transform: scale(1, 1);
  }
}
</style>
