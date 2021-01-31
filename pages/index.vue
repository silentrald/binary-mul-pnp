<template>
  <div id="index">
    <div class="input-container">
      <h1>
        Binary Multiplication<br>
        (Pencil and Paper)
      </h1>
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
        <!-- <button :disabled="show || help" @click="showHelp()"> -->
        <button disabled>
          <u>H</u>ELP
        </button>
      </div>
      <div class="button-container no-select mb-8">
        <button :disabled="show || help" @click="toggleShow()">
          Show Se<u>t</u>ting: {{ showSettings }}
        </button>
      </div>
      <select v-model="algorithm" class="select-container">
        <option value="normal" selected>
          Normal
        </option>
        <option value="booth">
          Booth's
        </option>
        <option value="extbooth">
          Extended Booth's
        </option>
      </select>
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
          <select v-model="algorithm" class="select-container" @change="algo()">
            <option value="normal">
              Normal
            </option>
            <option value="booth">
              Booth's
            </option>
            <option value="extbooth">
              Extended Booth's
            </option>
          </select>
        </div>

        <div class="mb-8" :style="{ 'font-size': '20px' }">
          +M: {{ mTextUsed }} <br>
          -M: {{ mNegText }}
        </div>

        <div v-if="showSettings === 'ALL'" class="center">
          <table class="mb-8">
            <tr><td>{{ mTextUsed }}</td></tr>
            <tr v-if="algorithm === 'normal'">
              <td class="border-bottom">
                {{ qTextUsed }}
              </td>
            </tr>
            <tr v-else>
              <td class="border-bottom">
                {{ boothString() }}
              </td>
            </tr>
            <tr v-for="bin in steps[algorithm]" :key="bin">
              <td class="left-align">
                {{ bin }}
              </td>
            </tr>
            <tr>
              <td class="border-top">
                {{ dec2bin(ans).padStart(size * 2, '0') }}
              </td>
            </tr>
          </table>
        </div>
        <div v-else-if="showSettings === 'STEPS'" class="center">
          <table id="table" class="mb-8">
            <tr><td>{{ mTextUsed }}</td></tr>
            <tr v-if="algorithm === 'normal'">
              <td class="border-bottom steps">
                <span
                  v-for="(b, i) in qTextUsed"
                  :key="i"
                >
                  <span v-if="i === qTextUsed.length - index" style="color: red;">
                    {{ b }}
                  </span>
                  <span v-else>
                    {{ b }}
                  </span>
                </span>
              </td>
            </tr>
            <tr v-else>
              <td class="border-bottom steps">
                <span
                  v-for="(b, i) in (algorithm === 'booth' ? qBooth : qExtBooth)"
                  :key="i"
                >
                  <span v-if="i === (algorithm === 'booth' ? qBooth : qExtBooth).length - index" style="color: red;">
                    {{ b }}
                  </span>
                  <span v-else>
                    {{ b }}
                  </span>
                </span>
              </td>
            </tr>
            <tr v-for="bin in currentSteps" :key="bin">
              <td class="left-align">
                {{ bin }}
              </td>
            </tr>
            <tr v-if="index === steps[algorithm].length">
              <td class="border-top">
                {{ dec2bin(ans).padStart(size * 2, '0') }}
              </td>
            </tr>
          </table>
        </div>
        <div v-if="showSettings === 'STEPS'">
          <div class="mb-8" :style="{ width: '100%' }">
            <button
              v-if="index < steps[algorithm].length"
              :style="{ width: '100%' }"
              @click="index = steps[algorithm].length"
            >
              SHOW ALL
            </button>
          </div>
          <div>
            <button v-if="index > 1" @click="changeIndex(-1)">
              &lt;- <u>P</u>REV
            </button>
            <button
              v-if="index < steps[algorithm].length"
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

const BOOTH_MAP = {
  '00': 0,
  '01': 1,
  10: -1,
  11: 0
}

const EXT_BOOTH_MAP = {
  '000': 0,
  111: 0,
  '001': 1,
  '010': 1,
  101: -1,
  110: -1,
  '011': 2,
  100: -2
}

export default {
  data () {
    return {
      show: false,
      mText: '0',
      mTextUsed: '',
      qText: '0',
      qTextUsed: '',
      qBooth: [],
      qExtBooth: [],
      mNegText: '0',
      m: 0,
      mNeg: 0,
      a: 0,
      q: 0,
      ans: 0,
      size: 0,
      steps: {
        normal: undefined,
        booth: undefined,
        extbooth: undefined
      },
      algorithm: 'normal', // normal, booth, extbooth
      index: 0,
      showSettings: 'ALL',
      help: false,
      saving: false,
      errors: {}
    }
  },

  computed: {
    currentSteps () {
      return this.show && this.steps[this.algorithm].length > 0
        ? this.steps[this.algorithm].slice(0, this.index)
        : []
    },

    boothString () {
      return () => {
        const booth = this.algorithm === 'booth' ? this.qBooth : this.qExtBooth
        const n = booth.length
        let temp = ''
        let t
        for (let i = 0; i < n; i++) {
          t = booth[i]
          temp += t > 0
            ? `+${t}`
            : (t === 0 ? '0' : t)
        }

        return temp
      }
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

    repeatStr (str, n) {
      let temp = ''
      for (let i = 0; i < n; i++) {
        temp += str
      }
      return temp
    },

    // Saves the steps into a text file and can be downloaded
    save () {
      if (this.steps[this.algorithm].length === 0) { return }
      // Convert steps to string
      // Set Given
      let text = `Binary Multiplication (Pencil&Paper)
${this.algorithm === 'booth' ? 'Booth\'s Algorithm' : 'Extended Booth\'s Algorithm'}

+M: ${this.mTextUsed}
-M: ${this.mNegText}

`
      if (this.algorithm === 'normal') {
        const space = this.repeatStr(' ', this.size)
        text += `${space}${this.mTextUsed}\n${space}${this.qTextUsed}\n`
      } else {
        const space = this.repeatStr(' ', this.size * 2 - this.boothString().length)
        text += `${this.repeatStr(' ', this.size)}${this.mTextUsed}\n${space}${this.boothString()}\n`
      }

      const line = this.repeatStr('=', this.size * 2)

      text += `${line}\n`

      const n = this.steps[this.algorithm].length
      for (let i = 0; i < n; i++) {
        text += `${this.steps[this.algorithm][i]}\n`
      }

      text += `${line}
${this.dec2bin(this.ans).padStart(this.size * 2, '0')}
`
      const blob = new Blob([text], { type: 'text/plain;charset=utf-8' })
      saveAs(blob, 'results.txt')
    },

    // Updates the step counter
    changeIndex (val) {
      this.index = Math.min(this.steps[this.algorithm].length, Math.max(1, this.index + val))
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

    zeroExt (bin, size) {
      return bin.padStart(size, '0')
    },

    signExt (bin, size) {
      return bin.padStart(size, bin.charAt(0) === '0' ? '0' : '1')
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
      this.qTextUsed = this.qText.padStart(this.size, '0')

      // get neg
      this.mNeg = this.shiftLeft(1, this.size) - this.m
      this.mNegText = this.dec2bin(this.mNeg, this.size)

      this.algo()
    },

    algo () {
      this.index = 1
      switch (this.algorithm) {
        case 'normal':
          if (this.steps.normal === undefined) { this.normalAlgo() }
          break
        case 'booth':
          if (this.steps.booth === undefined) {
            this.boothAlgo()
          }
          break
        case 'extbooth':
          if (this.steps.extbooth === undefined) { this.extBoothAlgo() }
          break
        default:
          break
      }
    },

    normalAlgo () {
      const currentSize = this.size * 2
      let addNeg = false

      // check if q is negative
      if (this.qTextUsed.charAt(0) === '1') {
        addNeg = true
      }

      // reverse string
      const tempQ = this.qTextUsed.split('').reverse()

      this.steps.normal = []
      let accum = 0
      let current

      // Produce the steps in the program
      for (let i = 0; i < this.size; i++) {
        current = tempQ[i] === '1'
          ? this.signExt(this.mTextUsed, currentSize - i)
          : this.signExt('0', currentSize - i)

        this.steps.normal.push(current)
        accum += this.shiftLeft(parseInt(current, 2), i)
      }

      // Add the neg of m negative number
      if (addNeg) {
        current = this.shiftLeft(1, this.size) - this.m
        accum += this.shiftLeft(current, this.size)
        this.steps.normal.push(this.dec2bin(current, this.size))
      }

      this.ans = accum % this.shiftLeft(1, currentSize)
    },

    boothAlgo () {
      // convert multiplier to extended booth
      const currentSize = this.size * 2
      let current
      let accum = 0
      const temp = this.qTextUsed + '0'
      const max = this.shiftLeft(1, currentSize)

      this.qBooth = []
      const same = this.mNegText === this.mTextUsed

      for (let i = 0; i < this.size; i++) {
        this.qBooth.push(BOOTH_MAP[temp.substr(i, 2)])
      }

      this.steps.booth = []
      let j = 0

      for (let i = this.size - 1; i > -1; i--) {
        switch (this.qBooth[i]) {
          case 1:
            current = this.signExt(this.mTextUsed, currentSize - j)
            break
          case -1:
            current = same
              ? this.zeroExt(this.mTextUsed, currentSize - j)
              : this.signExt(this.mNegText, currentSize - j)
            break
          case 0:
            current = this.signExt('0', currentSize - j)
            break
        }

        this.steps.booth.push(current)
        accum += this.shiftLeft(parseInt(current, 2), j++)
      }

      this.ans = accum % max
    },

    extBoothAlgo () {
      const currentSize = this.size * 2
      let current
      let accum = 0
      const max = this.shiftLeft(1, currentSize)
      let temp = this.qTextUsed + '0'

      if ((temp.length & 1) === 0) {
        temp = temp.charAt(0) + temp
      }

      this.qExtBooth = []

      for (let i = 0; i < temp.length - 1; i += 2) {
        this.qExtBooth.push(EXT_BOOTH_MAP[temp.substr(i, 3)])
      }

      this.steps.extbooth = []
      const same = this.mNegText === this.mTextUsed

      let j = 0

      for (let i = this.qExtBooth.length - 1; i > -1; i--) {
        switch (this.qExtBooth[i]) {
          case 2:
            current = this.signExt(this.mTextUsed + '0', currentSize - j)
            break
          case -2:
            current = same
              ? this.zeroExt(this.mTextUsed + '0', currentSize - j)
              : this.signExt(this.mNegText + '0', currentSize - j)
            break
          case 1:
            current = this.signExt(this.mTextUsed, currentSize - j)
            break
          case -1:
            current = same
              ? this.zeroExt(this.mTextUsed, currentSize - j)
              : this.signExt(this.mNegText, currentSize - j)
            break
          case 0:
            current = this.signExt('0', currentSize - j)
            break
        }

        this.steps.extbooth.push(current)
        accum += this.shiftLeft(parseInt(current, 2), j)
        j += 2
      }

      this.ans = accum % max
    },

    // Hides the result table
    hideShow () {
      this.show = false
      this.steps = {}
    },

    showHelp () {
      this.help = true
    }
  },

  head: {
    title: 'Binary Multiplication'
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

select {
  border: 2px solid black;
  padding: 4px;
  font-family: Commodore;
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

.center {
  display: flex;
  justify-content: center;
}

.steps {
  display: flex;
  justify-content: right;
}

table {
  border-collapse: collapse;
}

/* tr > td {
  border: 2px solid black;
  background-color: white;
} */

td {
  vertical-align: bottom;
  text-align: right;

  font-size: 20px;
}

.left-align {
  text-align: left !important;
}

.border-top {
  border-top: 2px solid black;
}

.border-bottom {
  border-bottom: 2px solid black;
}

/* Selections */
td::selection,
div::selection,
h1::selection,
label::selection,
input::selection,
span::selection {
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
