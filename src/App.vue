<script setup lang="ts">
import Resize from './components/Resize'
import { ref } from 'vue';

const open = ref(true);
const open2 = ref(true);

</script>

<template>
  <Resize style="height:100%">
      <template #left v-if="open">
        <div class="demo-split-pane" style="background-color: red;">
          111
          <button @click="open2 = !open2">{{ !open2 ? '打开右' : '关闭右' }}</button>
        </div>
      </template>
    <template #right v-if="open2">
      <div class="demo-split-pane" style="background-color: greenyellow;">
        <button @click="open = !open">{{ !open ? '打开左' : '关闭左' }}</button>
        Right Pane
      </div>
    </template>
  </Resize>
</template>

<style scoped>
.marin {
  width: 100vw;
  height: 100vh;
}
.demo-split-pane{
width:100%;
}

</style>
