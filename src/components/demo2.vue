<template>
    <div>
        <div id="container"></div>
    </div>
</template>

<style scoped>
    
</style>

<script>
import Konva from 'konva'
export default {
    data(){
        return {
            width:'',
            height:'',
            stage:''
        }
    },
    methods:{
        init(){
            var width = window.innerWidth;
            var height = window.innerHeight;
            var stage = new Konva.Stage({
                container: 'container',
                width: width,
                height: height
            });

            var layer = new Konva.Layer();
            stage.add(layer);

            // var rect1 = new Konva.Rect({
            // x: 60,
            // y: 60,
            // width: 100,
            // height: 90,
            // fill: 'red',
            // name: 'rect',
            // draggable: true
            // });

            let path = [[30,30],[200,30],[250,250],[30,30]]

            let ploygon = new Konva.Line({
				// sceneFunc:(context,shape) => {
				// 	context.beginPath()
				// 	path.map((item,index) => {
				// 		if(index === 0){
				// 			context.moveTo(item[0],item[1])
				// 		}else{
				// 			context.lineTo(item[0],item[1])
				// 		}
				// 	})
				// 	context.closePath()
				// 	context.fillStrokeShape(shape)
                // },
                
				// fill: 'rgba(0, 132, 255, 0.2)',
    			// stroke: '#4C98FF',
                // strokeWidth: 2,
                // name:'rect',
                // draggable:true
                points: [23, 20, 23, 160, 70, 93, 150, 109, 290, 139, 270, 93], 
                //points是多边形各个点的坐标，两两成对，分别代表x,y
                fill: '#00D2FF',
                stroke: 'black',
                strokeWidth: 5,
                closed: true, //是否要闭合，如果要就需要给true
                draggable:true,
                name:'rect'
            })
            


            layer.add(ploygon);

            var rect2 = new Konva.Rect({
            x: 250,
            y: 100,
            width: 150,
            height: 90,
            fill: 'green',
            name: 'rect',
            draggable: true
            });
            layer.add(rect2);

            var star = new Konva.Star({
            x: stage.width() / 2,
            y: stage.height() / 2,
            draggable: true,
            numPoints: 5,
            innerRadius: 20,
            outerRadius: 40,
            fill: 'yellow',
            stroke: 'black',
            strokeWidth: 4
            });

            var transformer = new Konva.Transformer();
            layer.add(transformer);
            transformer.attachTo(star);
            
        
            layer.add(star);
            layer.draw();

            stage.on('click tap', function (e) {
            // if click on empty area - remove all transformers
            if (e.target === stage) {
                stage.find('.rectTransformer').destroy();
                layer.draw();
                return;
            }
            // do nothing if clicked NOT on our rectangles
            if (!e.target.hasName('rect')) {
                return;
            }
            // remove old transformers
            // TODO: we can skip it if current rect is already selected
            stage.find('.rectTransformer').destroy();

            // create new transformer
            var tr = new Konva.Transformer({name: 'rectTransformer'});
            layer.add(tr);
            tr.attachTo(e.target);
            layer.draw();
            })
        }
    },
    mounted(){
        this.init()
    }
}
</script>