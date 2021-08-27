<template>
	<canvas 
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
				default: 700
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
				
				startX: 0,
				moveX: 0,
				showValue: 0
			}
		},
		methods: {
			initCtrl(){
				this.rWidth = uni.upx2px(this.rulerWidth);
				this.rHeight = uni.upx2px(this.rulerHeight);
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
				ctx.lineTo(this.rWidth/2, y);
				ctx.lineTo(this.rWidth/2 + x, 0);
				ctx.lineTo(this.rWidth/2 - x, 0);
				ctx.fillStyle = 'red';
				ctx.fill(); 
				ctx.closePath();
			},
			drawRulerScale(ctx){
				let _markHeight = uni.upx2px(this.markHeight+10);
				let scaleCountHalfView = (this.rWidth-20)/this.step/2;
				let startX = parseInt(this.showValue - scaleCountHalfView);
				let totalScaleCountPerView = parseInt(this.showValue + scaleCountHalfView);
				
				ctx.beginPath();
				ctx.lineWidth = uni.upx2px(1);
				ctx.strokeStyle = '#666666';
				ctx.fontsize = '2em';
				ctx.fontFamily = '微软雅黑';
				ctx.textAlign = 'center';
				ctx.textBaseline = 'top';
				for (let i = startX; i < totalScaleCountPerView; i++) {
					let posX = (i - startX) * this.step + 10; //后面的10像素为做边距，距canvas左边10像素开始画
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
				evt.stopPropagation();
				this.startX = evt.changedTouches[0].x;
			},
			onTouchMove(evt){
				evt.preventDefault();
				this.moveX = evt.changedTouches[0].x;
				let distance = this.moveX - this.startX;
				let scaleCount = distance/this.step;
				if(this.value+scaleCount <= this.min){
					this.showValue = this.min;
				}else if(this.value+scaleCount >= this.max){
					this.showValue = this.max;
				}else{
					this.showValue = parseInt(this.value+scaleCount);
				}
				if(this.value !== this.showValue){
					this.drawRuler();
				}
			},
			onTouchEnd(evt){
				let newValue = this.showValue;
				this.$emit('input', newValue);
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
		display: inline-block;
		margin: 30rpx;
	}
	.cur-value{
		text-align: center;
		font-size: 16px;
	}
</style>
