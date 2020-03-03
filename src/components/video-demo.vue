<template>
	<div class="wrapper">
		<div class="box" @click="clickVideo"> 
			<!-- 仍然是容器，带黑框 -->
			<video-player :options="options">
				
			</video-player>
		</div>
		

	</div>
</template>

<style scoped>
	.box{
		width: 1000px;
		height: 800px;
		background-color: orange;
		/*transform: scale(0.5);*/
		/*overflow: hidden;*/
	}
</style>

<script>
	require('videojs-flash');
	import VideoPlayer from './VideoPlayer.vue'
	export default {
		components:{VideoPlayer},
		data(){
			return {
				options:{
					autoplay: true,
			        controls: false,
			        preload: 'auto',
			        width: 722,
			        height: 592,
			        notSupportedMessage: '此视频暂无法播放，请稍后再试', // 允许覆盖Video.js无法播放媒体源时显示的默认信息。
			        // techOrder: ['html5', 'flash'],
			        notSupportedMessage: true,
			        sources: [
			          {
			            // src: 'rtmp://pili-publish.hls.xlink.cn/bgycctonghuhls/2002',
			            src:'rtmp://pili-publish.hls.xlink.cn/bgycctonghuhls/1995',
			            type: "rtmp/flv"
			          }
			        ]
				}
			}
		},
		computed:{

		},
		methods:{
			clickVideo(e){
				console.log(e)
				console.log('click new click')
			}
		},
		mounted(){

		},
		created(){

		}
	}
</script>