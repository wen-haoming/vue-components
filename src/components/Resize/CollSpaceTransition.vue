<template>
    <transition :name="ns.b()" v-on="on">
        <slot />
    </transition>
</template>
<script lang="ts" setup>

defineOptions({
    name: 'ElCollapseTransition',
})

const reset = (el) => {
    el.style.maxHeight = ''
    el.style.overflow = el.dataset.oldOverflow
    el.style.paddingTop = el.dataset.oldPaddingTop
    el.style.paddingBottom = el.dataset.oldPaddingBottom
}

const on = {
    beforeEnter(el) {
        if (!el.dataset) el.dataset = {}

        el.dataset.oldPaddingTop = el.style.paddingTop
        el.dataset.oldPaddingBottom = el.style.paddingBottom

        el.style.maxHeight = 0
        el.style.paddingTop = 0
        el.style.paddingBottom = 0
    },

    enter(el) {
        el.dataset.oldOverflow = el.style.overflow
        if (el.scrollHeight !== 0) {
            el.style.maxHeight = `${el.scrollHeight}px`
        } else {
            el.style.maxHeight = 0
        }
        el.style.paddingTop = el.dataset.oldPaddingTop
        el.style.paddingBottom = el.dataset.oldPaddingBottom
        el.style.overflow = 'hidden'
    },

    afterEnter(el) {
        el.style.maxHeight = ''
        el.style.overflow = el.dataset.oldOverflow
    },

    enterCancelled(el) {
        reset(el)
    },

    beforeLeave(el) {
        if (!el.dataset) el.dataset = {}
        el.dataset.oldPaddingTop = el.style.paddingTop
        el.dataset.oldPaddingBottom = el.style.paddingBottom
        el.dataset.oldOverflow = el.style.overflow

        el.style.maxHeight = `${el.scrollHeight}px`
        el.style.overflow = 'hidden'
    },

    leave(el) {
        if (el.scrollHeight !== 0) {
            el.style.maxHeight = 0
            el.style.paddingTop = 0
            el.style.paddingBottom = 0
        }
    },

    afterLeave(el) {
        reset(el)
    },

    leaveCancelled(el) {
        reset(el)
    },
}
</script>
<style lang="sass">
// .collapse-transition {
//     transition: getCssVar('transition-duration') height ease-in-out,
//         getCssVar('transition-duration') padding-top ease-in-out,
//         getCssVar('transition-duration') padding-bottom ease-in-out;
// }

// .collapse-transition-leave-active,
// .collapse-transition-enter-active {
//     transition: getCssVar('transition-duration') max-height ease-in-out,
//         getCssVar('transition-duration') padding-top ease-in-out,
//         getCssVar('transition-duration') padding-bottom ease-in-out;
// }
</style>