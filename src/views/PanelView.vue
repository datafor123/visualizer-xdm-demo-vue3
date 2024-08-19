<script setup lang="ts">
import dayjs, { Dayjs } from 'dayjs'
import { ref, computed } from 'vue'
type PanelSelectType = {
  family: Array<string>
  date: {
    start: number
    end: number
  }
}
const props = defineProps<{
  init: PanelSelectType
}>()
const emit = defineEmits<{
  onApply: [msg: PanelSelectType]
}>()

const families = ['Drink', 'Food', 'Non-Consumable']
const family = ref<Array<string>>(props.init.family)
const date = ref<[Dayjs, Dayjs]>([dayjs(props.init.date.start), dayjs(props.init.date.end)])
const isApplyEnable = computed(() => {
  if (
    [...props.init.family].sort().toString() === [...family.value].sort().toString() &&
    props.init.date.start === date.value[0].valueOf() &&
    props.init.date.end === date.value[1].valueOf()
  ) {
    return false
  } else {
    return true
  }
})
const onApplyEvent = () => {
  const msg = {
    family: family.value,
    date: {
      start: date.value[0].valueOf(),
      end: date.value[1].valueOf()
    }
  }
  emit('onApply', msg)
}
</script>

<template>
  <a-row class="flex items-center h-full p-1 justify-between px-5">
    <a-col span="10" offset="0">
      <label class="pr-1">Product Family</label>
      <a-select
        v-model:value="family"
        mode="tags"
        size="large"
        style="width: 260px"
        :token-separators="[',']"
        placeholder="Select a family"
        :options="families.map((value) => ({ value }))"
      ></a-select>
    </a-col>
    <a-col span="10">
      <label class="pr-1">Date</label>
      <a-range-picker size="large" style="width: 300px" v-model:value="date" />
    </a-col>
    <a-col span="4" class="text-right">
      <a-button
        type="primary"
        size="large"
        :disabled="!isApplyEnable"
        v-model:value="date"
        @click="onApplyEvent"
        >Apply</a-button
      >
    </a-col>
  </a-row>
</template>
