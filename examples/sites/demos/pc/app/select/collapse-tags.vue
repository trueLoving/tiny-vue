<template>
  <div>
    <br />
    <div>场景1： collapse-tags折叠</div>
    <br />
    <tiny-select v-model="value1" :show-proportion="true" multiple collapse-tags>
      <tiny-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value"> </tiny-option>
    </tiny-select>
    <br />
    <br />
    <div>场景2： hover-expand 折叠</div>
    <br />
    <tiny-select v-model="value2" multiple hover-expand>
      <tiny-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value"> </tiny-option>
    </tiny-select>
    <br />
    <br />
    <br />
    <div>场景3： click-expand 折叠</div>
    <br />
    <tiny-select v-model="value3" :max-visible-rows="2" multiple click-expand>
      <tiny-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value"> </tiny-option>
    </tiny-select>
  </div>
</template>

<script>
import { Select, Option } from '@opentiny/vue'

export default {
  components: {
    TinySelect: Select,
    TinyOption: Option
  },
  data() {
    return {
      options: [
        { value: '选项1', label: '北京' },
        { value: '选项2', label: '上海' },
        { value: '选项3', label: '天津' },
        { value: '选项4', label: '重庆超长超长超长超长超长超长超长超长超长' },
        { value: '选项5', label: '深圳' }
      ],
      value1: ['选项1', '选项2'],
      value2: ['选项1', '选项2', '选项3', '选项4'],
      value3: ['选项1', '选项2', '选项3', '选项4', '选项5']
    }
  }
}
</script>

<style lang="less" scoped>
.tiny-select {
  width: 280px;
}
</style>
