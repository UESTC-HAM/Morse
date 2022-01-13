<template>
  <v-dialog v-model="dialog" persistent max-width="290">
    <v-card>
      <v-card-title class="text-h5"> 启动音频服务 </v-card-title>
      <v-card-text>
        由于浏览器限制，您可能需要点击某个“允许播放音频”窗口并与以下按钮互动。
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="green darken-1" text @click="init"> 启动音频 </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  data: () => ({
    dialog: true,
  }),
  methods: {
    init: function () {
      this.Bus.$emit("initAudio");
      this.dialog = false;
    },
  },
};
</script>

<style></style>
