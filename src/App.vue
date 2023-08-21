<script setup lang="ts">
import { Split, SplitPanel } from './components/splitpanes'
import { ref } from 'vue';

const show = ref(true);

</script>
<template>
  <Split style="height: 300px" class="default-theme">
    <SplitPanel size="80">1</SplitPanel>
    <SplitPanel size="20">
      <button @click="() => show = !show">show{{ show ? 'show' : 'no-show' }}</button>
    </SplitPanel>
  </Split>
</template>

<style>
.splitpanes__pane {
  justify-content: center;
  align-items: center;
  display: flex;
  position: relative;
}

em.specs {
  font-size: 0.2em;
  line-height: 1;
  position: absolute;
  bottom: 0.5em;
  left: 0;
  right: 0;
  text-align: center;
}


html,
body,
#app {
  height: 100%;
  margin: 0;
}

body {
  font-family: Helvetica, Arial, sans-serif;
  font-size: 5em;
}

p {
  position: absolute;
  bottom: 5px;
  right: 5px;
  z-index: 10;
  font-size: 12px;

  a {}
}
</style>
