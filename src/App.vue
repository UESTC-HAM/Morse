<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <v-toolbar-title>Morse</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn icon @click="toggleTheme"
        ><v-icon>mdi-theme-light-dark</v-icon></v-btn
      >
    </v-app-bar>

    <v-main>
      <v-container>
        <StartMorse />
        <EchoOutput />
        <MorseBtn />
        <MorseExec />
      </v-container>
    </v-main>

    <v-footer padless>
      <v-col class="text-center" cols="12">
        {{ new Date().getFullYear() }} — <strong>UESTC ARA</strong>
      </v-col>
    </v-footer>
  </v-app>
</template>

<script>
import StartMorse from "./components/StartMorse";
import MorseExec from "./components/MorseExec";
import MorseBtn from "./components/MorseBtn";
import EchoOutput from "./components/EchoOutput";

export default {
  name: "App",

  components: {
    StartMorse,
    MorseExec,
    MorseBtn,
    EchoOutput,
  },
  methods: {
    toggleTheme: function () {
      const dark = localStorage.dark;
      if (dark === "true") {
        this.$vuetify.theme.dark = false;
        localStorage.setItem("dark", false);
      } else {
        this.$vuetify.theme.dark = true;
        localStorage.setItem("dark", true);
      }
    },
  },
  mounted() {
    if (localStorage.colortheme) {
      this.$vuetify.theme.themes.light.primary = localStorage.colortheme;
      this.$vuetify.theme.themes.dark.primary = localStorage.colortheme;
    }
    const dark = localStorage.dark;
    if (dark === "true") {
      this.$vuetify.theme.dark = true;
    } else if (dark === "false") {
      this.$vuetify.theme.dark = false;
    } else {
      if (
        window.matchMedia &&
        window.matchMedia("(prefers-color-scheme: dark)").matches
      ) {
        this.$vuetify.theme.dark = true;
      }
    }
  },
};
</script>
