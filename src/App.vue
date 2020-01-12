<template>
  <div id="app">
    <video ref="video"></video>
    <canvas ref="canvas"></canvas>
  </div>
</template>

<script>
import tfjs_yolo from 'tfjs-yolo'
import * as tf from '@tensorflow/tfjs'

export default {
  name: 'app',
  async mounted(){
    await this.videoInit()
    await this.yoloInit()
    await this.loopRun()
    console.log(tf.getBackend()) // eslint-disable-line no-console
    setTimeout(()=>{
      this.loopRun()
    },50)
  },
  data(){
    return {
      yolo : null,
    }
  },
  methods: {
    async videoInit(){
      let constraints = {
        audio: false,
        video: { width: 416, height: 416 }
      };

      let video = this.$refs['video'];

      let mediaStream = await navigator.mediaDevices.getUserMedia(constraints)
      console.log('mediaStream',mediaStream) // eslint-disable-line no-console
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
      console.log({results}) // eslint-disable-line no-console
      const canvas = this.$refs['canvas'];
      const ctx = canvas.getContext('2d');

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
      await this.loopRun()
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
</style>
