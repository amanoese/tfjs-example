<template>
  <div id="app">
    <p>{{message}}</p>
    <div class="contents">
      <video ref="video"></video>
      <canvas ref="canvas"></canvas>
    </div>
  </div>
</template>

<script>
import tfjs_yolo from 'tfjs-yolo'
import * as tf from '@tensorflow/tfjs'

export default {
  name: 'app',
  async mounted(){
    await this.videoInit()

    this.message = 'model download ...'
    await this.yoloInit()
    this.message = 'webgl init ...'
    await this.runYolo(this.$refs['video'])

    this.message = 'now running!'
    console.log(tf.getBackend()) // eslint-disable-line no-console

    this.loopRun()
  },
  data(){
    return {
      yolo : null,
      message : 'ready'
    }
  },
  methods: {
    async videoInit(){
      let constraints = {
        audio: false,
        video: true
      };

      let video = this.$refs['video'];

      let mediaStream = await navigator.mediaDevices.getUserMedia(constraints)
      video.srcObject = mediaStream;
      await new Promise((resolve)=>{
        video.onloadedmetadata = e => {
          resolve(e)
        }
      })
      video.play();
    },
    async yoloInit(){
      this.yolo = await tfjs_yolo.v3tiny();
    },
    async runYolo(htmlElement){
      return await this.yolo.predict(htmlElement)
    },
    renderCanvas(results){
      const video = this.$refs['video'];
      const canvas = this.$refs['canvas'];
      const ctx = canvas.getContext('2d');
      canvas.width = video.videoWidth
      canvas.height = video.videoHeight

      ctx.clearRect(0,0,canvas.width,canvas.height);
      ctx.beginPath();

      results.forEach(r => {
        ctx.lineWidth = 2;
        ctx.fillStyle = "red";
        ctx.strokeStyle = "red";

        ctx.rect(r["left"], r["top"], r["width"], r["height"]);
        ctx.fillText(r["class"], r["left"] + 5, r["top"] + 10);
      });

      ctx.stroke();
    },
    async loopRun(){
      let results = await this.runYolo(this.$refs['video'])
      this.renderCanvas(results)

      setTimeout(()=>{ this.loopRun() },50)
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.contents {
  position: relative;
  display: inline-flex;
}
.contents canvas {
  position: absolute;
  left:0;
  top:0;
}
</style>
