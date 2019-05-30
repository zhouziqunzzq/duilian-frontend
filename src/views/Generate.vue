<template>
  <v-container>
    <v-tabs
        color="red darken-4"
        dark
        slider-color="cyan"
        grow
    >
      <v-tab ripple>对对联</v-tab>
      <v-tab-item>
        <DuilianGenerator/>
      </v-tab-item>

      <v-tab ripple>关键字生成</v-tab>
      <v-tab-item>
        <KeywordGenerator/>
      </v-tab-item>

    </v-tabs>
  </v-container>
</template>

<script lang="ts">
  import Vue from "vue";
  import DuilianGenerator from "@/components/DuilianGenerator.vue";
  import KeywordGenerator from "@/components/KeywordGenerator.vue";

  export default Vue.extend({
    name: "translate",
    components: {
      DuilianGenerator,
      KeywordGenerator,
    },
    data: () => ({
      //
    }),
    computed: {
      //
    },
    methods: {
      //
      showInfo(msg: string) {
        this.$store.dispatch("showInfo", msg);
      },
      showError(msg: string) {
        this.$store.dispatch("showError", msg);
      },
    },
  });
</script>

<style scoped lang="stylus">

</style>
