<template>
  <div style="overflow: hidden">
    <div class="video-box">
      <video
        id="video"
        width="320"
        height="240"
        preload
        autoplay
        loop
        muted
      ></video>
      <canvas id="canvas" width="320" height="240"></canvas>
    </div>
    <canvas id="screenshotCanvas" width="320" height="240"></canvas>
    <div class="switch-button">
      <span type="primary" @click="destroyed">关闭摄像头</span>
      <span type="primary" @click="init">开始识别</span>
    </div>
  </div>
</template>

<script>
import tracking from '@/assets/tracking/build/tracking-min.js';
import '@/assets/tracking/build/data/face-min.js';
export default {
  props: {},
  data() {
    return {
      trackerTask: null,
      mediaStreamTrack: null,
      video: null,
      screenshotCanvas: null,
      uploadLock: true, // 上传锁
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      this.video = this.mediaStreamTrack = document.getElementById("video");
      this.screenshotCanvas = document.getElementById("screenshotCanvas");

      let canvas = document.getElementById("canvas");
      let context = canvas.getContext("2d");

      // 固定写法
      let tracker = new window.tracking.ObjectTracker("face");
      tracker.setInitialScale(4);
      tracker.setStepSize(2);
      tracker.setEdgesDensity(0.1);
      //摄像头初始化
      this.trackerTask = window.tracking.track("#video", tracker, {
        camera: true,
      });

      let _this = this;
      tracker.on("track", function (event) {
        // 检测出人脸 绘画人脸位置
        context.clearRect(0, 0, canvas.width, canvas.height);
        event.data.forEach(function (rect) {
          context.strokeStyle = "#0764B7";
          context.strokeRect(rect.x, rect.y, rect.width, rect.height);
        });
        // event.data.length长度为多少代表检测几张人脸
        if (_this.uploadLock && event.data.length) {
          //上传图片
          _this.screenshotAndUpload();
        }
      });
    },
    // 上传图片
    screenshotAndUpload() {
      // 上锁避免重复发送请求
      this.uploadLock = false;

      // 绘制当前帧图片转换为base64格式
      let canvas = this.screenshotCanvas;
      let video = this.video;
      let ctx = canvas.getContext("2d");
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
      let base64Img = canvas.toDataURL("image/jpeg");

      // 打印出 base64Img
      console.log("base64Img:", base64Img);

      // 请求接口成功以后打开锁
      // this.uploadLock = true;
    },
    //关闭摄像头
    destroyed() {
      if (!this.mediaStreamTrack) {
        return;
      }
      this.mediaStreamTrack.srcObject.getTracks()[0].stop();
      this.trackerTask.stop();
    },
  },
  components: {},
};
</script>

<style scoped lang="scss">
/* 绘图canvas 不需显示隐藏即可 */
#screenshotCanvas {
  display: none;
}

.video-box {
  position: relative;
  // margin-left: 30px;
  width: 330px;
  height: 250px;
  margin: 0 auto;
  margin-top: 50px;
}

.switch-button {
  margin-top: 30px;
}
video,
canvas {
  position: absolute;
  top: 0;
  left: 0;
  border: #000000 5px solid;
}

.switch-button {
  display: flex;
  align-content: center;
  justify-content: center;

  span {
    background: aqua;
    color: #fff;
    border-radius: 8px;
    display: flex;
    align-content: center;
    justify-content: center;
    padding: 5px 10px;
    &:not(:first-child) {
      margin-left: 15px;
    }
  }
}
</style>
