<script setup lang="ts">
import { computed, ref, watch, provide } from 'vue'
const props = defineProps({
  horizontal: { type: Boolean }
});
const slots = defineSlots();
const slotsList = computed(() => slots.default()); //必须添加响应式
const panesLength = computed(() => slotsList.length);

const panes = ref([]) // 承载所有 pane 的所有状态
const container = ref()
// 如果 slot 变化了，需要重新设置panes 的状态
watch(() => slotsList.value, () => {
  panes.value = slotsList.value.map((slot, index) => {
    const { size, maxSize, minSize } = slot.props;
    return {
      slot,
      index,
      data: { size, maxSize, minSize }
    }
  })
}, {
  immediate: true,
})

const bindEvents = (panel, activeSplitter) => {

  const sumPrevPanesSize = (splitterIndex) => {
    return panes.value.reduce((total, pane, i) => total + (i < splitterIndex ? pane.data.size : 0), 0)
  }

  const sumNextPanesSize = (splitterIndex) => {
    return panes.value.reduce((total, pane, i) => total + (i > splitterIndex + 1 ? pane.data.size : 0), 0)
  }

  const getCurrentDragPercentage = (drag) => {
    drag = drag[props.horizontal ? 'y' : 'x']
    // In the code bellow 'size' refers to 'width' for vertical and 'height' for horizontal layout.
    const containerSize = container.value[props.horizontal ? 'clientHeight' : 'clientWidth']
    return drag * 100 / containerSize
  }

  const getCurrentMouseDrag = (event) => {
    const rect = container.value.getBoundingClientRect()
    const { clientX, clientY } = ('ontouchstart' in window && event.touches) ? event.touches[0] : event

    return {
      x: clientX - rect.left,
      y: clientY - rect.top
    }
  }


  const calculatePanesSize = (drag) => {
    const splitterIndex = activeSplitter;
    let sums = {
      prevPanesSize: sumPrevPanesSize(splitterIndex),
      nextPanesSize: sumNextPanesSize(splitterIndex),
      prevReachedMinPanes: 0,
      nextReachedMinPanes: 0
    }
    const minDrag = 0 + sums.prevPanesSize
    const maxDrag = 100 - sums.nextPanesSize
    const dragPercentage = Math.max(Math.min(getCurrentDragPercentage(drag), maxDrag), minDrag)
    console.log(dragPercentage,'dragPercentage')
    let paneBefore = panes.value[activeSplitter].data || null
    let paneAfter = panes.value[activeSplitter + 1].data || null

    // const paneBeforeMaxReached = paneBefore.max < 100 && (dragPercentage >= (paneBefore.max + sums.prevPanesSize))
    // const paneAfterMaxReached = paneAfter.max < 100 && (dragPercentage <= 100 - (paneAfter.max + this.sumNextPanesSize(splitterIndex + 1)))
    // // Prevent dragging beyond pane max.
    // if (paneBeforeMaxReached || paneAfterMaxReached) {
    //   if (paneBeforeMaxReached) {
    //     paneBefore.size = paneBefore.max
    //     paneAfter.size = Math.max(100 - paneBefore.max - sums.prevPanesSize - sums.nextPanesSize, 0)
    //   }
    //   else {
    //     paneBefore.size = Math.max(100 - paneAfter.max - sums.prevPanesSize - this.sumNextPanesSize(splitterIndex + 1), 0)
    //     paneAfter.size = paneAfter.max
    //   }
    //   return
    // }

    // // When pushOtherPanes = true, find the closest expanded pane on each side of the splitter.
    // if (this.pushOtherPanes) {
    //   const vars = this.doPushOtherPanes(sums, dragPercentage)
    //   if (!vars) return // Prevent other calculation.

    //   ({ sums, panesToResize } = vars)
    //   paneBefore = this.panes[panesToResize[0]] || null
    //   paneAfter = this.panes[panesToResize[1]] || null
    // }
    if (paneBefore !== null) {
      paneBefore.size = Math.min(Math.max(dragPercentage - sums.prevPanesSize - sums.prevReachedMinPanes, paneBefore.min), paneBefore.max)
    }
    if (paneAfter !== null) {
      paneAfter.size = Math.min(Math.max(100 - dragPercentage - sums.nextPanesSize - sums.nextReachedMinPanes, paneAfter.min), paneAfter.max)
    }
    console.log(paneBefore,paneAfter)
  }


  const onMouseMove = (event) => {
    event.preventDefault();
    calculatePanesSize(getCurrentMouseDrag(event))
  }

  const onMouseUp = () => {
    document.removeEventListener('mousemove', onMouseMove)
    document.removeEventListener('mouseup', onMouseUp)
  }

  document.addEventListener('mousemove', onMouseMove, { passive: false })
  document.addEventListener('mouseup', onMouseUp)
}

</script>
<template>
  <div :class="['splitpanes', props.horizontal ? 'splitpanes--horizontal' : 'splitpanes--vertical']" ref="container">
    <template v-for="panel in panes">
      <component :is="panel.slot" :size="panel.data.size"
        :max-size="panel.data.maxSize" :min-size="panel.data.minSize" />
      <div @mousedown="() => bindEvents(panel, panel.index)" class="splitpanes__splitter"
        v-if="panesLength - 1 !== panel.index" />
    </template>
  </div>
</template>

<style lang="less">
.splitpanes {
  display: flex;
  width: 100%;
  height: 100%;

  &--vertical {
    flex-direction: row;
  }

  &--horizontal {
    flex-direction: column;
  }

  &--dragging * {
    user-select: none;
  }

  &__pane {
    width: 100%;
    height: 100%;
    overflow: hidden;

    .splitpanes--vertical & {
      transition: width 0.2s ease-out;
    }

    .splitpanes--horizontal & {
      transition: height 0.2s ease-out;
    }

    .splitpanes--dragging & {
      transition: none;
    }
  }

  // Disable default zoom behavior on touch device when double tapping splitter.
  &__splitter {
    touch-action: none;
  }

  &--vertical>.splitpanes__splitter {
    min-width: 1px;
    cursor: col-resize;
  }

  &--horizontal>.splitpanes__splitter {
    min-height: 1px;
    cursor: row-resize;
  }
}

.splitpanes.default-theme {
  .splitpanes__pane {
    background-color: #f2f2f2;
  }

  .splitpanes__splitter {
    background-color: #fff;
    box-sizing: border-box;
    position: relative;
    flex-shrink: 0;

    &:before,
    &:after {
      content: "";
      position: absolute;
      top: 50%;
      left: 50%;
      background-color: rgba(0, 0, 0, .15);
      transition: background-color 0.3s;
    }

    &:hover:before,
    &:hover:after {
      background-color: rgba(0, 0, 0, .25);
    }

    &:first-child {
      cursor: auto;
    }
  }
}

.default-theme {
  &.splitpanes .splitpanes .splitpanes__splitter {
    z-index: 1;
  }

  &.splitpanes--vertical>.splitpanes__splitter,
  .splitpanes--vertical>.splitpanes__splitter {
    width: 7px;
    border-left: 1px solid #eee;
    margin-left: -1px;

    &:before,
    &:after {
      transform: translateY(-50%);
      width: 1px;
      height: 30px;
    }

    &:before {
      margin-left: -2px;
    }

    &:after {
      margin-left: 1px;
    }
  }

  &.splitpanes--horizontal>.splitpanes__splitter,
  .splitpanes--horizontal>.splitpanes__splitter {
    height: 7px;
    border-top: 1px solid #eee;
    margin-top: -1px;

    &:before,
    &:after {
      transform: translateX(-50%);
      width: 30px;
      height: 1px;
    }

    &:before {
      margin-top: -2px;
    }

    &:after {
      margin-top: 1px;
    }
  }
}
</style>