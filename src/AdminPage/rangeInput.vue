<template>
  <div :class="['range-input-container', validateError ? 'error' : '']">
    <el-input v-model="start" :size="size" @input="(v) => setVal(v, 'start')" placeholder="输入起始值" />
    <span class="range-tips">至</span>
    <el-input v-model="end" :size="size" @input="(v) => setVal(v, 'end')" placeholder="输入结束值" />
  </div>
</template>

<script>
  export default {
    name: 'range-input',
    props: {
      value: {
        type: Array,
        default: () => []
      },
      size: {
        type: String,
        default: 'mini'
      }
    },
    watch: {
      value: {
        handler(v = []) {
          const [a, b] = v;
          this.start = a;
          this.end = b;
        },
        immediate: true
      },
      arrResult: {
        handler(v) {
          const [a, b] = v;
          if (a <= b) {
            this.validateError = false;
            this.$emit('input', this.arrResult);
            this.$emit('change', this.arrResult);
          } else {
            this.validateError = true;
          }
        }
      }
    },
    computed: {
      arrResult() {
        const startNums = this.start || 0;
        const endNums = this.end || 0;
        return [startNums, endNums];
      }
    },
    methods: {
      setVal(v, key) {
        const nums = v.replace(/\D*/g, '');
        this[key] = Number(nums);
      }
    },
    data() {
      return {
        start: '',
        end: '',
        validateError: false
      };
    }
  };
</script>

<style lang="scss">
.range-input-container {
  display: flex;
  flex-wrap: nowrap;
  align-items: center;
  &.error {
    &::after {
      content: '范围错误';
      position: absolute;
      top: 22px;
      font-size: 12px;
      color: #f56c6c;
    }
    .el-input__inner {
      border-color: #f56c6c;
    }
  }

  .el-input {
    width: 80px;
    &:first-child {
      .el-input__inner {
        border-radius: 5px 0 0 5px;
      }
    }
    &:last-child {
      .el-input__inner {
        border-radius: 0 5px 5px 0;
      }
    }
  }
  .range-tips {
    background-color: #f5f7fa;
    color: #909399;
    border: 1px solid #dcdfe6;
    padding: 0 10px;
    height: 28px;
    line-height: 28px;
    box-sizing: border-box;
    font-size: 12px;
    border-left: none;
    border-right: none;
  }
}
</style>
