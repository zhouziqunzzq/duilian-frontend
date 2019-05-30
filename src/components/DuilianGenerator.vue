<template>
  <v-container>
    <v-layout row wrap>
      <v-flex xs12>
        <v-text-field
            v-model="firstLine"
            :error-messages="firstLineErrors"
            label="请输入上联"
            required
            @input="$v.firstLine.$touch()"
            @blur="$v.firstLine.$touch()"
        ></v-text-field>
      </v-flex>

      <v-flex xs12>
        <v-text-field
            v-model="secondLine"
            label="下联"
            readonly
        ></v-text-field>
      </v-flex>

      <v-flex xs12>
        <v-text-field
            v-model="inscription"
            label="横批"
            readonly
        ></v-text-field>
      </v-flex>

      <v-flex xs12>
        <v-btn @click="generatePair" color="success">生成</v-btn>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script lang="ts">
  import Vue from "vue";
  import {validationMixin} from "vuelidate";
  import {required, maxLength, minLength, sameAs} from "vuelidate/lib/validators";
  import config from "@/config.ts";
  import {get, post} from "@/helpers.ts";

  export default Vue.extend({
    name: "DuilianGenerator",
    mixins: [validationMixin],
    validations: {
      firstLine: {required},
    },
    data: () => ({
      firstLine: "",
      secondLine: "",
      inscription: "",
    }),
    computed: {
      firstLineErrors() {
        const errors: any[] = [];
        if (!this.$v.firstLine!.$dirty) {
          return errors;
        }
        !this.$v.firstLine!.required && errors.push("请输入上联");
        return errors;
      },
    },
    methods: {
      async generatePair() {
        this.$v.$touch();
        if (this.$v.firstLine!.$invalid) {
          return;
        } else {
          this.secondLine = "";
          this.inscription = "";
          try {
            const response = await post<{
              result: boolean,
              code: number,
              msg: string,
              data: {
                xialian: string,
                hengpi: string,
              },
            }>(config.baseUrl + "pair",
              {
                input: this.firstLine,
              });
            if (response.parsedBody!.result) {
              this.showInfo(response.parsedBody!.msg);
              this.secondLine = response.parsedBody.data.xialian;
              this.inscription = response.parsedBody.data.hengpi;
            } else {
              this.showError(response.parsedBody!.msg);
            }
          } catch (response) {
            this.showError("出错啦！详细信息：" + response);
          }
        }
      },
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
  .fade-enter-active, .fade-leave-active
    transition: opacity .5s

  .fade-enter, .fade-leave-to
    opacity: 0
</style>
