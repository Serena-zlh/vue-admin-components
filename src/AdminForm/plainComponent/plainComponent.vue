<template>
  <div class="plain-display-component">
    <div v-if="item.type === UPLOAD" class="plain-img-container">
      <div v-if="data && data.length > 0">
        <img
          v-for="(img, index) in data"
          :key="index"
          :src="computeImg(img)"
          alt="暂无图片"
          @click="() => showModal(img)"
        />
      </div>
      <img v-else :src="computeImg({})" alt="暂无图片" />
    </div>
    <div v-else-if="item.type === RADIO">{{getRadio(item)}}</div>
    <div v-else-if="item.type === SELECT">{{getSelect(item)}}</div>
    <div v-else-if="item.type === CHECKBOX">{{getCheckbox(item)}}</div>
    <div v-else>{{data || item.value}}</div>
    <el-dialog title="图片预览" :visible.sync="dialogVisible" width="40%">
      <img class="dialog-img" :src="currentUrl" />
    </el-dialog>
  </div>
</template>

<script>
import {
 UPLOAD, RADIO, SELECT, CHECKBOX
} from '../typeConst';

export default {
  name: 'plain-display-testeeee',
  props: ['item', 'data'],
  methods: {
    getRadio(item) {
      const findVal = item.option.find(x => x.value === this.data);
      if (findVal) {
        return findVal.label;
      }
      return this.data || this.item.value;
    },
    getSelect(item) {
      const {
 option, multiple
} = item;
      if (multiple) {
        return this.getCheckbox(item);
      }
        const currentItem = option.find(x => x.value === this.data);
        return currentItem ? currentItem.label : this.data;
    },
    getCheckbox(item) {
      const { option } = item;
      const textList = [];
      this.data.forEach((x) => {
        const text = option.find(y => y.value === x).label;
        textList.push(text);
      });
      return textList.join(',') || '';
    },
    computeImg({ url }) {
      if (url) {
        return url;
      }
      return require('../../resources/images/no-image.png');
    },
    showModal({ url }) {
      if (!url) {
        return;
      }
      this.dialogVisible = true;
      this.currentUrl = url;
    }
  },
  data() {
    return {
      dialogVisible: false,
      currentUrl: '',
      UPLOAD,
      RADIO,
      SELECT,
      CHECKBOX
    };
  }
};
</script>

<style>
</style>
