<template>
  <div class="field-input">
    <template v-if="[3,7].includes(field.fieldType)">
      <slot>
        <el-date-picker
          v-on="$listeners"
          v-bind="{...datePickerOption[field.fieldType],...$attrs}"
          placeholder="选择日期">
        </el-date-picker>
      </slot>
    </template>
    <template v-else-if="[2,6].includes(field.fieldType)">
      <slot>
        <select-common
          v-bind="$attrs"
          v-on="$listeners"
          :multiple="field.fieldType===6"
          :opts-list="field.fieldOptsList"
        ></select-common>
      </slot>
    </template>
    <template v-else-if="[1,5].includes(field.fieldType)">
      <slot>
        <input-common
          v-bind="$attrs"
          v-on="$listeners"
          :limit-type="limitType"
          show-word-limit
          :type="field.fieldType===5?'textarea':'text'"
          :maxlength="getLimit(field)"
          :placeholder="filedPlaceHolder(field)"
        ></input-common>
      </slot>
    </template>
    <template v-else-if="[4].includes(field.fieldType)">
      <slot>
        <input-common
          type="number"
          v-bind="$attrs"
          v-on="$listeners"
          :precision="field.decimals"
          placeholder="请输入数字类型的值"></input-common>
      </slot>
    </template>
  </div>
</template>

<script>
/**
 * 自定义字段输入框
 */
import InputCommon from "./InputCommon";
import SelectCommon from "./selectCommon";
export default {
  name: "FieldInput",
  components: { SelectCommon, InputCommon },
  props: {
    field: {
      type: Object,
      default: () => ({})
    }
  },
  data () {
    return {
      // 时间选择配置，3：日期类型，7：日期时间类型
      datePickerOption: {
        3: {
          type: "date",
          valueFormat: "yyyy-MM-dd"
        },
        7: {
          type: "datetime",
          valueFormat: "yyyy-MM-dd HH:mm:ss",
          defaultTime: '00:00:00'
        }
      },
      placeholder: undefined,
      editValue: null // 上一次编辑的值
    }
  },
  computed: {
    limitType () {
      return this.field?.singleLineLimit?.characterType
    }
  },
  mounted () {
  },
  methods: {
    getLimit (field) {
      if (field.fieldType === 1) {
        return field.singleLineLimit.characterLimit
      } else {
        return field.decimals
      }
    },
    filedPlaceHolder (field) {
      if (field.fieldType === 1) {
        const characterType = field?.singleLineLimit?.characterType || []
        const typeName = []
        if (characterType.includes(1)) typeName.push('汉字')
        if (characterType.includes(2)) typeName.push('数字')
        if (characterType.includes(3)) typeName.push('字母')
        if (characterType.includes(4)) typeName.push('符号')
        if (characterType.length) return '请输入' + typeName.join('/')
        else return '请输入内容'
      } else {
        return '请输入内容'
      }
    }
  }
}
</script>

<style scoped>
.field-input{
  width: 100%;
}
</style>
