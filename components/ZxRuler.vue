<template>
	<canvas
	 class="ruler-canvas"
	:style="{'width': rWidth+'px', 'height': rHeight+'px'}" 
	canvas-id="rulerCanvas" 
	id="rulerCanvas"
	@touchstart="onTouchStart"
	@touchmove="onTouchMove"
	@touchend="onTouchEnd"></canvas>
</template>

<script>
	export default {
		name: 'ZxRuler',
		model:{
			peop: 'value',
			event: 'input'
		},
		props:{
			value:{
				type: Number,
				default: 0
			},
			CTRLID:{
				type: String,
				default: 'rulerid'
			},
			rulerWidth:{
				type: Number,
				default: 600
			},
			rulerHeight:{
				type: Number,
				default: 120
			},
			orientation:{
				type: String,
				default: 'horizontal'		// horizontal: 水平的；vertical：垂直的
			},
			max:{
				type: Number,
				default: 999
			},
			min:{
				type: Number,
				default: 0
			},
			// 标尺显示比例
			ratio:{
				type: Number,
				default: 1
			},
			markHeight:{
				type: Number,
				default: 16
			}
		},
		data() {
			return {
				rWidth: 0,
				rHeight: 0,
				stepInterval: 5,
				
				startX: 0,
				moveX: 0,
				showValue: 0
			}
		},
		watch: {
			value(newValue, oldValue) {
				this.initCtrl();
			}
		},
		methods: {
			initCtrl(){
				this.rWidth = parseInt(uni.upx2px(this.rulerWidth));
				this.rHeight = parseInt(uni.upx2px(this.rulerHeight));
				this.showValue = this.value;
				this.drawRuler();
			},
			drawRuler(){
				const context = uni.createCanvasContext('rulerCanvas');
				context.clearRect(0, 0, this.rWidth, this.rHeight); //每次更新画布之前先清除画布
				this.drawMark(context);
				this.drawRulerScale(context);
				setTimeout(() => {
					context.draw();
				},0);
			},
			drawMark(ctx){
				let scale = 3/4;		//三角形的三条边分别为x（底边）、y（高）、z（斜边）、
				let x = uni.upx2px(this.markHeight*3/4);
				let y = uni.upx2px(this.markHeight);
				ctx.beginPath();
				ctx.moveTo(this.rWidth/2 - x, 0);
				ctx.lineTo(Math.floor(this.rWidth/2), y);
				ctx.lineTo(this.rWidth/2 + x, 0);
				ctx.lineTo(this.rWidth/2 - x, 0);
				ctx.fillStyle = 'red';
				ctx.fill(); 
				ctx.closePath();
			},
			drawRulerScale(ctx){
				let _markHeight = uni.upx2px(this.markHeight+10);
				let scaleCountHalfView = (this.rWidth-20)/this.stepInterval/2;
				let startX = (this.showValue - scaleCountHalfView).toFixed(0);
				let totalScaleCountPerView = this.showValue + scaleCountHalfView;
				
				ctx.beginPath();
				ctx.lineWidth = uni.upx2px(1);
				ctx.strokeStyle = '#666666';
				ctx.fontsize = '2em';
				ctx.fontFamily = '微软雅黑';
				ctx.textAlign = 'center';
				ctx.textBaseline = 'top';
				for (let i = startX; i < totalScaleCountPerView; i++) {
					let posX = (i - startX) * this.stepInterval + 10; //后面的10像素为做边距，距canvas左边10像素开始画
					ctx.moveTo(posX, _markHeight);
					if(i % 10 === 0){
						ctx.lineTo(posX, _markHeight+15);
						ctx.fillStyle = (i < this.min || i > this.max) ? '#999999' : '#000000';
						ctx.fillText(i, posX, _markHeight+20);
					}else if(i%5 === 0){
						ctx.lineTo(posX, _markHeight+10);
					}else{
						ctx.lineTo(posX, _markHeight+5);
					}					
					ctx.stroke();
				}
				ctx.closePath();
			},
			onTouchStart(evt){
				this.startX = evt.changedTouches[0].x;
				evt.stopPropagation();
			},
			onTouchMove(evt){
				this.moveX = evt.changedTouches[0].x;
				let distance = this.startX - this.moveX;
				let scaleCount = distance/this.stepInterval;
				let curRulerValue = this.value+scaleCount;
				if(curRulerValue <= this.min){
					this.showValue = this.min;
				}else if(curRulerValue >= this.max){
					this.showValue = this.max;
				}else{
					this.showValue = parseInt(curRulerValue);
				}
				if(this.value !== this.showValue){
					this.drawRuler();
					this.$emit('onScrollChange', this.showValue);
				}
				evt.stopPropagation();
			},
			onTouchEnd(evt){
				let newValue = this.showValue;
				this.$emit('input', newValue);
				this.startX = this.moveX;
				evt.preventDefault();
			}
		},
		created() {
			this.initCtrl();
		}
	}
</script>
<style scoped>
	.ruler-canvas{
		position: relative;
		overflow: hidden;
	}
	.ruler-canvas:after {
      width: 100%;
      height: 100%;
      content: "";
      position: absolute;
      z-index: 1;
      top: 0;
      left: 0;
      pointer-events: none;
      background-image: linear-gradient(to right, rgba(255,255,255,1),rgba(255,255,255, 0) 30%, rgba(255,255,255, 0) 70%,rgba(255,255,255, 1) 100%);
    }
</style>
