<script setup lang="ts">
import { Logger } from '@171h/log'

interface Props {
  value: any
  version: number
}
const props = defineProps<Props>()

const logger = new Logger('CellShow.vue')

const isChange = ref(false)
watch([() => props.value, () => props.version], ([value, version], [preValue, preVersion]) => {
  if ((value !== preValue) && (version !== preVersion))
    isChange.value = true
  else 
    isChange.value = false
  
  logger.debug('value, version, preValue, preVersion', {value, version, preValue, preVersion})
  logger.debug('isChange', isChange.value)
})
</script>

<template>
  <div class="flex flex-col justify-center h-100%!" :style="{ 'background-color': isChange ? '#eff3ff' : 'transparent' }" >
    <span>{{ value }}</span>
  </div>
</template>