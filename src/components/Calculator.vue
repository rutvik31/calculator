<template>
  <div>
    <v-row class="ma-0 overflow">
      <v-col v-if="lastExp" cols="12" class="text-right p-1">
        <span class="caption">
          {{ lastExp }}
        </span>
      </v-col>
      <v-col cols="12" class="text-right pt-1">
        <span class="font-style">
          {{ result }}
        </span>
      </v-col>
    </v-row>
    <v-row id="btns" class="ma-0">
      <v-col
        v-for="button in buttons"
        :key="button.label"
        cols="3"
        class="pa-1"
      >
        <v-card
          elevation="0"
          outlined
          tile
          :color="button.color"
          @click="handleClick(button.value)"
        >
          <v-card-text class="text-center">
            <span
              class="cal-font black--text"
              :class="{ 'white--text': button.label == '=' }"
            >
              {{ button.label }}
            </span>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </div>
</template>

<script>
export default {
  data() {
    return {
      lastExp: "",
      result: "0",
      history: [],
      panel: false,
      historyVisible: false,
      hasDecimal: false,
      buttons: [
        { label: "C", value: "clear", dark: true },
        { label: "CE", value: "clearLast", dark: true },
        { label: "%", value: "%", dark: true },
        { label: "÷", value: "/", dark: false },
        { label: "7", value: "7", dark: true },
        { label: "8", value: "8", dark: true },
        { label: "9", value: "9", dark: true },
        { label: "×", value: "*", dark: false },
        { label: "4", value: "4", dark: true },
        { label: "5", value: "5", dark: true },
        { label: "6", value: "6", dark: true },
        { label: "-", value: "-", dark: false },
        { label: "1", value: "1", dark: true },
        { label: "2", value: "2", dark: true },
        { label: "3", value: "3", dark: true },
        { label: "+", value: "+", dark: false },
        { label: "0", value: "0", dark: true },
        { label: ".", value: ".", dark: true },
        { label: "Del", value: "delete", dark: true },
        { label: "=", value: "=", color: "primary", dark: false },
      ],
      chars: ["+", "-", "*", "/"],
    };
  },
  methods: {
    handleClick(value) {
      switch (value) {
        case "clear":
          this.result = "0";
          break;
        case "clearLast":
          while (true) {
            if (!this.result) {
              this.result = "0";
              break;
            }

            const char = this.result[this.result.length - 1];
            if (this.chars.indexOf(char) > -1) break;
            this.result = this.result.substring(0, this.result.length - 1);
          }
          break;
        case "%":
          const numbers = this.result.split(/[-+*/]/g).filter((v) => !!v);
          const chars = this.result.split(/[0-9]/g).filter((v) => !!v);

          if (numbers.length == chars.length) chars.pop();

          let number;
          let char;

          if (chars.length > 1) {
            number = numbers.splice(numbers.length, 1)[0];
            char = numbers.splice(chars.length, 1)[0];
          }

          this.calculateValue();
          const v = this.result / 100;
          if (number && v) {
            v = v * number;
          }

          if (char) {
            this.result = `${this.number}${char}${v}`;
            this.calculateValue();
          } else {
            this.lastExp = `${this.result}/100`;
            this.result = v;
          }

          break;

        case "delete":
          this.deleteInput();
          break;

        case "-":
          this.checkChar("-");
          break;

        case "+":
          this.checkChar("+");
          break;

        case "*":
          this.checkChar("*");
          break;

        case "/":
          this.checkChar("/");
          break;

        case "=":
          this.calculateValue();
          break;

        default:
          if (this.result === "0" || isNaN(parseInt(this.result))) {
            this.result = value;
          } else {
            this.result += value;
          }
      }
    },
    calculateValue() {
      const numbers = this.result.split(/[-+*/]/g).filter((v) => !!v);
      const chars = this.result.split(/[0-9]/g).filter((v) => !!v && v != ".");

      if (numbers.length == chars.length) chars.pop();
      while (true) {
        if (numbers.length == 1) break;

        let index = chars.indexOf("/");
        if (index == -1) index = chars.indexOf("*");
        if (index == -1) index = chars.indexOf("+");
        if (index == -1) index = chars.indexOf("-");

        const char = chars.splice(index, 1)[0];

        const val1 = parseFloat(numbers.splice(index, 1)[0]);
        const val2 = parseFloat(numbers.splice(index, 1)[0]);

        switch (char) {
          case "/":
            numbers.splice(index, 0, `${val1 / val2}`);
            break;

          case "*":
            numbers.splice(index, 0, `${val1 * val2}`);
            break;

          case "+":
            numbers.splice(index, 0, `${val1 + val2}`);
            break;

          case "-":
            numbers.splice(index, 0, `${val1 - val2}`);
            break;

          default:
            break;
        }
      }

      const history = JSON.parse(localStorage.getItem("history")) || [];
      history.push({
        expression: this.result,
        result: numbers[0],
      });
      localStorage.setItem("history", JSON.stringify(history));
      this.$emit("refereshHistory");

      this.lastExp = this.result;
      this.result = numbers[0];
    },
    checkChar(sChar) {
      if (this.result == "0") return;

      const char = this.result[this.result.length - 1];

      if (this.chars.indexOf(char) > -1) {
        this.result = this.result.substring(0, this.result.length - 1);
      }
      this.result += sChar;
    },
    deleteInput() {
      if (this.result == "0") return;
      this.result = this.result.substring(0, this.result.length - 1);
      if (!this.result) {
        this.result = "0";
        return;
      }
    },
  },
  mounted() {
    document.addEventListener("keydown", (el) => {
      if (el.key == "Enter") {
        this.calculateValue();
      }
      if (/^(?!f)[-+*/%.0-9]$/.test(el.key)) {
        this.handleClick(`${el.key}`);
      }
      if (el.key == "Backspace") {
        this.deleteInput();
      }
    });
  },
};
</script>
<style>
.font-style {
  font-size: calc(50px + 2vw);
  font-family: "Roboto" !important;
}
.cal-font {
  font-size: 20px !important;
  font-family: "Roboto" !important;
}
.overflow {
  word-break: break-all;
  max-height: calc(100vh - 355px);
  overflow-y: auto;
}
.backgroud {
  background-color: whitesmoke !important;
}
</style>
