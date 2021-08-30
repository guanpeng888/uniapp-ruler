<template>
	<view class="ruler-wrap">
		<!-- 水平尺 -->
		<HorizontalRuler
		v-if="isShowHorizontalRuler"
		:CTRLID="CTRLID"
		:min="min"
		:max="max"
		:ratio="ratio"
		:markHeight="markHeight"
		:rulerWidth="rulerWidth"
		:rulerHeight="rulerHeight"
		v-model="showValue"
		@input="getRulerValue"
		@onScrollChange="onRulerValueChange"/>
		
		<!-- 竖直尺 -->
		<VerticalRuler
		v-else
		:CTRLID="CTRLID"
		:min="min"
		:max="max"
		:ratio="ratio"
		:markHeight="markHeight"
		:rulerWidth="rulerWidth"
		:rulerHeight="rulerHeight"
		v-model="showValue"
		@input="getRulerValue"
		@onScrollChange="onRulerValueChange"/>
	</view>
	
</template>

<script>
	import HorizontalRuler from './HorizontalRuler.vue'
	import VerticalRuler from './VerticalRuler.vue'
	export default {
		name: 'ZxRuler',
		components:{
			HorizontalRuler,
			VerticalRuler
		},
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
		computed:{
			isShowHorizontalRuler(){
				return this.orientation === 'horizontal';
			}
		},
		watch: {
			value(newValue, oldValue) {
				this.initCtrl();
			}
		},
		methods: {
			initCtrl(){
				this.showValue = this.value;
			},
			getRulerValue(result){
				this.$emit('input', result);
			},
			onRulerValueChange(result){
				this.$emit('onScrollChange', result);
			},
		},
		created() {
			this.initCtrl();
		}
	}
</script>
<style scoped>

</style>
