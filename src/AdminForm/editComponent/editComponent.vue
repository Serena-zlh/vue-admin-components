<template>
  <div :class="['edit-component']">
    <div class="value-container">
      <!-- 输入框 -->
      <div v-if="currentConfig.type === INPUT">
        <p v-if="showEdit && !editMode">{{outputVal}}</p>
        <el-input
          v-else
          v-model="outputVal"
          @change="handleChange"
          :placeholder="$attrs.placeholder ? $attrs.placeholder : `请输入${currentConfig.label}`"
          v-bind="$attrs"
        />
      </div>
      <!-- 计数器 -->
      <div v-if="currentConfig.type === INPUTNUMBER">
        <p v-if="showEdit && !editMode">{{outputVal}}</p>
        <el-input-number
          v-else
          v-model="outputVal"
          @change="handleChange"
          :placeholder="$attrs.placeholder ? $attrs.placeholder : `请输入${currentConfig.label}`"
          v-bind="$attrs"
        />
      </div>
      <!-- 文本框 -->
      <div v-if="currentConfig.type === TEXTAREA">
        <p v-if="showEdit && !editMode">{{outputVal}}</p>
        <el-input
          v-else
          type="textarea"
          v-model="outputVal"
          @change="handleChange"
          :placeholder="$attrs.placeholder ? $attrs.placeholder : `请输入${currentConfig.label}`"
          v-bind="$attrs"
        />
      </div>
      <!-- 单选按钮组 -->
      <div v-if="currentConfig.type === RADIO">
        <p v-if="showEdit && !editMode">{{computeCurVal(currentConfig)}}</p>
        <el-radio-group v-else v-model="outputVal" @change="handleChange" v-bind="$attrs">
          <el-radio
            v-for="radioItem in currentConfig.option"
            :label="radioItem.value"
            :key="radioItem.value"
            v-bind="radioItem"
          >{{radioItem.label}}</el-radio>
        </el-radio-group>
      </div>
      <!-- Switch开关 -->
      <div v-if="currentConfig.type === SWITCH">
        <p v-if="showEdit && !editMode">{{outputVal}}</p>
        <el-switch
          v-else
          v-model="outputVal"
          @change="handleChange"
          active-color="#13ce66"
          inactive-color="#ff4949"
          v-bind="$attrs"
        />
      </div>
      <!-- 多选组 -->
      <div v-if="currentConfig.type === CHECKBOX">
        <p v-if="showEdit && !editMode">{{computeCurVal(currentConfig)}}</p>
        <el-checkbox-group v-else v-model="outputVal" @change="handleChange" v-bind="$attrs">
          <el-checkbox
            v-for="checkboxItem in currentConfig.option"
            :label="checkboxItem.value"
            :key="checkboxItem.value"
            v-bind="checkboxItem"
          >{{checkboxItem.label}}</el-checkbox>
        </el-checkbox-group>
      </div>
      <!-- 下拉框 -->
      <div v-if="currentConfig.type === SELECT">
        <p v-if="showEdit && !editMode">{{computeCurVal(currentConfig)}}</p>
        <el-select
          v-else
          v-model="outputVal"
          @change="handleChange"
          :placeholder="$attrs.placeholder ? $attrs.placeholder : `请选择${currentConfig.label}`"
          v-bind="$attrs"
        >
          <el-option
            v-for="selectItem in currentConfig.option"
            :key="selectItem.value"
            :label="selectItem.label"
            :value="selectItem.value"
            v-bind="selectItem"
          ></el-option>
        </el-select>
      </div>
      <!-- 图片上传 -->
      <div v-if="currentConfig.type === UPLOAD">
        <div v-if="showEdit && !editMode">
          <span v-for="(img, index) in $attrs['file-list']" :key="index">
            <img v-if="img.url" class="display-img" :src="img.url" />
          </span>
        </div>
        <el-upload
          v-else
          :list-type="$attrs['list-type'] || 'picture-card'"
          v-bind="$attrs"
          v-model="outputVal"
        >
          <i class="el-icon-plus"></i>
        </el-upload>
      </div>
      <!-- 单个日期选择 -->
      <div v-if="currentConfig.type === DATE">
        <p v-if="showEdit && !editMode">{{outputVal}}</p>
        <el-date-picker
          v-else
          v-model="outputVal"
          type="date"
          :value-format="$attrs['value-format'] ? $attrs['value-format'] : 'yyyy-MM-dd'"
          :placeholder="$attrs.placeholder ? $attrs.placeholder : '请选择日期'"
          @change="handleChange"
          v-bind="$attrs"
        />
      </div>
      <!-- 单个日期时间选择 -->
      <div v-if="currentConfig.type === DATETIME">
        <p v-if="showEdit && !editMode">{{outputVal}}</p>
        <el-date-picker
          v-else
          v-model="outputVal"
          type="datetime"
          :value-format="$attrs['value-format'] ? $attrs['value-format'] : 'yyyy-MM-dd HH:mm:ss'"
          :placeholder="$attrs.placeholder ? $attrs.placeholder : '请选择日期时间'"
          @change="handleChange"
          v-bind="$attrs"
        />
      </div>
      <!-- 日期段 -->
      <div v-if="currentConfig.type === DATERANGE">
        <p v-if="showEdit && !editMode">{{outputVal}}</p>
        <el-date-picker
          v-else
          v-model="outputVal"
          type="daterange"
          :value-format="$attrs['value-format'] ? $attrs['value-format'] : 'yyyy-MM-dd'"
          :end-placeholder="$attrs['end-placeholder'] ? $attrs['end-placeholder'] : '结束日期'"
          :start-placeholder="$attrs['start-placeholder'] ? $attrs['start-placeholder'] : '开始日期'"
          :range-separator="$attrs['range-separator'] ? $attrs['range-separator'] : '至'"
          @change="handleChange"
          v-bind="$attrs"
        />
      </div>
      <!-- 日期时间段 -->
      <div v-if="currentConfig.type === DATETIMERANGE">
        <p v-if="showEdit && !editMode">{{outputVal}}</p>
        <el-date-picker
          v-else
          v-model="outputVal"
          type="datetimerange"
          :value-format="$attrs['value-format'] ? $attrs['value-format'] : 'yyyy-MM-dd HH:mm:ss'"
          :end-placeholder="$attrs['end-placeholder'] ? $attrs['end-placeholder'] : '结束日期'"
          :start-placeholder="$attrs['start-placeholder'] ? $attrs['start-placeholder'] : '开始日期'"
          :range-separator="$attrs['range-separator'] ? $attrs['range-separator'] : '至'"
          @change="handleChange"
          v-bind="$attrs"
        />
      </div>
      <!-- 自定义组件 -->
      <div v-if="currentConfig.type === CUSTOM_COMPONENT">
        <component
          v-bind:is="currentConfig.component"
          v-model="outputVal"
          @change="handleCustom"
          v-bind="$attrs"
        ></component>
      </div>
    </div>
    <i
      v-if="showEdit"
      :class="['el-icon-edit', editMode ? 'edit-highlight' : '']"
      @click="changeEditStatus"
    ></i>
  </div>
</template>

<script>
import {
  INPUT,
  INPUTNUMBER,
  TEXTAREA,
  RADIO,
  SWITCH,
  CHECKBOX,
  SELECT,
  UPLOAD,
  DATE,
  DATETIME,
  DATERANGE,
  DATETIMERANGE,
  CUSTOM_COMPONENT
} from '../typeConst';

export default {
  name: 'edit-component',
  props: {
    value: {
      required: true
    },
    config: {
      type: Object,
      required: true
    }
  },
  methods: {
    handleCustom(val, ...rest) {
      this.outputVal = val;
      this.$emit('change', this.outputVal, this.currentConfig, rest);
      this.$emit('input', this.outputVal);
    },
    handleChange() {
      const { type } = this.currentConfig;
      if (this.showEdit && type !== 'INPUTNUMBER') {
        // 数字控件，需要手动点[编辑]按钮触发更新，其他控件自动改了就emit
        this.editMode = false;
        return;
      }
      this.emitVal();
    },
    emitVal() {
      this.$emit('change', this.outputVal, this.currentConfig);
      this.$emit('input', this.outputVal);
    },
    // 改变编辑状态
    changeEditStatus() {
      this.editMode = !this.editMode;
      if (!this.editMode) {
        this.emitVal();
      }
    },
    onUpload(rest, config) {
      this.$emit('change', rest, config);
    },
    // 在非编辑状态时计算展示值
    computeCurVal(item) {
      const {
          option, value, type, multiple
      } = item;

      if (!option || option.length === 0) {
        return '' || this.value || this.outputVal;
      }

      const currentVal = value || this.value;

      if (type === 'CHECKBOX' || (type === 'SELECT' && multiple)) {
        const textList = [];
        currentVal.forEach((x) => {
          const text = option.find(y => y.value === x).label;
          textList.push(text);
        });
        return textList.join(',') || '';
      }
      const currentItem = option.find(x => x.value === currentVal);

      if (currentItem) {
        return currentItem.label;
      }
      return '' || this.value;
    }
  },
  computed: {
    //  true展示编辑按钮
    showEdit() {
      const { showEdit } = this.currentConfig || {};
      return showEdit;
    }
  },
  watch: {
    outputVal: {
      handler() {
        this.emitVal();
      }
    },
    config: {
      handler(newConfig, oldConfig = {}) {
        if (!newConfig) {
          return;
        }
        // 有option一般证明是下拉框和CheckBox，radio这类选项是动态的
        if ('option' in newConfig) {
          const { option: newOption } = newConfig;
          const { option: oldOption } = oldConfig;
          if (JSON.stringify(newOption) !== JSON.stringify(oldOption)) {
            const { type, multiple, defaultVal } = newConfig;
            if ((type === 'SELECT' && multiple) || type === 'CHECKBOX') {
              this.outputVal = [];
            } else if (type === 'CUSTOM_COMPONENT') {
              this.outputVal = defaultVal;
            } else {
              this.outputVal = '';
            }
          }
        }
        this.currentConfig = newConfig;
      },
      immediate: true,
      deep: true
    },
    value: {
      handler(newVal) {
        this.outputVal = newVal;
      },
      immediate: true,
      deep: true
    }
  },
  data() {
    return {
      currentConfig: {},
      outputVal: '',
      editMode: false,
      INPUT,
      INPUTNUMBER,
      TEXTAREA,
      RADIO,
      SWITCH,
      CHECKBOX,
      SELECT,
      UPLOAD,
      DATE,
      DATETIME,
      DATERANGE,
      DATETIMERANGE,
      CUSTOM_COMPONENT
    };
  }
};
</script>

<style lang="scss">
.edit-component {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  max-width: 500px;
  .value-container {
    p {
      height: 40px;
      line-height: 40px;
      margin: 0;
    }
    .display-img {
      width: 140px;
      height: 140px;
      margin-right: 30px;
    }
  }
  .el-icon-edit {
    margin-left: 20px;
    cursor: pointer;
  }
  .edit-highlight {
    color: #409eff;
  }
}
</style>
