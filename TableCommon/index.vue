<template>
  <div
    class="table-common"
    :style="{height: height}"
  >
    <el-table
      ref="table"
      :height="tableHeight"
      :default-sort="defaultSort"
      :data="data"
      :class="[`is-scrolling-${scrollPosition}`]"
      v-bind="$attrs"
      v-on="$listeners"
    >
      <el-table-column
        v-if="selection"
        fixed
        type="selection"
        width="55"
        :selectable="selectable"
      >
      </el-table-column>
      <el-table-column
        v-for="(item, index) in columns"
        :key="index + '' + item.prop"
        v-bind="item"
        :prop="item.prop+''"
      >
        <template #header="headData">
          <slot :name="item.prop+'Header'" v-bind="headData" :col="item">
            <span>{{ item.label }}</span>
          </slot>
          <el-tooltip placement="top" v-if="item.tooltip">
            <div slot="content">
              <span style="line-height: 1.5" v-html="item.tooltip"></span>
            </div>
            <i class="iconfont iconquestion-circle-fill c-gray f-ml10 f-csp fw-nml"></i>
          </el-tooltip>
        </template>
        <template #default="scope">
          <slot v-bind="scope" :col="item">
            <slot v-bind="scope" :col="item" :name="item.prop">
              <hjTableItem :content="baseFilter(scope.row[item.prop],item)" />
            </slot>
          </slot>
        </template>
      </el-table-column>
      <template #empty v-if="$slots.empty">
        <slot name="empty" />
      </template>
    </el-table>
    <hj-pagination v-if="pagination" :pagination="pagination" @change="$emit('changePageNo',$event)"
                   @changePageSize="$emit('changePageSize',$event)"></hj-pagination>
    <config-header v-if="showHeaderConfig" ref="configHeader" v-bind="headerConfigOption" @updateList="$emit('updateList')"></config-header>
  </div>
</template>

<script>
/*
* 表格加分页
* slot : item.prop+'Header' // 自定义表头的插槽
* slot : item.prop // 自定义内容的插槽
* slot : empty
* slot : default //默认插槽，外部判断col自定义显示内容，默认显示hjTableItem
* $emit('changePageNo') // 改变页码分页组件返回结果
* $emit('changePageSize') // 分页组件返回结果
* $emit('headerConfig') // 点击配置表头按钮
* $emit('updateList') // 配置完表头刷新列表
* 额外的attr和事件监听自动绑定到table
* */
import { throttle } from "throttle-debounce";

export default {
  name: "TableCommon",
  components: { ConfigHeader },
  props: {
    baseFilter: {
      type: Function,
      default: val => val
    },
    selection: {
      type: Boolean,
      default: false
    },
    height: {
      type: String
    },
    pagination: {
      type: Object,
      default: undefined
    },
    data: {
      type: Array,
      default: () => []
    },
    selectable: {
      type: Function,
      default: undefined
    },
    // [ {
    //    ...table-column的attr
    // tooltip: 额外属性tooltip自定义头显示提示
    // }]
    columns: {
      type: Array,
      default: () => [],
      required: true
    },
    defaultSort: {
      type: Object,
      default: undefined
    },
    showHeaderConfig: { // 显示配置表头
      type: Boolean,
      default: false
    },
    headerConfigOption: {
      type: Object,
      default: undefined
    }
  },
  data () {
    return {
      scrollPosition: 'none'
    }
  },
  computed: {
    tableHeight () {
      return this.pagination ? `calc(${this.height}  - 30px)` : this.height
    }
  },
  mounted () {
    if (this.showHeaderConfig) {
      this.bindEvents()
      this.syncPostion()
      const btnstr = `<div class="table-header-config--btn"><i class="iconfont iconcog-fill" title="自定义表头字段"></i></div>`
      const btn = document.createElement('div')
      btn.className = 'table-header-config--btn--wrap'
      btn.innerHTML = btnstr
      btn.onclick = this.headerConfig
      this.$nextTick(() => {
        btn.style.height = this.$refs.table.$el.querySelector('.el-table__header-wrapper').offsetHeight + 'px'
        this.$refs.table.$el.appendChild(btn)
      })
    }
  },
  beforeDestroyed () {
    this.unbindEvents();
  },
  methods: {
    headerConfig () {
      this.$emit('headerConfig')
      this.$refs.configHeader.handleDialogShow()
    },
    syncPostion: throttle(20, function () {
      this.scrollPosition = this.$refs.table.scrollPosition
    }),
    bindEvents () {
      this.$refs.table.$el.querySelector('.el-table__body-wrapper').addEventListener('scroll', this.syncPostion, { passive: true });
    },
    unbindEvents () {
      this.$refs.table.$el.querySelector('.el-table__body-wrapper').removeEventListener('scroll', this.syncPostion, { passive: true });
    },
    clearSelection () {
      this.$refs.table.clearSelection();
    }
  }
}
</script>

<style scoped lang="scss">
.table-common {
  flex: 1;
  height: 100%;
  display: flex;
  flex-direction: column;
  &::v-deep{
    .el-table__header-wrapper{
      position: relative;
    }
    .is-scrolling-right,.is-scrolling-none{
      .table-header-config--btn{
        box-shadow: none
      }
    }
    .table-header-config--btn--wrap{
      position: absolute;
      overflow-y: hidden;
      width: 46px;
      height: 60px;
      right: 0;
      top: 0;
      display: flex;
      justify-content: flex-end;
      z-index: 100;
    }
    .table-header-config--btn {
      width: 36px;
      height: 100%;
      //position: absolute;
      display: flex;
      align-items: center;
      //right: 0;
      //top: 0;
      padding-right: 10px;
      padding-left: 10px;
      cursor: pointer;
      box-shadow: 0 0 10px rgba(0, 0,0, 0.12);
      background: #f5f5fa;
    }
  }
}

.c-gray {
  color: rgba(193, 193, 202, .5) !important;
}

.fw-nml {
  font-weight: normal;
}
</style>
