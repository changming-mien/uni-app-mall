<script>
export default {
  onLaunch: function () {
    console.log("App Launch");
  },
  onShow: function () {
    console.log("App Show");
  },
  onHide: function () {
    console.log("App Hide");
  },
};
</script>

<style>
/*每个页面公共css */
/* 全局引入图标 */
@import "/static/iconfont/iconfont.css";
page {
  position: relative;
  background-color: #ffffff;
}
</style>
