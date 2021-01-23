<template>
<!--pages/edit/edit.wxml-->
<view class="container edit-container">
  <textarea :maxlength="edit_text_max" class="edit_textarea" auto-focus="true" :focus="is_focus" @input="editInput" @confirm="editConfirm" :value="edit_text" adjust-position="true" @focus="editFocus" @blur="editBlur"></textarea>
  <view class="bottom-wrap" :style="'padding-bottom: ' + bottomHeight + 'px'">
    <view class="tips-wrapper">
      <text class="edit-tips">{{tips}}</text>
      <view class="delete-content" @tap="deleteContent">
        <image src="/static/image/delete_all.png" class="img-delete-all">
        </image>
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
const initBottomHeight = 0;
var app = getApp();

export default {
  data() {
    return {
      edit_text_max: 200,
      remain_length: 200,
      edit_text: "",
      is_focus: false,
      tips: "",
      index: -1,
      bottomHeight: initBottomHeight,
      dialogList: "",
      recordStatus: 0,
      oldText: ""
    };
  },

  components: {},
  props: {},

  /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function (options) {
    this.setEditText(options.content);
    let index = parseInt(options.index);
    this.setData({
      index: index,
      oldText: options.content
    });
  },
  methods: {
    /**
     * 设置初始内容
     */
    setEditText: function (text) {
      this.edit_text = text;
      this.setData({
        edit_text: this.edit_text
      });
      this.setData({
        is_focus: true
      });
    },

    /**
     * bindconfirm
     */
    editConfirm: function (event) {
      if (this.edit_text.length > 0 && this.edit_text != this.oldText) {
        // 得到页面栈
        let pages = getCurrentPages();
        let prevPage = pages[pages.length - 2]; //上一个页面

        let dialogList = prevPage.data.dialogList.slice(0);
        let editItem = dialogList[dialogList.length - 1];
        editItem.text = this.edit_text;
        prevPage.setData({
          dialogList: dialogList,
          recordStatus: 2
        });
        uni.navigateBack();
      } else {//文本输入为空时提示
      }
    },

    /**
     * 点击文本输入框修改底下按钮行高，让提示和按钮始终在键盘上方   *
     */
    editFocus: function (e) {
      let {
        value,
        height
      } = e.detail;
      console.log(value, height);

      if (!isNaN(height)) {
        this.setData({
          bottomHeight: height + initBottomHeight
        });
      }
    },

    /**
     * 输入框失焦
     */
    editBlur: function () {
      this.setData({
        bottomHeight: initBottomHeight
      });
    },

    /**
     * 清空内容
     */
    deleteContent: function () {
      this.setEditText("");
      this.setData({
        is_focus: true
      });
    }
  }
};
</script>
<style>
@import "./edit.css";
</style>