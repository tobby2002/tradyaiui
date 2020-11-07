<template>
  <q-page padding>
<!--    <h4 class="q-ma-xs">Start a new gekko</h4>-->
    <gekko-config-builder v-on:config="updateConfig" />
    <div v-if="config.valid" class="row justify-center q-mt-md">
      <q-btn outline  color="teal" @click.prevent="start" :disabled="pendingStratrunner" :loading="pendingStratrunner && pendingStratrunner !== ''">start trade</q-btn>
    </div>
  </q-page>
</template>

<script>
import gekkoConfigBuilder from "./gekkoConfigBuilder.vue";
import liveMixin from '../mixins/LiveGekkoSharedMixin'

export default {
  mixins:[liveMixin],
  components: {
    gekkoConfigBuilder
  },
  data: () => {
    return {
      pendingStratrunner: false,
      config: {}
    };
  }
};
</script>
