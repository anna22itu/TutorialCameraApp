<template>
  <ion-page>

    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Photo App</ion-title>
      </ion-toolbar>
    </ion-header>    

    <div style="display:flex; justify-content: center;">
      <div style ="display: flex; justify-content: center;">
          <canvas ref="canvasOut" style="border-style: solid; width: 300px; height: 400px;" id="output"></canvas>
      </div>
    </div>
    <div style="background-color: blue; display: none">
      <video ref="myVideo"></video>
      <canvas ref="myCanvas"></canvas>
    </div>
    <div style="display:flex; justify-content: center;">
      <div style ="display: flex; justify-content: center;">
        <ion-button style="border-style: solid; width: 200px; height: 100px;" @click="selectFront" v-if="!front">Back camera</ion-button>
        <ion-button style="border-style: solid; width: 200px; height: 100px;" @click="selectBack" v-if="front">Front camera</ion-button>
      </div>
    </div>
    <div style="display:flex; justify-content: center;">
      <div style ="display: flex; justify-content: center;">
        <ion-button style="border-style: solid; width: 200px; height: 100px;" @click="startCapture" v-if="!capturing">Take picture</ion-button>
      </div>
    </div>
      
  </ion-page>
</template>
  
<script>
import { IonHeader, IonPage, IonTitle, IonToolbar, IonButton } from '@ionic/vue';
import { defineComponent, ref, onMounted } from 'vue';
import { useMQTT } from 'mqtt-vue-hook'
import * as cv from 'opencv.js'
export default defineComponent({
  name: 'HomePage',
  components: { 
    IonHeader, IonToolbar, IonPage, IonTitle, IonButton
  },
  mounted(){
    this.myVideo = this.$refs.myVideo
    this.myCanvas = this.$refs.myCanvas 
    this.capturing = false
    this.canvasOut = this.$refs.canvasOut.getContext('2d');
    this.front = false
  },
  methods:{
    startCapture() {    
      let camera = undefined        
      if (this.front)
        camera = { video: true, audio: false, facingMode: 'environment'}
      else
        camera = { audio: false, video: { facingMode: { exact: "environment" } } }
      //navigator.mediaDevices.getUserMedia({ video: true, audio: false, facingMode: 'environment'}).then(stream => {
      navigator.mediaDevices.getUserMedia(camera).then(stream => {
      //  navigator.mediaDevices.getUserMedia({ audio: false, video: { facingMode: { exact: "environment" } } }  ).then(stream => {
        if(this.myVideo){
          this.myStream = stream;
          this.myVideo.srcObject = this.myStream;
          const playPromise = this.myVideo.play();
          if(playPromise !== undefined){
            setTimeout(() => {
              this.initCanvas();
            }, 500);
          }
        }     
      }).catch(error => {
          console.log(error);
      })
    },
    initCanvas(){
      this.myCanvas.setAttribute('width', 300)
      this.myCanvas.setAttribute('height', 400)
      const context = this.myCanvas.getContext('2d');
      let jpg_as_text;
      let data;
      context.drawImage(this.myVideo, 0,0, 300, 400);
      jpg_as_text = this.myCanvas.toDataURL("image/jpeg").split(';base64,')[1];               
      this.mqttHook.publish("screenshotFrameAnna",jpg_as_text,2);  
      this.showImage(this.myCanvas.toDataURL("image/jpeg")); 
    },
    showImage(image) {   
      const img = new Image();        
      img.src = image;
      let dst;
      img.onload = () => {   
        dst = cv.imread (img);
        cv.imshow ('output',dst);
      }
    },    
    selectFront (){
      this.front = true
    },
    selectBack () {
      this.front = false
    }
  },
  data() {        
    return{
      myVideo: null,
      myCanvas: null,            
      myStream: null,
      interval: null,
      capturing: null,
      canvasOut: null,       
      front: null
    }
  }, 
  setup () {
    const mqttHook = useMQTT(); 
    return {
      mqttHook
    };
  }
})
</script>

<style scoped>
#container {
  text-align: center;
  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;
  color: #8c8c8c;
  margin: 0;
}

#container a {
  text-decoration: none;
}
</style>
