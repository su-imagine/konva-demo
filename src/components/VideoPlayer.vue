<template>
    <div style="height:100%;display:flex;justify-content:center;align-items:center;">
        <div class="video-wrapper">
           <video ref="videoPlayer" style="height:100%;" class="video-js"></video>
        </div>
    </div>
</template>

<script>
import videojs from 'video.js';
import'video.js/dist/video-js.css';


export default {
    name: "VideoPlayer",
    props: {
        options: {
            type: Object,
            default() {
                return {};
            }
        }
    },
    watch: {
      options(val) {
          if(this.player) {
            this.player.src(this.options.sources[0].src); 
            console.log(this.options.sources[0].src);
          }
      }
    },
    data() {
        return {
            player: null
        }
    },
    mounted() {
        this.player = videojs(this.$refs.videoPlayer, this.options, function onPlayerReady() {});
    },
    beforeDestroy() {
        if (this.player) {
            this.player.dispose()
        }
    }
}
</script>

 <!-- <style scoped src='../assets/styles/video-js.css'></style> -->

  <style scoped>
    .video-wrapper{
      width:100%;
      height:100%;
    }
  </style>