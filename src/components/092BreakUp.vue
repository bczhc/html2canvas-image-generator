<script setup lang="ts">
import {takeScreenshot} from "../utils";
import breakupData from '/092-break-up/092拆分.txt?raw'
import codeData from '/092-break-up/092编码.txt?raw'
import {computed, ref} from "vue";
import {registerSimpleImageGenerator} from "../handler";

const appName = '092breakup';

let imageView = ref(null);

interface ListData {
  char: string,
  code?: string[],
  breakup?: string,
}

let props = defineProps<{
  params: object,
}>()

let inputText = computed(() => props.params['text'])

let breakupMap = {}
let codeMap: { [key: string]: string[] } = {}
for (let x of breakupData.split('\n') as string[]) {
  let split = x.split('\t');
  breakupMap[split[0]] = split[1]
}
for (let x of codeData.split('\n') as string[]) {
  let split = x.split('\t')
  if (codeMap[split[0]] == undefined) {
    codeMap[split[0]] = []
  }
  codeMap[split[0]].push(split[1])
}

function getListData(text: string): ListData[] {
  let chars = [...text]
  return chars.map(x => {
    let breakup: string | null = breakupMap[x] || null
    let code: string[] | null = codeMap[x] || null
    return {
      char: x,
      code: code,
      breakup: breakup,
    }
  })
}

registerSimpleImageGenerator(appName, async () => {
  return await takeScreenshot(imageView.value)
})
</script>

<template>
  <div id="view" class="td-view" ref="imageView">
    <ul id="data-ul">
      <li v-for="x in getListData(inputText)" class="字根-display">
        <span>{{ x.char }}</span>
        <span v-if="x.code != null">&nbsp;</span>
        <span v-if="x.code != null">{{ x.code.join(', ') }}</span>
        <span>：</span>
        <span>{{ x.breakup || "？" }}</span>
      </li>
    </ul>
  </div>
</template>

<style scoped lang="scss">
#data-ul {
  list-style-type: none;
  padding: 0;
  margin: 0;
  display: inline-block;
}

#data-ul {
  $border-color: green;

  li {
    border-left: $border-color 1px solid;
    border-right: $border-color 1px solid;
    border-top: $border-color 1px solid;
  }

  li:last-child {
    border-bottom: $border-color 1px solid;
  }
}

@font-face {
  font-family: "〇九二字根专用";
  src: url('/092-break-up/breakup.otf');
}

@font-face {
  font-family: 'Noto Sans';
  src: url('/NotoSans-Regular.ttf');
}

@font-face {
  font-family: 'Noto Sans CJK SC';
  src: url('/NotoSansSC-Regular.otf');
}

.字根-display {
  font-family: "Noto Sans", "〇九二字根专用", "Noto Sans CJK SC", sans-serif, Sans, serif;
}

.td-view {
  padding: 10px;
  display: inline-block;
}
</style>
