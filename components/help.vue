<template>
  <div class="help-container">
    <div
      :style="{
        width:'100%'
      }"
    >
      <button class="float-right" @click="$emit('close')">
        X
      </button>
      <h1>HOW TO USE</h1>
    </div>
    <div class="mb-8">
      <h2>Basic Usage</h2>
      <img src="~assets/img/help-input.png">
      <img src="~assets/img/help-submit.png">
    </div>
    <div class="mb-8">
      <center>
        <h2>
          Toggle to ALL or STEPS <br>
          <img src="~assets/img/help-show-settings.png"> <br>
          SETTINGS DEFINITION <br>
          ALL Settings: Shows all steps <br>
          STEPS Settings: Show step by step <br>
          NOTE: You can toggle this in the result table
        </h2>
      </center>
    </div>
    <div class="mb-8">
      <h2>Sample Results(ALL)</h2> <br>
      <center>
        <img src="~assets/img/results-table-all.png">
      </center>
    </div>
    <div>
      <h2>Sample Results(STEPS)</h2><br>
      <img src="~assets/img/results-table-steps1.png">
      <img src="~assets/img/results-table-steps2.png">
    </div>
    <div>
      <img src="~assets/img/results-table-steps-final.png">
    </div>
    <div>
      <h2>
        Shortcuts <br>
        <div class="left-align">
          Enter: Get Results <br>
          H: Show Help <br>
          T: Toggle Settings <br>
          N or Right Key: Next Step <br>
          P or Left Key: Prev Step <br>
          X or Esc: Exit Result/Help <br>
          S: Save Results <br>
        </div>
      </h2>
    </div>
  </div>
</template>

<script>
export default {}
</script>

<style scoped>
h1 {
  margin: 0;
  padding: 0;
  text-align: center;
}

h2 {
  text-align: center;
}

.help-container {
  position: sticky;
  margin: 24px auto;
  padding: 12px 24px;
  width: max-content;

  display: flex;
  flex-direction: column;

  border: 4px solid black;
  background-color: white;

  align-items: center;
  justify-content: center;
}

.left-align {
   text-align: left;
}

.float-right {
   float: right;
}

.mb-8 {
  margin-bottom: 8px;
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

button {
  margin: 4px;
  border: 2px solid black;
  font-family: Commodore;
  outline: none;
}

button:hover {
  background-color: black;
  color: white;
}
</style>
