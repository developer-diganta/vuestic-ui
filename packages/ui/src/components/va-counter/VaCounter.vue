<template>
  <va-input-wrapper
    class="va-counter"
    v-bind="{ ...fieldListeners, ...inputWrapperPropsComputed }"
    :class="classComputed"
    :style="styleComputed"
    :focused="isFocused"
    :error="computedError"
    :error-messages="computedErrorMessages"
    @keydown.up.prevent="increaseCount"
    @keydown.right.prevent="increaseCount"
    @keydown.down.prevent="decreaseCount"
    @keydown.left.prevent="decreaseCount"
  >
    <template v-if="$props.buttons" #prepend="slotScope">
      <div
        class="va-counter__prepend-wrapper"
        :style="{ marginRight: marginComputed }"
        @mousedown.prevent="focus"
      >
        <slot name="decreaseAction" v-bind="{ ...slotScope, decreaseCount }">
          <va-button
            class="va-counter__button-decrease"
            v-bind="decreaseButtonProps"
            ref="decreaseButtonRef"
          />
        </slot>
      </div>
    </template>

    <template v-else #prependInner="slotScope">
      <div
        class="va-counter__prepend-inner"
        @mousedown.prevent="focus"
      >
        <slot name="decreaseAction" v-bind="{ ...slotScope, decreaseCount }">
          <va-button
            v-bind="decreaseIconProps"
            ref="decreaseButtonRef"
          />
        </slot>
      </div>
    </template>

    <template v-if="$props.buttons" #append="slotScope">
      <div
        class="va-counter__append-wrapper"
        :style="{ marginLeft: marginComputed }"
        @mousedown.prevent="focus"
      >
        <slot name="increaseAction" v-bind="{ ...slotScope, increaseCount }">
          <va-button
            class="va-counter__button-increase"
            v-bind="increaseButtonProps"
            ref="increaseButtonRef"
          />
        </slot>
      </div>
    </template>

    <template v-else #appendInner="slotScope">
      <div
        class="va-counter__append-inner"
        @mousedown.prevent="focus"
      >
        <slot name="increaseAction" v-bind="{ ...slotScope, increaseCount }">
          <va-button
            v-bind="increaseIconProps"
            ref="increaseButtonRef"
          />
        </slot>
      </div>
    </template>

    <template v-if="$slots.content" #default="slotScope">
      <div ref="input" tabindex="0" class="va-counter__content-wrapper">
        <slot
          name="content"
          v-bind="{ ...slotScope, value: Number(valueComputed) }"
        />
      </div>
    </template>

    <input
      v-if="!$slots.content"
      ref="input"
      class="va-input__content__input"
      type="number"
      inputmode="decimal"
      v-bind="{ ...inputAttributesComputed, ...inputListeners }"
      :value="valueComputed"
      :aria-live="$props.disabled ? 'off' : 'polite'"
      @input="setCountInput"
      @change="setCountChange"
    />
  </va-input-wrapper>
</template>

<script lang="ts">
import { toRefs, computed, shallowRef, InputHTMLAttributes, ComputedRef, toRef, useAttrs, useSlots, ref } from 'vue'
import omit from 'lodash/omit'
import pick from 'lodash/pick'

import { safeCSSLength } from '../../utils/css'
import {
  useComponentPresetProp,
  useFormFieldProps,
  useEmitProxy,
  useFocus, useFocusEmits,
  useStateful, useStatefulProps,
  useColors,
  useTranslation,
  useLongPress,
  useTemplateRef,
  useValidation,
  useClearableProps,
  useValidationEmits,
} from '../../composables'
import useCounterPropsValidation from './hooks/useCounterPropsValidation'

import { VaInputWrapper } from '../va-input-wrapper'
import { VaButton } from '../va-button'
import { extractComponentProps, filterComponentProps } from '../../utils/component-options'
import isNil from 'lodash/isNil'

const { createEmits: createInputEmits, createListeners: createInputListeners } = useEmitProxy(
  ['change'],
)

const { createEmits: createFieldEmits, createListeners: createFieldListeners } = useEmitProxy([
  { listen: 'click-prepend', emit: 'click:decrease-button' },
  { listen: 'click-append', emit: 'click:increase-button' },
  { listen: 'click-prepend-inner', emit: 'click:decrease-icon' },
  { listen: 'click-append-inner', emit: 'click:increase-icon' },
])

const VaInputWrapperProps = extractComponentProps(VaInputWrapper)
</script>

<script lang="ts" setup>

defineOptions({
  name: 'VaCounter',
  inheritAttrs: false,
})

const props = defineProps({
  ...useFormFieldProps,
  ...useStatefulProps,
  ...useComponentPresetProp,
  ...useClearableProps,
  ...VaInputWrapperProps,
    // input
  modelValue: { type: [String, Number], default: 0 },
  manualInput: { type: Boolean, default: false },
  min: { type: Number },
  max: { type: Number },
  step: { type: Number, default: 1 },
  color: { type: String, default: 'primary' },
    // icons & buttons
  increaseIcon: { type: String, default: 'va-plus' },
  decreaseIcon: { type: String, default: 'va-minus' },
  buttons: { type: Boolean, default: false },
  flat: { type: Boolean, default: true },
  rounded: { type: Boolean, default: false },
  margins: { type: [String, Number], default: '4px' },
  longPressDelay: { type: Number, default: 500 },

  ariaLabel: { type: String, default: '$t:counterValue' },
  ariaDecreaseLabel: { type: String, default: '$t:decreaseCounter' },
  ariaIncreaseLabel: { type: String, default: '$t:increaseCounter' },
})

const emit = defineEmits([
  'update:modelValue',
  ...useValidationEmits,
  ...createInputEmits(),
  ...createFieldEmits(),
  ...useFocusEmits,
])

const input = shallowRef<HTMLInputElement | HTMLDivElement>()

const { min = ref(undefined), max = ref(undefined), step } = toRefs(props)

const {
  isFocused,
  focus,
  blur,
} = useFocus(input, emit)

const { valueComputed } = useStateful(props, emit)

const reset = () => withoutValidation(() => {
  emit('update:modelValue', props.clearValue)
  emit('clear')
  resetValidation()
})

const {
  computedError,
  computedErrorMessages,
  withoutValidation,
  resetValidation,
} = useValidation(props, emit, { reset, focus, value: valueComputed })

const setCountInput = ({ target }: Event) => {
  valueComputed.value = Number((target as HTMLInputElement | null)?.value)
}

const setCountChange = ({ target }: Event) => {
  calculateCounterValue(Number((target as HTMLInputElement | null)?.value))
}

const getRoundDownWithStep = (value: number) => {
  if (typeof min.value === 'undefined' || !step.value) { return value }

  // If the user enters a value manually, then we must round it to the nearest valid value,
  // taking into account the initial value (`props.min`) and the step size (`props.step`)
  return min.value + step.value * Math.floor((value - min.value) / step.value)
}

const calculateCounterValue = (counterValue: number) => {
  if (typeof min.value !== 'undefined' && counterValue < min.value) {
    valueComputed.value = min.value
    return
  }

  if (max.value && (counterValue > max.value)) {
    // since the `props.step` may not be a multiple of `(props.max - props.min)`,
    // we must round the result taking into account the allowable value
    valueComputed.value = getRoundDownWithStep(max.value)
    return
  }

  valueComputed.value = getRoundDownWithStep(counterValue)
}

const isMinReached = computed(() => {
  if (isNil(min.value)) { return false }

  return Number(valueComputed.value) <= min.value
})

const isMaxReached = computed(() => {
  if (isNil(max.value)) { return false }

  return step.value
    ? Number(valueComputed.value) > (max.value - step.value)
    : Number(valueComputed.value) >= max.value
})

const tabIndexComputed = computed(() => props.disabled ? -1 : 0)

const isDecreaseActionDisabled = computed(() => (
  isMinReached.value || props.disabled
))

const isIncreaseActionDisabled = computed(() => (
  isMaxReached.value || props.disabled
))

const decreaseCount = () => {
  if (isDecreaseActionDisabled.value) { return }
  calculateCounterValue(Number(valueComputed.value) - step.value)
}

const increaseCount = () => {
  if (isIncreaseActionDisabled.value) { return }
  calculateCounterValue(Number(valueComputed.value) + step.value)
}

useLongPress(useTemplateRef('decreaseButtonRef'), {
  onUpdate: decreaseCount,
  delay: toRef(props, 'longPressDelay'),
})

useLongPress(useTemplateRef('increaseButtonRef'), {
  onUpdate: increaseCount,
  delay: toRef(props, 'longPressDelay'),
})

const { getColor } = useColors()
const colorComputed = computed(() => getColor(props.color))

const decreaseIconProps = computed(() => ({
  class: { 'va-counter__icon--inactive': isDecreaseActionDisabled.value },
  color: colorComputed.value,
  icon: props.decreaseIcon,
  plain: true,
  disabled: isDecreaseActionDisabled.value,
  readonly: props.readonly,
  tabindex: -1,
  'aria-label': tp(props.ariaDecreaseLabel),
  ...(!isDecreaseActionDisabled.value && { onClick: decreaseCount }),
}))

const increaseIconProps = computed(() => ({
  class: { 'va-counter__icon--inactive': isIncreaseActionDisabled.value },
  color: colorComputed.value,
  icon: props.increaseIcon,
  plain: true,
  disabled: isIncreaseActionDisabled.value,
  readonly: props.readonly,
  tabindex: -1,
  'aria-label': tp(props.ariaIncreaseLabel),
  ...(!isIncreaseActionDisabled.value && { onClick: increaseCount }),
}))

const isSquareCorners = computed(() => (
  (typeof props.margins === 'string' ? parseFloat(props.margins) : props.margins) === 0
))

const buttonsColor = () => {
  if (isFocused.value) { return props.color }

  return 'background-border'
}

const buttonProps = computed(() => ({
  ...pick(props, ['color', 'textColor']),
  round: props.rounded,
  preset: props.flat ? 'secondary' : '',
  borderColor: (props.flat) ? buttonsColor() : '',
}))

const decreaseButtonProps = computed(() => ({
  ...buttonProps.value,
  icon: props.decreaseIcon,
  disabled: isDecreaseActionDisabled.value,
  'aria-label': tp(props.ariaDecreaseLabel),
  ...(!isDecreaseActionDisabled.value && { onClick: decreaseCount }),
}))

const increaseButtonProps = computed(() => ({
  ...buttonProps.value,
  icon: props.increaseIcon,
  disabled: isIncreaseActionDisabled.value,
  'aria-label': tp(props.ariaIncreaseLabel),
  ...(!isIncreaseActionDisabled.value && { onClick: increaseCount }),
}))

const { tp } = useTranslation()

const attrs = useAttrs()
const slots = useSlots()

const inputAttributesComputed = computed(() => (({
  tabindex: tabIndexComputed.value,
  'aria-label': tp(props.ariaLabel),
  'aria-valuemin': min.value,
  'aria-valuemax': max.value,
  ...omit(attrs, ['class', 'style']),
  ...pick(props, ['disabled', 'min', 'max', 'step']),
  readonly: props.readonly || !props.manualInput,
}) as InputHTMLAttributes))

const classComputed = computed(() => ([
  attrs.class,
  { 'va-counter--input-square': isSquareCorners.value },
  { 'va-counter--content-slot': slots.content && props.buttons },
].filter(Boolean)))

const styleComputed: ComputedRef<Partial<CSSStyleDeclaration>> = computed(() => ({
  ...((attrs.style as Partial<CSSStyleDeclaration>) || {}),
}))

const marginComputed = computed(() => safeCSSLength(props.margins))

useCounterPropsValidation(props)

const fieldListeners = createFieldListeners(emit)
const inputListeners = createInputListeners(emit)
const inputWrapperPropsComputed = filterComponentProps(VaInputWrapperProps)

defineExpose({
  focus,
  blur,
  decreaseCount,
  increaseCount,
  reset,
})
</script>

<style lang="scss">
@import "variables";

.va-counter {
  --va-input-wrapper-width: var(--va-form-element-default-width-small);

  &.va-input-wrapper {
    min-width: unset;
    flex: none;

    .va-input-wrapper__field {
      width: unset;
    }
  }

  .va-input-wrapper__field > *,
  .va-input-wrapper__container > * {
    margin-right: 0;
  }

  &.va-counter--input-square {
    .va-input-wrapper__field {
      border-radius: 0;
      border-left: none;
      border-right: none;
    }

    .va-counter__prepend-wrapper {
      z-index: 1;

      .va-counter__button-decrease {
        border-top-right-radius: 0;
        border-bottom-right-radius: 0;
      }

      .va-counter__button-decrease:not(.va-button--square) {
        width: unset;
      }
    }

    .va-counter__append-wrapper {
      z-index: 1;

      .va-counter__button-increase {
        border-top-left-radius: 0;
        border-bottom-left-radius: 0;
      }

      .va-counter__button-increase:not(.va-button--square) {
        width: unset;
      }
    }
  }

  .va-counter__prepend-wrapper,
  .va-counter__append-wrapper {
    .va-counter__button-decrease,
    .va-counter__button-increase {
      .va-button__content {
        padding: unset;
      }
    }
  }

  .va-counter__content-wrapper {
    width: 100%;
    display: flex;
    justify-content: center;

    &:focus-visible {
      outline: none;
    }
  }

  .va-input__content__input {
    text-align: center;

    // Chrome, Safari, Edge, Opera
    &::-webkit-outer-spin-button,
    &::-webkit-inner-spin-button {
      -webkit-appearance: none;
      margin: 0;
    }

    // Firefox
    &[type=number] {
      -moz-appearance: textfield;
    }
  }

  .va-input-wrapper__field {
    padding: 0;
  }

  &__prepend-inner,
  &__append-inner {
    display: flex;
    align-items: stretch;
    height: 100%;
    margin: 0 0.5rem;
  }
}
</style>
