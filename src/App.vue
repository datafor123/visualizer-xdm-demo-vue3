<script setup lang="ts">
import dayjs from 'dayjs'
import { onMounted, onUnmounted, ref } from 'vue'
import XDMWorker, { packageDaterangeMessage, packageNormalMessage } from './common/send.xdm'
import Panel from './views/PanelView.vue'
type PanelSelectType = {
  family: Array<string>
  date: {
    start: number
    end: number
  }
}
const PARAMETERNAME = {
  productFamily: '[product_class].[hierarchy_product_family1].[product_family]',
  date: '[time_by_day].[AGG_the_date].[the_date]'
}
const DEFAULTS = ref<PanelSelectType>({
  family: [],
  date: {
    start: dayjs('1997-03-25', 'YYYY-MM-DD').valueOf(),
    end: dayjs('1997-03-29', 'YYYY-MM-DD').valueOf()
  }
})
const xdm = ref<XDMWorker>()
const iframeRef = ref<HTMLIFrameElement>()
const onApplyEvent = (msg: PanelSelectType) => {
  DEFAULTS.value = { ...msg }
  xdm.value?.send(encodeQueryString(msg), iframeRef.value?.contentWindow as Window)
}

const encodeQueryString = (data: PanelSelectType) => {
  const { family: p, date: d } = data
  const sender = [
    ...packageDaterangeMessage(PARAMETERNAME.date, [
      [
        { i: 1, v: d.start },
        { i: 1, v: d.end }
      ]
    ])
  ]
  if (p.length) sender.push(...packageNormalMessage(PARAMETERNAME.productFamily, [...p]))
  return sender
}

onMounted(() => {
  xdm.value = new XDMWorker({
    onPageInitEvent: () => {
      xdm.value?.send(
        encodeQueryString(DEFAULTS.value),
        iframeRef.value?.contentWindow as Window,
        true
      )
    }
  })
})

onUnmounted(() => {
  xdm.value?.destroy()
})
</script>

<template>
  <div class="flex flex-col h-full w-full">
    <div class="h-16 bg-slate-100" theme="light">
      <Panel @onApply="onApplyEvent" :init="DEFAULTS" />
    </div>
    <div class="h-full bg-white">
      <iframe
        src="https://aws.datafor.com.cn:448/datafor/plugin/datafor/api/share/open?shareid=hrpHe8ZLIm9QLnZ2XIRofIpy&__xdmTimeout=150"
        style="border: none; width: 100%; height: 100%"
        ref="iframeRef"
      />
    </div>
  </div>
</template>

<style scoped></style>
