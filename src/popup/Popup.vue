<template>
  <main class="flex flex-col p-3 pb-0 text-gray-800 w-190px justify-center items-center">
    <div class="cursor-pointer flex justify-center items-center" @click="openOptions">
      <img src="../assets/icon.svg" w-22px>
      <p ml-6px text-14px>
        Color Picker
      </p>
    </div>

    <div class="bg-gray-500 h-1px mt-3 w-full" />

    <div
      class="cursor-pointer bg-blue-500 rounded-2 my-3 text-center py-5px text-18px text-light-50 w-80% on-hover"
      @click="open()"
    >
      Click to pick
    </div>

    <div v-if="notSupported">
      Eye Dropper is not supported
    </div>

    <div v-if="sRGBHex" class="border rounded flex flex-wrap mb-3 w-full p-1 justify-center" :style="{ backgroundColor: sRGBHex }">
      <div
        v-for="color in colorList"
        :key="color"
        class="border rounded cursor-pointer bg-light-200 my-3px mr-4px py-2px px-6px"
        :title="copySupported ? 'copy' : ''"
        @click="copy(color)"
      >
        {{ text === color && copied ? 'copied!' : color }}
      </div>
    </div>
  </main>
</template>

<script setup lang="ts">
import { useClipboard, useEyeDropper } from '@vueuse/core'

const { isSupported, open, sRGBHex } = useEyeDropper()
const notSupported = computed(() => isSupported.value ? '' : 'Eye Dropper is not supported')

const colorList = computed(() => {
  const Hex = sRGBHex.value.replace('#', '')
  const rgb = colorHexToRgb(sRGBHex.value)
  const rgba = rgb.replace('rgb', 'rgba').replace(')', ',1)')
  return [Hex, sRGBHex.value, rgb, rgba]
})

const { text, copy, copied, isSupported: copySupported } = useClipboard()
watch(sRGBHex, (value) => {
  if (value)
    copy(value)
})

// 16进制颜色转换为RGB颜色
function colorHexToRgb(color: string) {
  const reg = /^#([0-9a-fA-f]{3}|[0-9a-fA-f]{6})$/
  let sColor = color.toLowerCase()
  if (sColor && reg.test(sColor)) {
    if (sColor.length === 4) {
      let sColorNew = '#'
      for (let i = 1; i < 4; i += 1)
        sColorNew += sColor.slice(i, i + 1).concat(sColor.slice(i, i + 1))

      sColor = sColorNew
    }
    // 处理六位的颜色值
    const sColorChange = []
    for (let i = 1; i < 7; i += 2)
      sColorChange.push(parseInt(`0x${sColor.slice(i, i + 2)}`))

    return `rgb(${sColorChange.join(',')})`
  }
  return sColor
}

function openOptions() {
  browser.runtime.openOptionsPage()
}
</script>

<style>
.on-hover{
  transition: all 0.05s;
}
.on-hover:hover {
    box-shadow: 0 0 20px 0 rgb(44 71 146 / 40%);
    transform: scale(1.05);
  }
</style>
