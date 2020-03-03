<template>
	<div class="add-content">
		<div class="body-content">
			<!-- 左侧视频区域 -->
			<div class="content-left">
				<div ref="operate" class="operate-wrapper" :style="{height:stageHeight + 'px',width:stageWidth + 'px'}">
					<div style="width: 100%;height: 100%;">
						<video-player :options="videoOptions" class="video" ref="videoWrapper" v-if="videoSrc && showVideo" />
						<div v-else class="no-video">暂无视频源</div>
                    </div>
					<div id="stage" ref="stage" class="konva">
					</div>
				</div>
			</div>
			<!-- 右边设置区域 -->
			<div class="content-right">
				<!-- <span class="tip">请在画面中框选出需要重点识别的区域</span> -->
				<button class="setting-item" @click="resetDraw">重置</button>
				<button class="setting-item" @click="saveTask">保存</button>
				<select class="setting-item" v-model="typeSelected" @change="changeType">
					<option value="rect">矩形</option>
					<option value="polygon">多边形</option>
				</select>
			</div>	
		</div>
	</div>
</template>


<script>
    require('videojs-flash');
	import VideoPlayer from "./VideoPlayer.vue";
	import Konva from 'konva'

    export default {
        components: {
			VideoPlayer
        },
        props: {
            // dialogVisible: {
            //     type: Boolean,
            //     default: false
            // },
            algoName: {
                type: String,
                default: ''
			},
			projectId:{
				type:Number,
				default:0
			}
        },
        
        data () {
            return {
				task:{
					"algoName": "garbageExposure",
					"algoParam": {
						"alertDuration": 1200,
						"detectArea": [
							{
								"areaDirection": 1,
								"areaIndex": 1,
								"areaPoint": [
									{
										"point": [
											0.3851,
											0.6051
										]
									},
									{
										"point": [
											0.6655,
											0.6051
										]
									},
									{
										"point": [
											0.6655,
											0.9339
										]
									},
									{
										"point": [
											0.3851,
											0.9339
										]
									}
								],
								"areaType": 1,
								"targetType": 1
							},
							{
								"areaDirection": 1,
								"areaIndex": 2,
								"areaPoint": [
									{
										"point": [
											0.4375,
											0.6772
										]
									},
									{
										"point": [
											0.6258,
											0.6772
										]
									},
									{
										"point": [
											0.6258,
											0.8619
										]
									},
									{
										"point": [
											0.4375,
											0.8619
										]
									}
								],
								"areaType": 1,
								"targetType": 1
							}
						],
						"firstDuration": 1200,
						"inferenceDuration": 0
					},
					"createTime": "Wed, 15 Jan 2020 01:29:17 GMT",
					"deviceId": 138,
					"jobDesc": "",
					"jobId": 327,
					"jobName": "幼儿园游乐场1垃圾检测",
					"name": "幼儿园游乐场1",
					"status": 0,
					"updateTime": "Wed, 15 Jan 2020 01:29:17 GMT"
				},
				stage: null,
                formData: {
                    deviceId: '',
                    algoName: '',
                    name: '',
                    jobName: '',
                    jobDesc: '',
                    firstDuration: 60,
                    alertDuration: 3600,
                    algoParam: {}
                },
                videoSrc: 'rtmp://pili-publish.hls.xlink.cn/bgycctonghuhls/2800',
				typeSelected:'rect',
				drawTypeList:[
					{label:'矩形',value:'rect'},
					{label:'多边形',value:'polygon'}
				],
				detectArea:[],
				shapeConfig:{
					shapeArr:[]
				},
				polygonArr:[],
				circleArr:[],
				targetIndex:'',
				clickTag:0,
				dbCickFlag:false,
				startDrawFlag:false,
				polygonType:1,
				movePath:[],
				showVideo:false,
				stageHeight:'',
				stageWidth:'',
				topGap:20,
				leftGap:20,

			}
        },

        computed: {
			overShapeLimit(){
				return !(this.shapeConfig.shapeArr.length < 2)
			},
			videoOptions() {
                return {
                    autoplay: true,
                    controls: false,
                    width:this.stageWidth,
                    height:this.stageHeight,
                    sources: [
                      {
                        src: this.videoSrc,
                        type: "rtmp/flv"
                      }
                    ]
                }
            }
		},
		watch: {
            // dialogVisible(val){
			// 	if(val){
			// 		this.$nextTick(() => {
			// 			this.getStageHeight()
			// 			this.stageInit()
			// 		})
			// 		this.init()
			// 	}
			// }, 
       	},
        methods: {
			/**
			 * 
			 */
			changeType(val){
				this.polygonType = this.typeSelected == 'rect' ? 1 : 2
				if(this.polygonType === 1){
					this.startDrawRect()
				}else{
					this.startDrawPolygon()
				}
			},
			/**
			 * 舞台初始化
			 */
			stageInit(){
				this.stage = new Konva.Stage({
					container:'stage',
					width:this.stageWidth,
					height:this.stageHeight
				})
				this.shapeLayer = new Konva.Layer({
					x:0,
					y:0,
					width:this.stageWidth,
					height:this.stageHeight
				})
				this.stage.add(this.shapeLayer)
				this.shapeLayer.setZIndex(1)
				
			},
			/**
			 * 绑定画多边形的事件
			 */
			startDrawPolygon(){
				console.log('开始画多边形')
				this.polygonIndex = this.shapeConfig.shapeArr.length
				if(!this.shapeConfig.shapeArr[this.polygonIndex]){
					this.shapeConfig.shapeArr[this.polygonIndex] = {}
					this.shapeConfig.shapeArr[this.polygonIndex].polygonType = 2
				}
				let movePath = []
				this.resetEventListener()

				this.stage.on('click', e => {
					if(!this.startDrawFlag){
						this.startDrawFlag = true
					}
					const ev = e.evt
					const x = ev.clientX - this.leftGap
					const y = ev.clientY  - this.topGap // 减去 tab 栏的高度
					this.targetIndex = this.polygonIndex

					if(this.clickTag === 0){
						this.clickTag = 1
						movePath.push([x,y])
						setTimeout(() => {
							this.clickTag = 0
						},500)
					}else{
						this.dbCickFlag = true
						this.startDrawFlag = false
					}
					this.shapeConfig.shapeArr[this.polygonIndex].path = movePath
					if(!this.shapeConfig.endX){
						return
					}
					this.drawFence()
					if(this.dbCickFlag){
						this.dbCickFlag = false
						this.shapeConfig.endX = 0
						this.shapeConfig.endY = 0
						this.resetEventListener()
						this.addAdjustListener()
						this.addStageMoveHandler()
					}
				})

				this.stage.on('mousemove',e => {
					console.log(this.startDrawFlag)
					if(!this.startDrawFlag){
						return
					}
					const ev = e.evt
					this.shapeConfig.endX = ev.clientX - this.leftGap
					this.shapeConfig.endY = ev.clientY - this.topGap

					this.drawFence(this.shapeConfig.path)
				})



				// console.log('画多边形')
				// if(this.dbCickFlag){
				// 	let path = this.shapeConfig.path[1] || []
				// 	this.dbCickFlag = false
				// }else{
				// 	let path = this.shapeConfig.path[0] || []
				// }
				// this.resetEventListener()
				// this.stage.on('click',e => {
				// 	const ev = e.evt
				// 	const x = ev.clientX - this.leftGap
				// 	const y = ev.clientY - 50 - this.topGap // 减去 tab 栏的高度
				// 	this.targetIndex = 0
				// 判断是否在可操作的区域内
				// if(){
					// todo
				// }
				// 	// 点击过快则为双击
				// 	if(this.clickTag === 0) {
				// 		this.clickTag = 1
				// 		path.push([x,y])
				// 		setTimeout(() => {
				// 			this.clickTag = 0
				// 		},500)
				// 	}else{
				// 		this.dbCickFlag = true
				// 	}

				// 	// this.shapeConfig.path[0] = path
				// 	this.drawArea()
				// })
				// this.stage.on('mousemove',e => {
				// 	const ev = e.evt
				// 	const x = ev.clientX - this.leftGap
				// 	const y = ev.clientY - this.topGap
				// 	this.shapeConfig.endX = x
				// 	this.shapeConfig.endY = y - 50
				// 	this.drawArea()
				// })
			},

			/**
			 * 绑定画矩形的事件
			 */
			startDrawRect(flag = 0){
				let serial = this.shapeConfig.shapeArr.length
				if(!this.shapeConfig.shapeArr[serial]){
					this.shapeConfig.shapeArr[serial] = {}
					this.shapeConfig.shapeArr[serial].polygonType = 1
				}
				let x1 = '',x2 = '',y1 = '',y2 = ''
				let movePath = []
				this.resetEventListener()
				this.stage.on('mousemove',e => {
					if(this.clickTag === 0 && flag === 0){
						return
					}
					const ev = e.evt
					x2 = ev.clientX - this.leftGap
					y2 = ev.clientY - this.topGap
					movePath = [[x1,y1],[x2,y1],[x2,y2],[x1,y2]]
					this.shapeConfig.shapeArr[serial].path = movePath
					this.drawFence()
				})
				this.stage.on('click',e => {
					const ev = e.evt
					if(this.clickTag === 1){
						this.clickTag = 0
						x2 = ev.clientX - this.leftGap
						y2 = ev.clientY - this.topGap
						movePath = [[x1,y1],[x2,y1],[x2,y2],[x1,y2]]
						this.shapeConfig.shapeArr[serial].path = movePath
						console.log(this.shapeConfig.shapeArr[serial].path)
						this.drawFence()
						this.resetEventListener()
						this.addAdjustListener()
						this.addStageMoveHandler()
					}else{
						this.targetIndex = serial
						this.clickTag = 1
						x1 = ev.clientX - this.leftGap
						y1 = ev.clientY - this.topGap // 减去 tab 栏的高度
					}
					
				})
				
			},

			/**
			 * 重置,将舞台初始化
			 */
			resetDraw(){
				this.polygonType = this.typeSelected == 'rect' ? 1 : 2
				this.dbCickFlag = false
				this.clickTag = 0
				this.targetIndex = ''
				this.resetEventListener()
				this.shapeLayer.remove()
				this.stage.remove()
				this.shapeConfig.shapeArr = []
				this.shapeConfig.endX = ''
				this.shapeConfig.endY = ''
				this.stageInit()
				
				if(this.polygonType == 1){
					this.startDrawRect()
				}else{
					this.startDrawPolygon()
				}
			},
			/**
			 * stage事件重置
			 */
			resetEventListener(){
				this.stage.removeEventListener('click')
				this.stage.removeEventListener('mousemove')
				this.stage.removeEventListener('mouseup')
				this.stage.removeEventListener('mousedown')
			},
			/**
			 * 手动描绘区域
			 */
			drawArea(){
				// console.log(this.targetIndex == this.polygonIndex)
				const shapeConfig = this.shapeConfig
				let polygonPath = shapeConfig.polygonArr[this.polygonIndex].path
				let tempPath = []
				let path = []
				tempPath = [...polygonPath,[shapeConfig.endX,shapeConfig.endY]]
				tempPath.map(item => {
					if(path.length === 0){
						path.push(item)
					}else if(!path.find(p => p[0] === item[0] && p[1] === item[1])){  // 没有去重
						path.push(item)
					}
				})
				// path.push(path[0])  // 首尾相连
				// 画端点
				let dotArr = []
				path.map((item,index) => {
					if(path.length < 2 ){
						return
					}
					let dot = this.drawCircle(item[0],item[1],this.polygonIndex)
					this.circleArr.push(dot)
				})
				let ploygon = this.drawPolygon(path)
				this.shapeLayer.destroyChildren()
				this.shapeLayer.add(ploygon)
				dotArr.map(dot => this.shapeLayer.add(dot))
				this.shapeLayer.batchDraw()

				// 结束
				if(this.dbCickFlag){
					this.dbCickFlag = false
					this.resetEventListener()
					this.addAdjustListener()
					this.addStageMoveHandler()
				}
			},

			/**
			 * 进行微调的事件绑定
			 */
			addAdjustListener(){
				this.stage.on('mouseup',e => {
					this.stage.removeEventListener('mousemove')
					this.addStageMoveHandler()
				})
				this.stage.on('mousedown',e => {
					this.stage.removeEventListener('mousemove')
					let target = e.target
					console.log(target)
					console.log(this.polygonType)
					if(target.attrs.name === 'circle'){
						if(target.attrs.polygonType == 1){
							this.adjustRect(target)
						}
						if(target.attrs.polygonType == 2){
							this.adjustPolygon(target)
						}
					}

					if(target.attrs.name === 'polygon'){
						// this.startDrawRect(1)
						if(this.overShapeLimit){
							return
						}
						if(this.polygonType == 1){
							console.log('画矩形')
							this.startDrawRect(1)
						}else{
							console.log('画多边形')
							this.startDrawPolygon()
						}
						// console.log(this.clickTag)
						// if(!this.clickTag){
						// 	this.clickTag = 1
						// 	setTimeout(() => {
						// 		this.clickTag = 0
						// 	},500)
						// }else{
						// 	this.dbCickFlag = true
						// }

						// // 双击画图，单击调整位置
						// if(this.dbCickFlag){
						// 	if(this.polygonType == 1){
						// 		this.startDrawRect(1)
						// 	}else{
						// 		this.startDrawPolygon()
						// 	}
						// }else{
						// 	let startX = e.evt.clientX
						// 	let startY = e.evt.clientY
						// 	this.movePolygon(target,startX,startY)
						// }
					}

					if(!target.attrs.name){  // 点击区域之外画图
						// if(this.polygonType == 1){
						// 	this.startDrawRect(1)
						// }else{
						// 	console.log('画多边形')
						// }
					}
				
				})
			},

			movePolygon(target,startX,startY){
				let serial = target.attrs.serial
				this.stage.on('mousemove',e => {
					let ev = e.evt
					let moveX = ev.clientX - startX
					let moveY = ev.clientY - startY

					startX = e.evt.clientX
					startY = e.evt.clientY
					
					let movePath = this.shapeConfig.polygonArr[serial].path.map(item => {
						let path = []
						path[0] = item[0] + moveX
						path[1] = item[1] + moveY
						return path
					})
					this.shapeConfig.shapeArr[serial].path = movePath
					
					this.drawFence()
				})
			},

			adjustRect(target){
				let serial = target.attrs.serial
				let x0,y0,x1,y1
				let circleArr = this.stage.find('Circle')
				let ind = circleArr.findIndex((circle,index) => {
					return circle._id === target._id
				})
				ind = ind % 4
				let movePath = this.shapeConfig.shapeArr[serial].path
				x0 = movePath[0][0]
				y0 = movePath[0][1]
				x1 = movePath[2][0]
				y1 = movePath[2][1]
				this.stage.on('mousemove',e => {
					let ev = e.evt
					let x = ev.clientX - this.leftGap
					let y = ev.clientY - this.topGap
					if(ind == 0){
						x0 = x
						y0 = y
					}
					if(ind == 1){
						x1 = x
						y0 = y
					}
					if(ind == 2){
						x1 = x
						y1 = y
					}
					if(ind == 3){
						x0 = x
						y1 = y
					}
					movePath = [[x0,y0],[x1,y0],[x1,y1],[x0,y1]]
					this.shapeConfig.shapeArr[serial].path = movePath
					this.drawFence()
				})
			},
			adjustPolygon(target){
				let serial = target.attrs.serial
				let circleArr = this.stage.find('Circle').filter(circle => {
					return circle.attrs.serial === serial
				})
				let ind = circleArr.findIndex((circle,index) => {
					return circle._id === target._id
				})
				console.log(ind)
				this.stage.on('mousemove',e => {
					let ev = e.evt
					let x = ev.clientX - this.leftGap
					let y = ev.clientY - 50 - this.topGap
					this.shapeConfig.shapeArr[serial].path[ind] = [x,y]
					this.drawFence()
				})
			},			
			
			addStageMoveHandler(){
				this.stage.on('mousemove',e => {
					let targetName = e.target.attrs.name
					if(targetName !== 'circle' && targetName !== 'polygon'){
						this.targetIndex = ''
						this.drawFence()

					}else{
						let index = e.target.attrs.serial
						if(index !== this.targetIndex){
							this.targetIndex = index
							this.drawFence()
						}else{

						}
					}
				})
			},
			
            init() {
				let operateWrapper = this.$refs.operate
				// this.videoWidth = this.stageWidth
				// this.videoHeight = this.stageHeight
				this.initRegion()  // 初始化界面
				this.showVideo = true
			},

			/**
			 * 初始化的时候去画矩形和多边形
			 */
			initRegion(){
				let algoParam = this.task.algoParam || null
				if(!algoParam){  // 没有数据,进行事件的绑定
					this.resetDraw()
					return 
				}
				this.detectArea = algoParam.detectArea
				for(let i=0;i<algoParam.detectArea.length;i++){
					let area = algoParam.detectArea[i]
					if(!area.areaPoint){
						continue
					}
					let shape = {}
					let tempArr = []
					// this.polygonType = area.targetType    // targetType  1是矩形   2是多边形
					shape.polygonType = area.areaType

					area.areaPoint.forEach(point => {
						let x = parseFloat((point.point[0] * this.stageWidth).toFixed(5))
						let y = parseFloat((point.point[1]*this.stageHeight).toFixed(5))
						tempArr.push([x,y])
					})
					shape.path = tempArr
					this.shapeConfig.shapeArr.push(shape)
				}

				console.log('----------------------'+'shapeConfig的path初始化为')
				console.log(this.shapeConfig.shapeArr)
				if(this.shapeConfig.shapeArr.length == 0){
					this.resetDraw()
					return
				}

				this.drawFence()
				this.addAdjustListener()
				this.addStageMoveHandler()
			},

			// 
			drawFence(){
				let shapeArr = this.shapeConfig.shapeArr
				let path = []
				shapeArr.map((shape,ind) => {
					let pathArr = []
					shape.path = shape.path || []
					shape.path.map(item => {
						if(pathArr.length === 0){
							pathArr.push(item)
						}else if(!pathArr.find(p => p[0] === item[0] && p[1] === item[1])){  // 没有去重
							pathArr.push(item)
						}
					})
					if(ind == shapeArr.length -1 && this.startDrawFlag){
						let movePoint = [this.shapeConfig.endX,this.shapeConfig.endY]
						pathArr.push(movePoint)
					}
					path.push(pathArr)
				})
				// 画端点
				let dotArr = []
				// console.log(shapeArr)
				path.map((areaPath,index) => {
					areaPath.map(item => {
						let dot = this.drawCircle(parseFloat(item[0]),parseFloat(item[1]),index,this.shapeConfig.shapeArr[index].polygonType)
						dotArr.push(dot)
						this.circleArr.push(dot)
					})
				})

				// 画区域
				let areaArr = []
				path.map((areaPath,index) => {
					let area = this.drawPolygon(areaPath,index)
					areaArr.push(area)
					this.polygonArr.push(area)
				})

				this.shapeLayer.destroyChildren()
				areaArr.map(area => this.shapeLayer.add(area))
				dotArr.map(dot => this.shapeLayer.add(dot))
				this.shapeLayer.batchDraw()

			},

			/**
			 * 画端点
			 */
			drawCircle(x,y,index=0,polygonType){
				let dot = new Konva.Circle({
					x: x,
					y: y,
					radius: 8,
					fill: 'rgb(255,255,255)',
					stroke: 'green',
					strokeWidth: 1,
					// opacity:this.circleOpacity,
					opacity:this.targetIndex === index ? 1 : 0,
					name:'circle',
					serial:index,
					polygonType:polygonType
				})
				return dot
			},

			/**
			 * 传入一个二维数组，画多边形
			 */
			drawPolygon(path,index = 0){
				let polygon = new Konva.Shape({
					sceneFunc:(context,shape) => {
						context.beginPath()
						path.map((item,index) => {
							if(index === 0){
								context.moveTo(item[0],item[1])
							}else{
								context.lineTo(item[0],item[1])
							}
						})
						context.closePath()
						context.fillStrokeShape(shape)
					},
					fill: 'rgba(0, 132, 255, 0.2)',
					stroke: '#4C98FF',
					strokeWidth: 2,
					name:'polygon',
					serial:index
				})
				return polygon
			},
			
			
			/**
			 * 保存编辑的区域
			 */
			saveTask(){
				let shapeArr = this.shapeConfig.shapeArr
				this.addPolygon(shapeArr)
				let {algoName,algoParam,jobDesc,jobId,jobName,name,deviceId} = this.task
				if(!algoParam){
					algoParam = {}
				}
				algoParam.detectArea = this.detectArea
				let params = {
					algoName,
					algoParam,
					jobDesc,
					jobId,
					jobName,
					name,
					deviceId,
					projectId:this.projectId
				}

				console.log(params)
				
			},
			arrTrans(num,arr){
				let resArr = []
				arr.forEach((item,index) => {
					let page = Math.floor(index/num)
					item = index%2 == 0 ? (item/this.stageWidth).toFixed(4) : (item/this.stageHeight).toFixed(4) 
					if(!resArr[page]){
						resArr[page] = []
					}
					resArr[page].push(parseFloat(item))
				})
				return resArr
			},
			/**
			 * 添加图形的点位（相对点位）
			 */
			addPolygon(shapeArr){
				console.log('保存的polygon数据是多少')
				console.log(shapeArr)
				this.detectArea = []
				shapeArr.map((shape,ind) => {
					let polygon = {}
					polygon.areaDirection = 2
					polygon.areaIndex = ind + 1
					polygon.areaType = shape.polygonType
					polygon.targetType = shape.polygonType
					let areaPoint = []
					shape.path.map(endPoint => {
						let tempArr = this.arrTrans(2,endPoint)  // 除以宽高
						tempArr.forEach(item => {
							let obj = {}
							obj.point = item
							areaPoint.push(obj)
						})
					})
					
					polygon.areaPoint = areaPoint
					this.detectArea.push(polygon)

				})
			},      
            close () {
				this.videoSrc = ''
				this.shapeConfig.shapeArr = []
                this.$emit('close')
			},
			getStageHeight(){
				let h = document.body.clientHeight || document.documentElement.clientHeight
				this.stageHeight = h - 50 - 20 - 50
				this.stageWidth = parseInt(this.stageHeight * 1.235)  // 计算得出的一个视频宽高比
				console.log('height-------------')
				console.log(this.stageHeight)
			},
		},
		
       	mounted() {
			this.$nextTick(() => {
				this.getStageHeight()
				this.stageInit()
				this.init()
			})
		},
        created () { 
        }
    }
</script>
<style>
  .video-js .vjs-big-play-button{
    position: absolute;
    top:50%;
    left: 50%;
    transform: translate(-50%,-50%);
  }
</style>
<style scoped>

.setting-item{
	width: 150px;
	margin:15px;
}
.body-content {
   display: flex;
   width: 100%;
   height: calc(100vh - 50px) ;
   flex: 1;
   /* margin: 0px 0px 0px 0 !important; */
   justify-content: center;
   align-items: center;
   /* flex-direction: column; */
}
.content-left{
	display: flex;
	/* justify-content: center; */
	/* align-items: center; */
   	/* background-color: #222; */
	height: 100%;
	flex: 1;
}
.content-right{
	display: flex;
	flex-direction: column;
	align-items: center;
	min-width: 250px;
	height: 100%;
	/* background-color: #222; */
}

  .add-content {
    overflow-y: auto;
  }

  .operate-wrapper {
      position: relative;
	  margin-top: 20px;
	  margin-left: 20px;
      width: 100%;
	  height:100%;
      display: flex;
  }

  .konva {
    position: absolute;
    left: 0;
    top:0; 
	width: 100%;
	height: 100%;
	background-color: transparent;
	overflow: hidden;
	z-index: 1;
	/* cursor: crosshair; */
}
  .title-content .main {
    background-color: #0092fe;
    color: #fff;
    height: 30px;
    line-height: 30px;
    padding: 10px 20px;
    overflow: hidden;
  }
  .title-content .el-icon-close {
    float: right;
    line-height: 30px;
    cursor: pointer;
  }

  .bottons {
     height: 50px;
     display: flex;
     justify-content: center;
     align-items: center;
     text-align: center
  }
  

  .video {
     display: block;
	 /* background-color: ; */
  }
  .video-wrapper{
	
  }

  .tip {
      padding-bottom: 10px;
      padding-left: 20px;
     color: #C0C5D2;
     font-size: 12px;
     text-align: left;
     display: inline-block;
     width: 100%;
  }

  .el-drawer__header {
     display: none !important; 
  }

  .add-content >>> .el-drawer__header {
    display: none;
  }



 .x-form--label-right  >>> .x-form-item__label {
    color: #FFF;
}

.form-area {
    width: 100%;
    padding-left: 20px;
    padding-top: 20px;
}
.no-video{
	width: 100%;
	height:100%;
	display: flex;
	justify-content:center;
	align-items:center;
	background-color: black;
	color: #fff;
	font-size: 36px;
}

</style>
