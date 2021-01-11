<template>
  <div class="edit-info-config-container">
    <el-form
      ref="ruleForm"
      :model="form"
      :class="['info-form-container', className ? className : '']"
      label-width="120px"
      v-bind="$attrs"
    >
      <el-form-item
        v-for="item in editConfig"
        :key="item.key"
        :label="item.label"
        :prop="item.key"
        :class="item.itemClassName ? item.itemClassName : ''"
        :rules="item.rules"
        v-bind="item"
      >
        <div v-if="item.isText">
          <plain-component :item="item" :data="form[item.key]" />
        </div>

        <edit-component v-else v-model="form[item.key]" :config="item" @change="changeItemVal" v-bind="item" />
      </el-form-item>
    </el-form>
    <el-dialog title="图片预览" :visible.sync="dialogVisible" width="40%">
      <img class="dialog-img" :src="currentUrl" />
    </el-dialog>
  </div>
</template>

<script>
import plainComponent from './plainComponent/plainComponent';
import editComponent from './editComponent/editComponent';
import './main.scss';

export default {
  name: 'edit-config',
  props: {
    config: {
      required: true,
      type: Array
    },
    value: {
      required: true
    },
    className: {
      type: String
    }
  },
  components: {
    plainComponent,
    editComponent
  },
  watch: {
    config: {
      handler(newConfig = [], oldConfig) {
        this.editConfig = newConfig;
        if (JSON.stringify(newConfig) !== JSON.stringify(oldConfig)) {
          // 1. 第一次发生在初始化，old=undefined
          // 2. 第二次发生在select(比如ajax)的option初始化了
          this.restructForm();
        }
      },
      deep: true,
      immediate: true
    },
    value: {
      handler(newForm, oldForm = {}) {
        if (!this.editConfig || this.editConfig.length < 0) {
          return;
        }
        const newStr = JSON.stringify(newForm);
        const oldStr = JSON.stringify(oldForm);
        // 组件内部更新changeItemVal，emit input出去，这里接住的watch是一样的，所以不更新
        if (newStr === oldStr) {
          return;
        }

        this.form = Object.assign({}, this.initForm, this.form, newForm);
      },
      immediate: true,
      deep: true
    }
  },
  methods: {
    // 要帮助form初始化，不然新增一个key，也会触发watch,因为vue不知道v.a是否是新增的
    restructForm() {
      for (let i = 0; i < this.editConfig.length; i++) {
        const item = this.editConfig[i];
        const {
          key, type, defaultVal, multiple
          } = item;
        if (key in this.initForm) {
          break;
        }

        if ((type === 'SELECT' && multiple) || type === 'CHECKBOX') {
          this.initForm[key] = [];
        } else if (type === 'CUSTOM_COMPONENT') {
          this.initForm[key] = defaultVal;
        } else {
          this.initForm[key] = '';
        }
      }
    },
    changeItemVal(val, item, ...rest) {
      this.$emit('change', val, item, rest);
      this.$emit('input', this.form);
    },
    validate() {
      return new Promise((resolve) => {
        this.$refs.ruleForm.validate((valid) => {
          resolve(valid);
        });
      });
    }
  },
  data() {
    return {
      form: {},
      editConfig: [],
      dialogVisible: false,
      currentUrl: '',
      initForm: {}
    };
  }
};
</script>
