<template>
  <v-card class="my-2">
    <v-expansion-panels>
      <v-expansion-panel>
        <v-expansion-panel-header>
          <v-card-title>嘤频配置</v-card-title>
        </v-expansion-panel-header>
        <v-expansion-panel-content>
          <v-card-actions>
            <v-row>
              <v-col cols="12" md="6">
                <v-slider
                  v-model="freq"
                  label="Frequency (Hz)"
                  max="2500"
                  min="800"
                  thumb-label
                ></v-slider>
              </v-col>
              <v-col cols="12" md="6">
                <v-range-slider
                  v-model="holdtime"
                  label="Time (ms)"
                  max="600"
                  min="100"
                  thumb-label
                />
              </v-col>
              <v-col cols="12">
                <v-btn tile block @click="test">测试 (CQCQ)</v-btn>
              </v-col>
            </v-row>
          </v-card-actions>
        </v-expansion-panel-content>
      </v-expansion-panel>
    </v-expansion-panels>
  </v-card>
</template>

<script>
import morse_code from "@/assets/morse.json";

export default {
  name: "MorseExec",
  data: () => ({
    audioCtx: null,
    oscillator: null,
    gainNode: null,
    queue: [],
    freq: 1500,
    holdtime: [100, 500],
    sound: false,
  }),
  watch: {
    freq: (freq) => {
      localStorage.setItem("freq", freq);
    },
    holdtime: (holdtime) => {
      localStorage.setItem("holdtimeLower", holdtime[0]);
      localStorage.setItem("holdtimeUpper", holdtime[1]);
    },
  },
  methods: {
    test: function () {
      const cq = "-.-. --.- -.-. --.- ";
      for (var i = 0; i < cq.length; i++) {
        this.Bus.$emit("keyPress", cq[i]);
      }
    },
    beepLoop: function () {
      if (!this.sound) {
        if (this.queue.length != 0) {
          this.sound = true;
          let keyPressed = this.queue.shift();
          this.Bus.$emit("soundOut", keyPressed);
          let length = this.holdtime[0];
          switch (keyPressed) {
            case ".": {
              length = this.holdtime[0];
              break;
            }
            case "-": {
              length = this.holdtime[1];
              break;
            }
            case " ": {
              length = 0;
              break;
            }
            default:
              break;
          }
          console.log(`Beep [${keyPressed}]!`);
          if (length !== 0) {
            this.beep(length, this.freq);
            setTimeout(() => {
              this.sound = false;
            }, length + 100);
          } else {
            setTimeout(() => {
              this.sound = false;
            }, length + this.holdtime[0] + 100);
          }
        }
      }
    },
    init: function () {
      this.audioCtx = new (window.AudioContext ||
        window.webkitAudioContext ||
        window.audioContext)();
    },
    beep: function (duration, frequency, volume, type, callback) {
      this.oscillator = this.audioCtx.createOscillator();
      this.gainNode = this.audioCtx.createGain();
      this.oscillator.connect(this.gainNode);
      this.gainNode.connect(this.audioCtx.destination);

      if (volume) {
        this.gainNode.gain.value = volume;
      }
      if (frequency) {
        this.oscillator.frequency.value = frequency;
      }
      if (type) {
        this.oscillator.type = type;
      }
      if (callback) {
        this.oscillator.onended = callback;
      }

      this.oscillator.start(this.audioCtx.currentTime);
      this.oscillator.stop(
        this.audioCtx.currentTime + (duration || 500) / 1000
      );
    },
  },
  mounted: function () {
    this.freq = localStorage.getItem("freq");
    this.holdtime = [
      parseInt(localStorage.getItem("holdtimeLower")),
      parseInt(localStorage.getItem("holdtimeUpper")),
    ];

    this.Bus.$on("initAudio", () => {
      this.init();
    });

    this.Bus.$on("keyPress", (key) => {
      this.queue.push(key);
    });

    this.Bus.$on("keyBatchPress", (char) => {
      let keys = morse_code[char];
      for (var i = 0; i < keys.length; i++) {
        this.queue.push(keys[i]);
      }
      this.queue.push(" ");
    });

    setInterval(this.beepLoop, 10);
  },
};
</script>

<style scoped>
.v-expansion-panel-header {
  padding: unset;
}

.v-expansion-panels {
  z-index: unset;
}
</style>
