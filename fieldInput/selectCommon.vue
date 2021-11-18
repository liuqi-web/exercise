<template>
  <el-select
    :value="value"
    v-bind="$attrs"
    v-on="$listeners"
  >
    <el-option v-for="option in optsList" :key="option.name" :label="option.name" :value="option.id"></el-option>
  </el-select>
</template>

<script>
/***
 * 下拉选择 选项没有时自动过滤
 */
export default {
  name: "SelectCommon",
  props: ['value', 'optsList'],
  mounted () {
    const { value, optsList } = this
    if (this.$attrs.multiple) {
      if (value && value?.length) {
        const newVal = value.filter(id => optsList.find(i => i.id === id))
        this.$nextTick(() => {
          this.$emit('input', newVal);
        })
      }
    } else {
      if (value && !optsList.find(i => i.id === value)) {
        this.$nextTick(() => {
          this.$emit('input', '');
        })
      }
    }
  }
}
</script>

<style scoped>

</style>
