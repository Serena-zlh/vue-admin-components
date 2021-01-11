<template>
  <div class="classify-tree-selector">
    <div class="handler-block" @click="showTree">
      <div class="tags-container">
        <el-tag v-for="tag in selectedList" :key="tag.key" type="info" size="mini">
          {{ tag.name }}
        </el-tag>
      </div>
    </div>
    <div :class="['filter-block', showDialog ? 'absolute-filter' : '', hide ? 'hide' : '']" @click="showTree">
      <el-input v-model="filterText" placeholder="输入文本搜索节点" size="mini"  />
      <el-tree
        class="filter-tree"
        :data="treeData"
        :props="defaultProps"
        default-expand-all
        show-checkbox
        node-key="key"
        @check="handleCheckChange"
        :filter-node-method="filterNode"
        ref="tree"
      />
    </div>
  </div>
</template>

<script>
  export default {
    name: 'classify-selector',
    props: {
      value: {
        type: Array,
        default: () => []
      },
      showDialog: {
        type: Boolean,
        default: true
      }
    },
    watch: {
      value: {
        handler(v = []) {
          this.$nextTick(() => {
            this.$refs.tree.setCheckedKeys(v);
            const selectedNode = this.$refs.tree.getCheckedNodes();
            this.selectedList = selectedNode.filter(x => x.children === undefined);
          });
        },
        immediate: true
      },
      filterText(val) {
        this.$refs.tree.filter(val);
      }
    },
    methods: {
      showTree(e) {
        e.stopPropagation();
        this.hide = false;
      },
      handleCheckChange(node, { checkedNodes }) {
        this.selectedList = checkedNodes.filter(x => x.children === undefined);
        this.$emit(
          'input',
          this.selectedList.map(x => x.key)
        );
        this.$emit(
          'change',
          this.selectedList.map(x => x.key)
        );
      },
      filterNode(value, data) {
        if (!value) return true;
        return data.name.indexOf(value) !== -1;
      }
    },
    mounted() {
      if (this.showDialog) {
        document.addEventListener('click', () => {
          this.hide = true;
        });
      }
    },
    data() {
      return {
        hide: true,
        selectedList: [],
        filterText: '',
        defaultProps: {
          children: 'children',
          label: 'name'
        },
        treeData: [
          {
            children: [
              {
                key: '1-1',
                name: '其他'
              },
              {
                key: '1-2',
                name: '资源产出'
              }
            ],
            key: '1',
            name: '意见类型'
          },
          {
            children: [
              {
                key: '2-1',
                name: '角色天赋'
              },
              {
                key: '2-2',
                name: '角色等级'
              },
              {
                key: '2-3',
                name: '角色命座'
              }
            ],
            key: '2',
            name: '数值养成'
          }
        ]
      };
    }
  };
</script>

<style lang="scss">
.classify-tree-selector {
  position: relative;
  .filter-block {
    position: relative;
    &.hide {
      display: none;
    }
    &.absolute-filter {
      position: absolute;
      top: 33px;
    }
    z-index: 10;
    border: 1px solid #e4e7ed;
    border-radius: 4px;
    background-color: #fff;
    box-shadow: 0 2px 12px 0 rgba(0,0,0,.1);
    box-sizing: border-box;
    margin: 15px 0;
    padding: 10px 20px;
    min-width: 260px;
    &::before {
      content: '';
      width: 0;
      height: 0;
      border: 10px solid #e4e7ed;
      position: absolute;
      top: -10px;
      left: 15px;
      transform: rotate(45deg);
      border-right-color: transparent;
      border-bottom-color: transparent;
    }
    &::after {
      content: '';
      width: 0;
      height: 0;
      border: 10px solid #fff;
      position: absolute;
      top: -8px;
      left: 15px;
      transform: rotate(45deg);
      border-right-color: transparent;
      border-bottom-color: transparent;
    }
    .filter-tree {
      height: 200px;
      overflow-y: auto;
    }
  }
  .handler-block {
    position: relative;
    cursor: pointer;
    .tags-container {
      min-height: 28px;
      min-width: 220px;
      &:empty {
        background-color: #f2f2f2;
        &::after {
          content: '请从下方树节点中选择分类';
          margin-left: 15px;
          font-size: 12px;
          color: #bbb;
        }
      }
      .el-tag {
        margin-right: 5px;
      }
    }
  }
}
</style>
