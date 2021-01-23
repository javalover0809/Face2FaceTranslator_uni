<template>
<view>
<view v-if="modalShow" style="height:100%;width:100%">
  <view class="modal-wrapper">
    <view class="modal-triangle"> </view>
    <view class="menu-modal">
      <view v-for="(item, index) in modalItems" :key="index" class="menu-modal-item  " :data-type="item.type" @tap="itemTap">{{item.text}}
      </view>
    </view>
  </view>
</view>
<view v-if="modalShow" class="modal-hidden" @touchstart="leaveBubbleModal"></view>
</view>
</template>

<script>
/*
Tencent is pleased to support the open source community by making Face-2-Face Translator available.

Copyright (C) 2018 THL A29 Limited, a Tencent company. All rights reserved.

Licensed under the MIT License (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at
http://opensource.org/licenses/MIT

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.
*/
import { language } from "../../utils/conf";
const tips_language = language[0];
let modalItems = [{
  type: 'copySource',
  text: tips_language.copy_source_text
}, {
  type: 'delete',
  text: tips_language.delete_item
}, {
  type: 'copyTarget',
  text: tips_language.copy_target_text
}];

export default {
  data() {
    return {
      // tips_language: language[0], // 目前只有中文
      modalItems: modalItems
    };
  },

  components: {},
  props: {
    item: {
      type: Object,
      default: () => ({})
    },
    modalShow: {
      type: Boolean,
      default: true
    },
    index: {
      type: Number
    }
  },
  mounted: function () {},
  methods: {
    deleteBubbleModal: function () {
      this.$emit('modaldelete', {
        detail: {
          item: this.data.item,
          index: this.data.index
        }
      }, {
        bubbles: true,
        composed: true
      });
      this.leaveBubbleModal();
    },

    /**
     * 点击
     */
    itemTap: function (e) {
      let itemType = e.currentTarget.dataset.type;
      let item = this.item;

      switch (itemType) {
        case 'copySource':
          this.setClip(item.text);
          break;

        case 'copyTarget':
          this.setClip(item.translateText);
          break;

        case 'delete':
          this.deleteBubbleModal();
          break;

        default:
          break;
      }
    },

    /**
     * 复制到剪贴板
     *
     * @param      {string}  text    需要复制到剪贴板的文字
     */
    setClip: function (text) {
      uni.setClipboardData({
        data: text,
        success: res => {
          this.leaveBubbleModal();
          uni.showToast({
            title: "已复制到剪切板",
            icon: "success",
            duration: 1000,
            success: function (res) {
              console.log("show succ");
            },
            fail: function (res) {
              console.log(res);
            }
          });
        }
      });
    },
    leaveBubbleModal: function () {
      this.$emit('modalleave', {
        detail: {
          modalShow: this.data.modalShow
        }
      });
    }
  }
};
</script>
<style>
@import "./index.css";
</style>