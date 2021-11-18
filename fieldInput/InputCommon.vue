<template>
  <el-input :value="$attrs.value" @input="singleLineLimit($event)" v-bind="newAttr" v-on="newListeners" :type="filterType" :maxlength="maxlength"></el-input>
</template>

<script>
/**
 * 通用输入框
 * limitType 限制输入类型
 * precision type为number时生效,小数位数
 */
export default {
  name: "InputCommon",
  props: {
    // [text | textarea]
    type: {
      type: String,
      default: 'text'
    },
    // 1 汉字  2 数字  3 字母   4符号
    limitType: {
      type: Array,
      default: () => [1, 2, 3, 4]
    },
    // 数值类型时候的小数精度
    precision: {
      type: Number,
      default: 0
    }
  },
  data () {
    return {
      editValue: null // 上一次编辑的值
    }
  },
  computed: {
    filterType () {
      if (this.type === 'number') {
        return 'text'
      }
      return this.type
    },
    newAttr () {
      const attrs = { ...this.$attrs }
      delete attrs.value
      return attrs
    },
    newListeners () {
      const listeners = { ...this.$listeners }
      delete listeners.input
      return listeners
    },
    maxlength () {
      if (this.type === 'number') {
        return 15
      }
      return this.$attrs.maxlength
    }
  },
  methods: {
    singleLineLimit ($event) {
      // 单行文本
      if (this.type === 'text') {
        // 1 汉字  2 数字  3 字母   4符号
        const value = $event.trim()
        const isVaild = this.$tool.singleLineLimit(this.limitType, value)
        if (isVaild) {
          this.$emit('input', value)
          this.editValue = value
        } else {
          if (!value || value.length === 1) {
            this.$emit('input', '')
          } else {
            this.$emit('input', this.editValue)
          }
        }
      } else if (this.type === 'number') { // 数值
        let value = $event
        value = value.replace(/[^\d.]/g, '')
        const dot1 = value.indexOf('.')
        const dot2 = value.indexOf('.', dot1 + 1)
        if (~dot2) { // 只有一个小数点
          value = value.slice(0, dot2)
        }
        if (~dot1) { // 保留precision为小数
          let endIndex = dot1
          if (this.precision !== 0) {
            endIndex = endIndex + 1 + (this.precision)
          }
          value = value.slice(0, endIndex)
        }
        this.$emit('input', value)
      } else {
        this.$emit('input', $event)
      }
    }
  }
}
</script>
