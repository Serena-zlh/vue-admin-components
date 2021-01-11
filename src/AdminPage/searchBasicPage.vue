<template>
  <div class="search-basic-page">
    <div class="search-block shadow-container" v-if="searchConfig && searchConfig.length > 0">
      <edit-config v-model="searchForm" :config="searchConfig" inline @input="emitForm" />
      <div class="btn-block">
        <el-button size="mini" plain icon="el-icon-refresh-right" @click="resetQuery">重置</el-button>
        <el-button type="primary" size="mini" plain icon="el-icon-search" @click="doQuery">查询</el-button>
        <slot name="other-btns" />
      </div>
    </div>

    <div class="table-block shadow-container">
      <slot name="header" />
      <el-table border stripe :data="tableData" style="width: 100%" class="table-container" v-on="$listeners">
        <slot name="extra-table-col-type" />
        <!-- 这里约定都用value-label 这样的格式， 适配所有element组件 -->
        <el-table-column
          v-for="item in renderCol"
          :key="item.value"
          :prop="item.value"
          :label="item.label"
          v-bind="item"
        >
          <template slot="header">
            <slot :name="item.headerSlotName" :config="item">{{ item.label }}</slot>
          </template>
          <template slot-scope="scope">
            <slot :name="item.contentSlotName" :config="item" :row="scope.row" :index="scope.$index">
              <div v-if="item.custom" v-html="item.custom(scope.row)"></div>
              <div v-else>{{ scope.row[item.value] }}</div>
            </slot>
          </template>
        </el-table-column>
        <slot name="operator" />
      </el-table>
      <el-pagination
        v-if="total > 0"
        @size-change="sizeChange"
        @current-change="pageChange"
        :current-page="searchForm.page"
        :page-sizes="[10, 20, 50, 100]"
        :page-size="searchForm.page_size"
        layout="total, sizes, prev, pager, next, jumper"
        :total="Number(total)"
      />
    </div>
  </div>
</template>

<script>
  import editConfig from '../AdminForm/main';

  export default {
    name: 'basic-page',
    props: {
      // 搜索框的配置
      config: {
        type: Array,
        default: () => []
      },
      // 表格列
      columns: {
        type: Array,
        default: () => []
      },
      // 搜索接口
      query: {
        type: Function,
        default: () => {}
      },
      // 外层控制刷新列表
      doRefresh: {
        type: Boolean,
        default: false
      },
      // 初始化是否需要请求列表
      immediateQuery: {
        type: Boolean,
        default: true
      },
      value: {
        type: Object,
        default: () => {}
      }
    },
    watch: {
      value: {
        handler(v, oldv = { page: 1, page_size: 10 }) {
          if (v && typeof v === 'object') {
            const a = JSON.stringify(v);
            const b = JSON.stringify(oldv);
            if (a !== b) {
              this.searchForm = Object.assign({}, this.searchForm, v);
            }
          }
        },
        immediate: false,
        deep: false
      },
      config: {
        handler(v) {
          if (v && typeof v === 'object') {
            this.searchConfig = v;
          }
        },
        immediate: true
      },
      doRefresh: {
        handler(v) {
          if (v) {
            this.doQuery();
          }
        },
        immediate: false
      }
    },
    computed: {
      renderCol() {
        return this.columns.filter(x => !x.hide);
      }
    },
    components: {
      editConfig
    },
    mounted() {
      if (this.immediateQuery) {
        this.doQuery();
      }
    },
    methods: {
      emitForm() {
        this.$emit('sync-query', this.searchForm);
        this.$emit('input', this.searchForm);
      },
      sizeChange(size) {
        this.searchForm.page_size = size;
        this.doQuery();
      },
      pageChange(page) {
        this.searchForm.page = page;
        this.doQuery();
      },
      resetQuery() {
        this.searchForm = {
          page_size: 10,
          page: 1
        };
        this.doQuery();
      },
      // 搜索还是用业务代码搜
      async doQuery() {
        this.$emit('update:doRefresh', false);
        this.emitForm();
        const res = await this.query(this.searchForm);
        if (res) {
          ({ tableList: this.tableData, total: this.total } = res);
        }
      }
    },
    data() {
      return {
        searchForm: {
          page: 1,
          page_size: 10
        },
        searchConfig: [],
        total: 0,
        tableData: []
      };
    }
  };
</script>

<style lang="scss" scoped>
.search-basic-page {
  .search-block {
    display: flex;
    justify-content: space-between;
    align-items: flex-end;
  }
  .table-block {
    margin-top: 20px;
    text-align: right;
    .table-container {
      margin-top: 15px;
    }
  }
}
.btn-block {
  text-align: right;
  flex: 1 0 auto;
}
.el-pagination {
  margin-top: 20px;
}
</style>
