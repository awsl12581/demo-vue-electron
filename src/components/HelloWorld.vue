<script setup lang="ts">
import { ref } from 'vue';
import { Icon } from '@iconify/vue';
defineProps<{ msg: string }>();
const icons = ref('solar:accumulator-outline');
const changeIcon = () => {
  icons.value = 'solar:accessibility-bold';
};
const count = ref(0);
</script>

<template>
  <h1 class="bg-slate-400">{{ msg }}</h1>

  <div class="card">
    <ElButton
      @click="
        () => {
          count++;
          changeIcon();
        }
      "
    >
      <Icon :icon="icons"></Icon>
      count is {{ count }}
    </ElButton>
    <p>
      Edit
      <code>components/HelloWorld.vue</code> to test HMR
    </p>
  </div>

  <p>
    Check out
    <a href="https://vuejs.org/guide/quick-start.html#local" target="_blank">create-vue</a>, the
    official Vue + Vite starter
  </p>
  <p>
    Install
    <a href="https://github.com/johnsoncodehk/volar" target="_blank">Volar</a>
    in your IDE for a better DX
  </p>
  <p class="read-the-docs">Click on the Vite and Vue logos to learn more</p>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
