<!--
 * @Author: ZRMYDYCG
 * @Date: 2024-11
 * @LastEditors: ZRMYDYCG
 * @LastEditTime: 2024-11
 * @Description: 
-->
<script setup lang="ts">
import type { CSSProperties } from 'vue'
import type { IAffixProps } from './affix'
import { ref, watchEffect, watch, onMounted, computed } from 'vue'
import { useElementBounding, useWindowSize, useEventListener } from '@vueuse/core'

const props = defineProps<IAffixProps>()
const emits = defineEmits(['change', 'scroll'])

const target = ref<HTMLElement>()
const root = ref<HTMLDivElement>()
const scrollContainer = ref<HTMLElement | Window>() // 滚动目标容器
const rootStyle = computed<CSSProperties>(() => {
  return {
    height: fixed.value ? `${rootHeight.value}px` : '',
    width: fixed.value ? `${rootWidth.value}px` : ''
  }
})
const targetStyle = computed<CSSProperties>(() => {
  if (!fixed.value) return {}

  const { offset } = props

  return {
    height: `${rootHeight.value}px`,
    width: `${rootWidth.value}px`,
    top: props.position === 'top' ? offset + 'px' : '',
    bottom: props.position === 'bottom' ? offset + 'px' : '',
    transform: transform.value ? `translateY(${transform.value}px)` : '',
    zIndex: props.zIndex
  }
})

const fixed = ref(false)
const scrollTop = ref(0)
const transform = ref(0)

const {
  height: rootHeight,
  width: rootWidth,
  top: rootTop,
  bottom: rootBottom,
  // update: 用于更新元素的边界信息
  update: updateRoot
} = useElementBounding(root, { windowScroll: false })
const { height: windowHeight } = useWindowSize()
useEventListener(scrollContainer, 'scroll', () => {
  updateRoot()
  emits('scroll', {
    scrollTop: scrollTop.value,
    fixed: fixed.value
  })
})

const update = () => {
  if (!scrollContainer.value) return
  scrollTop.value =
    scrollContainer.value instanceof Window
      ? document.documentElement.scrollTop
      : scrollContainer.value.scrollTop || 0
  const { position, offset = 0 } = props

  if (position === 'top') {
    if (offset > rootTop.value) {
      fixed.value = true
    }
  } else if (position === 'bottom') {
    fixed.value = windowHeight.value - offset < rootBottom.value
  }
}

watch(fixed, (value) => {
  emits('change', value)
})
watchEffect(update)

onMounted(() => {
  scrollContainer.value = window
  target.value = document.documentElement
  updateRoot()
})
</script>

<template>
  <div ref="root" class="yq-affix" :style="rootStyle">
    <div :class="['yq-affix-target', { 'yq-affix--fixed': fixed }]" :style="targetStyle">
      <slot></slot>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.yq-affix--fixed {
  position: fixed;
}
</style>
