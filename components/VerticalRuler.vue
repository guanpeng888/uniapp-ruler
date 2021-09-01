<template>
	<canvas
	 class="ruler-canvas"
	:style="{'width': rWidth+'px', 'height': rHeight+'px'}" 
	:canvas-id="CTRLID" 
	:id="CTRLID"
	:disable-scroll="true"
	@touchstart="onTouchStart"
	@touchmove="onTouchMove"
	@touchend="onTouchEnd"></canvas>
</template>

<script>
	export default {
		name: 'VerticalRuler',
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
				default: 160
			},
			rulerHeight:{
				type: Number,
				default: 480
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
				
				startY: 0,
				moveY: 0,
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
				const context = uni.createCanvasContext(this.CTRLID);
				context.clearRect(0, 0, this.rWidth, this.rHeight); //每次更新画布之前先清除画布
				this.drawVerticalMark(context);
				this.drawVerticalRulerDial(context);
				setTimeout(() => {
					context.draw();
				},0);
			},
			drawVerticalMark(ctx){
				let scale = 3/4;		//三角形的三条边分别为x（底边）、y（高）、z（斜边）、
				let x = uni.upx2px(this.markHeight) *3/4;
				let y = uni.upx2px(this.markHeight);
				ctx.beginPath();
				ctx.moveTo(0, parseInt(this.rHeight/2 - x));
				ctx.lineTo(y, this.rHeight/2);
				ctx.lineTo(0, this.rHeight/2 + x);
				ctx.lineTo(0, this.rHeight/2 - x);
				ctx.fillStyle = 'red';
				ctx.fill(); 
				ctx.closePath();
			},
			drawVerticalRulerDial(ctx){
				let _markHeight = uni.upx2px(this.markHeight+10);
				let scaleCountHalfView = (this.rHeight-20)/this.stepInterval/2;		//20为上下边距
				let _startY = this.showValue - scaleCountHalfView;
				let totalScaleCountPerView = parseInt(this.showValue + scaleCountHalfView);
				
				ctx.beginPath();
				ctx.lineWidth = uni.upx2px(1);
				ctx.strokeStyle = '#666666';
				ctx.fontsize = '2em';
				ctx.fontFamily = '微软雅黑';
				ctx.textAlign = 'center';
				ctx.textBaseline = 'middle';
				for (let i = totalScaleCountPerView; i > _startY; i--) {
					// 计算Y轴坐标，将大值放在上面，小值放在下面
					let posY = Math.abs(i - totalScaleCountPerView) * this.stepInterval + 12; //后面的12像素为上边距，距canvas上边12像素开始画
					ctx.moveTo(_markHeight, posY);
					if(i % 10 === 0){
						ctx.lineTo(_markHeight+15, posY);
						ctx.fillStyle = (i < this.min || i > this.max) ? '#999999' : '#000000';
						ctx.fillText(i, _markHeight+28, posY);		//28为文字距刻度线的距离
					}else if(i%5 === 0){
						ctx.lineTo(_markHeight+10, posY);
					}else{
						ctx.lineTo(_markHeight+5, posY);
					}					
					ctx.stroke();
				}
				ctx.closePath();
			},
			onTouchStart(evt){
				this.startY = evt.changedTouches[0].y;
				evt.stopPropagation();
				evt.preventDefault();
			},
			onTouchMove(evt){
				this.moveY = evt.changedTouches[0].y;
				let distance = this.moveY - this.startY;
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
				evt.preventDefault();
			},
			onTouchEnd(evt){
				let newValue = this.showValue;
				this.$emit('input', newValue);
				this.startY = this.moveY;
				evt.stopPropagation();
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
</style>
