<template>
  <v-card>
    <v-card-title>
      输出结果
      <v-icon class="ms-4" @click="clearData">mdi-delete</v-icon>
      <v-icon class="ms-4" @click="stopAudio">mdi-stop</v-icon>
      <v-icon class="ms-4" @click="repeatData">mdi-restart</v-icon>
    </v-card-title>
    <v-card-text class="morse">
      <ruby v-for="(txt, idx) in translated" :key="idx">
        {{ txt.text }}<rt>{{ txt.code }}</rt>
      </ruby>
      <ruby v-if="buffer.length !== 0">
        #<rt>{{ buffer.join("") }}</rt>
      </ruby>
    </v-card-text>
  </v-card>
</template>

<script>
import morse_code from "@/assets/morse.json";

export default {
  data: () => ({
    buffer: [],
    translated: [],
  }),
  methods: {
    clearData: function () {
      this.buffer = this.buffer.slice(0, 0);
      this.translated = this.translated.slice(0, 0);
    },
    repeatData: function () {
      const tmp = this.translated
      this.clearData()
      tmp.forEach(c => {
        this.Bus.$emit("keyBatchPress", c.text);
      });
    },
    stopAudio: function () {
      this.Bus.$emit("stopAudio");
    },
    translateBuffer: function () {
      let joined = this.buffer.join("");
      let txt =
        Object.keys(morse_code).find((k) => morse_code[k] === joined) || "?";
      this.translated.push({
        text: txt,
        code: joined,
      });
    },
  },
  mounted: function () {
    this.Bus.$on("keyPress", (k) => {
      if (k !== " ") this.buffer.push(k);
      if (this.buffer.length == 0 && k === " ") {
        this.translated.push({
          text: "_",
          code: "",
        });
        this.buffer = this.buffer.slice(0, 0);
      } else if (this.buffer.length >= 6 || k === " ") {
        this.translateBuffer();
        this.buffer = this.buffer.slice(0, 0);
      }
    });
    this.Bus.$on("keyBatchPress", (char) => {
      this.translated.push({
        text: char,
        code: morse_code[char],
      });
    });
  },
};
</script>

<style scoped>
.morse {
  font-size: 20px;
}
ruby {
  margin-inline: 4px;
}
</style>
