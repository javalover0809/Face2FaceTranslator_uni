<template>
<view class="button-wrap" :hidden="hidden">
  <view class="img-big-wrap">
    <view class="button-container">
      <view v-for="(button, index) in buttons" :key="index" class="button-item">
		<view @touchstart="streamRecord" @touchend="endStreamRecord" :data-conf="button" class="button-press">
          <span :class="'text-in-button ' + ( button.buttonType == 'press' ? 'text-press': '' )">{{button.buttonText}}</span>
          <image class="button-background" src="http://www.wetalk.ltd/Users/Oraida/Downloads/test/test/ns2.jpeg"></image>
        </view>
        <view class="button-label">{{button.msg}}</view>
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
let buttons = []; // 按钮配置
// 按钮配置
language.forEach(item => {
  buttons.push({
    buttonText: item.lang_name,
    lang: item.lang_content,
    lto: item.lang_to[0],
    msg: item.hold_talk,
    buttonType: 'normal'
  });
}); // 按钮对应图片
// 按钮对应图片
let buttonBackground = {
  zh_CN: {
    normal: '../../image/button_zh.png',
    press: '../../image/button_zh_press.png',
    disabled: '../../image/button_zh_disabled.png'
  },
  en_US: {
    normal: '../../image/button_en.png',
    press: '../../image/button_en_press.png',
    disabled: '../../image/button_en_disabled.png'
  }
};

export default {
  data() {
    return {
      buttons: buttons,
      buttonBackground: buttonBackground,
      currentButtonType: 'normal'
    };
  },

  components: {},
  props: {
    buttonDisabled: {
      type: Boolean,
      default: false
    }
  },
  watch: {
    buttonDisabled: {
      handler: function (newVal, oldVal) {
        let buttonType = newVal ? 'disabled' : 'normal';
        this.changeButtonType(buttonType);
      },
      immediate: true
    }
  },
  mounted: function () {// console.log(this.data.buttonEvent,)
  },
  methods: {
    /**
     * 按下按钮开始录音
     */
    streamRecord(e) {
      if (this.buttonDisabled) {
        return;
      } // 先清空背景音


      uni.stopBackgroundAudio();
      let currentButtonConf = e.currentTarget.dataset.conf;
      this.changeButtonType('press', currentButtonConf.lang);
      this.$emit('recordstart', {
        detail: {
          buttonItem: currentButtonConf
        }
      });
    },

    /**
     * 松开按钮结束录音
     */
    endStreamRecord(e) {
	  console.log("停止语音输入")
      let currentButtonConf = e.currentTarget.dataset.conf;
      console.log("currentButtonConf", currentButtonConf);
      this.$emit('recordend', {
        detail: {
          buttonItem: currentButtonConf
        }
      });
    },

    /**
     * 修改按钮样式
     */
    changeButtonType(buttonType, buttonLang) {
      let tmpButtons = this.buttons.slice(0);
      tmpButtons.forEach(button => {
        if (!buttonLang || buttonLang == button.lang) {
          button.buttonType = buttonType;
        }
      });
      this.setData({
        buttons: tmpButtons
      });
    }

  }
};
</script>
<style>
@import "./index.css";
</style>