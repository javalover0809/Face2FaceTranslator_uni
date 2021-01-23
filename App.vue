<script>
//app.js
const utils = require("./utils/util.js");

export default {
  onLaunch: function () {
    uni.getStorage({
      key: 'history',
      success: res => {
        this.globalData.history = res.data;
      },
      fail: res => {
        console.log("get storage failed");
        console.log(res);
        this.globalData.history = [];
      }
    });
  },
  globalData: {
    // 权限询问
    getRecordAuth: function () {
      uni.getSetting({
        success(res) {
          console.log("succ");
          console.log(res);

          if (!res.authSetting['scope.record']) {
            uni.authorize({
              scope: 'scope.record',

              success() {
                // 用户已经同意小程序使用录音功能，后续调用 wx.startRecord 接口不会弹窗询问
                console.log("succ auth");
              },

              fail() {
                console.log("fail auth");
              }

            });
          } else {
            console.log("record has been authed");
          }
        },

        fail(res) {
          console.log("fail");
          console.log(res);
        }

      });
    },
    history: []
  },
  onHide: function () {
    uni.stopBackgroundAudio();
  },
  methods: {}
};
</script>
<style>
@import "./app.css";
</style>