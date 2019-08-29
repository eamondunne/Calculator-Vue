<template>
  <v-app id="inspire">
    <v-content>
      <v-container class="fill-height" fluid>
        <v-row align="center" justify="center">
          <v-col cols="12" sm="8" md="4">
            <v-card class="elevation-12">
              <div id="calculator">
                <div class="calculator-warning" v-if="failedConnection" disabled>UNABLE TO CONNECT TO API</div>
                <div class="calculator-input" disabled>{{display}}</div>
                <div class="calculator-row">
                  <div class="calculator-col">
                    <button class="calculator-btn" v-on:click="setValue(7)">7</button>
                  </div>
                  <div class="calculator-col">
                    <button class="calculator-btn" v-on:click="setValue(8)">8</button>
                  </div>
                  <div class="calculator-col">
                    <button class="calculator-btn" v-on:click="setValue(9)">9</button>
                  </div>
                  <div class="calculator-col">
                    <button class="calculator-btn arithmetic" v-on:click="handleOperator('/')">÷</button>
                  </div>
                </div>
                <div class="calculator-row">
                  <div class="calculator-col">
                    <button class="calculator-btn" v-on:click="setValue(4)">4</button>
                  </div>
                  <div class="calculator-col">
                    <button class="calculator-btn" v-on:click="setValue(5)">5</button>
                  </div>
                  <div class="calculator-col">
                    <button class="calculator-btn" v-on:click="setValue(6)">6</button>
                  </div>
                  <div class="calculator-col">
                    <button class="calculator-btn arithmetic" v-on:click="handleOperator('*')">×</button>
                  </div>
                </div>
                <div class="calculator-row">
                  <div class="calculator-col">
                    <button class="calculator-btn" v-on:click="setValue(1)">1</button>
                  </div>
                  <div class="calculator-col">
                    <button class="calculator-btn" v-on:click="setValue(2)">2</button>
                  </div>
                  <div class="calculator-col">
                    <button class="calculator-btn" v-on:click="setValue(3)">3</button>
                  </div>
                  <div class="calculator-col">
                    <button class="calculator-btn arithmetic" v-on:click="handleOperator('-')">-</button>
                  </div>
                </div>
                <div class="calculator-row">
                  <div class="calculator-col">
                    <button class="calculator-btn action" v-on:click="getResult()">=</button>
                  </div>
                  <div class="calculator-col">
                    <button class="calculator-btn" v-on:click="setValue(0)">0</button>
                  </div>
                  <div class="calculator-col">
                    <button class="calculator-btn action" v-on:click="clear()">AC</button>
                  </div>
                  <div class="calculator-col">
                    <button class="calculator-btn arithmetic" v-on:click="handleOperator('+')">+</button>
                  </div>
                </div>
              </div>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
const axios = require("axios");
export default {
  name: "Calculator",
  data: () => ({
    value: "",
    display: "",
    operand1: null,
    operand2: null,
    operator: null,
    result: null,
    finished: false,
    failedConnection: false
  }),
  methods: {
    setValue(val) {
      // Clear data if calculation is finished
      if (this.finished) {
        this.clear();
        this.finished = false;
      }
      this.value += val;
      this.display += val;
    },

    handleOperator(op) {
      // Handle if user changes operator
      if (this.operator !== null && this.value === "") {
        this.operator = op;
        this.display = this.display.slice(0, this.display.length - 1);
        this.display += op;
        return;
      }
      // Disable operator selection until operand value is inputted
      if (this.value === "" && this.display === "") {
        return;
      }
      // Allows for operation chaining (will compute current first)
      if (this.operator !== null) {
        this.getResult();
      }
      this.operator = op;
      if (this.finished === true) {
        this.operand1 = this.result;
        this.display = "" + this.operand1 + op;
        this.finished = false;
      } else {
        this.display += op;
        this.operand1 = parseInt(this.value);
      }
      this.value = "";
    },

    getResult() {
      // Deactivate result query if no inputs entered or already computed
      if ((this.value === "" && this.display === "") || this.finished) {
        return;
      }
      // Divide by 0
      if (this.operand1 === "Infinity" || this.result === "Infinity"){
        this.result = "Infinity";
        this.display += "=" + this.result;
        this.operator = null;
        this.finished = true;
        return;
      }
      this.operand2 = parseInt(this.value);
      this.value = "";
      this.callAPI();
    },

    async callAPI() {
      let requestURL = "http://localhost:10000/api/v1/";
      switch (this.operator) {
        case "+":
          requestURL += "add";
          break;
        case "-":
          requestURL += "subtract";
          break;
        case "*":
          requestURL += "multiply";
          break;
        case "/":
          requestURL += "divide";
          break;
      }
      let request =
        requestURL + "?num1=" + this.operand1 + "&num2=" + this.operand2;
      await axios.get(request).then(response => {
        this.result = response["data"];
        this.display += "=" + this.result;
        this.operator = null;
        this.finished = true;
      })
      // .catch(error =>{
      //   this.failedConnection = true;
      // });
    },

    clear() {
      this.value = "";
      this.display = "";
      this.operand1 = null;
      this.operand2 = null;
      this.operator = null;
      this.result = null;
    }
  },
};
</script>

<style lang="scss" scoped>
$warning: #ffb6b9;
$p-light: #fae3d9;
$secondary: #bbded6;
$s-dark: #8ac6d1;
$dark: #424345;
$gray: #616163;
$white: #fff;
$black: #000;
$light: #d4d4d2;

#calculator {
  width: 100%;
  margin: 0 auto;
  display: flex;
  padding: 0;
  flex-direction: column;
  background-color: $dark;

  .calculator-input {
    color: $light;
    width: 100%;
    min-height: 5.2rem;
    border-radius: 0;
    padding: 0.8rem;
    display: block;
    font-size: 2.4rem;
    background-color: $dark;
    text-align: right;
    font-weight: lighter;
    &:focus,
    &:active {
      outline: none;
    }
    box-shadow: 0 0 0 1px $gray;
  }
    .calculator-warning {
    color: $dark;
    width: 100%;
    min-height: 2.4rem;
    // border: none;
    padding: 0.8rem;
    display: block;
    font-size: 1.2rem;
    background: $warning;
    text-align: center;
    font-weight: bold;
    &:focus,
    &:active {
      outline: none;
    }
  }

  .calculator-row {
    display: flex;
    padding: 0;
    justify-content: space-around;
    .calculator-col {
      flex: 1;
      box-shadow: 0 0 0 1px $gray;
      &.wide {
        flex: 2;
      }
    }
  }

  .calculator-btn {
    width: 100%;
    color: white;
    border: none;
    cursor: pointer;
    padding: 0.8rem;
    outline: none;
    font-size: 1.6rem;
    transition: all 0.3s ease-in-out;
    font-weight: 200;
    justify-content: center;
    background-color: $dark;
    &.arithmetic {
      background-color: $light;
      color: black;
    }
    &.gray {
      background-color: $dark;
    }
    &.action {
      background-color: $secondary;
      color: black;
    }
    &:active {
      background-color: $white;
    }
  }
}
</style>