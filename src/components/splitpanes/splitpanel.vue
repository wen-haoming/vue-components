<script setup lang="ts">
// import { watch, toRefs } from 'vue';

const props = defineProps({
  size: { type: [Number, String], default: null },
  minSize: { type: [Number, String], default: 0 },
  maxSize: { type: [Number, String], default: 100 },
  horizontal: { type: Boolean },
})

// const emit = defineEmits(['change'])

// // watch(() => [props.size, props.minSize, props.maxSize], () => {
// //   emit('change', props)
// // })

</script>
<template>
  <div class="splitpanes__pane" :style="!props.horizontal ? `width:${props.size}% `:`height:${props.size}% `" >
    <slot />
  </div>
</template>

  