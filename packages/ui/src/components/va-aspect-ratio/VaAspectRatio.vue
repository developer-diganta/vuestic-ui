<template>
  <div class="va-aspect-ratio">
    <div v-if="stylesComputed" :style="stylesComputed" />
    <slot />
  </div>
</template>

<script lang="ts" setup>
import { computed, type PropType } from 'vue'

import { useComponentPresetProp } from '../../composables'

defineOptions({
  name: 'VaAspectRatio',
})

const props = defineProps({
  ...useComponentPresetProp,
  ratio: {
    type: [Number, String] as PropType<number | 'auto'>,
    default: 'auto',
    validator: (v: number | 'auto') => {
      if (typeof v === 'number') {
        return v > 0
      }

      return v === 'auto'
    },
  },
  contentHeight: { type: Number, default: 1 },
  contentWidth: { type: Number, default: 1 },
  maxWidth: {
    type: Number,
    default: 0,
    validator: (v: number) => v >= 0,
  },
})

const aspectRatio = computed(() => {
  if (props.ratio === 'auto' && props.contentHeight === 1 && props.contentWidth === 1) { return 0 }
  if (!isNaN(+props.ratio)) { return props.ratio as number }
  return props.contentWidth / props.contentHeight
})

const stylesComputed = computed(() => {
  if (!aspectRatio.value) { return }

  return { paddingBottom: `${1 / aspectRatio.value * 100}%` }
})

const maxWidthComputed = computed(() => props.maxWidth ? `${props.maxWidth}px` : undefined)
</script>

<style lang="scss">
@import 'variables';

.va-aspect-ratio {
  position: var(--va-aspect-ratio-position);
  overflow: var(--va-aspect-ratio-overflow);
  display: flex;
  max-width: v-bind(maxWidthComputed);
}
</style>
