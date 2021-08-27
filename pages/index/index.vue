<template>
	<view class="ruler-wrap">
		<canvas 
		:style="{'width': rWidth+'px', 'height': rHeight+'px'}" 
		canvas-id="rulerCanvas" 
		id="rulerCanvas"
		@touchstart="onTouchStart"
		@touchmove="onTouchMove"
		@touchend="onTouchEnd"></canvas>		
	</view>
</template>

<script>
	export default {
		props:{
			rulerWidth: {
				type: Number,
				default: 700
			},
			rulerHeight: {
				type: Number,
				default: 120
			},
			max: {
				type: Number,
				default: 300
			},
			min: {
				type: Number,
				default: 50
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
				step: 5,
				
				
				start: 0,
				end: 0,
				startX: 0,
				moveX: 0,
				mainValue: 0
			}
		},
		onLoad() {

		},
		methods: {
			initCtrl(){
				this.rWidth = uni.upx2px(this.rulerWidth);
				this.rHeight = uni.upx2px(this.rulerHeight);
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
				ctx.lineTo(this.rWidth/2, y);
				ctx.lineTo(this.rWidth/2 + x, 0);
				ctx.lineTo(this.rWidth/2 - x, 0);
				ctx.fillStyle = 'red';
				ctx.fill(); 
				ctx.closePath();
			},
			drawRulerScale(ctx){
				let scaleInterval = this.max - this.min;
				let _markHeight = uni.upx2px(this.markHeight+10);
				ctx.beginPath();
				ctx.lineWidth = 0.1;
				ctx.strokeStyle = '#666666';
				ctx.font = '2em 微软雅黑';
				ctx.textAlign = 'center';
				ctx.textBaseline = 'top';
				let scalePerView = (this.rWidth-20)/this.step;
				console.log('scalePerView::::',(this.rWidth-20),JSON.stringify(scalePerView))
				for (let i = 0; i < scalePerView; i++) {
					let posX = i*this.step + 10; //后面的10像素为做边距，距canvas左边10像素开始画
					ctx.moveTo(posX, _markHeight);
					if(i%10 === 0){
						ctx.lineTo(posX, _markHeight+15);
						ctx.fillText((i+this.min), posX, _markHeight+20);
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
				const that = this;
				evt.stopPropagation();
				that.start = 0;
				that.end = -((this.max - this.min) * this.step);
				that.startX = evt.changedTouches[0].x;
			},
			onTouchMove(evt){
				this.moveX = evt.changedTouches[0].x;
				
			},
			onTouchEnd(evt){
				console.log(this.moveX - this.startX);
				this.startX = this.moveX;
			},
		},
		created() {
			this.initCtrl();
		}
	}
</script>

<style>
	.ruler-wrap{
		border: 1px solid #efefef;
		box-sizing: border-box;
		display: inline-block;
		margin: 30rpx;
	}
</style>
