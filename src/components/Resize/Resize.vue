<script setup lang="ts">
import { ref, computed } from 'vue';

const isDragging = ref(false);
const startX = ref(0);
const leftPanelWidth = ref(300); // 初始左侧面板宽度

const slots = defineSlots();


const startDrag = (event) => {
    isDragging.value = true;
    startX.value = event.clientX;
    document.addEventListener('mousemove', drag);
    document.addEventListener('mouseup', stopDrag);
};

const drag = (event) => {
    if (!isDragging.value) return;
    const offsetX = event.clientX - startX.value;
    leftPanelWidth.value += offsetX;
    startX.value = event.clientX;
};

const stopDrag = () => {
    isDragging.value = false;
    document.removeEventListener('mousemove', drag);
    document.removeEventListener('mouseup', stopDrag);
};

const hasLeftSlot = computed(() => !!slots.left)
const hasRightSlot = computed(() => !!slots.right)
</script>

<template>
    <div class="split-container">
        <!-- 如果右边消失，则左边要 flex 1 来自适应 -->
        <div v-if="!!slots.left" class="left-panel"
            :style="{ width: leftPanelWidth + 'px', flex: !slots.right ? `1` : undefined }">
            <!-- 左侧区域内容 -->
            <slot name="left" />
        </div>
        <div v-if="hasLeftSlot && hasRightSlot" class="splitter" @mousedown="startDrag"></div>
        <div v-if="!!slots.right" class="right-panel">
            <!-- 右侧区域内容 -->
            <slot name="right" />
        </div>
    </div>
</template>
<style scoped>
.split-container {
    display: flex;
    width: 100%;
    height: 100%;
}

.left-panel {
    flex: 0 0 auto;
    will-change: width flex;
}

.splitter {
    flex: 0 0 5px;
    background-color: #ccc;
    cursor: ew-resize;
}

.right-panel {
    flex: 1 1 auto;
    will-change: width flex;
}
</style>