<template>
  <v-card class="my-2">
    <v-card-title
      >发送面板
      <v-spacer />
      <v-btn-toggle v-model="btnMode" dense>
        <v-btn value="text">
          <v-icon>mdi-text</v-icon>
        </v-btn>

        <v-btn value="3btn">
          <v-icon>mdi-card</v-icon>
        </v-btn>

        <v-btn value="1btn">
          <v-icon>mdi-format-list-group</v-icon>
        </v-btn>
      </v-btn-toggle>
    </v-card-title>
    <v-card-actions v-if="btnMode == 'text'">
      <v-textarea v-model="msg" @keydown.prevent @keydown.exact="keyDown" />
    </v-card-actions>
    <v-card-actions v-if="btnMode == '3btn'" v-hotkey="keymap3btn">
      <v-row no-gutters>
        <v-col cols="12" md="4">
          <v-btn tile block @click="doBeep('.')">Short ( . )</v-btn>
        </v-col>
        <v-col cols="12" md="4">
          <v-btn tile block @click="doBeep('-')">Long ( - )</v-btn>
        </v-col>
        <v-col cols="12" md="4">
          <v-btn tile block @click="doBeep(' ')">Space ( )</v-btn>
        </v-col>
      </v-row>
    </v-card-actions>
    <v-card-actions v-if="btnMode == '1btn'" v-hotkey="keymap1btn">
      <v-btn
        tile
        block
        v-long-press="150"
        @long-press-stop="doBtnBeep('.')"
        @long-press-start="doBtnBeep('-')"
        >Send</v-btn
      >
    </v-card-actions>
  </v-card>
</template>

<script>
import LongPress from "vue-directive-long-press";
import morse_code from "@/assets/morse.json";

export default {
  name: "MorseBtn",
  directives: {
    "long-press": LongPress,
  },
  data: () => ({
    btnMode: "3btn",
    btnState: false,
    awaitNext: false,
    msg: "",
    timeout: null,
    spaceTimer: 0,
  }),
  computed: {
    keymap3btn() {
      return {
        z: this.doBtnBeepShort,
        x: this.doBtnBeepLong,
        space: this.doBtnBeepSpace,
      };
    },
    keymap1btn() {
      return {
        space: { keydown: this.keytimerDown, keyup: this.keytimerUp },
      };
    },
  },
  methods: {
    keytimerDown: function () {
      this.spaceTimer = setTimeout(() => {
        this.doBtnBeep("-");
      }, 150);
    },
    keytimerUp: function () {
      clearTimeout(this.spaceTimer);
      this.doBtnBeep(".");
    },
    doBtnBeepShort: function () {
      this.doBeep(".");
    },
    doBtnBeepLong: function () {
      this.doBeep("-");
    },
    doBtnBeepSpace: function () {
      this.doBeep(" ");
    },
    doBeep: function (type) {
      this.Bus.$emit("keyPress", type);
    },
    keyDown: function (e) {
      let c = e.key.toUpperCase();
      if (c in morse_code || c === " ") {
        this.msg = this.msg + c;
        if (c === " ") {
          this.Bus.$emit("keyPress", " ");
        } else {
          this.Bus.$emit("keyBatchPress", c);
        }
      } else if (c === "BACKSPACE") {
        this.msg = this.msg.slice(0, -1);
      }
    },
    doBtnBeep: function (type) {
      clearTimeout(this.timeout);
      this.awaitNext = false;
      if (type === ".") {
        if (!this.btnState) this.Bus.$emit("keyPress", type);
        this.btnState = false;
      } else {
        this.Bus.$emit("keyPress", type);
        this.btnState = true;
      }
      this.awaitNext = true;
      this.timeout = setTimeout(() => {
        if (this.awaitNext) {
          this.awaitNext = false;
          this.Bus.$emit("keyPress", " ");
        }
      }, 500);
    },
  },
};
</script>
