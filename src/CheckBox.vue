<script setup lang="ts">
import { Icon } from '@iconify/vue'

defineProps<{ status: boolean | 'any' }>()
const icons = {
  any: 'mdi:checkbox-blank',
  true: 'mdi:checkbox-marked',
  false: 'mdi:cancel-box',
}
</script>

<template>
  <Icon :icon="icons[status.toString() as keyof typeof icons]" />
</template>

<style scoped></style>
