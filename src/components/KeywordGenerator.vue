<template>
  <v-container>
    <v-layout row wrap>
      <v-flex xs12>
        <v-text-field
            v-model="keyword"
            :error-messages="keywordErrors"
            label="请输入关键字（2 - 4 字）"
            required
            @input="$v.keyword.$touch()"
            @blur="$v.keyword.$touch()"
        ></v-text-field>
      </v-flex>

      <v-flex xs12>
        <v-text-field
            v-model="firstLine"
            label="上联"
            readonly
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
        <v-btn @click="generateKeyword" color="success">生成</v-btn>
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
    name: "KeywordGenerator",
    mixins: [validationMixin],
    validations: {
      keyword: {required, minLength: minLength(2), maxLength: maxLength(4)},
    },
    data: () => ({
      keyword: "",
      firstLine: "",
      secondLine: "",
      inscription: "",
    }),
    computed: {
      keywordErrors() {
        const errors: any[] = [];
        if (!this.$v.keyword!.$dirty) {
          return errors;
        }
        !this.$v.keyword!.required && errors.push("请输入关键字（2 - 4 字）");
        !this.$v.keyword!.minLength && errors.push("关键字不能少于两个字");
        !this.$v.keyword!.maxLength && errors.push("关键字不能多于四个字");
        return errors;
      },
    },
    methods: {
      async generateKeyword() {
        this.$v.$touch();
        if (this.$v.$invalid) {
          return;
        } else {
          this.firstLine = "";
          this.secondLine = "";
          this.inscription = "";
          try {
            const response = await post<{
              result: boolean,
              code: number,
              msg: string,
              data: {
                shanglian: string,
                xialian: string,
                hengpi: string,
              },
            }>(config.baseUrl + "keyword",
              {
                input: this.firstLine,
              });
            if (response.parsedBody!.result) {
              this.showInfo(response.parsedBody!.msg);
              this.firstLine = response.parsedBody.data.shanglian;
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
