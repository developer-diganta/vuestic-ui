<template>
  <div v-for="data, key in form" class="item">
    <span class="v-title va-text-capitalize">
      {{ key }}:
    </span>

    <VaValue v-slot="v">
      <input v-if="v.value" v-model="form[key]">
      <span v-else>
        {{ form[key] }}
      </span>

      <VaButton :icon="v.value ? 'save' : 'edit'" preset="plain" size="small" @click="v.value = !v.value" />
    </VaValue>
  </div>
</template>

<script setup lang="ts">
  const form = reactive({
    name: 'Maksim',
    surname: 'Nedoshev'
  })
</script>

<style>
  .item {
    height: 2rem;
  }

  input {
    outline: 1px solid var(--va-background-border);
    box-sizing: border-box;
    padding: 0 4px;
    margin: 0 -4px;
  }
</style>
