<template>
<view class="bubble-wrap" @longpress="showModal">
  <view class="modal-wrap" v-if="recordStatus == 2">
    <modal :modal-show="modalShow" :index="index" :item="item" @modalleave="modalLeave"></modal>
  </view>

  <view class="create-time">{{item.create}}</view>
  <view class="section-body" :data-index="index">
    <view class="send-message">
      <view :data-id="item.id" class="text-content" :data-index="index">
        <view :class="'text-detail  text-detail-' + item.lfrom">{{item.text}}<waiting-icon v-if="recordStatus == 0"></waiting-icon></view>
      </view>
      <navigator hover-class="navigator-hover" :data-text="item.text" :data-id="item.id" :data-index="index" class="edit-icon" v-if="editShow" :data-item="item" :url="'/pages/edit/edit?content='+item.text+'&index='+index">
          <image class="edit-icon-img" :src="edit_icon_path"></image>
      </navigator>
    </view>
    <view class="line-between" v-if="recordStatus > 0"></view>
    <view class="translate-message">
      <view class="text-content">
        <view :class="'text-detail text-detail-' + item.lto">{{item.translateText}}<waiting-icon v-if="recordStatus == 1"></waiting-icon>
      </view>
      </view>
      <view class="play-icon" @tap.stop="playTranslateVoice" catchtouchstart="playTranslateVoice" v-if="recordStatus == 2">
        <play-icon :play-type="playType"></play-icon>
      </view>
    </view>
  </view>
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
import modal from "../modal/index";
import waitingIcon from "../waiting-icon/index";
import playIcon from "../play-icon/index";

export default {
  data() {
    return {
      tips_language: language[0],
      // 目前只有中文
      modalShow: false,
      // 展示悬浮框
      playType: 'wait',
      // 语音播放状态
      waiting_animation: {},
      waiting_animation_1: {},
      edit_icon_path: "/static/image/edit.png"
    };
  },

  components: {
    modal,
    waitingIcon,
    playIcon
  },
  props: {
    /*
    item 格式
    {
      create: '04/27 15:37',
      text: '一二三四五',
      translateText: '12345',
      voicePath: '',
      translateVoicePath: '',
      id: 0,
    },*/
    item: {
      type: Object,
      default: () => ({})
    },
    editShow: {
      type: Boolean,
      default: false
    },
    index: {
      type: Number
    },
    currentTranslateVoice: {
      type: String
    },
    recordStatus: {
      type: Number,
      default: 2 // 0：正在识别，1：正在翻译，2：翻译完成

    }
  },
  watch: {
    item: {
      handler: function (newVal, oldVal) {
        // 翻译完成后，文字有改变触发重新翻译
        if (this.recordStatus == 2 && oldVal.text && oldVal.text != '' && newVal.text != oldVal.text) {
          this.$emit('translate', {
            detail: {
              item: this.data.item,
              index: this.data.index
            }
          });
        } // 翻译内容改变触发播放


        if (newVal.autoPlay && newVal.translateVoicePath != oldVal.translateVoicePath) {
          this.autoPlayTranslateVoice();
        } else if (newVal.translateVoicePath == "") {
          this.playAnimationEnd();
        }
      },
      immediate: true,
      deep: true
    },
    currentTranslateVoice: {
      handler: function (newVal, oldVal) {
        if (newVal != '' && newVal != this.item.translateVoicePath) {
          this.playAnimationEnd();
        }
      },
      immediate: true
    }
  },
  mounted: function () {
    if (this.item.autoPlay) {
      this.autoPlayTranslateVoice();
    }
  },
  // 组件生命周期函数，在组件实例被从页面节点树移除时执行
  destroyed: function () {// console.log("detach")
  },
  methods: {
    /**
     * 显示悬浮框
     */
    showModal: function () {
      this.setData({
        modalShow: true
      });
    },

    /**
     * 离开悬浮框
     */
    modalLeave: function () {
      this.setData({
        modalShow: false
      });
    },

    /**
     * 点击播放图标
     */
    playTranslateVoice: function () {
      let nowTime = parseInt(+new Date() / 1000);
      let voiceExpiredTime = this.item.translateVoiceExpiredTime || 0;

      if (this.playType == 'playing') {
        uni.stopBackgroundAudio();
        this.playAnimationEnd();
      } else if (nowTime < voiceExpiredTime) {
        this.autoPlayTranslateVoice();
      } else {
        this.setData({
          playType: 'loading'
        });
        this.$emit('expired', {
          detail: {
            item: this.data.item,
            index: this.data.index
          }
        });
      }
    },

    /**
     * 播放背景音乐
     */
    autoPlayTranslateVoice: function (path, index) {
      let play_path = this.item.translateVoicePath;

      if (!play_path) {
        console.warn("no translate voice path");
        return;
      }

      uni.onBackgroundAudioStop(res => {
        console.log("play voice end", res);
        this.playAnimationEnd();
      });
      this.playAnimationStart();
      uni.playBackgroundAudio({
        dataUrl: play_path,
        title: '',
        success: res => {
          this.playAnimationStart();
        },
        fail: res => {
          // fail
          console.log("failed played", play_path);
          this.playAnimationEnd();
        },
        complete: function (res) {
          console.log("complete played");
        }
      });
    },

    /**
     * 开始播放
     */
    playAnimationStart: function () {
      this.setData({
        playType: 'playing'
      });
    },

    /**
     * 结束播放
     */
    playAnimationEnd: function () {
      this.setData({
        playType: 'wait'
      });
    }
  }
};
</script>
<style>
@import "./index.css";
</style>