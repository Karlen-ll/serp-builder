<script setup lang="ts">
import { ref, nextTick } from 'vue'
import VInput from './VInput.vue'
import VButton from '../button/VButton.vue'

const props = withDefaults(
  defineProps<{
    modelValue: string[]
    legend: string
    placeholder?: string
    min?: number
    max?: number
  }>(),
  {
    min: 1,
    max: 4,
  }
)

const emit = defineEmits<{
  (e: 'update:modelValue', contacts: string[]): void
}>()

const handleAdd = () => {
  emit('update:modelValue', props?.modelValue?.length ? [...props.modelValue, ''] : [''])

  nextTick(() => {
    const inputs = fieldsetReference.value!.querySelectorAll('input')
    inputs[props.modelValue.length - 1].focus()
  })
}

const fieldsetReference = ref<HTMLFieldSetElement | null>(null)

const handleUpdate = (value: string, index: number) => {
  const result = [...props.modelValue!]
  result[index] = value

  emit('update:modelValue', result)
}

const handleRemove = (index: number) => {
  const result = [...props.modelValue!]
  result.splice(index, 1)

  emit('update:modelValue', result)
}
</script>

<template>
  <fieldset ref="fieldsetReference" class="input-group">
    <legend class="input-group__title">{{ legend }}</legend>

    <v-button
      class="input-group__add-button"
      :mode="['secondary', 'circle']"
      icon="plus"
      :aria-label="`Add ${placeholder}`"
      :state="modelValue.length >= max ? 'disabled' : undefined"
      @click="handleAdd"
    />

    <TransitionGroup name="list" tag="ul" class="input-group__list">
      <li v-for="(item, index) in modelValue" :key="index" class="input-group__item">
        <v-input
          :model-value="item"
          :placeholder="`${placeholder} ${index + 1}`"
          @update:modelValue="handleUpdate($event, index)"
        />

        <transition name="button-slide">
          <v-button
            v-show="modelValue.length > min"
            class="input-group__remove-button"
            mode="icon"
            icon="close"
            aria-label="Remove"
            @click="handleRemove(index)"
          />
        </transition>
      </li>
    </TransitionGroup>
  </fieldset>
</template>

<style scoped lang="scss">
.input-group {
  border: none;
  position: relative;
  padding: 0;
  margin: 0;

  &__list {
    padding: 0;
    margin: 0;
  }

  &__item {
    display: flex;
    justify-content: center;
    position: relative;
  }

  &__remove-button.button {
    width: $input-field-height;
    height: $input-field-height;
    color: $color--danger;
    padding: 0;
    margin: #{$margin-lg + $input-padding-y} 0 0 $margin-xs;

    @include hover() {
      color: #ff2344;
    }
  }

  &__add-button {
    right: 0;
    bottom: 100%;
    position: absolute;
  }
}

.button-slide {
  &-enter-active,
  &-leave-active {
    transition: opacity $timeout-lg ease, transform $timeout-lg ease;
  }

  &-enter-from,
  &-leave-to {
    opacity: 0;
    transform: scaleX(0);
  }
}
</style>