<template>
  <div id="app">
    <div class="row">
      <div class="col-3 offset-3 q-mt-lg">
        <vue3-q-tel-input v-model:tel="input" searchText="Search using code/country" dense filled default-country="ar" />
      </div>
    </div>
    <div>entered telephone number : {{ input.iso }}</div>
    <div>entered telephone number : {{ input.valor }}</div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';
import Vue3QTelInput from '@/component/Index.vue';

export default defineComponent({
  name: 'ServeDev',
  components: {
    Vue3QTelInput,
  },
  setup() {
    const input= ref({
      iso: "",
      valor: ""
    })

    return {
      input
    }
  }
});
</script>
