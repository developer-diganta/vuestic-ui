<template>
  <div
    class="va-avatar-group"
    :class="classComputed"
    role="group"
  >
    <va-avatar
      v-for="(option, idx) in maxOptions"
      :key="idx"
      v-bind="{ ...avatarProps, ...option }"
      role="listitem"
    />
    <slot v-if="restOptionsCount > 0" name="rest" v-bind="avatarProps">
      <va-avatar
        v-bind="avatarProps"
        :color="restColor"
        class="va-avatar-group__rest"
        role="listitem"
      >
        +{{ restOptionsCount }}
      </va-avatar>
    </slot>
  </div>
</template>

<script lang="ts">
import { computed, PropType } from 'vue'

import { VaAvatar } from '../va-avatar'

import pick from 'lodash/pick.js'
import { extractComponentProps, filterComponentProps } from '../../utils/component-options'
import { useBem, useComponentPresetProp, useSize, useSizeProps } from '../../composables'

const VaAvatarProps = extractComponentProps(VaAvatar)
</script>

<script lang="ts" setup>

defineOptions({
  name: 'VaAvatarGroup',
})

const props = defineProps({
  ...useSizeProps,
  ...useComponentPresetProp,
  ...VaAvatarProps,

  max: {
    type: Number,
    default: 0,
  },
  vertical: {
    type: Boolean,
    default: false,
  },
  options: {
    type: Array as PropType<Record<string, unknown>[]>,
    default: () => [],
  },
    /** If there are more avatars that can be displayed we show rest number. This prop changes color of rest indicator. */
  restColor: {
    type: String,
    default: 'secondary',
  },
})

const classComputed = useBem('va-avatar-group', () => ({
  ...pick(props, ['vertical']),
}))

const maxOptions = computed(() => props.max && props.max <= props.options.length ? props.options.slice(0, props.max) : props.options)
const restOptionsCount = computed(() => {
  const hasOptions = props.options.length > 0
  const canAddMoreOptions = maxOptions.value.length < props.options.length
  const remainingOptions = props.options.length - (props.max || 0)

  return hasOptions && canAddMoreOptions ? remainingOptions : 0
})
const { sizeComputed, fontSizeComputed } = useSize(props, 'VaAvatarGroup')

const filteredAvatarProps = filterComponentProps(VaAvatarProps)
const avatarProps = computed(() => ({
  ...filteredAvatarProps.value,
  fontSize: fontSizeComputed.value,
  size: sizeComputed.value,
}))
</script>

<style lang="scss">
@import "variables";

.va-avatar-group {
  display: flex;
  flex-wrap: nowrap;

  .va-avatar + .va-avatar {
    margin-left: var(--va-avatar-group-gap);
  }

  &--vertical {
    flex-direction: column;

    .va-avatar + .va-avatar {
      margin-left: 0;
      margin-top: var(--va-avatar-group-gap);
    }
  }
}
</style>
