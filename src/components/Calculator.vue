<template>
  <div>
    <v-toolbar color="primary">
      <v-toolbar-title>Calculator</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-menu offset-y :close-on-content-click="false">
        <template v-slot:activator="{ on }">
          <v-btn icon v-on="on">
            <v-icon>mdi-menu</v-icon>
          </v-btn>
        </template>
        <v-list>
          <v-list-item>
            <v-list-item-title>History</v-list-item-title>
            <v-list-item-icon @click="clearHistory()">
              <v-icon>mdi-history</v-icon>
            </v-list-item-icon>
          </v-list-item>
          <v-list-item v-for="(entry, index) in history" :key="index">
            <v-list-item-title>{{ entry.expression }}</v-list-item-title>
            <v-list-item-subtitle>{{ entry.result }}</v-list-item-subtitle>
          </v-list-item>
        </v-list>
      </v-menu>
    </v-toolbar>
    <v-container fluid>
      <v-row>
        <v-col cols="12" md="8" offset-md="2">
          <v-card class="mb-3">
            <v-card-text>
              <v-text-field
                v-model="result"
                class="text-right"
                readonly
              ></v-text-field>
            </v-card-text>
            <v-divider></v-divider>
            <v-card-text>
              <v-row>
                <v-col
                  v-for="button in buttons"
                  :key="button.label"
                  cols="3"
                  class="mb-2"
                >
                  <v-btn
                    :color="button.color"
                    :dark="button.dark"
                    @click="handleClick(button.value)"
                    :ripple="false"
                  >
                    {{ button.label }}
                  </v-btn>
                </v-col>
              </v-row>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
export default {
  data() {
    return {
      result: "0",
      history: [],
      historyVisible: false,
      buttons: [
        { label: "C", value: "clear", color: "grey", dark: true },
        { label: "CE", value: "clearLast", color: "grey", dark: true },
        { label: "%", value: "%", color: "grey", dark: true },
        { label: "÷", value: "/", color: "orange", dark: false },
        { label: "7", value: "7", color: "grey", dark: true },
        { label: "8", value: "8", color: "grey", dark: true },
        { label: "9", value: "9", color: "grey", dark: true },
        { label: "×", value: "*", color: "orange", dark: false },
        { label: "4", value: "4", color: "grey", dark: true },
        { label: "5", value: "5", color: "grey", dark: true },
        { label: "6", value: "6", color: "grey", dark: true },
        { label: "-", value: "-", color: "orange", dark: false },
        { label: "1", value: "1", color: "grey", dark: true },
        { label: "2", value: "2", color: "grey", dark: true },
        { label: "3", value: "3", color: "grey", dark: true },
        { label: "+", value: "+", color: "orange", dark: false },
        { label: "0", value: "0", color: "grey", dark: true },
        { label: ".", value: ".", color: "grey", dark: true },
        { label: "=", value: "=", color: "orange", dark: false },
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
            this.result = v;
          }

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
      const chars = this.result.split(/[0-9]/g).filter((v) => !!v);

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
            numbers.splice(index, 0, val1 / val2);
            break;

          case "*":
            numbers.splice(index, 0, val1 * val2);
            break;

          case "+":
            numbers.splice(index, 0, val1 + val2);
            break;

          case "-":
            numbers.splice(index, 0, val1 - val2);
            break;
        }
      }

      const history = JSON.parse(localStorage.getItem("history")) || [];
      history.push({
        expression: this.result,
        result: numbers[0],
      });
      localStorage.setItem("history", JSON.stringify(history));
      this.refereshHistory();

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
    refereshHistory() {
      this.history = JSON.parse(localStorage.getItem("history")) || [];
    },
    clearHistory() {
      localStorage.removeItem("history");
      this.refereshHistory();
    },
  },
  mounted() {
    document.addEventListener("keydown", (el) => {
      if (el.key == "Enter") {
        this.calculateValue();
      }
      if (/[-+*/%0-9]/.test(el.key)) {
        this.handleClick(`${el.key}`);
      }
    });
    this.refereshHistory();
  },
};
</script>
